FROM centos:centos7
MAINTAINER 1ocate <msyeoenliple@gmail.com>

# 컨테이너 생성시 사용할 User & Group 가져오기
ARG USER_ID
ARG GROUP_ID

RUN if [ ${USER_ID:-0} -ne 0 ] && [ ${GROUP_ID:-0} -ne 0 ]; then \
    userdel -f www-data &&\
    if getent group develop ; then groupdel develop; fi &&\
    groupadd -g ${GROUP_ID} develop &&\
    useradd -l -u ${USER_ID} -g develop www-data &&\
    install -d -m 0755 -o www-data -g develop /home/www-data &&\
    chown --changes --silent --no-dereference --recursive \
          --from=33:600 ${USER_ID}:${GROUP_ID} \
        /var/www/app; \
    else  groupadd  develop &&\
    useradd -g develop www-data \
;fi

        
ENV OS_LOCALE="en_US.UTF-8" 

ENV LANG=${OS_LOCALE} \
    LANGUAGE=${OS_LOCALE} \
    LC_ALL=${OS_LOCALE}

ENV GITLAB_CONF_DIR=/etc/gitlab-runner \
    PHP_RUN_DIR=/run/php \
    PHP_LOG_DIR=/var/log/php \
    PHP_CONF_DIR=/etc/php.ini \
    PHP_DATA_DIR=/var/lib/php \
	NGINX_CONF_DIR=/etc/nginx \
	APACHE_CONF_DIR=/etc/httpd 


COPY ./configs/nginx.repo /etc/yum.repos.d/
COPY ./configs/index.php /var/www/html/


Run \
    yum update -y \
    #Install Apache
    &&yum install -y httpd \
    && yum install -y vim \
    # Install PHP
    && yum install -y http://ftp.riken.jp/Linux/remi/enterprise/remi-release-7.rpm \
    && yum-config-manager --enable remi-php72 \
    && yum install -y curl php php-cli php-readline php-mbstring php-zip php-intl php-json php-xml php-curl php-gd php-pgsql php-mysql php-pear \
    # Install composer
    && php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');" \ 
    && php -r "if (hash_file('sha384', 'composer-setup.php') === '55ce33d7678c5a611085589f1f3ddf8b3c52d662cd01d4ba75c0ee0459970c2200a51f492d557530c71c15d8dba01eae') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;" \ 
    && php composer-setup.php \
    && mv composer.phar /usr/local/bin/composer \
    && php -r "unlink('composer-setup.php');" \
    # Install supervisor
	&& yum install -y supervisor && mkdir -p /var/log/supervisor \
    && chown www-data:develop /var/www/html/ -Rf \
    # Cleaning
    && yum clean all
    
COPY ./configs/supervisord.conf  /etc/
COPY ./configs/httpd.conf ${APACHE_CONF_DIR}/conf/
COPY ./configs/vhost.conf ${APACHE_CONF_DIR}/conf.d/
COPY ./configs/php.ini ${PHP_CONF_DIR}
WORKDIR /var/www/html/
EXPOSE 80 443 

CMD ["/usr/bin/supervisord"]
