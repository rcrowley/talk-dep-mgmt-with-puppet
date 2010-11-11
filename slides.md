!SLIDE bullets

# Dependency management with Puppet



!SLIDE bullets

# Hi, I&#8217;m Richard Crowley

* Equal opportunity technology hater
* DevStructure&#8217;s operator and UNIX hacker



!SLIDE bullets

# Installing Puppet

	@@@ sh
	apt-get install build-essential \
		ruby ruby-dev rubygems
	gem install puppet puppet-pip

* <https://github.com/rcrowley/puppet-pip>



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
		"mysql-python":
			ensure   => "1.2.3",
			provider => pip;
		"django":
			ensure   => "1.2.3",
			provider => pip;
	}



!SLIDE bullets

# A bug!

	@@@ puppet
	package {
		"mysql-python":
			ensure   => "1.2.3",
			provider => pip;
	}

* Missing `libmysqlclient-dev` package which is needed to build `mysql-python` package.



!SLIDE bullets

# Declare all dependencies

	@@@ puppet
	package {
		"libmysqlclient-dev":
			ensure => "5.1.49-1ubuntu8";
		"mysql-python":
			ensure   => "1.2.3",
			provider => pip;
	}



!SLIDE bullets

# Nondeterminism!

	@@@ puppet
	package {
		"libmysqlclient-dev":
			ensure => "5.1.49-1ubuntu8";
		"mysql-python":
			ensure   => "1.2.3",
			provider => pip;
	}



!SLIDE bullets

# Declare that dep

	@@@ puppet
	package {
		"libmysqlclient-dev":
			ensure => "5.1.49-1ubuntu8";
		"mysql-python":
			ensure   => "1.2.3",
			provider => pip,
			require  =>
				Package["libmysqlclient-dev"];
	}



!SLIDE bullets

# Realistic example



!SLIDE bullets smaller

# Puppet in your project

	(master) rcrowley@wd-40:~/work/example$ ls -l
	total 20
	-rw-r--r-- 1 rcrowley rcrowley    0 Nov  9 01:36 __init__.py
	-rw-r--r-- 1 rcrowley rcrowley 2700 Nov  9 01:40 deps.pp
	-rw-r--r-- 1 rcrowley rcrowley   79 Nov  9 01:37 fabfile.py
	-rw-r--r-- 1 rcrowley rcrowley  546 Nov  9 01:36 manage.py
	-rw-r--r-- 1 rcrowley rcrowley 3388 Nov  9 01:36 settings.py
	-rw-r--r-- 1 rcrowley rcrowley  484 Nov  9 01:36 urls.py
	(master) rcrowley@wd-40:~/work/example$



!SLIDE bullets small

# Python and `pip`

	@@@ puppet
	stage { "pre": before => Stage["main"] }
	class pre {
		package {
			"build-essential": ensure => latest;
			"python": ensure => "";
			"python-dev": ensure => "";
			"python-setuptools": ensure => "";
		}
		exec { "easy_install pip":
			path => "/usr/local/bin:/usr/bin:/bin",
			refreshonly => true,
			require => Package["python-setuptools"],
			subscribe => Package["python-setuptools"],
		}
	}
	class { "pre": stage => "pre" }



!SLIDE bullets small

# `deps.pp`

	@@@ puppet
	# Python and pip from before goes here.

	package {
		"django":
			ensure   => "1.2.3",
			provider => pip;
		"libmysqlclient-dev":
			ensure => "5.1.49-1ubuntu8";
		"mysql-python":
			ensure   => "1.2.3",
			provider => pip,
			require  => Package["libmysqlclient-dev"];
		"nginx":
			ensure => "0.7.67-3ubuntu1";
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

	@@@ sh
	# As root!
	export GEMS="/usr/lib/ruby/gems/1.8/gems"
	export RUBYLIB=$GEMS/"puppet-pip-0.0.1/lib"
	puppet apply deps.pp

* Bring a new server up to speed.
* Incrementally manage your devbox.



!SLIDE bullets

* <https://gist.github.com/668598>



!SLIDE bullets

# Thank you

* <richard@devstructure.com> or [@rcrowley](http://twitter.com/rcrowley)
* P.S. use DevStructure.
