MaNPP-buildout
==============
Install Nginx /PostgreSQL / PHP-FPM stack on your Mac for development purposes.

Currently supports following versions:
Nginx 1.5.3
PHP 5.4.17
PostgreSQL 9.2.4


Installation
-------------

### Prerequisites

1. Make sure you have latest Xcode installed, otherwise all compilation stuff will fail.
2. At least Python 2.6 is installed (python2.7 is recommended)
3. [homebrew](http://mxcl.github.com/homebrew/) installed


### Install binary dependencies

```bash

[bananos@amber]$ brew install postgresql gettext

```

### Run bootstrap.py

```bash

[bananos@amber]$ git clone git@github.com:bananos/MaNPP-buildout.git
[bananos@amber]$ cd MaNPP-buildout

[bananos@amber]$ /usr/bin/python2.7 bootstrap.py
Creating directory '/Users/bananos/Projects/github/MaNPP-buildout/bin'.
Creating directory '/Users/bananos/Projects/github/MaNPP-buildout/parts'.
Creating directory '/Users/bananos/Projects/github/MaNPP-buildout/eggs'.
Creating directory '/Users/bananos/Projects/github/MaNPP-buildout/develop-eggs'.
Generated script '/Users/bananos/Projects/github/MaNPP-buildout/bin/buildout'.
```

### Specify your user/group and timezone

Open ``buildout.cfg``,  navigate to ``[buildout]`` section and modify user/group/timezone lines:

```ini

[buildout]
# We extend the buildout-development.cfg configuration
extends = buildout-base.cfg

# you should specify your user here
user  = bananos
group = staff

timezone = Europe/Kiev
```
Save and proceed to next step.

### Run buildout

```bash

[bananos@amber]$ bin/buildout
```

Grab a cup of coffee & wait for a while for parts to compile. On core i5 mac it takes about 20min.


### Run supervisor

Once compilation is done, run [supervisor](http://supervisord.org), a client/server system that allows its users
to monitor and control a number of processes on UNIX-like operating systems.

Since we're going to run nginx on default 80 port, we're going to use `sudo`.
```bash

[bananos@amber]$ sudo bin/supervisord
```

Once supervisor daemon starts, you've already started PostgreSQL, Nginx and php-fpm processes. You can check them out:

```bash

[bananos@amber]$ bin/supervisorctl status
nginx                            RUNNING    pid 54481, uptime 0:00:07
php-fpm                          RUNNING    pid 54482, uptime 0:00:07
postgresqld                      RUNNING    pid 54487, uptime 0:00:07

```


### Test it in your browser

For testing purposes a simple ``index.php`` is created at ``var/www``.
Open your browser at ``http://localhost``, you should see PHP info page.


Project structure
------------------

After successful buildout you'll end up with something like this:

```bash

[bananos@amber]$ ls -l
 README.md
 bin
 bootstrap.py
 buildout-base.cfg
 buildout.cfg
 develop-eggs
 eggs
 etc             # <---- configuration for postgresql/php/nginx/supervisor
 parts           # <---- binary distributions live there
 templates       # <---- configuration templates
 thirdparty      # <---- source code & recipe downloads
 var             # <---- psql data, PIDs, logs, www-root, etc.
```

**Note:**  To tune configuration for postgresql/php or nginx you should modify files in ``templates/``,
not in ``etc/`` which is recreated each time buildout is run.


Working with PostgreSQL
----------------------
``psql`` is symlinked into ``bin/`` for convenience. A database folder is created for you at ``var/pgdata``, you may 
change this setting later by setting up ``[postgresql-conf]`` section in ``buildout.cfg``.


```bash

[bananos@amber]$ bin/psql --dbname=postgres

psql (9.2.4)
Type "help" for help.

postgres=# 

```

**Note:**  Make sure to correctly specify PostgreSQL base dir ``[postgresql-conf:basedir]`` or buildout won't 
be able to compile php PDO bindings and make correct symlinks. By default it is set to follow brew conventions:
``/usr/local/opt/postgresql/``



Logs
-----
All logging happens to be in ``var/log/`` directory:

```bash

[bananos@amber]$ ls -l var/log/
total 48
-rw-r--r--  1 root     staff   118 Aug 15 11:56 nginx.stdout.log
-rw-r--r--  1 root     staff     0 Aug 15 11:56 nginx_access.log
-rw-r--r--  1 root     staff   798 Aug 15 11:56 nginx_error.log
-rw-------  1 bananos  staff   321 Aug 15 11:56 php-fpm-error.log
-rw-------  1 bananos  staff     0 Aug 15 11:56 php-fpm-www.access.log
-rw-------  1 bananos  staff     0 Aug 15 11:56 php-fpm-www.log.slow
-rw-r--r--  1 root     staff   205 Aug 15 11:56 php-fpm.stdout.log
-rw-r--r--  1 root     staff   151 Aug 15 11:56 postgresqld.stdout.log
-rw-r--r--  1 root     staff  1014 Aug 15 11:56 supervisord.log
```

PostgreSQL/Nginx/php-fpm stdout/stderr logs are intercepted by supervisor:

```bash

[bananos@amber]$ bin/supervisorctl tail postgresqld
LOG:  database system was shut down at 2013-08-15 11:43:21 EEST
LOG:  database system is ready to accept connections
LOG:  autovacuum launcher started

```


FAQ
----

### What is buildout?

[Buildout](http://www.buildout.org/) is a Python-based build system for creating, assembling
and deploying applications from multiple parts, some of which may be non-Python-based. 
It lets you create a buildout configuration and reproduce the same software later.
