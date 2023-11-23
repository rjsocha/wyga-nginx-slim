ARG VERSION
FROM nginx:${VERSION}-alpine-slim AS mold
RUN apk --no-cache -U upgrade
RUN apk del libintl libc-utils ca-certificates-bundle alpine-keys apk-tools
RUN rm -rf /etc/group- /etc/passwd- /etc/shadow- /lib/apk /lib/firmware /lib/mdev rm /lib/modules-load.d /lib/sysctl.d /etc/sysctl.conf /etc/udhcpd.conf /media /mnt /opt /srv /var/spool /var/opt /var/mail /var/cache/apk /usr/sbin/nginx-debug /etc/init.d /etc/apk /etc/logrotate.d  /etc/modprobe.d /etc/modules-load.d /etc/network /etc/opt /etc/periodic /etc/secfixes.d /etc/ssl /etc/sysctl.d /home /var/lib/misc /var/lib/udhcpd  /usr/lib/modules-load.d /etc/conf.d /etc/crontabs /etc/profile.d /etc/inittab  /etc/motd /etc/fstab /usr/share/licenses /usr/share/man /usr/share/misc /usr/share/udhcpc /docker-entrypoint.sh /docker-entrypoint.d /usr/local/bin/envsubst
COPY --chmod=755 wyga-entrypoint.sh /
COPY --chmod=755 env-conf /bin/env-conf
#COPY files/nginx/wyga/nginx-slim.conf /etc/nginx/nginx.conf
RUN mkdir -p /etc/nginx/stream.d
RUN mkdir -p /etc/nginx/nginx.d

FROM scratch
COPY --from=mold / /
EXPOSE 80
ENTRYPOINT [ "/wyga-entrypoint.sh" ]
CMD ["nginx", "-g", "daemon off;"]
