# syntax=docker/dockerfile:1
ARG RUBY_VERSION=3.2.3
FROM docker.io/library/ruby:$RUBY_VERSION-slim AS base

WORKDIR /rails

# Install essential runtime packages
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y curl libjemalloc2 libvips postgresql-client && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

ENV RAILS_ENV="production" \
    BUNDLE_DEPLOYMENT="1" \
    BUNDLE_PATH="/usr/local/bundle" \
    BUNDLE_WITHOUT="development"

FROM base AS build

# Node.js install karne ka behtar aur asaan tareeka
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y build-essential git libpq-dev libyaml-dev pkg-config nodejs npm && \
    npm install --global yarn && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

COPY Gemfile* ./
RUN if [ ! -f Gemfile.lock ]; then bundle lock --add-platform x86_64-linux; fi
RUN bundle install

COPY package.json yarn.lock* ./
RUN if [ -f yarn.lock ]; then yarn install; else yarn install --no-lockfile; fi

COPY . .

# Assets precompile (Database ke bagair)
RUN SECRET_KEY_BASE_DUMMY=1 bundle exec rails assets:precompile

FROM base
COPY --from=build /usr/local/bundle /usr/local/bundle
COPY --from=build /rails /rails

RUN useradd rails --create-home --shell /bin/bash && \
    chown -R rails:rails db log storage tmp
USER rails:rails

EXPOSE 80
CMD ["./bin/rails", "server", "-b", "0.0.0.0", "-p", "80"]