---
layout: post
title: "Apache Web Server: 8 Tipps to secure your server"
category: "Security"
---

In year 2016 Apache runs [50%](http://de.statista.com/statistik/daten/studie/181588/umfrage/marktanteil-der-meistgenutzten-webserver/) of all websites on the internet. More than 440.000 new websites are [launched daily](https://www.quora.com/How-many-websites-are-created-each-year-month-week) while about 45.000 [are hacked again that day](http://www.forbes.com/sites/jameslyne/2013/09/06/30000-web-sites-hacked-a-day-how-do-you-host-yours/#44c1de113a8c). Not relevant for your site? Please think again. We will show you how to configure your freshly obtained hosting package or server with a secure Apache server.

## Impacts of lacking server security

Many server owners, especially when hosting small websites, are not aware of the fact that this terrain can be very hard and dangerous. Small servers and hosting packets can be obtained within minutes for a few bucks a year to run blogs, forums, shops or small websites. In most cases, owners do not have any clue about what is going on with their servers and how to avoid getting hacked. They do not configure web servers and other services at all. It only takes hours for malicious crawlers, portscanners and other software to spot these possibilities. If security threats are found, servers may get hacked in only a few seconds, automatically.
“Not that big of a deal, we ain’t nothing to hide and it doesn’t harm our business. Calm down”. I’m pretty used to hear sentences like this from customers. This is foolish and pure greenness. Every day your server is not in your control (or also in control of others) can have a lot of impact:

- Websites and email addresses being not available to potential customers
- Penalties from Google and other search engines
- Distribution of thousands of spam mails via your domain
- Domain/IP being known to blacklists and all other impacts that derive from that
- Passive criminality by approving possible denial of service and other attacks and bunches of spam

This list has no end. That’s not all possible trouble by far. To overcome these impacts, hosting should be only done by experienced and competent persons. No matter if you dive into the topics on your own very dedicated or want to outsource it to another party. In any case, get help by an expert if you don’t know what you’re doing or what to do.


## Apache – Always baiting the bees

For the operations of websites, web servers are the first point being attacked once you expose them to the internet. In this case Apache 2.x offers its services via http and https (port 80 and 443). During the next sections we will show you how to properly configure your Apache.

### 1 – Disable Server Side Includes and CGI

Most likeable you do not need these features, so let’s disable them. We should be always aware of reducing the amount of possible risks. So are these. Therefor we will edit the main config.

``` shell
sudo nano /etc/apache2/apache2.conf
```

Add or edit (depending on if they are present) the following parameters:

``` apache
Options -FollowSymLinks -Includes -ExecCGI
AllowOverride None
Require all granted
```

### 2 – Limit request size

Per default the body size of incoming HTTP/HTTPS requests is not event limited. This makes it easily possible to overload the server. You do not even need large DoS attacks for that. We will limit the request body size to a maximum of 200kb in the apache2.conf file.

``` apache
LimitRequestBody 204800
``` 

### 3 – Deactivate unneeded modules

Like any other software and service, Apache comes with many and mighty default functionality. We should disable all unneeded modules to prevent exploits from using them for attacks. Let us first see which modules are currently active.

``` shell
sudo ls /etc/apache2/mods-enabled/
```

"status" and "autoindex" for instance are not needed. We will disable them and restart Apache afterwards.

``` shell
sudo a2dismod autoindex
sudo a2dismod status
sudo service apache2 restart
```

### 4 – Hide Apaches Version to visitors

Per default, users can see the version of Apache and the operating system we are running. This may be cool but can be dangerous due to the reason that known attacks and exploits are often version specific. To get to know which versions a server is running, is part of a potential attackers analysis. We should not want to offer that.

``` shell
sudo nano /etc/apache2/conf-enabled/security.conf
```

We should disable that in our security configuration and restart the server afterwards with ‘service apache2 restart’.

``` apache
ServerSignature Off
ServerTokens Prod
```

### 5 – Deactivate Directory Browsing and Symbolic Links

Directory browsing or listing make it possible for us to browse the file system on exposed directories. It offers navigation through the directories. Possible attackers can analyse directories, files and applications regarding heavier security leaks for their attacks. Symbolic links are practically of course but can give a chance to access linked programs and scripts on other locations of the server.
Apache directory browsing – useful feature or only dangerous?

We edit the following options in the `apache2.conf` file and restart Apache, again.

``` apache
Options -FollowSymLinks
AllowOverride None
Require all granted
```

If we try to repeat the above screenshot by accessing the directory in our browser again, we should see a "Forbidden" message. This expresses pretty much what it should.

### 6 – Keep your software up to date

This is a general component of server operations. If there is any update, there is a reason for that. Daily developers and companies are facing new security issues and are about to fix them as soon as possible. This fixes are only useful, if you update your software artefacts, of course. Luckily there is something like package managers in Linux, so there is no lazy excuses here. In some Linux distributions we are also getting hints about updates when a new SSH session was started.

``` shell
sudo apt-get upgrade
sudo apt-get update
```

### 7 – Make use of ModSecurity

ModSecurity is a very nice and useful Apache module. It knows basic security measures against well known attacking schemes like SQL injections, Cross-Site-Scripting, Brute Force and Session Hijacking. It's easy to install, configure and maintain.

``` shell
sudo apt-get install libapache2-modsecurity
```

Once the module is installed, we should move the recommended config file, that is already included, to the correct config-files destination. Afterwards we enable the module.

``` shell
sudo mv /etc/modsecurity/modsecurity.conf-recommended /etc/modsecurity/modsecurity.conf
sudo nano /etc/modsecurity/modsecurity.conf # Edit/Add 'SecRuleEngine On' in this file
```

ModSecurity ships with a big bunch of security rules which are just waiting to get applied in Apache. That is why we extend the '/etc/apache2/mods-enabled/security2.conf' file with the following:

``` apache
IncludeOptional /etc/modsecurity/*.conf
IncludeOptional "/usr/share/modsecurity-crs/*.conf"
IncludeOptional "/usr/share/modsecurity-crs/base_rules/*.conf
```

This module is giving us huge progress in securing the server. With 'service apache2 restart' we will restart the server again to apply our changes.

### 8 – Secure admin areas of applications

Of course we should only operate secure applications on our web servers but furthermore we can and should also protect their backends. I am talking about popular CMS-, shop- and blog-applications like WordPress, Joomla, Shopware and co. At first they have to be configured in a secure way themselves (correct file permissions, strong passwords, secure databases, v-host configurations). Afterwards we make us of Apaches own security measures and protect login areas which we only use for ourselves. In-built concepts like .htaccess and .htpasswd files enable us to do so. The .htpasswd file defines all possible users and passwords while .htaccess defines the authentication kind and meta data. To create them, I would like to [redirect you to one of the available tools](http://www.web2generators.com/apache-tools/htpasswd-generator) out there. It makes much sense to secure e.g. WordPress' 'wp-admin' folder using this approach.


## Conclusion

Using a well configured and therefor safe Apache web server, you will be able to sleep better in the future, again. Nobody can guarantee, that these recommended measures are enough. But you're 99% safe with them and better of than most of the insecure server owners. Furthermore the security is heavily influenced by other services like databases, SSH, mail and applications that you're hosting. Please also harden and secure them in a good way.
