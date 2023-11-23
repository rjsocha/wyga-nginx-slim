FROM alpine:3 AS build
ARG VERSION
ARG ZLIB=1.3
ARG PCRE=10.42
WORKDIR /build
ADD https://nginx.org/download/nginx-${VERSION}.tar.gz nginx.tar.gz
ADD https://github.com/PCRE2Project/pcre2/releases/download/pcre2-${PCRE}/pcre2-${PCRE}.tar.gz pcre.tar.gz
ADD http://zlib.net/zlib-${ZLIB}.tar.gz zlib.tar.gz
RUN apk add gcc libc-dev make linux-headers
RUN tar --strip-components 1 -xzf nginx.tar.gz 
RUN tar xzf pcre.tar.gz
RUN tar xzf zlib.tar.gz
RUN ./configure --with-zlib=./zlib-${ZLIB} --with-pcre=./pcre2-${PCRE} --prefix=/etc/nginx --sbin-path=/usr/sbin/nginx --modules-path=/usr/lib/nginx/modules --conf-path=/etc/nginx/nginx.conf --error-log-path=/var/log/nginx/error.log --http-log-path=/var/log/nginx/access.log --pid-path=/var/run/nginx.pid --lock-path=/var/run/nginx.lock --http-client-body-temp-path=/var/cache/nginx/client_temp --http-proxy-temp-path=/var/cache/nginx/proxy_temp --http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp --http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp --http-scgi-temp-path=/var/cache/nginx/scgi_temp --user=nginx --group=nginx --with-compat --with-file-aio --with-threads --with-http_addition_module --with-http_auth_request_module --with-http_flv_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_mp4_module --with-http_random_index_module --with-http_realip_module --with-http_secure_link_module --with-http_slice_module --with-http_stub_status_module --with-http_sub_module --with-http_v2_module --with-stream --with-stream_realip_module  --with-cc-opt='-Os -fomit-frame-pointer -g' --with-ld-opt=-Wl,--as-needed,-O1,--sort-common
RUN make
RUN strip -x objs/nginx
RUN make install
FROM alpine:3 AS mold
COPY --from=build /build/objs/nginx /usr/sbin/nginx
RUN apk --no-cache -U upgrade
RUN apk del zlib ssl_client scanelf musl-utils libssl3 libcrypto3 libc-utils ca-certificates-bundle alpine-keys apk-tools
COPY --chmod=755 entrypoint.sh /entrypoint.sh
COPY --chmod=755 env-conf /bin/env-conf
RUN mkdir -m 755 -p /etc/nginx /etc/nginx/stream.d /etc/nginx/nginx.d /etc/nginx/conf.d /var/cache/nginx /var/log/nginx
RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/error.log
COPY --from=build /etc/nginx/fastcgi.conf /etc/nginx/fastcgi_params /etc/nginx
COPY --from=build /etc/nginx/mime.types /etc/nginx/scgi_params /etc/nginx/uwsgi_params /etc/nginx
COPY --from=build /etc/nginx/html /usr/share/nginx/
COPY nginx.conf /etc/nginx/nginx.conf
COPY default.conf /etc/nginx/conf.d/default.conf
RUN addgroup -S nginx 
RUN adduser -SDH \
  -h /var/cache/nginx \
  -s /sbin/nologin \
  -G nginx \
  -g nginx \
   nginx && addgroup nginx www-data
RUN rm -rf /etc/group- /etc/passwd- /etc/shadow- /lib/apk /lib/firmware /lib/mdev rm /lib/modules-load.d /lib/sysctl.d /etc/sysctl.conf /etc/udhcpd.conf /media /mnt /opt /srv /var/spool /var/opt /var/mail /var/cache/apk /usr/sbin/nginx-debug /etc/init.d /etc/apk /etc/logrotate.d  /etc/modprobe.d /etc/modules-load.d /etc/network /etc/opt /etc/periodic /etc/secfixes.d /etc/ssl /etc/sysctl.d /home /var/lib/misc /var/lib/udhcpd  /usr/lib/modules-load.d /etc/conf.d /etc/crontabs /etc/profile.d /etc/inittab  /etc/motd /etc/fstab /usr/share/licenses /usr/share/man /usr/share/misc /usr/share/udhcpc /docker-entrypoint.sh /docker-entrypoint.d /usr/local/bin/envsubst /etc/busybox-paths.d /usr/local

FROM scratch
COPY --from=mold / /
EXPOSE 80
ENV PATH=/bin:/sbin:/usr/bin:/usr/sbin
STOPSIGNAL SIGQUIT
ENTRYPOINT [ "/entrypoint.sh" ]
CMD ["nginx", "-g", "daemon off;"]
