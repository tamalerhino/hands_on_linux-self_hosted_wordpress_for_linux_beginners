# Install Required software for our Hosting Platform
On your Webserver (as **root**)

### 1. Update and Upgrade software repos.

### 2. Install the following software
    1. mysql-server
    2. nginx php-mysql php-fpm monit

### 3. Start the following software
```
nginx php7.4-fpm monit
```
### 4. Enable the following software
```
mysql nginx php7.4-fpm monit
```

# Basic nginx Configuration

### 1. Go to `/etc/nginx/`.

### 2. Rename `nginx.conf` to `nginx.conf.ORIG` to create a backup.

### 3. Create a new `nginx.conf` file and copy the follwing data into the file:


    user  www-data;
    worker_processes  auto;

    pid /run/nginx.pid;

    events {
        worker_connections  1024;
    }

    http {
        include       mime.types;
        default_type  application/octet-stream;

        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';
        error_log  /var/log/nginx_error.log error;
        #access_log  logs/access.log  main;

        sendfile        on;
        #tcp_nopush     on;

        keepalive_timeout  65;

        # SSL
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # no sslv3 (poodle etc.)
        ssl_prefer_server_ciphers on;

        # Gzip Settings
        gzip on;
        gzip_disable "msie6";
        gzip_vary on;
        gzip_min_length 512;
        gzip_types text/plain text/html application/x-javascript text/javascript application/javascript text/xml text/css application/font-sfnt;

        fastcgi_cache_path /usr/share/nginx/cache/fcgi levels=1:2 keys_zone=microcache:10m max_size=1024m inactive=1h;

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
    }

### 4. Now create the cache directory: `/usr/share/nginx/cache/fcgi`


## Check for configuration errors

Test the nginx configuration (or attempt to reload nginx) to make sure you don't have any errors:

Test the configuration without restarting/reloading the nginx service:

```
nginx -t
```
Or **reload** nginx, if it's running and you want to apply the new configuration right away

*Important*: When testing an nginx config, `[warn]`s are okay -- `[err]` means there's a problem that you need to troubleshoot and fix, and will prevent the nginx service from starting successfully.

# Basic php-fpm and PHP configuration
### 1. Install the PHP extensions we need for WordPress

    php-json php-xmlrpc php-curl php-gd php-xml php-mbstring

### 2. Now ensure that the directory for php-fpm sockets exists: `/run/php-fpm` if it does not exist create it.

### 3. Backup the original `/etc/php/7.4/fpm/php-fpm.conf` by renaming it to `/etc/php/7.4/fpm/php-fpm.conf.ORIG`

