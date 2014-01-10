whm-cpanel-hide-git-repos
=========================

I use git for deployment to staging and production WHM/cPanel servers. 

I've recently discovered that the .git folder inside of the web root directory was web accessible, thereby exposing my wordpress installs, database username & passwords etc.:( 

The solution? Reconfigure Apache to deny access to any directories or files starting with .git. 

This can be done on a site by site basis by creating a .htaccess file that denies access to the .git directory.

However, I don't like that as it's too easy to forget. 

I'd much rather do it on a global level by modifying the httpd.conf

## Modifying WHM / cPanel Apache httpd.conf

WHM doesn't have a single httpd.conf that we can modify.   Instead it builds the main apache.conf from a bunch of templates documented here: 

[EasyApache documentation](http://docs.cpanel.net/twiki/bin/view/EasyApache3/)

The specific documentation we want is how to modify the VirtualHost directives for each website. 

[Changes Contained Within A Virutal Host Directive](http://docs.cpanel.net/twiki/bin/view/EasyApache3/InsideVHost)

We need to modify two files. 

* Copy /var/cpanel/templates/apache2/vhost.default to vhost.local 
* Copy /var/cpanel/templates/apache2/ssl_vhost.default to ssl_vhost.local 

Edit each file and add the following into the <Virualhost> directive. 

I do it after **ServerAdmin**

```# do not allow .git version control files to be issued
<Directorymatch "^/.*/\.git+/">
  Order deny,allow
  Deny from all
</Directorymatch>
<Files ~ "^\.git">
    Order allow,deny
    Deny from all 
</Files>```

Next, make a backup of /usr/local/apache/conf/http.conf just in case anything goes wrong so you can immediately restore it. 

Then run:

```
/scripts/rebuildhttpdconf
service http restart
```

Immediately test access to a .git directory on a ssl and non ssl site.  Any problems, restore /usr/local/apache/conf/httpd.conf from your backup and restart httpd. 




