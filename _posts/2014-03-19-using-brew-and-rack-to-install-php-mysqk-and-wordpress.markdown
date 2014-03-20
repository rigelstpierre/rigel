---
layout: post
title: Using Brew and Rack to Install PHP, MySQL, and Wordpress
date: 2014-03-19
---

Most people use MAMP or MAMP Pro when it comes to developing Wordpress sites. I’ve never liked it and have tried a few solutions but always come back to it. Recently I tried something a bit different. I used Brew to install PHP, and MySQL and then used Rack to serve it locally vs using Apache or NGINX. To do this first you need to install Brew.

`ruby -e "$(curl -fsSL https://raw.github.com/Homebrew/homebrew/go/install)”`

If you already have brew installed I encourage you to run `brew update`and `brew doctor`.
Next you want to tap:

`brew tap homebrew/dupes`<br />
`brew tap josegonzalez/homebrew-php`

Once you have those tapped you’ll want to install PHP with MySQL and CGI. Feel free to add on to this as you may require.

`brew install php55 --with-mysql --with-cgi`

To add PHP to your start up routine

`cp /usr/local/Cellar/php54/5.4.15/homebrew-php.josegonzalez.php55.plist ~/Library/LaunchAgents/`

To Start PHP

`launchctl load -w ~/Library/LaunchAgents/homebrew-php.josegonzalez.php55.plist`

To Stop PHP

`launchctl unload -w ~/Library/LaunchAgents/homebrew-php.josegonzalez.php55.plist`

If you want you to have MySQL to start on start up add this.

`cp /usr/local/Cellar/mysql/5.6.10/homebrew.mxcl.mysql.plist ~/Library/LaunchAgents/`

To start to MySQL

`launchctl load -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist`

And to stop

`launchctl unload -w ~/Library/LaunchAgents/homebrew.mxcl.mysql.plist `

Manual Start/Stop<br />
  - Start mysql: `/usr/local/mysql/support-files/mysql.server start`<br />
  - Stop mysql: `/usr/local/mysql/support-files/mysql.server stop`<br />
  - See other options - `/usr/local/mysql/support-files/mysql.server -h`<br />

Run following command to improve security of your mysql setup. It will present you wizard to set mysql root password among other things.

`mysql_secure_installation`

Adding the MySQL workbench will make working with MySQL that much easier.

```ln -s /usr/local/Cellar/mysql/5.6.10 /usr/local/mysql
sudo ln -s /usr/local/Cellar/mysql/5.6.10/my.cnf /etc/my.cnf```

Next will need to install the gems needed to support the Rack setup. I’m assuming you have Ruby, and I recommend Rbev installed.

```
gem install rack
gem install rack-legacy
gem install rack-rewrite
```

The last thing you’ll need to do is add a `config.ru` to the default the root folder of your wordpress or other PHP app. This will also work for static sites. This is the `config.ru`.

```ruby
 config.ru for Rackup + Wordpress
# added hackery to work around wordpress issues - Patrick Anderson (patrick@trinity-ai.com)
# clearly this could be cleaner, but it does work 
 
require 'rack'
require 'rack-legacy'
require 'rack-rewrite'
 
# patch Php from rack-legacy to substitute the original request so 
# WP's redirect_canonical doesn't do an infinite redirect of /
module Rack
  module Legacy
    class Php
      def run(env, path)
        
        config = {'cgi.force_redirect' => 0}
        config.merge! HtAccess.merge_all(path, public_dir) if @htaccess_enabled
        config = config.collect {|(key, value)| "#{key}=#{value}"}
        config.collect! {|kv| ['-d', kv]}
        
        script, info = *path_parts(path)
        env['SCRIPT_FILENAME'] = script
        env['SCRIPT_NAME'] = strip_public script
        env['PATH_INFO'] = info
        env['REQUEST_URI'] = strip_public path
        env['REQUEST_URI'] = env['ORIGINAL_REQUEST'] unless env['ORIGINAL_REQUEST'].nil?
        
        super env, @php_exe, *config.flatten
      end
    end
  end
end
 
INDEXES = ['index.html','index.php', 'index.cgi']
 
use Rack::Rewrite do
 
  rewrite %r{(.*$)}, lambda {|match, rack_env|
    rack_env['ORIGINAL_REQUEST'] = rack_env['PATH_INFO']
 
    to_return = rack_env['PATH_INFO']
    if !File.exists?(File.join(Dir.getwd, rack_env['PATH_INFO']))
      to_return = '/index.php'
    end
    INDEXES.each do |index|
      if File.exists?(File.join(Dir.getwd, rack_env['PATH_INFO'], index))
        to_return = File.join(rack_env['PATH_INFO'], index)
      end
    end
    to_return
  }
end
 
use Rack::Legacy::Php, Dir.getwd
use Rack::Legacy::Cgi, Dir.getwd
run Rack::File.new Dir.getwd
```

To start up the app you’ll need to cd into the directory and run `rackup` and then navigate to `http://localhost:9292/`

This file works pretty well with `http://pow.cx/` development server
