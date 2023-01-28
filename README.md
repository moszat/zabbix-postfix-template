# zabbix-postfix-template
Zabbix template for Postfix SMTP server

Works for Zabbix 4.x

Forked from https://github.com/kstka/zabbix-postfix-template

# Requirements
* [pflogsum](http://jimsun.linxnet.com/postfix_contrib.html)
* [pygtail](https://pypi.org/project/pygtail/)

# Installation on Zabbix client
    # for Ubuntu / Debian
    apt-get install pgtail pflogsumm

    # ! check the following paths in zabbix-postfix-stats.sh
    # MAILLOG=/var/log/mail.log
    # PFLOGSUMM=/usr/sbin/pflogsumm
    # PYGTAIL=/bin/pygtail

    # install plugin

    cp zabbix-postfix-stats.sh /usr/bin/
    chmod +x /usr/bin/zabbix-postfix-stats.sh

    # make readable the maillog for zabbix

        # method 1 - add zabbix user into the adm group
	
	    usermod -a -G adm zabbix

        # method 2 - create unique logrotate conf (if it does not exist) for maillog as follows, important is: create 644

	    echo "/var/log/mail.log {
	    weekly
	    rotate 4
	    create
	    compress
	    delaycompress
	    create 644 syslog adm
	    }" > /etc/logrotate.d/mail

	    #until it takes effect
            chmod 644 /var/log/mail.log

        # method 3 - use sudo

	    echo "Defaults:zabbix !requiretty
	    zabbix ALL=(ALL) NOPASSWD: /usr/bin/zabbix-postfix-stats.sh" >> /etc/sudoers

            # modify postfix.conf as follows

	    UserParameter=postfix[*],/usr/bin/zabbix-postfix-stats.sh $1	-> UserParameter=postfix[*],sudo /usr/bin/zabbix-postfix-stats.sh $1
	    UserParameter=postfix.update_data,/usr/bin/zabbix-postfix-stats.sh	-> UserParameter=postfix.update_data,sudo /usr/bin/zabbix-postfix-stats.sh

    # for Zabbix agent
    cp postfix.conf /etc/zabbix/zabbix_agentd.d/
    systemctl restart zabbix-agent

    # for Zabbix agent2
    cp postfix.conf /etc/zabbix/zabbix_agent2.d/plugins.d/
    systemctl restart zabbix-agent2

Finally import template_app_zabbix.xml and attach it to your host