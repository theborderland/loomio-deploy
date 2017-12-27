FROM ruby:2.3.5

COPY Gemfile.lock Gemfile private_pub.ru /loomio-faye/
RUN ls
WORKDIR /loomio-faye
RUN ls
RUN gem install private_pub
RUN gem install thin
RUN bundle install
EXPOSE 3000

CMD thin --port 3000 \
         --environment production \
         --rackup private_pub.ru \
         start
