# -------- Base Image --------
ARG RUBY_VERSION=3.2.3
FROM ruby:${RUBY_VERSION}-slim AS base

WORKDIR /rails

# Runtime deps
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
    curl libjemalloc2 libvips postgresql-client && \
    rm -rf /var/lib/apt/lists/*

ENV RAILS_ENV=production \
    BUNDLE_DEPLOYMENT=1 \
    BUNDLE_PATH=/usr/local/bundle \
    BUNDLE_WITHOUT=development:test

# -------- Build Stage --------
FROM base AS build

# Build deps + Node 20
RUN apt-get update -qq && \
    apt-get install --no-install-recommends -y \
    build-essential git libpq-dev libyaml-dev pkg-config curl && \
    curl -fsSL https://deb.nodesource.com/setup_20.x | bash - && \
    apt-get install -y nodejs && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /rails

# Copy full project
COPY . .

# Verify scripts exist (safe guard)
RUN cat package.json

# Install JS dependencies
RUN npm install

# Build JS (esbuild)
RUN npm run build

# Install Ruby gems
RUN bundle install

# Precompile Rails assets
RUN SECRET_KEY_BASE_DUMMY=1 bundle exec rails assets:precompile

# -------- Final Production Image --------
FROM base

WORKDIR /rails

# Copy gems
COPY --from=build /usr/local/bundle /usr/local/bundle

# Copy app (with built assets)
COPY --from=build /rails /rails

# Non-root user
RUN useradd rails --create-home --shell /bin/bash && \
    chown -R rails:rails /rails
USER rails

EXPOSE 80

CMD ["./bin/rails", "server", "-b", "0.0.0.0", "-p", "80"]