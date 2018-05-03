#
# Builds a docker image for i-doit 1.8
#
FROM ubuntu:14.04
ENV idoit_home /var/www/i-doit
ENV idoit_download_url https://sourceforge.net/projects/i-doit/files/i-doit/1.8/idoit-open-1.8.zip/download 

COPY policy-rc.d /usr/sbin/policy-rc.d

RUN apt-get update && apt-get dist-upgrade -y

# install wget
RUN apt-get update && apt-get install -y -q wget

# install mysql 5.6
RUN apt-get update && apt-get install -y -q mysql-server-5.6

# configure mysql 5.6 for i-doit 1.8
COPY mysql-for-i-doit-1-8.cnf /etc/mysql/conf.d

# install required packages
RUN add-apt-repository ppa:ondrej/php
RUN apt-get update
RUN apt-get update
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y -q  \
	apache2 \
	libapache2-mod-php5.6 \
	curl \
	php5.6 \
	php5.6-cli \
	php5.6-xmlrpc \
	php5.6-ldap \
	php5.6-gd \
	php5.6-mysql \
	php5.6-curl \
	mcrypt \
	php5.6-mcrypt \
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
	/etc/php5/apache2/php.ini

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
EXPOSE 80

# install configuration file for supervisord
COPY supervisord.conf /etc/supervisor/conf.d/supervisord.conf

CMD ["/usr/bin/supervisord"]
