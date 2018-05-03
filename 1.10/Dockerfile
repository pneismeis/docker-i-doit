#
# Builds a docker image for i-doit 1.10.2
#
FROM ubuntu:16.04
ENV idoit_home /var/www/i-doit
ENV idoit_download_url https://vorboss.dl.sourceforge.net/project/i-doit/i-doit/1.10.2/idoit-open-1.10.2.zip

COPY policy-rc.d /usr/sbin/policy-rc.d

RUN apt-get update && apt-get dist-upgrade -y

# install wget
RUN apt-get update && apt-get install -y -q wget
# install curl
RUN apt-get update && apt-get install -y -q curl

ENV MYSQL_PWD Pwd123
RUN echo "mysql-server mysql-server/root_password password $MYSQL_PWD" | debconf-set-selections
RUN echo "mysql-server mysql-server/root_password_again password $MYSQL_PWD" | debconf-set-selections

# install mysql 5.7
RUN apt-get update && apt-get install -y -q mysql-server
#RUN apt-get update && apt-get install -y -q mariadb-server
# configure mysql 5.6 for i-doit 1.10.2
COPY mysql-for-i-doit-1-8.cnf /etc/mysql/conf.d

# install required packages
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y -q  \
	apache2 \
	libapache2-mod-php \
	curl \
	php \
	php-cli \
	php-xmlrpc \
	php-ldap \
	php-gd \
	php-mysql \
	php-curl \
	mcrypt \
	php-mcrypt \
	unzip \
	supervisor

# enable mcrypt in php5
RUN phpenmod mcrypt

# create the home directory for i-doit
RUN mkdir -p ${idoit_home}

# fix two php configuration options, required for i-doit
RUN sed -i \
	-e "s#; max_input_vars = 1000#max_input_vars = 10000#" \
	-e "s#post_max_size = 8M#post_max_size = 128M#" \
	/etc/php/7.0/apache2/php.ini

# make sure apache module mod_rewrite is enabled
RUN a2enmod rewrite

# download i-doit 1.8 and upack it
RUN wget -O i-doit.zip ${idoit_download_url} \
	&&  unzip i-doit.zip -d ${idoit_home} \
	&& rm i-doit.zip

# default rights
RUN chmod +x ${idoit_home}/idoit-rights.sh
RUN cd ${idoit_home} && ./idoit-rights.sh

# apache2 configuration for the virtual host on port 80
# includes the alias definition for '/i-doit'
COPY 000-default.conf /etc/apache2/sites-enabled/000-default.conf

# expose i-doit webinterface running on port 80
EXPOSE 88

# install configuration file for supervisord
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["/usr/bin/supervisord"]
