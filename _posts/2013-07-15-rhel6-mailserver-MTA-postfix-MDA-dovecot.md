---
layout: post
title: rhel6-simplest-mail-server-deployment-using-postfix-for-MTA-and-dovecot-for-MDA
---

Simplest Mail Server Deployment using postfix for MTA and dovecot for MDA
=========================================================================

A simplest mail server deployment doc presents smallest config using postfix for MTA, dovecot for MDA and Evolution for MUA, which *DONOT* need DNS setting.
I am looking such virtual machine as a test/devel-server for project development.

Prerequisites
-------------

* Assume MTA and MDA server is the same one

        mail.example.com

using the same server can lead *NO-DNS* setting for simplest config.

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

###Test mail(or postfix) server###

* Add user vitan to *mail.example.com*

        useradd vitan
        passwd vitan

* In any other machine which can look up the *mail.example.com*, type the following command:

        echo "Test the smtp server" | mail -s "from `hostname`" vitan@mail.example.com

* Check the mail on mail(or postfix) server *mail.example.com*

        cat /var/spool/mail/vitan

The following output is welcome.

        From: Weitao Zhou <weizhou@test.example.com>
        ......
        To: vitan@mail.example.com
        Subject: from test.example.com
        User-Agent: Heirloom mailx 12.5 7/5/10
        MIME-Version: 1.0
        Content-Type: text/plain; charset=us-ascii
        Content-Transfer-Encoding: 7bit

        Test the smtp server

###dovecot basic config###

I am still logging in *mail.example.com* as *root*.

* Backup dovecot config file firstly

        cp /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.bak

* Config */etc/dovecot/dovecot.conf*, and diff output to the following:

        diff /etc/dovecot/dovecot.conf /etc/dovecot/dovecot.conf.bak

*OUTPUT:*
        20c20
        < protocols = imap pop3
        ---
        > #protocols = imap pop3 lmtp
        32c32
        < login_greeting = Dovecot ready.
        ---
        > #login_greeting = Dovecot ready.

* Backup dovecot another config file

        cp /etc/dovecot/conf.d/10-ssl.conf /etc/dovecot/conf.d/10-ssl.conf.bak

* Config */etc/dovecot/conf.d/10-ssl.conf* to turn off ssl, and diff output to the following:

        diff /etc/dovecot/conf.d/10-ssl.conf /etc/dovecot/conf.d/10-ssl.conf.bak

*OUTPUT:*
        6c6
        < ssl = no
        ---
        > #ssl = yes

* Backup dovecot another config file */etc/dovecot/conf.d/10-auth.conf*

        cp /etc/dovecot/conf.d/10-auth.conf /etc/dovecot/conf.d/10-auth.conf.bak

* Config */etc/dovecot/conf.d/10-auth.conf* to enable plain text auth, and diff output to the following:

        diff /etc/dovecot/conf.d/10-auth.conf /etc/dovecot/conf.d/10-auth.conf.bak

*OUTPUT:*

        10d9
        < disable_plaintext_auth = no

* Backup dovecot another config file */etc/dovecot/conf.d/auth-system.conf.ext*

        cp /etc/dovecot/conf.d/auth-system.conf /etc/dovecot/conf.d/auth-system.conf.bak

* Config */etc/dovecot/conf.d/auth-system.conf* for PAM authentication, and diff output to the following:

        diff /etc/dovecot/conf.d/auth-system.conf /etc/dovecot/conf.d/auth-system.conf.bak

*OUTPUT:*

        15d14
        <   args = session=yes dovecot

* Backup dovecot another config file */etc/dovecot/conf.d/10-mail.conf*

        cp /etc/dovecot/conf.d/10-mail.conf /etc/dovecot/conf.d/10-mail.conf.bak

* Config */etc/dovecot/conf.d/10-mail.conf* to mail_location, and diff output to the following:

        diff /etc/dovecot/conf.d/10-mail.conf /etc/dovecot/conf.d/10-mail.conf.bak

*OUTPUT:*

        31d30
        < mail_location = mbox:~/mail:INBOX=/var/mail/%u

* More config settings refer to [dovecot v2.0 configuration](http://wiki2.dovecot.org/FrontPage?action=show#Dovecot_configuration)

* Reload the config files

        doveadm reload

* Always drop all the Iptables rules now.

* A *trouble* caused by file permission on dovecot

        ls -la /var/spool/mail/vitan

unless the directory mode is *600*, run

        chmod 600 /var/spool/mail/vitan

make sure that all the directorys(in */var/spool/mail/) mode is 600 for using MUA(Evolution or Thunderbird).

*REMEMBER* to change the dir mod after adding new user, if not, the new user fail to config MUA.

###MUA Evolution basic config###

Open the Evolution Interface and add the mail account step by step, leave others default except the followings:

* Receiving Email
        Server Type: IMAP
        Server: mail.example.com

* Sending Email
        Server: mail.example.com

* Now, *vitan* can receive and sending his mails on mail client Evolution.
