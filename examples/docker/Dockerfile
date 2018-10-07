FROM centos:centos7

RUN rpm -Uvh http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && \
    rpm -Uvh http://rpms.famillecollet.com/enterprise/remi-release-7.rpm

# Install desired PHP version (remi-php) #
RUN yum update -y && \
  yum install --enablerepo="remi,remi-php56" -y \
  php \
  php-devel \
  php-cli \
  php-pdo \
  php-gd \
  php-mbstring \
  php-xml && \
  yum clean all && \
  rm -rf /var/cache/yum/* && \
  rm -rf /tmp/* && \
  rm -rf /var/tmp/*

# other build dependencies
RUN yum update -y && \
  yum install -y \
  libicu-devel \
  readline-devel \
  openssl-devel \
  tcl-devel \
  build-essential \
  pkg-config \
  git \
  vim \
  nano \
  gcc \
  make \
  cmake \
  autoconf \
  automake \
  m4 \
  nasm \
  wget \
  curl && \
  echo "set number" >> /root/.vimrc && \
  echo "set hlsearch" >> /root/.vimrc && \
  echo "export TERM=xterm" >> /root/.bashrc && \
  echo "export EDITOR=vim" >> /root/.bashrc && \
  yum clean all && \
  rm -rf /var/cache/yum/* && \
  rm -rf /tmp/* && \
  rm -rf /var/tmp/*

# Include apache just to keep container running
COPY include/apache_default /etc/httpd/conf.d/vhosts.conf
COPY include/run /usr/local/bin/run
RUN chmod +x /usr/local/bin/run

RUN mkdir /project

WORKDIR /project

EXPOSE 80 22

CMD ["/usr/local/bin/run"]
