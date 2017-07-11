FROM ruby:2.3.1

COPY Gemfile.lock Gemfile private_pub.ru /loomio-faye/
ENV BUNDLE_GITHUB__HTTPS=true
WORKDIR /loomio-faye
RUN bundle install
EXPOSE 3000

CMD thin --port 3000 \
         --environment production \
         --rackup private_pub.ru \
         start
