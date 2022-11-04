FROM ubuntu:20.04

ARG NGINX_VERSION=1.19.2
ARG GEOIP2_VERSION=3.3

WORKDIR /opt

RUN adduser --system --no-create-home --disabled-login --disabled-password --group nginx

RUN ["mkdir", "/var/log/nginx"]

RUN apt-get update \
    && apt-get install -y \
        build-essential \
        libpcre++-dev \
        zlib1g-dev \
        libgeoip-dev \
        libpcre3-dev \
        libssl-dev \
        libmaxminddb-dev \
        wget \
        git

RUN cd /opt \
    && git clone --depth 1 -b $GEOIP2_VERSION --single-branch https://github.com/leev/ngx_http_geoip2_module.git \
    && wget -O - http://nginx.org/download/nginx-$NGINX_VERSION.tar.gz | tar zxfv - \
    && mv /opt/nginx-$NGINX_VERSION /opt/nginx \
    && cd /opt/nginx \
    && ./configure --with-compat --add-dynamic-module=/opt/ngx_http_geoip2_module --with-stream --user=nginx --group=nginx --with-http_ssl_module --with-ipv6 --with-threads --with-stream --with-stream_ssl_module --with-stream_ssl_preread_module \
    && make modules \
    && make && make install

RUN ["mkdir", "-p", "/usr/lib/nginx/modules"]
RUN ["cp", "/opt/nginx/objs/ngx_http_geoip2_module.so", "/usr/lib/nginx/modules"]
RUN ["cp", "/opt/nginx/objs/ngx_stream_geoip2_module.so", "/usr/lib/nginx/modules"]

RUN apt-get update \
    && apt-get install -y --no-install-recommends --no-install-suggests libmaxminddb0 \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && chmod -R 644 /usr/lib/nginx/modules/ngx_http_geoip2_module.so \
    && chmod -R 644 /usr/lib/nginx/modules/ngx_stream_geoip2_module.so

EXPOSE 443

CMD ["/opt/nginx/objs/nginx", "-c", "/etc/nginx/conf.d/nginx.conf", "-g", "daemon off;"]