### 4. Create a new file `/etc/php/7.4/fpm/php-fpm.conf` and copy the following content:

    [global]
    pid = /run/php/php7.4-fpm.pid
    error_log = /var/log/php-fpm.log
    include=/etc/php/7.4/fpm/pool.d/*.conf

### 5. Remove(delete) the original (default) pool file: `/etc/php/7.4/fpm/pool.d/www.conf`

### 6. Create a *new* default pool configuration at /etc/php/7.4/fpm/pool.d/www.conf with the following content:

    [default]
    security.limit_extensions = .php
    listen = /run/php/yourserverhostname.sock
    listen.owner = www-data
    listen.group = www-data
    listen.mode = 0660
    user = www-data
    group = www-data
    pm = dynamic
    pm.max_children = 75
    pm.start_servers = 8
    pm.min_spare_servers = 5
    pm.max_spare_servers = 20
    pm.max_requests = 500

This new default pool won't be used, but I'm creating it here to prevent new students from getting confused when they try to restart php-fpm at this point in the course. We'll remove this file again later -- you don't REALLY need it.

### 7. Backup the original `/etc/php/7.4/fpm/php.ini` file by renaming it to `/etc/php/7.4/fpm/php.ini.ORIG`

### 8. Create a new `/etc/php/7.4/fpm/php.ini` file and copy the following content:

    [PHP]
    engine = On
    short_open_tag = Off
    asp_tags = Off
    precision = 14
    output_buffering = 4096
    zlib.output_compression = Off
    implicit_flush = Off
    unserialize_callback_func =
    serialize_precision = 17
    disable_functions =
    disable_classes =
    zend.enable_gc = On
    expose_php = Off
    max_execution_time = 30
    max_input_time = 60
    memory_limit = 128M
    error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
    display_errors = Off
    display_startup_errors = Off
    log_errors = On
    log_errors_max_len = 1024
    ignore_repeated_errors = Off
    ignore_repeated_source = Off
    report_memleaks = On
    track_errors = Off
    html_errors = On
    variables_order = "GPCS"
    request_order = "GP"
    register_argc_argv = Off
    auto_globals_jit = On
    post_max_size = 8M
    auto_prepend_file =
    auto_append_file =
    default_mimetype = "text/html"
    default_charset = "UTF-8"
    doc_root =
    user_dir =
    enable_dl = Off
    file_uploads = On
    upload_max_filesize = 25M
    max_file_uploads = 20
    allow_url_fopen = On
    allow_url_include = Off
    default_socket_timeout = 60
    [CLI Server]
    cli_server.color = On
    [Date]
    [filter]
    [iconv]
    [intl]
    [sqlite]
    [sqlite3]
    [Pcre]
    [Pdo]
    [Pdo_mysql]
    pdo_mysql.cache_size = 2000
    pdo_mysql.default_socket=
    [Phar]
    [mail function]
    SMTP = localhost
    smtp_port = 25
    mail.add_x_header = On
    [SQL]
    sql.safe_mode = Off
    [ODBC]
    odbc.allow_persistent = On
    odbc.check_persistent = On
    odbc.max_persistent = -1
    odbc.max_links = -1
    odbc.defaultlrl = 4096
    odbc.defaultbinmode = 1
    [Interbase]
    ibase.allow_persistent = 1
    ibase.max_persistent = -1
    ibase.max_links = -1
    ibase.timestampformat = "%Y-%m-%d %H:%M:%S"
    ibase.dateformat = "%Y-%m-%d"
    ibase.timeformat = "%H:%M:%S"
    [MySQL]
    mysql.allow_local_infile = On
    mysql.allow_persistent = On
    mysql.cache_size = 2000
    mysql.max_persistent = -1
    mysql.max_links = -1
    mysql.default_port =
    mysql.default_socket =
    mysql.default_host =
    mysql.default_user =
    mysql.default_password =
    mysql.connect_timeout = 60
    mysql.trace_mode = Off
    [MySQLi]
    mysqli.max_persistent = -1
    mysqli.allow_persistent = On
    mysqli.max_links = -1
    mysqli.cache_size = 2000
    mysqli.default_port = 3306
    mysqli.default_socket =
    mysqli.default_host =
    mysqli.default_user =
    mysqli.default_pw =
    mysqli.reconnect = Off
    [mysqlnd]
    mysqlnd.collect_statistics = On
    mysqlnd.collect_memory_statistics = Off
    [OCI8]
    [PostgreSQL]
    pgsql.allow_persistent = On
    pgsql.auto_reset_persistent = Off
    pgsql.max_persistent = -1
    pgsql.max_links = -1
    pgsql.ignore_notice = 0
    pgsql.log_notice = 0
    [Sybase-CT]
    sybct.allow_persistent = On
    sybct.max_persistent = -1
    sybct.max_links = -1
    sybct.min_server_severity = 10
    sybct.min_client_severity = 10
    [bcmath]
    bcmath.scale = 0
    [browscap]
    [Session]
    session.save_handler = files
    session.use_strict_mode = 0
    session.use_cookies = 1
    session.use_only_cookies = 1
    session.name = PHPSESSID
    session.auto_start = 0
    session.cookie_lifetime = 0
    session.cookie_path = /
    session.cookie_domain =
    session.cookie_httponly =
    session.serialize_handler = php
    session.gc_probability = 1
    session.gc_divisor = 1000
    session.gc_maxlifetime = 1440
    session.referer_check =
    session.cache_limiter = nocache
    session.cache_expire = 180
    session.use_trans_sid = 0
    session.hash_function = 0
    session.hash_bits_per_character = 5
    url_rewriter.tags = "a=href,area=href,frame=src,input=src,form=fakeentry"
    [MSSQL]
    mssql.allow_persistent = On
    mssql.max_persistent = -1
    mssql.max_links = -1
    mssql.min_error_severity = 10
    mssql.min_message_severity = 10
    mssql.compatibility_mode = Off
    mssql.secure_connection = Off
    [Assertion]
    [COM]
    [mbstring]
    [gd]
    [exif]
    [Tidy]
    tidy.clean_output = Off
    [soap]
    soap.wsdl_cache_enabled=1
    soap.wsdl_cache_dir="/tmp"
    soap.wsdl_cache_ttl=86400
    soap.wsdl_cache_limit = 5
    [sysvshm]
    [ldap]
    ldap.max_links = -1
    [dba]
    [opcache]
    [curl]
    [openssl]


### 9. Restart the servie `php7.4-fpm`

# MySQL Setup

Perform the following actions as the root user.

### 1. Create mysql root pass

    echo -n @ && cat /dev/urandom | env LC_CTYPE=C tr -dc [:alnum:] | head -c 15 && echo

This password should pass the mysql validation plugin's requirements for a 'strong' password, should you choose to use that.

**SAVE THIS PASSWORD TO A TEXTFILE OR PASSWORD MANAGER RIGHT NOW! (not on your WordPress VM)**. You'll need it later.

*Note* You might notice the 'echo -n @' at the beginning of this password generation command. This is to get around the new password policy in MySQL version 5.5.6 and newer. Yes, it is predictable and the extra character doesn't add security, but it also doesn't make it any *less* secure. Being a sysadmin is also about being pragmatic.

### 2. Run mysql_secure_installation script

    /usr/bin/mysql_secure_installation

Answer 'y' to all questions and set the password policy (second question) to 0-3 (whichever one you prefer; it doesn't really affect you in this course). If this is your first time through the course, you can set the password security to 0 to ensure mysql doesn't surprise you when you're testing/experimenting with passwords.

### 3. Restart `mysql` service

Not strictly necessary but it's good to practice working with services.

# Set up a WordPress Site

Replace all instances of 'tutorialinux' with the system username that you'll use for this site. It makes sense to use a truncated version of your domain for this, e.g. for 'tutorialinux.com' I would use 'tutorialinux'.


### 1. Create a system user for this site called `tutorialinux`

### 2. Create the directory `/home/tutorialinux/logs`

### 3. Change the Owner of `/home/tutorialinux/logs/` to `tutorialinux`

### 4. Change the Group of /home/tutorialinux/logs/` to `www-data`

### 5. Create the file `/etc/nginx/conf.d/tutorialinux.conf` and copy the follwing data into it:

    server {
        listen       80;
        server_name  www.tutorialinux.com;

        client_max_body_size 20m;

        index index.php index.html index.htm;
        root   /home/tutorialinux/public_html;

        location / {
            try_files $uri $uri/ /index.php?q=$uri&$args;
        }

        # pass the PHP scripts to FastCGI server
        location ~ \.php$ {
                # Basic
                try_files $uri =404;
                fastcgi_index index.php;

                # Create a no cache flag
                set $no_cache "";

                # Don't ever cache POSTs
                if ($request_method = POST) {
                  set $no_cache 1;
                }

                # Admin stuff should not be cached
                if ($request_uri ~* "/(wp-admin/|wp-login.php)") {
                  set $no_cache 1;
                }

                # WooCommerce stuff should not be cached
                if ($request_uri ~* "/store.*|/cart.*|/my-account.*|/checkout.*|/addons.*") {
                  set $no_cache 1;
                }

                # If we are the admin, make sure nothing
                # gets cached, so no weird stuff will happen
                if ($http_cookie ~* "wordpress_logged_in_") {
                  set $no_cache 1;
                }

                # Cache and cache bypass handling
                fastcgi_no_cache $no_cache;
                fastcgi_cache_bypass $no_cache;
                fastcgi_cache microcache;
                fastcgi_cache_key $scheme$request_method$server_name$request_uri$args;
                fastcgi_cache_valid 200 60m;
                fastcgi_cache_valid 404 10m;
                fastcgi_cache_use_stale updating;


                # General FastCGI handling
                fastcgi_pass unix:/var/run/php/tutorialinux.sock;
                fastcgi_pass_header Set-Cookie;
                fastcgi_pass_header Cookie;
                fastcgi_ignore_headers Cache-Control Expires Set-Cookie;
                fastcgi_split_path_info ^(.+\.php)(/.+)$;
                fastcgi_param SCRIPT_FILENAME $request_filename;
                fastcgi_intercept_errors on;
                include fastcgi_params;         
        }

        location ~* \.(js|css|png|jpg|jpeg|gif|ico|woff|ttf|svg|otf)$ {
                expires 30d;
                add_header Pragma public;
                add_header Cache-Control "public";
                access_log off;
        }

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }

    server {
        listen       80;
        server_name  tutorialinux.com;
        rewrite ^/(.*)$ http://www.tutorialinux.com/$1 permanent;
    }


### 6. Disable default nginx vhost (only the first time you set up a website) by removing(deleting) `/etc/nginx/sites-enabled/default`

### 7. Create the new php-fpm vhost pool config file by creating the file `/etc/php/7.4/fpm/pool.d/tutorialinux.conf` and copying the following data into it. 

*Replace all occurrences of "tutorialinux" in the configuration file content below with your site name.*


    [tutorialinux]
    listen = /var/run/php/tutorialinux.sock
    listen.owner = tutorialinux
    listen.group = www-data
    listen.mode = 0660
    user = tutorialinux
    group = www-data
    pm = dynamic
    pm.max_children = 75
    pm.start_servers = 8
    pm.min_spare_servers = 5
    pm.max_spare_servers = 20
    pm.max_requests = 500

    php_admin_value[upload_max_filesize] = 25M
    php_admin_value[error_log] = /home/tutorialinux/logs/phpfpm_error.log
    php_admin_value[open_basedir] = /home/tutorialinux:/tmp


## Clean up the original php-fpm pool config file

### 8. We've kept this around just to prevent errors while restarting php-fpm. Since we just created a new php-fpm pool config file, let's clean the old one by deleting `/etc/php/7.4/fpm/pool.d/www.conf`

### 9. Create the php-fpm logfile `/home/tutorialinux/logs/phpfpm_error.log` as the user `tutorialinux` using `sudo -u tutorialinux`

# Create Site Database + DB User

### 1. First, create a new mysql password for your wordpress site. You can do this in any Linux shell, local or remote:

    # Create password
    echo -n @ && cat /dev/urandom | env LC_CTYPE=C tr -dc [:alnum:] | head -c 15 && echo

### 2. Now log into your mysql database with the root account, using the password you created and saved earlier (during the mysql_secure_installation script run):

    mysql -u root -p

This will prompt you for the MySQL root user’s password, and then give you a database shell. This shell will let you enter the following commands to create the WordPress database and user, along with appropriate permissions. Swap out ‘yoursite’ for your truncated domain name. This name can't contain any punctuation or special characters.

### 3. Replace `chooseapassword` with the strong password that you just created with the shell command above, and `tutorialinux` with your site name.

    # Log into mysql
    CREATE DATABASE tutorialinux;
    CREATE USER 'tutorialinux'@'localhost' IDENTIFIED BY 'chooseapassword';
    GRANT ALL PRIVILEGES ON tutorialinux.* TO tutorialinux@localhost;
    FLUSH PRIVILEGES;


Great; you’re done! Hit *ctrl-d* to exit the MySQL shell.

# Install WordPress

Now it's time to actually download and install the WordPress application.


## Download WordPress

Become your site user (named tutorialinux in my case) and download the WordPress application:

### 1. Switch to User `tutorialinux`

### 2. Go to tutorialinux's home directory
    
### 3. Download the wordpress package:

    wget https://wordpress.org/latest.tar.gz

### 4. Extract Wordpress Archive (+ Clean Up)

    latest.tar.gz
### 5. Once you have untared(extracted it) delete:    
    latest.tar.gz

### 6. Rename the extracted 'wordpress' directory to `public_html`


### Exit the unprivileged user's shell and become root again 

    ctrl-d (+ENTER)


### 7. Set proper file permissions on your site files

Make sure you're in your user's *home/public_html* directory -- I'm still using the `tutorialinux` user here for illustration:

    go to `/home/tutorialinux/public_html`
    chown -R tutorialinux:www-data .
    find . -type d -exec chmod 755 {} \;
    find . -type f -exec chmod 644 {} \;


## 8. Restart your services

    php7.4-fpm
    nginx


## Optional: Set up DNS
Log into your registrar's dashboard (wherever you purchased your domain name, usually) and point your domain at the WordPress server's IP.


## Open the WordPress Installer in your Browser
Use your browser to navigate to the wordpress install page at http://example.com/wp-admin/setup-config.php (replace example.com with your chosen domain name).

If you don't have a domain name yet (or it's not set up with an A record to point at your server's IP address), then use this trick to 'fake' it on your local machine.

Make these changes on the local Linux machine you're using as a 'base' to go through this course, **not* on your remote web server that's running the WordPress site**:

### 1. open `/etc/hosts`

## 2. Add two lines like the following, with the IP and hostnames replaced by your WordPress server's IP address and your domain name, respectively:
```
81.7.14.132    tutorialinux.com
81.7.14.132    www.tutorialinux.com
```

This will trick applications (only on the local machine) to use this mapping, INSTEAD of the public DNS system, to resolve your hostname.

It's a great way of testing things locally, before modifying DNS records at your domain registrar (e.g. namecheap).


## ONCE YOU HAVE RUN THE WORDPRESS INSTALLER...

You'll be able to run the installer by navigating to your server IP address in a browser. Once you've done that...

### 3. Secure the wp-config.php file so other users can’t read DB credentials by schanging the mode of the `/home/tutorialinux/public_html/wp-config.php` to `640`