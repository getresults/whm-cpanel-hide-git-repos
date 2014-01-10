## Deny web access to .git directories on WHM/cPanel


### The problem

I use git for deployment to staging and production WHM/cPanel servers. 

I've recently discovered that the .git folder inside of the web root directory was web accessible, thereby exposing my wordpress installs, database username & passwords etc.:( 

### The solution

Reconfigure Apache to deny access to any directories or files starting with .git. 

This can be done on a site by site basis by creating a .htaccess file that denies access to the .git directory.

However, I don't like that as it's too easy to forget when creating / cloning a repo.

I'd much rather do it on a global level by modifying the httpd.conf

### Modifying WHM / cPanel Apache httpd.conf

WHM doesn't have a single httpd.conf that we can modify.   Instead it builds the main apache.conf from a bunch of templates documented here: 

[EasyApache documentation](http://docs.cpanel.net/twiki/bin/view/EasyApache3/)

The specific documentation we want is how to modify the VirtualHost directives for each website. 

[Changes Contained Within A Virutal Host Directive](http://docs.cpanel.net/twiki/bin/view/EasyApache/EasyApacheChangesWithinVirtualHost)

We need to copy and modify two template files: 

```
cp /var/cpanel/templates/apache2/vhost.default to vhost.local 
cp /var/cpanel/templates/apache2/ssl_vhost.default to ssl_vhost.local 
```

Edit each file and add the following into the <Virualhost> directive. 

I do it after **ServerAdmin**

```
# do not allow .git version control files to be issued
<Directorymatch "^/.*/\.git+/">
  Order deny,allow
  Deny from all
</Directorymatch>
<Files ~ "^\.git">
    Order allow,deny
    Deny from all 
</Files>
```

Next, make a backup of /usr/local/apache/conf/http.conf just in case anything goes wrong so you can immediately restore it. 

Then run:

```
/scripts/rebuildhttpdconf
service http restart
```

Test access to a .git directory on a ssl and non ssl site.  

If you have any problems, restore /usr/local/apache/conf/httpd.conf from your backup and restart httpd, then re-edit the vhost.local and ssl_vhost.local templates you created. 


### From the EasyApache Documentation:

```
Custom templates that will apply to all virtual hosts when rebuilding an existing Apache configuration
To create custom template files that affect all virtual hosts:
Create a copy of one or more of the following files:
Apache 1 without SSL — /var/cpanel/templates/apache1/vhost.default
Apache 2 without SSL — /var/cpanel/templates/apache2/vhost.default
Apache 1 with SSL — /var/cpanel/templates/apache1/ssl_vhost.default
Apache 2 with SSL — /var/cpanel/templates/apache2/ssl_vhost.default
Rename the copied file to one of the following:
vhost.local — use this if you copied vhost.default.
ssl_vhost.local — use this if you copied ssl_vhost.default.
Edit the *.local files to make the changes you would like to your virtual host configuration.
PICK Important: This method affects all of your virtual hosts as the .local file(s) will be used in place of the .default file(s).
```




