!SLIDE bullets

# Dependency management with Puppet

* <http://rcrowley.org/talks/pycon-2011/>



!SLIDE bullets

# Hi, I&#8217;m Richard Crowley

* Equal opportunity technology hater.
* DevStructure&#8217;s operator and UNIX hacker.



!SLIDE bullets

# Install Puppet

	@@@ sh
	sudo apt-get -y install ruby rubygems
	sudo gem install --no-rdoc --no-ri puppet

* You want 2.6 or better.
* Ubuntu Lucid and pretty much<br />every Red Hat should use RubyGems.



!SLIDE bullets

# Configuration management

* Express _all_ dependencies in a<br />legible and iterable format.



!SLIDE bullets

# `pip freeze`

* Not good enough.



!SLIDE bullets

# Resources

* The smallest unit of configuration.
* Have a _type_, a _name_, and _parameters_.



!SLIDE bullets

# Resources

	@@@ Puppet
	package { "python":
		ensure => installed,
		provider => apt,
	}



!SLIDE bullets

# Resources

	@@@ Puppet
	package { "python":
		ensure => "2.6.6-2ubuntu1",
		provider => apt,
	}



!SLIDE bullets

# Resources

	@@@ Puppet
	package { "python":
		ensure => latest,
		provider => apt,
	}



!SLIDE bullets

# Resources

	@@@ Puppet
	package {
		"build-essential":
			ensure => installed,
			provider => apt;
		"python":
			ensure => installed,
			provider => apt;
		"python-dev":
			ensure => installed,
			provider => apt;
		"python-setuptools":
			ensure => installed,
			provider => apt;
	}



!SLIDE bullets small

# Resources

	@@@ Puppet
	package {
		"build-essential": ensure => installed;
		"python": ensure => installed;
		"python-dev": ensure => installed;
		"python-setuptools": ensure => installed;
	}



!SLIDE bullets

# Resource types

* There&#8217;s more to life than<br />package management.



!SLIDE bullets

# Users and groups

	@@@ Puppet
	group { "rcrowley":
		ensure => present,
		gid => 1000,
	}
	user { "rcrowley":
		ensure => present,
		gid => 1000,
		uid => 1000,
	}



!SLIDE bullets

# Authorized keys

	@@@ Puppet
	ssh_authorized_key { "Richard's VM":
		ensure => present,
		key => "AAAAB3NzaC1yc2EAAAABIwAAAQEA6NF8iallvQVp22WDkTkyrtvp9eWW6A8YVr+kz4TjGYe7gHzIw+niNltGEFHzD8+v1I2YJ6oXevct1YeS0o9HZyN1Q9qgCgzUFtdOKLv6IedplqoPkcmF0aYet2PkEDo3MlTBckFXPITAMzF8dJSIFo9D8HfdOV0IAdx4O7PtixWKn5y2hMNG0zQPyUecp4pzC6kivAIhyfHilFR61RGL+GPXQ2MWZWFYbAGjyiYJnAmCP3NOTd0jMZEnDkbUvxhMmBYSdETk1rRgm+R4LOzFUGaHqHDLKLX+FIPKcF96hrucXzcWyLbIbEgE98OHlnVYCzRdK8jlqm8tehUc9c9WhQ",
		type => "ssh-rsa",
		user => "rcrowley",
	}



!SLIDE bullets

# Services

	@@@ Puppet
	service { "apache2":
		enable => true,
		ensure => running,
		hasstatus => true,
	}



!SLIDE bullets

# Generic resource types

## `exec` and `file`



!SLIDE bullets

# Arbitrary commands

	@@@ Puppet
	exec { "easy_install pip":
		path => "/usr/local/bin:/usr/bin:/bin",
		unless => "which pip",
	}



!SLIDE bullets small

# Arbitrary files

	@@@ Puppet
	file {
		"/root/.pip":
			ensure => directory,
			group => "root",
			mode => 0755,
			owner => "root";
		"/root/.pip/pip.conf":
			content => template("python/pip.conf.erb"),
			ensure => file,
			group => "root",
			mode => 0644,
			owner => "root";
	}



!SLIDE bullets

# `template` function

	@@@ Puppet
	template("python/pip.conf.erb")

