FROM nginx:1.14-alpine
LABEL maintainer="maimidoudou6 <maimidoudou6@gmail.com>" app="httpd"

ENV NGX_DOC_ROOT="/data/web/html/"
COPY index.html $NGX_DOC_ROOT
COPY entrypoint.sh /bin/

EXPOSE 80/tcp
EXPOSE 8080/tcp

HEALTHCHECK --start-period=3s --interval=5s --timeout=3s CMD wget -O - -q http://${IP:-0.0.0.0}:${PORT:-80} || exit 1
#HEALTHCHECK --start-period=3s --interval=5s --timeout=3s CMD wget -O - -q http://${IP:-0.0.0.0}:10080 || exit 1

CMD ["/usr/sbin/nginx", "-g", "daemon off;"]
ENTRYPOINT ["/bin/entrypoint.sh"]
