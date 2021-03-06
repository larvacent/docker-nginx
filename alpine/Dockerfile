FROM larvacent/alpine:3.8

LABEL maintainer="support@larvacent.com"

# persistent / runtime deps
ENV BUILD_DEPS autoconf dpkg-dev file g++ gcc libc-dev make linux-headers curl pcre-dev zlib-dev libressl-dev libxml2-dev libxslt-dev gd-dev

RUN set -x \
	&& apk add --no-cache pcre zlib libressl libxml2 libxslt gd \
	&& addgroup -g 82 -S www-data \
	&& adduser -u 82 -D -S -G www-data www-data
# 82 is the standard uid/gid for "www-data" in Alpine

ARG NGINX_VERSION

# Environment settings
ENV NGINX_VERSION=${NGINX_VERSION}
ENV NGINX_TGZ_URL http://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz
ENV NGINX_INI_DIR=/usr/local/etc/nginx

RUN set -xe \
	&& apk add --no-cache --virtual .build-deps \
	    $BUILD_DEPS  \
	&& cd /tmp \
	&& curl -fSL ${NGINX_TGZ_URL} -o nginx-${NGINX_VERSION}.tar.gz \
	&& tar zxf nginx-${NGINX_VERSION}.tar.gz \
	&& rm -rf nginx-${NGINX_VERSION}.tar.gz \
	&& cd nginx-${NGINX_VERSION}/ \
	&& ./configure \
		--prefix=/usr/local \
		--conf-path=${NGINX_INI_DIR}/nginx.conf \
		--error-log-path=/var/log/nginx/error.log \
		--http-client-body-temp-path=/var/lib/nginx/body \
		--http-fastcgi-temp-path=/var/lib/nginx/fastcgi \
		--http-log-path=/var/log/nginx/access.log \
		--http-proxy-temp-path=/var/lib/nginx/proxy \
		--lock-path=/run/lock/nginx.lock \
		--pid-path=/run/nginx.pid \
		--with-http_ssl_module \
		--with-http_v2_module \
		--with-http_image_filter_module \
		--with-http_slice_module \
		--with-http_xslt_module \
		--with-http_realip_module \
		--with-http_stub_status_module \
		--with-pcre \
		--with-pcre-jit \
		--with-http_flv_module \
		--with-http_mp4_module \
		--with-http_addition_module \
		--with-threads \
		--with-http_secure_link_module \
		--with-http_degradation_module \
		--with-http_ssl_module \
		--with-http_gzip_static_module \
		--without-mail_imap_module \
		--without-mail_pop3_module \
		--without-mail_smtp_module \
		--without-http_uwsgi_module \
		--without-http_scgi_module \
		--without-select_module \
		--with-http_sub_module \
		--with-cc-opt='-O2' \
	&& make -j "$(nproc)" \
	&& make install \
	&& mkdir -p /var/lib/nginx \
	&& mkdir -p /usr/local/etc/nginx/sites \
	&& rm -f /usr/local/etc/nginx/scgi_* \
	&& rm -rf /tmp/nginx-${NGINX_VERSION} \
	\
	&& apk del .build-deps

# Add configuration files
COPY image-files /

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log && ln -sf /dev/stderr /var/log/nginx/error.log

EXPOSE 80

STOPSIGNAL SIGTERM

CMD ["nginx", "-c", "/usr/local/etc/nginx/nginx.conf"]