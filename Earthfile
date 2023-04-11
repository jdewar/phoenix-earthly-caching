VERSION 0.7

# This Earthfile was generated using docker2earthly
# the conversion is done on a best-effort basis
# and might not follow best practices, please
# visit https://docs.earthly.dev for Earthfile guides

ARG --global ELIXIR_VERSION=1.14.3
ARG --global OTP_VERSION=25.1
ARG --global DEBIAN_VERSION=bullseye-20220801-slim
ARG --global BUILDER_IMAGE="hexpm/elixir:${ELIXIR_VERSION}-erlang-${OTP_VERSION}-debian-${DEBIAN_VERSION}"
ARG --global RUNNER_IMAGE="debian:${DEBIAN_VERSION}"

subbuild1:
    FROM ${BUILDER_IMAGE}
    RUN apt-get update -y && apt-get install -y build-essential git     && apt-get clean && rm -f /var/lib/apt/lists/*_*
    WORKDIR /app
    RUN mix local.hex --force &&     mix local.rebar --force
    ENV MIX_ENV="prod"
    COPY mix.exs mix.lock ./
    RUN mix deps.get --only $MIX_ENV
    RUN mkdir config
    COPY config/config.exs config/${MIX_ENV}.exs config/
    RUN mix deps.compile
    COPY priv priv
    COPY lib lib
    COPY assets assets
    RUN mix assets.deploy
    RUN mix compile
    COPY config/runtime.exs config/
    COPY rel rel
    RUN mix release
    SAVE ARTIFACT /app/_build/${MIX_ENV}/rel/my_app my_app

subbuild3:
    FROM +subbuild1
    SAVE IMAGE --push ghcr.io/jdewar/my-app:cache

subbuild2:
    FROM ${RUNNER_IMAGE}
    RUN apt-get update -y && apt-get install -y libstdc++6 openssl libncurses5 locales   && apt-get clean && rm -f /var/lib/apt/lists/*_*
    RUN sed -i '/en_US.UTF-8/s/^# //g' /etc/locale.gen && locale-gen
    ENV LANG en_US.UTF-8
    ENV LANGUAGE en_US:en
    ENV LC_ALL en_US.UTF-8
    WORKDIR "/app"
    RUN chown nobody /app
    ENV MIX_ENV="prod"
    COPY --chown=nobody:root +subbuild1/my_app ./
    USER nobody
    CMD ["/app/bin/server"]
    SAVE IMAGE --push ghcr.io/jdewar/my-app:latest

build:
    BUILD +subbuild2
    BUILD +subbuild3