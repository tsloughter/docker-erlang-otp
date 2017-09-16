FROM alpine:3.6

ENV OTP_VERSION="20.0.1"

RUN set -xe \
    && OTP_DOWNLOAD_URL="https://github.com/erlang/otp/archive/OTP-${OTP_VERSION}.tar.gz" \
    && OTP_DOWNLOAD_SHA256="8b121b38102acd43f89afd786055461741522c3a13ee17ef1a795c0dbf6aa281" \
    && apk add --no-cache --virtual .fetch-deps \
        curl \
        ca-certificates \
    && curl -fSL -o otp-src.tar.gz "$OTP_DOWNLOAD_URL" \
    && echo "$OTP_DOWNLOAD_SHA256  otp-src.tar.gz" | sha256sum -c - \
    && apk add --no-cache --virtual .build-deps \
        gcc \
        openssl-dev \
        libc-dev \
        make \
        autoconf \
        ncurses-dev \
        tar \
    && export ERL_TOP="/usr/src/otp_src_${OTP_VERSION%%@*}" \
    && mkdir -vp $ERL_TOP \
    && tar -xzf otp-src.tar.gz -C $ERL_TOP --strip-components=1 \
    && rm otp-src.tar.gz \
    && ( cd $ERL_TOP \
      && ./otp_build autoconf \
      && ./configure \
        --enable-dirty-schedulers \
      && make -j$(getconf _NPROCESSORS_ONLN) \
      && make install ) \
    && rm -rf $ERL_TOP \
    && scanelf --nobanner -E ET_EXEC -BF '%F' --recursive /usr/local | xargs strip --strip-all \
    && scanelf --nobanner -E ET_DYN -BF '%F' --recursive /usr/local | xargs -r strip --strip-unneeded \
    && runDeps=$( \
        scanelf --needed --nobanner --recursive /usr/local \
            | awk '{ gsub(/,/, "\nso:", $2); print "so:" $2 }' \
            | sort -u \
            | xargs -r apk info --installed \
            | sort -u \
    ) \
    && apk add --virtual .erlang-rundeps $runDeps \
    && apk del .fetch-deps .build-deps

ENV CLOUD_SDK_VERSION 161.0.0
ENV PATH /google-cloud-sdk/bin:$PATH
RUN apk --no-cache add curl python py-crcmod bash libc6-compat openssh-client git && \
    curl -O https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz && \
    tar xzf google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz && \
    rm google-cloud-sdk-${CLOUD_SDK_VERSION}-linux-x86_64.tar.gz && \
    ln -s /lib /lib64 && \
    gcloud config set core/disable_usage_reporting true && \
    gcloud config set component_manager/disable_update_check true && \
    gcloud config set metrics/environment github_docker_image

RUN apk --no-cache add docker

RUN apk add --update tar curl git bash make libc-dev gcc g++ vim && \
    rm -rf /var/cache/apk/*

RUN set -xe \
    && curl -fSL -o rebar3 "https://s3.amazonaws.com/rebar3-nightly/rebar3" \
    && chmod +x ./rebar3 \
    && ./rebar3 local install \
    && rm ./rebar3

ENV PATH "$PATH:/root/.cache/rebar3/bin"

CMD ["erl"]
