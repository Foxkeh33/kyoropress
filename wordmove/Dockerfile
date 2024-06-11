#
# Wordmove Dockerfile
#
# Pull base image.
FROM ruby:2.6-slim

ENV DEBIAN_FRONTEND noninteractive
ENV WORDMOVE_WORKDIR /html

COPY mount-ssh.sh /bin/mount-ssh.sh
RUN chmod +x /bin/mount-ssh.sh

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    libssl-dev \
    libreadline-dev \
    zlib1g-dev \
    openssh-server \
    curl \
    rsync \
    mariadb-client \
    lftp \
    lsb-release \
    apt-transport-https \
    ca-certificates \
    git \
    wget \
    vim \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

RUN wget -O /etc/apt/trusted.gpg.d/php.gpg https://packages.sury.org/php/apt.gpg \
    && echo "deb https://packages.sury.org/php/ $(lsb_release -sc) main" | tee /etc/apt/sources.list.d/php.list \
    && apt update && apt -y install php7.4-cli php7.4-mysql \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*


RUN gem install ed25519 -v "~> 1.2"
RUN gem install bcrypt_pbkdf -v "~> 1.0"
RUN gem install specific_install
RUN gem specific_install https://github.com/murodon/wordmove_custom.git kusanagi
RUN curl -o /usr/local/bin/wp -L https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
    && chmod +x /usr/local/bin/wp

RUN eval "$(ssh-agent -s)"

WORKDIR ${WORDMOVE_WORKDIR}

ENTRYPOINT ["/bin/mount-ssh.sh"]

CMD ["/bin/bash", "-l"]