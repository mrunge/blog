Configuring Django to use a MySQL-Server via SSL
################################################
:date: 2012-07-12 12:45
:author: mrunge
:category: Linux
:tags: django, mysql, Python
:slug: configuring-django-to-use-a-mysql-server-via-ssl

Currently, I'm using Django to develop some internally used
applications. For one of them, I had to use a SSL connection to access a
MySQL database. As development environment, I'm using Fedora 17, of
course.

During that process, I ran into two bugs:

-  MySQL 5.5 doesn't export HAVE\_OPENSSL anymore, as it has done in
   earlier versions. This causes MySQL-python to assume, SSL is
   deactivated during build. Patching \_mysql.c from MySQL-python to
   ignore that, fixes it.
-  It seems, ssl-connection parameters in Django have to be specified
   something like this:

   ::

       'database': {
               'ENGINE':'django.db.backends.mysql',
               'NAME':'dbname',
               'USER':'dbuser',
               'PASSWORD':'xxxxxxxxxxxx',
               'HOST':'host',
               'OPTIONS': {'ssl':{
                           'key':None,
                           'cert':None,
                           'ca':None,
                           'cipher': 'DHE-RSA-AES256-SHA'
                               },
                           },


