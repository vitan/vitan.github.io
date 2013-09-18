---
layout: devel-theme
title: rhel6 apache deploy
tags: apache django wsgi
categories: Linux
excerpt: A simple deploy document which covered apache, django project, wsgi, rhel6.
---

Red Hat Linux Enterprise 6 Apache Deploy Guide
==============================================

A simple deploy document which covered apache, django project, wsgi, rhel6.

Prerequisites
-------------

* change to root

        su -

* install httpd

        yum install httpd

* start httpd

        chkconfig httpd on
        service httpd start

* install mod_wsgi

        yum install mod_wsgi
if successfully installed, run the following command

        find -L /etc/httpd/ -name *wsgi*
*OUTPUT:*

        /etc/httpd/modules/mod_wsgi.so
        /etc/httpd/conf.d/wsgi.conf

* install django, here you must download the related django version manully

        pip install Django==1.5.1
*Or:*

        pip install Django==1.4.3

Basic config
------------
Assume a django project named *lwepm*

* cp the project to related dirs

        cp lwepm /usr/lib/python2.6/site-packages/

* touch a config file for *lwepm*

        touch /etc/httpd/conf.d/lwepm.conf

* review the `lwepm.conf`

        # Deployment using mod_python
        #
        # Useful documentation:
        # http://docs.djangoproject.com/en/dev/howto/deployment/modpython/
        #
        
        Alias /favicon.ico /usr/lib/python2.6/site-packages/lwepm/static/images/favicon.ico
        # This should be configured as the correct location
        Alias /static/admin /usr/lib/python2.6/site-packages/django/contrib/admin/static/admin/
        Alias /static /usr/lib/python2.6/site-packages/lwepm/static/
        
        # Limit threads forked:
        # http://blog.webfaction.com/tips-to-keep-your-django-mod-python-memory-usage-down
        # prefork MPM
        StartServers 5
        MinSpareServers 5
        MaxSpareServers 10
        MaxClients 256
        MaxRequestsPerChild 0
        
        # Configurations for mod_wsgi
        #WSGIDaemonProcess daemon processes=5 threads=1
        #WSGIProcessGroup daemon
        WSGIScriptAlias / /usr/lib/python2.6/site-packages/lwepm/wsgi.py
        WSGIPassAuthorization On
        
        <Location "/">
            # ======================
            # Handler for mod_python
            # ======================
            #SetHandler python-program
            #PythonHandler django.core.handlers.modpython
            #SetEnv DJANGO_SETTINGS_MODULE tcms.product_settings
            #PythonDebug On
        
            # ====================
            # Handler for mod_wsgi
            # ====================
            SetHandler wsgi-script
        
            #order deny,allow
            #Deny from all
            #Allow from 10.66.65.110
        
            LimitRequestBody 10485760
            AddOutputFilterByType DEFLATE text/html text/plain text/xml text/javascript application/x-javascript text/css
        
            ErrorDocument 401 /static/errors/unauthorized.html
        </Location>

* restart the apache server

        service httpd restart

* check the result on the browser

* If Error, Debug steps:

        1. check whether the browser go to the apache server. If successful, you can find:

            Apache .....

        Or

            Internal Error 501/500 and so on.

        Then you can run the following command to debug:

            tail -n 20 /var/log/httpd/error_log

        You can get some info in the last lines of the output.

        2. if failed or you cannot got the apache server, you should check the url, the linux Network config, iptables, selinux and so on.


Tips
----

* file permission can cause many bugs.

* you should firstly confirm apache server is ok. After you installed&started the apache server, you can get the index.html on browser.


Refer Urls
----------

* [django guide for apache deployment](https://docs.djangoproject.com/en/1.3/howto/deployment/modwsgi/)

{% include disqus.html %}