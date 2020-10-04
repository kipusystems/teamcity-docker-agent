FROM jetbrains/teamcity-minimal-agent:latest

RUN export GITHUB_OAUTH_TOKEN=1b151267468ae1aed3dec5732fe2a388256c38bd
USER root

# no docs for gems
RUN mkdir -p /etc/ \
	&& { \
		echo 'install: --no-document'; \
		echo 'update: --no-document'; \
	} >> /etc/gemrc

# remove tty warning when executing login shells
RUN sed -i 's/^mesg n || true$/tty -s \&\& mesg n/g' /root/.profile

# install git, rvm, nodejs and yarn
ENV RUBY_VERSION=2.3.7 \
    BUNDLER_VERSION=1.17.3 \
    RAILS_ENV=test

ENV SOURCE_RVM="source /usr/share/rvm/scripts/rvm" \
    RUBY_INSTALL="rvm install ${RUBY_VERSION}" \
    BUNDLER_INSTALL="gem install bundler:${BUNDLER_VERSION} nokogiri" \
    BUNDLER_CONFIG="bundle config silence_root_warning 1 \
                    && bundle config build.nokogiri --use-system-libraries \
                    && bundle config github.com KipuDevGemsUser:dNsEPTUWMbTumnphFmtjjGb9tKKbtmxT8 \
                    && bundle config enterprise.contribsys.com aa8ac900:18a34bbd \
                    && bundle install"

# install main dependencies for environment
RUN apt-get update && apt-get install -y --no-install-recommends \
                                        software-properties-common \
                                        gpg-agent \
                                        apt-utils \
    && curl -sL https://deb.nodesource.com/setup_10.x | bash - \
    && curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-add-repository -y ppa:rael-gc/rvm \
    && apt-get update \
    && apt-get install -y --no-install-recommends \
        streamer1.0-plugins-base \
        postgresql-client \
        libxslt-dev \
        libxml2-dev \
        gstreamer1.0-tools \
        gstreamer1.0-x \
        build-essential \
        patch \
        ruby-dev \
        wget \
        rvm \
        xvfb \
        libpq-dev \
        imagemagick \
        zlib1g-dev \
        liblzma-dev \
        nodejs \
        yarn \
        git \
        jq \
        && rm -rf /var/lib/apt/lists/*

# install google chrome latest
RUN wget -q -O - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add - && \
    sh -c 'echo "deb http://dl.google.com/linux/chrome/deb/ stable main" >> /etc/apt/sources.list.d/google.list' && \
    apt-get update && apt-get install google-chrome-stable --assume-yes

# install redis server
RUN wget http://download.redis.io/redis-stable.tar.gz && \
    tar xvzf redis-stable.tar.gz && \
    make -C ./redis-stable install

# update TC agent properties
RUN sed -i "/^trap.*/i sed -i '/^systemDir=.*/a auto_authorize=true' /data/teamcity_agent/conf/buildAgent.properties" run-agent.sh

ENV APP /app
RUN mkdir $APP
WORKDIR $APP
ADD Gemfile* $APP/

# install ruby & bundler
RUN /bin/bash -c "${SOURCE_RVM} && ${RUBY_INSTALL} && ${BUNDLER_INSTALL} && ${BUNDLER_CONFIG}"
ADD . $APP