* Argument: special pathname.
* Result: string rendered from ERB.



!SLIDE bullets

# `pip.conf.erb` template

	[install]
	user-mirrors = true
	mirrors =
		http://pypi.<%= domainname %>
		http://pypi.python.org



!SLIDE bullets

# Facts

* `facter | less`
* <code>facter <em>factname</em></code>
* Normalized names for<br />dozens of system parameters.



!SLIDE bullets

# Running Puppet manifests

* <code>sudo puppet apply <em>pathname</em></code>
* `--noop` and `--verbose` are handy.



!SLIDE bullets

# Composing resources

## Classes

* Singleton collections of resources.



!SLIDE bullets small

# Classes

	@@@ Puppet
	class python {
		package {
			"build-essential": ensure => installed;
			"python": ensure => installed;
			"python-dev": ensure => installed;
			"python-setuptools": ensure => installed;
		}
		exec { "easy_install pip":
			path => "/usr/local/bin:/usr/bin:/bin",
			unless => "which pip",
		}
	}



!SLIDE bullets

# Using classes

	@@@ Puppet
	include python

* `include` is a shorthand function.



!SLIDE bullets

# Using classes

	@@@ Puppet
	class { "python": }

* Class instances are just resources.



!SLIDE bullets

# Parameterized classes

	@@@ Puppet
	class foo($bar = "baz") {}

	class { "foo": bar => "quux" }

* Like any resource,<br />classes may accept parameters.



!SLIDE bullets

# Composing resources

## Definitions

* Resource <em>type</em>s expressed as Puppet code.



!SLIDE bullets small

# Definitions

	@@@ Puppet
	define pip($ensure = installed) {
		case $ensure {
			installed: {
				exec { "pip install $name":
					path => "/usr/local/bin:/usr/bin:/bin",
				}
			}
			latest: {
				exec { "pip install --upgrade $name":
					path => "/usr/local/bin:/usr/bin:/bin",
				}
			}
			default: {
				exec { "pip install $name==$ensure":
					path => "/usr/local/bin:/usr/bin:/bin",
				}
			}
		}
	}



!SLIDE bullets

# Using definitions

	@@@ Puppet
	pip { "mysql-python": ensure => installed }



!SLIDE bullets

# Variables and expressions

## Further reading

* <http://bit.ly/puppet-variables>
* <http://bit.ly/puppet-expressions>
* <http://bit.ly/puppet-conditionals>



!SLIDE bullets

# Ruby plugins

* For when definitions aren&#8217;t good enough.
* Implementation details are<br />beyond the scope of this talk.



!SLIDE bullets

# `gem install puppet-pip`

* Ruby implementation of a _provider_<br />for the _package_ resource type.
* Submitted to be included in<br />the next release of Puppet.



!SLIDE bullets

# Using `puppet-pip`

	@@@ Puppet
	package { "mysql-python":
		ensure => installed,
		provider => pip,
	}



!SLIDE bullets

# Modules

	/etc/puppet/
		manifests/
			nodes.pp
			site.pp
		modules/
			python/
				manifests/
					init.pp
					foo.pp

* Code organization pays dividends.



!SLIDE bullets

# `modules/python/ manifests/init.pp`

	@@@ Puppet
	class python {
		# ...
	}



!SLIDE bullets

# `modules/python/ manifests/foo.pp`

	@@@ Puppet
	class python::foo {
		# ...
	}



!SLIDE bullets

# Module autoloading

* `include python` will expect the `python` class in `modules/python/manifests/init.pp`.
* Templates referenced as `template("python/foo.erb")` will render `modules/python/templates/foo.erb`.



!SLIDE bullets

# Running Puppet modules

* <code>sudo puppet apply -e "include <em>classname</em>"</code>
* `--noop` and `--verbose` are still handy.



!SLIDE bullets

# Which modules are applied to which servers?



!SLIDE bullets

# Nodes

	@@@ Puppet
	node default {
		include python
	}

	node /^www\d+\./ inherits default {
		include apache
		include django
	}

	node "mail.example.com" inherits default {
		include lamson
	}

* Conventionally in `manifests/nodes.pp`.



!SLIDE bullets

