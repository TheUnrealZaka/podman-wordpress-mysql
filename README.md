# Manual to install WordPress with MySQL using an Ubuntu image

## Index
1. [Introduction](#1-Introduction)
2. [Prepare the System and Download the Base Image](#2-Prepare-the-System-and-Download-the-Base-Image)
3. [Creation of the Containerfile and Image Building](#3-Creation-of-the-Containerfile-and-Image-Building)
4. [Execution and Configuration of the Pods](#4-Execution-and-Configuration-of-the-Pods)

### 1. Introduction

Podman is a container management tool that allows the development and deployment of containers without the need for a central daemon. This final practice aims to deepen the concepts of containers through the creation, management, and deployment of application containers using Podman.

### 2. Prepare the System and Download the Base Image

#### Podman Installation
On Ubuntu, Podman can be installed using the following command:

```bash
   sudo apt-get install -y podman
```

#### Verify the installation:

```bash
   podman --version
```

#### Download the base image of Ubuntu
```bash
   podman pull ubuntu
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
FROM ubuntu:latest

# Install Apache, PHP, PHP-FPM and other necessary packages for WordPress
RUN apt-get update && \
    apt-get install -y apache2 php libapache2-mod-php mysql-client php-mysql php-xml php-json php-gd php-mbstring php-fpm unzip wget

# Prepare the directory for the PHP-FPM socket
RUN mkdir -p /run/php && \
    chown -R www-data:www-data /run/php

# Download and install WordPress
RUN wget https://wordpress.org/latest.tar.gz -P /var/www/html/ && \
    tar -xvzf /var/www/html/latest.tar.gz -C /var/www/html/ && \
    rm /var/www/html/latest.tar.gz

# Copy custom PHP configuration file
COPY config.php /var/www/html/wordpress/

# Open port 80
EXPOSE 80

# Configure and start Apache and PHP-FPM, ensuring socket permissions
CMD ["sh", "-c", "service php7.4-fpm start && a2enmod proxy_fcgi setenvif && a2enconf php7.4-fpm && service apache2 restart && /bin/bash"]
```

We will create the config.php file, it is important to create it inside the WordPress folder:
```bash
nano WordPress/config.php
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
define( 'DB_HOST', 'localhost' );

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

define( 'AUTH_KEY',         '-=hF?c-)-me${4S,5)COb^B:a#t2;HW|c4+y=UP}{xx*_ZNzCz@MQR5#2gxJ)S~I' );
define( 'SECURE_AUTH_KEY',  'V`r5n8.DnE^/e:Ev;uX4-(uLNf1$elOkjaVQmq!-u7nfsU:`<Btpx[T@MVMft&dA' );
define( 'LOGGED_IN_KEY',    'Vf]Vl|EnxG8]/Y]4(U4*%~4O2?%N7+Z_p=h uE$qg}oT:uB&iS[zrx{nR5 EC4(*' );
define( 'NONCE_KEY',        'lm1p-oaGxg0`-tf8@(4TrW3Kt.k-=^mq|FFip]D]6XyEeS]>>>FJUn?P3m6R0+`v' );
define( 'AUTH_SALT',        'P7y7&pq=GFiO:%Ysd=t|3P8E>jr4-g[Ct]6y1DgKwi[?H1{uSSGm<|SEELZ}t]{V' );
define( 'SECURE_AUTH_SALT', 'E!fBqH07bYh8p-2~XU?0YAy4)V{ V@sX#jI.?5]J2pFvg` PMWyN+nX-J.);[VeK' );
define( 'LOGGED_IN_SALT',   'IVS-X^<ziM1TR+]NC]yw)+0n#Ml8iqvo^*RvR(+H-d_KjYTKuf)}5<R>yE<xuBOV' );
define( 'NONCE_SALT',       '@V; PF{eG7hO&u4zAMcy3$.q)WP}M,XRVSW+EE/F:5&C6B^oq,_g7{Uv~vHX+oxi' );

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

We will do the same for MySQL
```bash
   nano MySQL/Containerfile
```
**Containerfile for MySQL:**
```Dockerfile
# Use the base image of Ubuntu
FROM ubuntu:latest

# Install MySQL
RUN apt-get update && \
    apt-get install -y mysql-server

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
```

After configuring MySQL, we will exit the container and execute the following command:
```bash
    podman commit mymysql myfinalmysql
```

Now we will enter the browser and enter this URL: http://localhost:8080 to enter WordPress, where we must complete the following information.

In the database configuration, we will enter the following data.

Finally, we will commit:
```bash
    podman commit mywordpress myfinalwordpress
```
