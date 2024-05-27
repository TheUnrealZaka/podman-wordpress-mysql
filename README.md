# Manual to install WordPress with MySQL using a Fedora image 

## Index
1. [Introduction](#1-Introduction)
2. [Prepare the System and Download the Base Image](#2-Prepare-the-System-and-Download-the-Base-Image)
3. [Creation of the Containerfile and Image Building](#3-Creation-of-the-Containerfile-and-Image-Building)
4. [Execution and Configuration of the Pods](#4-Execution-and-Configuration-of-the-Pods) 

### 1. Introduction 

Podman is a container management tool that allows the development and deployment of containers without the need for a central daemon. This final practice aims to deepen the concepts of containers through the creation, management, and deployment of application containers using Podman. 

### 2. Prepare the System and Download the Base Image 

#### Podman Installation
On Fedora, Podman can be installed using the following command: 

```bash
sudo dnf install -y podman
``` 

#### Verify the installation: 

```bash
podman --version
``` 

#### Download the base image of Fedora
```bash
podman pull fedora
``` 

### 3. Creation of the Containerfile and Image Building 

We will create two directories as we will need to have two Containerfiles in different locations. 

```bash
mkdir WordPress
mkdir MySQL
``` 

We will create the Containerfile for WordPress:
```bash
nano WordPress/Containerfile
``` 

Inside we will add the following content: 

```Dockerfile
FROM fedora:latest 

# Install Apache, PHP and other necessary packages for WordPress
RUN dnf -y update && dnf -y install httpd php php-mysqlnd php-xml php-json php-gd php-mbstring php-fpm unzip wget 

# Prepare the directory for the PHP-FPM socket
RUN mkdir -p /run/php-fpm && chown -R apache:apache /run/php-fpm 

# Download and install WordPress
RUN wget https://wordpress.org/latest.tar.gz -P /var/www/html/ && \
    tar -xvzf /var/www/html/latest.tar.gz -C /var/www/html/ && \
    rm /var/www/html/latest.tar.gz && mv /var/www/html/wordpress/* /var/www/html/

# Copy custom PHP configuration file
COPY wp-config.php /var/www/html/ 

# Copy PHP configuration 

COPY php.ini /etc/php.ini 

# Open port 80
EXPOSE 80 

# Configure and start Apache and PHP-FPM, ensuring socket permissions
CMD ["sh", "-c", "php-fpm && chown apache:apache /run/php-fpm/www.sock && chmod 660 /run/php-fpm/www.sock && /usr/sbin/httpd -D FOREGROUND"]
``` 

We will create the config.php file, it is important to create it inside the WordPress folder:
```bash
nano WordPress/wp-config.php
``` 

And we copy the following content: 

```php
<?php
/**
* The base configuration for WordPress
*
* The wp-config.php creation script uses this file during the installation.
* You don't have to use the website, you can copy this file to "wp-config.php"
* and fill in the values.
*
* This file contains the following configurations:
*
* * Database settings
* * Secret keys
* * Database table prefix
* * ABSPATH
*
* @link https://wordpress.org/documentation/article/editing-wp-config-php/
*
* @package WordPress
*/ 

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'wordpress' ); 

/** Database username */
define( 'DB_USER', 'root' ); 

/** Database password */
define( 'DB_PASSWORD', 'root' ); 

/** Database hostname */
define( 'DB_HOST', '127.0.0.1' ); 

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' ); 

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' ); 

/**#@+
* Authentication unique keys and salts.
*
* Change these to different unique phrases! You can generate these using
* the {@link https://api.wordpress.org/secret-key/1.1/salt/ WordPress.org secret-key service}.
*
* You can change these at any point in time to invalidate all existing cookies.
* This will force all users to have to log in again.
*
* @since 2.6.0
*/ 

define( 'AUTH_KEY',         '' );
define( 'SECURE_AUTH_KEY',  '' );
define( 'LOGGED_IN_KEY',    '' );
define( 'NONCE_KEY',        '' );
define( 'AUTH_SALT',        '' );
define( 'SECURE_AUTH_SALT', '' );
define( 'LOGGED_IN_SALT',   '' );
define( 'NONCE_SALT',       '' ); 

/**#@-*/ 

/**
* WordPress database table prefix.
*
* You can have multiple installations in one database if you give each
* a unique prefix. Only numbers, letters, and underscores please!
*/
$table_prefix = 'wp_'; 

/**
* For developers: WordPress debugging mode.
*
* Change this to true to enable the display of notices during development.
* It is strongly recommended that plugin and theme developers use WP_DEBUG
* in their development environments.
*
* For information on other constants that can be used for debugging,
* visit the documentation.
*
* @link https://wordpress.org/documentation/article/debugging-in-wordpress/
*/
define( 'WP_DEBUG', false ); 

/* Add any custom values between this line and the "stop editing" line. */



/* That's all, stop editing! Happy publishing. */ 

/** Absolute path to the WordPress directory. */
if ( ! defined( 'ABSPATH' ) ) {
define( 'ABSPATH', __DIR__ . '/' );
} 

/** Sets up WordPress vars and included files. */
require_once ABSPATH . 'wp-settings.php';
``` 

Also we will create a config for PHP: 

```bash
nano WordPress/php.ini
``` 

And we copy the following content: 

```
Maximize the memory limit for PHP scripts
memory_limit = 256M
; Increase the maximum size of uploaded files
upload_max_filesize = 64M
; Increase the maximum size of POST data that PHP will accept
post_max_size = 64M
; Ensure that cookies are used for the session instead of passing the session ID in the URL
session.use_cookies = 1
session.use_only_cookies = 1
; Set the error handler to display less information in production
display_errors = Off
log_errors = On
error_reporting = E_ALL & ~E_DEPRECATED & ~E_STRICT
Set default timezone used by all date/time functions
date.timezone = "Europe/Madrid" ; Increase the maximum time for the date to "Europe/Madrid".
; Increase the maximum execution time of PHP scripts
max_execution_time = 300
; Specific settings to improve security
expose_php = Off
``` 

We will do the same for MySQL
```bash
nano MySQL/Containerfile
```
**Containerfile for MySQL:**
```Dockerfile
# Use the base image of Ubuntu
FROM fedora:latest 

# Install MySQL
RUN dnf update && \
    dnf install -y mysql-server 

# Prepare the data directory
RUN mkdir -p /var/lib/mysql/ && \
    chown mysql:mysql /var/lib/mysql/ 

# Copy the initialization script
COPY init-mysql.sh /usr/local/bin/
RUN chmod +x /usr/local/bin/init-mysql.sh 

# Expose port 3306
EXPOSE 3306 

# Set the initialization script as the default command
CMD ["init-mysql.sh"]
``` 

We should create the script, important to create it inside the MySQL folder
```bash
nano MySQL/init-mysql.sh
``` 

And we copy the following content
```sh
#!/bin/bash
# Initialize the database if the directory is empty
if [ -z "$(ls -A /var/lib/mysql)" ]; then
    echo "Initializing the database..."
    mysqld --initialize --user=mysql --datadir=/var/lib/mysql/
    echo "Database initialized."
fi 

# Start the MySQL server
echo "Starting MySQL..."
exec mysqld --datadir='/var/lib/mysql/' --user=mysql
``` 

**Building the images:** 

First, we will build the WordPress image with the name mywordpress:
```bash
podman build -t mywordpress ./WordPress
```
Then the same with MySQL:
```bash
podman build -t mymysql ./MySQL
``` 

### 4. Execution and Configuration of the Pods 

**Creation of the pod and execution of the containers:** 

```bash
podman pod create --name mypod -p 8080:80 -p 33060:3306
podman run --pod mypod -it --name mywordpress -d mywordpress
podman run --pod mypod -it --privileged --name mymysql -d mymysql
``` 

First, we will enter the MySQL container: 

```bash
podman exec -it mymysql /bin/bash
``` 

Once inside, we will execute the following command to obtain the temporary password that MySQL uses:
```bash
grep "temporary password" /var/log/mysql/error.log 
```

Once we have this password, we will enter MySQL using the following command:
```bash
mysql -u root -p
```

Inside MySQL, we will execute the following commands:
```bash
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root'; 

CREATE DATABASE wordpress; 

FLUSH PRIVILEGES;

exit

exit
``` 

After configuring MySQL, we will exit the container and execute the following command:
```bash
podman commit mymysql myfinalmysql
``` 

After that, we will restart the WordPress container to fix the database connection:
```bash
podman restart mywordpress
``` 

Finally, we will commit:
```bash
podman commit mywordpress myfinalwordpress
```
Now we will enter the browser and enter this URL: http://localhost:8080 to enter WordPress, where we must complete the installation. 

For any problems you can have with the steps, you can contact me on [Discord](https://discord.com/users/1114850055128629350) or write an email to: contact@theunrealzaka.me 

Made with ❤ by [TheUnrealZaka](https://www.theunrealzaka.me)
