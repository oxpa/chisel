#syntax=docker/dockerfile:1
ARG UV=24.04
FROM ubuntu:$UV AS builder

LABEL maintainer="NGINX Docker Maintainers <docker-maint@nginx.com>"

ENV NGINX_VERSION mainline

ARG SOURCE_DATE_EPOCH
ARG REPOHOST=packages.nginx.org

COPY files/chisel/* /

RUN groupadd --system --gid 101 nginx \
    && useradd --system --gid nginx --no-create-home --home /nonexistent --comment "nginx user" --shell /bin/false --uid 101 nginx \
    && usermod -aG tty nginx \
    && apt-get update && apt-get install -y --no-install-recommends --no-install-suggests gpg wget ca-certificates git \
    && git clone https://github.com/canonical/chisel-releases.git \
    && cd chisel-releases && git switch ubuntu-$UV && cd .. \
    && cp chisel.yaml ./chisel-releases \
    && rm ./chisel-releases/slices/*nginx* \
    && cp libxslt1.1.yaml nginx.yaml nginx-module* ./chisel-releases/slices \
    && mkdir /build \
    && ./chisel cut --release=/chisel-releases/ --root=/build \
         nginx_bins nginx-module-njs_bins nginx-module-otel_bins \
         nginx-module-acme_bins nginx-module-geoip_bins nginx-module-xslt_bins \
    && cp /etc/passwd /etc/group /build/etc \
    && ln -sf /dev/stdout /build/var/log/nginx/access.log \
    && ln -sf /dev/stderr /build/var/log/nginx/error.log 
 

FROM scratch
COPY --from=builder /build /

EXPOSE 80

STOPSIGNAL SIGQUIT

CMD ["nginx", "-g", "daemon off;"]

# vim:syntax=Dockerfile
