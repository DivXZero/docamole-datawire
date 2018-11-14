FROM alpine:3.8

ENV NODE_ENV=production
ENV RAILS_ENV=production

WORKDIR /docamole-web

COPY Gemfile Gemfile.lock package.json yarn.lock ./

RUN apk add --no-cache \
    postgresql-dev linux-headers zlib-dev ruby-dev make g++ \
    supervisor openrc nginx ruby-bigdecimal postgresql-libs ruby-json ruby-etc ruby yarn \
    && echo 'gem: --no-rdoc --no-ri' >> ~/.gemrc \
    && gem install bundler \
    && bundle install \
    && yarn install \
    && apk del \
    postgresql-dev linux-headers zlib-dev ruby-dev make g++

ADD . .
RUN bundle exec rake assets:precompile

# nginx Web Server
RUN mkdir -p /docamole-secure
COPY docamole.crt docamole.key dhparam.pem /docamole-secure/
COPY docamole.com.conf /etc/nginx/conf.d/default.conf

# supervisord
RUN mkdir -p /etc/supervisor.d
COPY supervisord.conf /etc/supervisord.conf

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisord.conf"]
