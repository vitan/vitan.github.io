---
layout: post
title: rhel6-simplest-mail-server-deployment-using-postfix-for-MTA-and-dovecot-for-MDA
---

Red Hat Linux Enterprise 6 Simplest Mail Server Deployment using postfix for MTA and dovecot for MDA
====================================================================================================

A simplest mail server deployment doc presents smallest config using postfix and dovecot, which *DONOT* need DNS setting.
I am looking such virtual machine as a test/devel-server for project development.

Prerequisites
-------------

* Assume MTA and MDA server is the same one

	mail.example.com

using the same server can lead NO-DNS setting for simplest config.

* change to root

	su -

* Install and start *postfix*

	yum install postfix
	service postfix start
	chkconfig postfix on

* Install and start *dovecot*

	yum install dovecot
	service dovecot start
	chkconfig dovecot on

Basic config
------------

###postfix basic config###

* Backup postfix config file firstly

	cp /etc/postfix/main.cf /etc/postfix/main.cf.bak

* Config */etc/postfix/main.cf*, and diff output to the following:

	diff /etc/postfix/main.cf /etc/postfix/main.cf.bak

*OUTPUT:*
	75c75
	< myhostname = mail.usersys.redhat.com
	---
	> #myhostname = host.domain.tld
	98c98
	< myorigin = $myhostname
	---
	> #myorigin = $myhostname
	113c113
	< inet_interfaces = all
	---
	> #inet_interfaces = all
	116c116
	< #inet_interfaces = localhost
	---
	> inet_interfaces = localhost
	119,120c119
	< #inet_protocols = all
	< inet_protocols = ipv4
	---
	> inet_protocols = all
	268d266
	< mynetworks = 10.66.0.0/16, 127.0.0.0/8

* Check the config file

	postfix check

any error message, you can refer [Postfix Basic Configuration](http://www.postfix.org/BASIC_CONFIGURATION_README.html) for trouble shooting.

* Restart the service *postfix* after successfully checking

	service postfix restart

* Add rule to iptables

	iptables -A INPUT -p tcp -s 0/0 --sport 1024:65535 -d mail.example.com --dport 25 -m state --state NEW,ESTABLISHED -j ACCEPT
	service iptables save

*WARNING:* In reality configuration I failed to make it, and at last using the following commands to clear all the firewall(or iptables) rules, as my virtual machine no security requirement.Shooting the trouble in future.

	iptables -F
	service iptables save

* Test the mail server.
Assume server *mail.example.com* having user *vitan*,
 In your local machine or any other machine which can look up the <em>mail.example.com</em>.</li>

