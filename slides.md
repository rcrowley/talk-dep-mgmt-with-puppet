!SLIDE bullets

# Dependency management with Puppet



!SLIDE bullets

# Hi, I&#8217;m Richard Crowley

* Equal opportunity technology hater
* DevStructure&#8217;s operator and UNIX hacker



!SLIDE bullets

# Installing Puppet

	sudo apt-get update
	sudo apt-get -y install ruby ruby-dev \
		rubygems libopenssl-ruby \
		libshadow-ruby1.8

	sudo gem install rubygems-update
	PATH=$PATH:/var/lib/gems/1.8/bin \
		sudo update_rubygems

	sudo gem install puppet



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

	package { "nginx":
		ensure => "0.7.65-1ubuntu2",
	}

	package {
		"mysql2":
			ensure   => "0.2.4",
			provider => gem;
		"rails":
			ensure   => "3.0.0",
			provider => gem;
	}



!SLIDE bullets

# A bug!

	package {
		"mysql2":
			ensure   => "0.2.4",
			provider => gem;
	}

* Missing `libmysqlclient-dev` package which is needed to build `mysql2` gem.



!SLIDE bullets

# Declare all dependencies

	package {
		"libmysqlclient-dev":
			ensure => "5.1.41-3ubuntu12.3";
		"mysql2":
			ensure   => "0.2.4",
			provider => gem;
	}



!SLIDE bullets

# Nondeterminism!

	package {
		"libmysqlclient-dev":
			ensure => "5.1.41-3ubuntu12.3";
		"mysql2":
			ensure   => "0.2.4",
			provider => gem;
	}



!SLIDE bullets

# Declare that dep

	package {
		"libmysqlclient-dev":
			before => Package["mysql2"],
			ensure => "5.1.41-3ubuntu12.3";
		"mysql2":
			ensure   => "0.2.4",
			provider => gem;
	}



!SLIDE bullets

# Classes

* Singletons.
* Collections of resources.
* May be named as dependencies.



!SLIDE bullets smaller

# RubyGems and friends in a `class`

	class rubygems {
		package {
			"build-essential":
				ensure => latest;
			"ruby":
				ensure => "4.2"; # 1.8.7
			"ruby-dev":
				ensure  => "4.2",
				require => Package["build-essential"];
			"rubygems": 
				ensure => latest;
			"rubygems-update":
				ensure   => latest,
				provider => gem,
				require  => Package["rubygems"];
		}
		exec { "/bin/sh -c 'PATH=$PATH:/var/lib/gems/1.8/bin
			update_rubygems'":
			before  => Class["deps"],
			require => Package["rubygems-update"],
		}
	}



!SLIDE bullets small

# Files

	file {
		"nginx-foo":
			content => "server {
		listen 80;
		root /var/www/foo;
	}
	",
			ensure  => file,
			group   => root,
			mode    => 644,
			owner   => root,
			path    => "/etc/nginx/sites-available/foo",
			require => Package["nginx"];
		"/etc/nginx/sites-enabled/foo":
			ensure  => "/etc/nginx/sites-available/foo",
			require => Package["nginx"];
	}



!SLIDE bullets

# Services

	service { "nginx":
		ensure    => running,
		subscribe => File["nginx-foo"],
	}



!SLIDE bullets small

# Arbitrary commands

	exec { "sh -c 'echo Hello | mail example@example.com'":
		path => "/usr/bin:/bin",
	}



!SLIDE bullets

# Definitions

* Build your own resource types.
* Encapsulate common patterns.



!SLIDE bullets small

# `authorized_user` type

	define authorized_user($user, $public_key) {
		file {
			"/home/$user/.ssh":
				ensure  => directory,
				require => User[$user];
			"/home/$user/.ssh/authorized_keys":
				content => "$public_key\n",
				ensure  => file,
				require => User[$user];
		}
		group { $user: ensure => present }
		user { $user:
			ensure     => present,
			group      => $user,
			managehome => true,
			require    => Group[$user],
		}
	}



!SLIDE bullets

# An authorized user

	authorized_user { "rcrowley":
		public_key => "ssh-rsa OH HAI",
	}



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



!SLIDE bullets small

# `deps.pp`

	# Put the rubygems class from above here.
	include rubygems
	class deps {
		"json":
			ensure   => "1.4.2",
			provider => gem;
		"libmysqlclient-dev":
			ensure => "5.1.41-3ubuntu12.3";
		"mysql2":
			ensure   => "0.2.4",
			provider => gem,
			require  => Package["libmysqlclient-dev"];
		"nginx":
			ensure => "0.7.65-1ubuntu2";
		"sinatra":
			ensure   => "1.0",
			provider => gem;
	}
	include deps



!SLIDE bullets

# Running Puppet

	sudo puppet apply deps.pp

* Bring a new server up to speed.
* Incrementally manage your devbox.



!SLIDE bullets

# Thank you

* <richard@devstructure.com> or [@rcrowley](http://twitter.com/rcrowley)
* P.S. use DevStructure.
