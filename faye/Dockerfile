FROM ruby:2.3.5

COPY Gemfile.lock Gemfile private_pub.ru /loomio-faye/
WORKDIR /loomio-faye
RUN gem install hello_world
EXPOSE 3000

CMD thin --port 3000 \
         --environment production \
         --rackup private_pub.ru \
         start