# `manifests/site.pp`

	@@@ Puppet
	Exec {
		path => "/usr/local/bin:/usr/bin:/bin",
	}

	import "nodes"

* Default entrypoint.



!SLIDE bullets

# Running Puppet catalog

* `sudo puppet apply /etc/puppet/manifests/site.pp`
* `--noop` and `--verbose` are still handy.



!SLIDE bullets

# Django module

	@@@ Puppet
	class django {
		package {
			"django":
				ensure => "1.2.5",
				provider => pip;
			"mysql-python":
				ensure => installed,
				provider => pip;
		}
	}



!SLIDE bullets

# A bug!

	@@@ Puppet
	class django {
		package {
			"django":
				ensure => "1.2.5",
				provider => pip;
			"mysql-python":
				ensure => installed,
				provider => pip;
		}
	}



!SLIDE bullets

# Declare _all_ dependencies

	@@@ Puppet
	class django {
		package {
			"django":
				ensure => "1.2.5",
				provider => pip;
			"libmysqlclient-dev":
				ensure => installed;
			"mysql-python":
				ensure => installed,
				provider => pip;
		}
	}



!SLIDE bullets

# Nondeterminism!

	@@@ Puppet
	class django {
		package {
			"django":
				ensure => "1.2.5",
				provider => pip;
			"libmysqlclient-dev":
				ensure => installed;
			"mysql-python":
				ensure => installed,
				provider => pip;
		}
	}



!SLIDE bullets small

# Declare interdependencies

	@@@ Puppet
	class django {
		package {
			"django":
				ensure => "1.2.5",
				provider => pip;
			"libmysqlclient-dev":
				ensure => installed;
			"mysql-python":
				ensure => installed,
				provider => pip,
				require => Package["libmysqlclient-dev"];
		}
	}



!SLIDE bullets small

# Declare interdependencies

	@@@ Puppet
	class python {
		package {
			"build-essential": ensure => installed;
			"python": ensure => installed;
			"python-dev": ensure => installed;
			"python-setuptools": ensure => installed;
		}
		exec { "easy_install pip":
			path => "/usr/local/bin:/usr/bin:/bin",
			require => Package["python-setuptools"],
			unless => "which pip",
		}
	}



!SLIDE bullets small

# Coarser dependencies

	@@@ Puppet
	class python {
		package {
			"build-essential": ensure => installed;
			"python": ensure => installed;
			"python-dev": ensure => installed;
			"python-setuptools": ensure => installed;
		}
		exec { "easy_install pip":
			alias => "pip",
			path => "/usr/local/bin:/usr/bin:/bin",
			require => Package["python-setuptools"],
			unless => "which pip",
		}
		Class["python"] -> Package<| provider == pip |>
	}

* All `pip` packages require `pip` to be setup.



!SLIDE bullets

# `before` and `require`

* One resource can `require` another.
* One resource can come `before` another.



!SLIDE bullets small

# Same difference

	@@@ Puppet
	class django {
		package {
			"django":
				ensure => "1.2.5",
				provider => pip;
			"libmysqlclient-dev":
				before => Package["mysql-python"],
				ensure => installed;
			"mysql-python":
				ensure => installed,
				provider => pip;
		}
	}



!SLIDE bullets

# Why all the fuss?

* Python packages are ignorant of<br />lower-level dependencies.
* For example, `libmysqlclient-dev`.
* Explicit is better than implicit.



!SLIDE bullets small

# Apache module

	@@@ Puppet
	class apache {
		package {
			"apache2-mpm-worker": ensure => installed;
			"libapache2-mod-wsgi": ensure => installed;
		}
		file {
			"/etc/apache2/sites-available/mysite":
				content => template("apache/mysite.erb"),
				ensure => file,
				require => Package["apache2-mpm-worker"];
			"/etc/apache2/sites-enabled/001-mysite":
				ensure =>
					"/etc/apache2/sites-available/mysite",
				require => Package["apache2-mpm-worker"];
		}
		# Next slide goes here.
	}



!SLIDE bullets small

