
ARG RUBY_VERSION=3.2.3
FROM docker.io/library/ruby:$RUBY_VERSION-slim AS base

WORKDIR /rails

# 1. Runtime packages install karein
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y curl libjemalloc2 libvips postgresql-client && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Production environment set karein
ENV RAILS_ENV="production" \
    BUNDLE_DEPLOYMENT="1" \
    BUNDLE_PATH="/usr/local/bundle" \
    BUNDLE_WITHOUT="development" \
    LD_PRELOAD="/usr/local/lib/libjemalloc.so"

# 2. Build stage
FROM base AS build

# Build packages (Node.js aur Yarn zaroori hain React build ke liye)
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y build-essential git libpq-dev libyaml-dev pkg-config nodejs npm && \
    npm install --global yarn && \
    rm -rf /var/lib/apt/lists /var/cache/apt/archives

# Ruby gems install karein
COPY Gemfile* ./
RUN if [ ! -f Gemfile.lock ]; then bundle lock --add-platform x86_64-linux; fi
RUN bundle install

# JS dependencies install karein
COPY package.json yarn.lock* ./
RUN yarn install --frozen-lockfile || yarn install

# Poora code copy karein
COPY . .

# Assets Precompile: Yeh ab aapki package.json ka "build" script chalayega
RUN SECRET_KEY_BASE_DUMMY=1 bundle exec rails assets:precompile

# 3. Final production image
FROM base

# Build artifacts copy karein
COPY --from=build /usr/local/bundle /usr/local/bundle
COPY --from=build /rails /rails

# Permissions aur User setup
RUN useradd rails --create-home --shell /bin/bash && \
    chown -R rails:rails db log storage tmp
USER rails:rails

# Port 80 expose karein
EXPOSE 80
CMD ["./bin/rails", "server", "-b", "0.0.0.0", "-p", "80"]