FROM amazonlinux:2017.03

RUN yum update -y && \
  yum install -y \
  php71 \
  php71-devel \
  php71-cli \
  php71-pdo \
  php71-gd \
  php71-mbstring \
  php71-xml \
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
  yum clean all && \
  echo "set number" >> /root/.vimrc && \
  echo "set hlsearch" >> /root/.vimrc && \
  echo "export TERM=xterm" >> /root/.bashrc && \
  echo "export EDITOR=vim" >> /root/.bashrc && \
  rm -rf /var/cache/yum/* && \
  rm -rf /tmp/* && \
  rm -rf /var/tmp/*

COPY include/apache_default /etc/httpd/conf.d/vhosts.conf
COPY include/run /usr/local/bin/run
RUN chmod +x /usr/local/bin/run

RUN mkdir /project

WORKDIR /project

EXPOSE 80 22

CMD ["/usr/local/bin/run"]