# Apache service

	@@@ Puppet
	service { "apache2":
		enable => true,
		ensure => running,
		hasstatus => true,
		require => Package["apache2-mpm-worker"],
		subscribe => [
			Package[
				"apache2-mpm-worker",
				"libapache2-mod-wsgi"],
			File[
				"/etc/apache2/sites-available/mysite",
				"/etc/apache2/sites-enabled/001-mysite"]],
	}



!SLIDE bullets

# `notify` and `subscribe`

* Complementary just like `before` and `require`.
* Notified `exec` resources run their command.
* Notified `service` resources<br />restart the service.



!SLIDE bullets smaller

# `modules/apache/templates/mysite.erb`

	<VirtualHost *:80>
		DocumentRoot /usr/local/share/wsgi/mysite/media
		Alias /media /usr/local/share/wsgi/mysite/media
		WSGIScriptAlias / /usr/local/share/wsgi/mysite/mysite.wsgi
		WSGIDaemonProcess mysite processes=2
		WSGIProcessGroup mysite
	</VirtualHost>



!SLIDE bullets smaller

# `modules/apache/templates/mysite.erb` with one process per core

	<VirtualHost *:80>
		DocumentRoot /usr/local/share/wsgi/mysite/media
		Alias /media /usr/local/share/wsgi/mysite/media
		WSGIScriptAlias / /usr/local/share/wsgi/mysite/mysite.wsgi
		WSGIDaemonProcess mysite processes=<%= processorcount %>
		WSGIProcessGroup mysite
	</VirtualHost>



!SLIDE bullets

# Deploy



!SLIDE bullets smaller

# Deploy with Fabric

	@@@ Python
	from fabric.api import *
	from fabric.contrib.project import rsync_project

	env.hosts = ['example.com']

	def deploy():
	    rsync_project(remote_dir='/etc/puppet')
	    sudo('RUBYLIB=/var/lib/gems/1.8/gems/puppet-pip-0.0.5/lib'
			 'puppet apply /etc/puppet/manifests/site.pp')



!SLIDE bullets

# Puppet master

* Deploy your Puppet code only to the master.
* Agents phone home periodically<br />to fetch their configuration.



!SLIDE bullets

# Puppet master

* `sudo puppet master`
* Drops privileges after starting.
* Can be run by Apache or Nginx if needed.



!SLIDE bullets

# Puppet agent

* Daemon or cron job that phones home.



!SLIDE bullets

# An agent&#8217;s `/etc/puppet/puppet.conf`

	[main]
		server = puppet.example.com

* Accept defaults otherwise.
* See <http://bit.ly/puppet-config-ref>.



!SLIDE bullets

# Puppet agent

* `sudo puppet agent --no-daemonize --onetime`
* First time will fail in SSL client verify.



!SLIDE bullets

# Sign SSL certificates

## Tell the master the agent is cool

* `sudo puppet cert -l`
* <code>sudo puppet cert -s <em>hostname</em></code>



!SLIDE bullets

# Puppet agent

* `sudo puppet agent --no-daemonize --onetime`
* If certificates verify, it receives<br />and executes a catalog.
* `--noop` and `--verbose` are still handy.



!SLIDE bullets

# Why should I bother with Puppet master?



!SLIDE bullets

# Exported resources

	[main]
		server = puppet.example.com

		dbadapter = mysql
		dbname = puppet
		dbpassword = puppet
		dbserver = localhost
		dbuser = puppet

	[master]
		thin_storeconfigs = true

* Run MySQL on the master.



!SLIDE bullets

# Exported resources

	@@@ Puppet
	node /^www\d+\./ inherits default {
		include apache
		include django
		@@nagios_host { "$hostname":
			address => "$ipaddress",
			hostgroups => ["www"],
			use => "generic-host",
			# ...
		}
	}

	node "ops.example.com" inherits default {
		include nagios
		Nagios_host<<||>>
	}



!SLIDE bullets

# Blueprint

* If all that sounds like a lot of trouble...
* `blueprint`(1) reverse engineers servers to generate Puppet code.
* <https://github.com/devstructure/blueprint>
* <http://devstructure.com/>



!SLIDE bullets

# Thank you

* <richard@devstructure.com> or [@rcrowley](http://twitter.com/rcrowley)
* P.S. I&#8217;m available for Puppet<br />and operations consulting.
