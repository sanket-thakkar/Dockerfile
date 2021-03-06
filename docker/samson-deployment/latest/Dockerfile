#+++++++++++++++++++++++++++++++++++++++
# Dockerfile for webdevops/samson-deployment:latest
#    -- automatically generated  --
#+++++++++++++++++++++++++++++++++++++++

FROM zendesk/samson:latest

MAINTAINER info@webdevops.io
LABEL vendor=WebDevOps.io
LABEL io.webdevops.layout=8
LABEL io.webdevops.version=0.56.1


###############################################################################
# Bootstrap
###############################################################################

# Ensure UTF-8
ENV LANG       en_US.UTF-8
ENV LC_ALL     en_US.UTF-8

ADD baselayout.tar /

# Init bootstrap (and install ansible)
RUN export DEBIAN_FRONTEND=noninteractive \
    && set -x \
    # Fix root terminal
    && echo "export TERM=xterm" >> /root/.bashrc \
    # Enable non-free
    && sed -ri 's/(deb.*\/debian jessie main)/\1 contrib non-free /' -- /etc/apt/sources.list \
    # System updatei
    && /usr/local/bin/apt-upgrade \
    # Base stuff
    && /usr/local/bin/apt-install \
        lsb-release \
        ca-certificates \
        locales \
        apt-transport-https \
    # Generate base locale
    && echo "en_US.UTF-8 UTF-8" > /etc/locale.gen \
    && locale-gen \
    && /usr/local/bin/apt-install \
        # Install ansible
        python-minimal \
        python-setuptools \
        python-pip \
        python-paramiko \
        python-jinja2 \
        python-dev \
        libffi-dev \
        libssl-dev \
        build-essential \
    && pip install --upgrade pip \
    && hash -r \
    && pip install ansible \
    && chmod 750 /usr/local/bin/ansible* \
    # Cleanup
    && apt-get purge -y -f --force-yes \
        python-dev \
        build-essential \
        libssl-dev \
        libffi-dev \
    && apt-get autoremove -y -f \
    && apt-get clean -y

###############################################################################
# Base
###############################################################################

ENV DOCKER_CONF_HOME /opt/docker/

COPY conf/ /opt/docker/

# Install packages
RUN chmod +x /opt/docker/bin/* \
    && /usr/local/bin/apt-install \
        supervisor \
        syslog-ng \
        syslog-ng-core \
        logrotate \
        cron \
        wget \
        curl \
        net-tools \
        gnupg \
    && /opt/docker/bin/provision run --tag bootstrap --role webdevops-base \
    && /opt/docker/bin/bootstrap.sh


###############################################################################
# Base-app
###############################################################################

ENV APPLICATION_USER  application
ENV APPLICATION_GROUP application
ENV APPLICATION_PATH  /app
ENV APPLICATION_UID   1000
ENV APPLICATION_GID   1000

# Install services
RUN /usr/local/bin/apt-install \
        # Install services
        openssh-server \
        mysql-client \
        sqlite \
        dnsmasq \
        postfix \
        # Install common tools
        sudo \
        zip \
        unzip \
        bzip2 \
        wget \
        curl \
        net-tools \
        moreutils \
        dnsutils \
        openssh-client \
        rsync \
        git \
        nano \
        vim \
    && /usr/local/bin/generate-locales \
    && /opt/docker/bin/provision run --tag bootstrap --role webdevops-base-app \
    && /opt/docker/bin/bootstrap.sh


###############################################################################
# Samson
###############################################################################

ENV RAILS_ENV="production"

ENV SQLITE_CLEANUP_DAYS=0

# NGINX reverse proxy
RUN export DEBIAN_FRONTEND=noninteractive && set -x \
       && echo deb https://apt.dockerproject.org/repo debian-jessie main > /etc/apt/sources.list.d/docker.list \
       && apt-key adv --keyserver hkp://pgp.mit.edu:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D \
    && /usr/local/bin/apt-install \
        # Install nginx
        nginx \
        # Install docker
        docker-engine \
        # Install php
        php5-cli \
        php5-mysqlnd \
        php5-mcrypt \
        php5-curl \
        php5-recode \
        php5-json \
        # Install java
        openjdk-7-jre \
        # SQlite 3
        sqlite3 \
    && pip install -U \
        docker-compose \
        python-dotenv \
    && curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin/ --filename=composer \
    ## Enable ansible for deployment user
    && chmod 755 /usr/local/bin/ansible*

# NPM stack
RUN npm install -g npm \
    && npm update -g \
    && npm install -g gulp \
    && npm install -g grunt-cli \
    && npm install -g bower \
    && npm install -g npm-cache

# Deployer stack
RUN set -x \
    # Deployer: Ansistratno (https://github.com/ansistrano)
    && ansible-galaxy install --force \
        carlosbuenosvinos.ansistrano-deploy \
        carlosbuenosvinos.ansistrano-rollback \
    # Deployer: PHP Deployer (http://deployer.org/)
    && wget --quiet -O/usr/local/bin/dep http://deployer.org/deployer.phar \
    && chmod +x /usr/local/bin/dep \
    # Deployer: capistrano (http://capistranorb.com/)
    && gem install capistrano \
    # Deplyer: Magallanes
    && wget http://download.magephp.com/magallanes.latest.tar.gz -O /tmp/magallanes.tar.gz \
    && tar -xzf /tmp/magallanes.tar.gz -C /tmp/ \
    && /tmp/magallanes/bin/mage install --installDir=/opt/magallanes \
    && rm -rf /tmp/magallanes.tar.gz /tmp/magallanes/ \
    && ln -s /opt/magallanes/latest/bin/mage /usr/bin/mage

# Upload
ADD database.yml    /app/config/database.yml
ADD web/            /app/public/assets/

ADD crontab         /etc/cron.d/webdevops-samson-deployment

RUN rake assets:precompile \
    && /opt/docker/bin/control.sh service.enable cron \
    && /opt/docker/bin/provision run --tag bootstrap --role webdevops-samson-deployment \
    && /opt/docker/bin/bootstrap.sh


EXPOSE 80

ENTRYPOINT ["/opt/docker/bin/entrypoint.sh"]
CMD ["supervisord"]
