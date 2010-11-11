!SLIDE bullets

# Dependency management with Puppet



!SLIDE bullets

# Hi, I&#8217;m Richard Crowley

* Equal opportunity technology hater
* DevStructure&#8217;s operator and UNIX hacker



!SLIDE bullets

# Installing Puppet

* `apt-get install puppet`

* *or*

* `gem install puppet`

* You really want 2.6 or better.



!SLIDE bullets

# Resources

* The smallest unit of configuration.
* Have a *type*, a *name*, and *attributes*.



!SLIDE bullets

# Packages

* The obvious resource type.
* Abstract differences between<br />package managers.



!SLIDE bullets

# Package resources

	@@@ puppet
	package { "nginx":
		ensure => "0.7.67-3ubuntu1",
	}

	package {
		"mysql2":
			ensure   => "0.2.6",
			provider => gem;
		"rails":
			ensure   => "3.0.1",
			provider => gem;
	}



!SLIDE bullets

# A bug!

	@@@ puppet
	package {
		"mysql2":
			ensure   => "0.2.6",
			provider => gem;
	}

* Missing `libmysqlclient-dev` package which is needed to build `mysql2` gem.



!SLIDE bullets

# Declare all dependencies

	@@@ puppet
	package {
		"libmysqlclient-dev":
			ensure => "5.1.49-1ubuntu8";
		"mysql2":
			ensure   => "0.2.6",
			provider => gem;
	}



!SLIDE bullets

# Nondeterminism!

	@@@ puppet
	package {
		"libmysqlclient-dev":
			ensure => "5.1.49-1ubuntu8";
		"mysql2":
			ensure   => "0.2.6",
			provider => gem;
	}



!SLIDE bullets

# Declare that dep

	@@@ puppet
	package {
		"libmysqlclient-dev":
			ensure => "5.1.49-1ubuntu8";
		"mysql2":
			ensure   => "0.2.6",
			provider => gem,
			require  =>
				Package["libmysqlclient-dev"];
	}



!SLIDE bullets

# Realistic example



!SLIDE bullets smaller

# Puppet in your project

	(master) rcrowley@wd-40:~/work/example$ ls -al
	total 44
	drwxr-xr-x 5 rcrowley rcrowley 4096 Sep 29 23:02 .
	drwxr-xr-x 3 rcrowley rcrowley 4096 Sep 29 22:42 ..
	drwxr-xr-x 7 rcrowley rcrowley 4096 Sep 29 22:42 .git
	-rw-r--r-- 1 rcrowley rcrowley    6 Sep 29 22:59 .gitignore
	-rw-r--r-- 1 rcrowley rcrowley 1672 Sep 29 23:00 Capfile
	-rw-r--r-- 1 rcrowley rcrowley 2986 Sep 29 23:02 app.rb
	-rw-r--r-- 1 rcrowley rcrowley  179 Sep 29 23:01 config.ru
	-rw-r--r-- 1 rcrowley rcrowley   38 Sep 29 22:58 deps.pp
	drwxr-xr-x 2 rcrowley rcrowley 4096 Sep 29 23:01 public
	-rw-r--r-- 1 rcrowley rcrowley  798 Sep 29 23:01 unicorn.conf.rb
	drwxr-xr-x 2 rcrowley rcrowley 4096 Sep 29 23:01 views
	(master) rcrowley@wd-40:~/work/example$



!SLIDE bullets

# Ruby and RubyGems

	@@@ puppet
	stage { "pre": before => Stage["main"] }
	class pre {
		package {
			"build-essential": ensure => latest;
			"ruby": ensure => "4.5"; # Ruby 1.8.7
			"ruby-dev": ensure => "4.5"; # Ruby 1.8.7
			"rubygems": ensure => "1.3.7-2";
		}
	}
	class { "pre": stage => "pre" }



!SLIDE bullets small

# `deps.pp`

	@@@ puppet
	# Ruby and RubyGems from before goes here.

	package {
		"json":
			ensure   => "1.4.6",
			provider => gem;
		"libmysqlclient-dev":
			ensure => "5.1.49-1ubuntu8";
		"mysql2":
			ensure   => "0.2.6",
			provider => gem,
			require  => Package["libmysqlclient-dev"];
		"nginx":
			ensure => "0.7.67-3ubuntu1";
		"sinatra":
			ensure   => "1.1.0",
			provider => gem;
	}



!SLIDE bullets small

# `deps.pp`, continued

	@@@ puppet
	file {
		"/etc/nginx/sites-available/example":
			content => "
	server {
		listen 80;
		root /var/www/example;
	}
	",
			ensure  => file;
		"/etc/nginx/sites-enabled/example":
			ensure => "/etc/nginx/sites-available/example";
		"/var/www/example":
			ensure => directory;
	}



!SLIDE bullets

# Running Puppet

* `sudo puppet apply deps.pp`

* Bring a new server up to speed.
* Incrementally manage your devbox.



!SLIDE bullets

* <https://gist.github.com/668403>



!SLIDE bullets

# Thank you

* <richard@devstructure.com> or [@rcrowley](http://twitter.com/rcrowley)
* P.S. use DevStructure.
