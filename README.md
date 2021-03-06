# Apache docker image

This Docker image be used for apache multi vhost for openshift. This inherits is based on the Debian OS version, with changes made to the configuration to ensure compatibility with the [OpenShift security policy](https://docs.openshift.com/container-platform/3.11/creating_images/guidelines.html). 

If you want to make modifications to the image, your `Dockerfile` should look something like this, ensuring the PHP version is updated in the FROM image descriptor:

```Dockerfile
FROM linkbn/apache-openshift:X.X
ARG USER_ID=2000
USER root
COPY vhost-conf-src/ /etc/apache2/conf-enabled
COPY vhost-doc-src/ /var/www/html/
RUN /docker-bin/docker-build.sh
USER ${USER_ID}
```

### Run Apache HTTPD

This command is ran from docker file. Apache is running on port `8080`.

## Image Configuration at buildtime

With docker build arguments (`docker build --build-arg VAR_NAME=VALUE`), if you want to change some of them you will need to run the command as root in your Dockerfile inheriting from the image in the script `/docker-bin/docker-build.sh`.

### System configuration (buildtime)

* **USER_ID**: Id of the user that will run the container (default: `2000`)
* **TZ**: System timezone will be used for cron and logs (default: `UTC`, done by `docker-build.sh`)
* **CA_HOSTS_LIST**: List of host CA certificate to add to system CA list, example: `my-server1.local.net:443 my-server2.local.local:8443` (default: `none`, done by `docker-build.sh`)

### Apache HTTPD configuration  (buildtime)

Log format by default is `combined` on container stdout, and apache is listening on port 8080 (http) or 8443 (https). Document root of Apache is `/var/`.

* **remoteip**: By default remoteip configuration is enabled, see runtime part of the documentation to configure it.
* **serve-cgi-bin**: Is disabled by default.
* **syslog**: You can enable Apache HTTPD logging to syslog, using `a2enconf syslog` in your docker build. See docker-compose.yml file for the configuration.

The [**mod_auth_openidc**](https://github.com/zmartzone/mod_auth_openidc) module is present in the image, it's no enabled by default you can add it by, adding into your `Dockerfile` :

```
a2enmod auth_openidc
```

## Image Configuration at runtime

With environment variables (`docker run  -e VAR_NAME=VALUE`).

### System configuration (runtime)

* **USER_NAME**: Name of the user that will run the container will have the id defined by **USER_ID** and home defined by **USER_HOME** (default: `default`)

### Apache HTTPD configuration (runtime)

* **APACHE_RUN_USER**: Username of the user that will run apache (default: `$USER_NAME`).
* **APACHE_SERVER_NAME**: Set Apache ServerName (default: `__default__`).

#### Apache HTTPD remoteip configuration (runtime)

* **APACHE_REMOTE_IP_HEADER**: Set `RemoteIPHeader` directive of the [remote_ip module](https://httpd.apache.org/docs/trunk/mod/mod_remoteip.html) (default: `X-Forwarded-For`)
* **APACHE_REMOTE_IP_TRUSTED_PROXY**: Set `RemoteIPtrustedProxy` directive of the [remote_ip module](https://httpd.apache.org/docs/trunk/mod/mod_remoteip.html) (default: `10.0.0.0/8 172.16.0.0/12 192.168.0.0/16`)
* **APACHE_REMOTE_IP_INTERNAL_PROXY**: Set `RemoteIPInternalProxy` directive of the [remote_ip module](https://httpd.apache.org/docs/trunk/mod/mod_remoteip.html) (default: `10.0.0.0/8 172.16.0.0/12 192.168.0.0/16`)

#### Apache HTTPD syslog configuration (runtime)

Will be used only if you add `a2enconf syslog` in your `Dockerfile`.

* **APACHE_SYSLOG_HOST**: IP or DNS of the UDP syslog server (default: `$SYSLOG_HOST`).
* **APACHE_SYSLOG_PORT**: Port of syslog server (default: `$SYSLOG_PORT or 514`).
* **APACHE_SYSLOG_PROGNAME**: Value of logsource field in syslog (default: `httpd`).

#### Apache HTTPD vhost config & document root configuration

Provide the command bellow to your dockerfile:

```
COPY vhost-conf-src/ /etc/apache2/conf-enabled
COPY vhost-doc-src/ /var/www/html/
```

Which:

* "vhost-conf-src" is a directory that contains your vhost configuration
* "vhost-doc-src" is a directory that contains your vhost web data (document root)

The Vhost configuration should have the following parameter :

```
<VirtualHost *:8080>
    ServerName test.com
    ServerAlias test-dev.fr
    DocumentRoot "/var/www/html/test.com/www"
    <Directory /var/www/html/test.com/www>
		Require all granted
    </Directory>
</VirtualHost>
```

#### This version is not included any php version. If you need an apache with php supported , please see the image linkbn/php-openshift