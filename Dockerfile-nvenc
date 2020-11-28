##### Building stage #####
FROM nvidia/cuda:11.1-devel-ubuntu20.04 as builder
MAINTAINER Kieran Harkin <kieran+git@harkin.me>

# Versions of nginx, rtmp-module and ffmpeg 
ENV  TZ=Europe/Dublin
ARG  NGINX_VERSION=1.19.4
ARG  NGINX_RTMP_MODULE_VERSION=1.2.1
ARG  FFMPEG_VERSION=4.3.1

# Install dependencies
RUN apt update && \
	apt install -y --no-install-recommends \
		wget gcc make autoconf automake ca-certificates \
		openssl libssl-dev yasm git pkg-config \
		libpcre3-dev librtmp-dev libtheora-dev && \
    rm -rf /var/lib/apt/lists/*

# Download nginx source
RUN mkdir -p /tmp/build && \
	cd /tmp/build && \
	wget https://nginx.org/download/nginx-${NGINX_VERSION}.tar.gz && \
	tar -zxf nginx-${NGINX_VERSION}.tar.gz && \
	rm nginx-${NGINX_VERSION}.tar.gz

# Download rtmp-module source
RUN cd /tmp/build && \
    wget https://github.com/arut/nginx-rtmp-module/archive/v${NGINX_RTMP_MODULE_VERSION}.tar.gz && \
    tar -zxf v${NGINX_RTMP_MODULE_VERSION}.tar.gz && \
	rm v${NGINX_RTMP_MODULE_VERSION}.tar.gz

# Build nginx with nginx-rtmp module
RUN cd /tmp/build/nginx-${NGINX_VERSION} && \
    ./configure \
        --prefix=/app/nginx \
        --conf-path=/etc/nginx/nginx.conf \
        --error-log-path=/var/log/nginx/error.log \
        --http-log-path=/var/log/nginx/access.log \		
        --pid-path=/var/run/nginx/nginx.pid \
        --lock-path=/var/lock/nginx.lock \
        --http-client-body-temp-path=/tmp/nginx-client-body \
        --with-http_ssl_module \
        --with-threads \
        --add-module=/tmp/build/nginx-rtmp-module-${NGINX_RTMP_MODULE_VERSION} \
        --with-cc-opt="-Wimplicit-fallthrough=0" && \
    make -j $(getconf _NPROCESSORS_ONLN) && \
    make install

# Download ffmpeg source
RUN cd /tmp/build && \
  wget http://ffmpeg.org/releases/ffmpeg-${FFMPEG_VERSION}.tar.gz && \
  tar -zxf ffmpeg-${FFMPEG_VERSION}.tar.gz && \
  rm ffmpeg-${FFMPEG_VERSION}.tar.gz
  
# Get ffnvcodec
RUN git clone https://git.videolan.org/git/ffmpeg/nv-codec-headers.git && \
  cd nv-codec-headers && \
  make -j $(getconf _NPROCESSORS_ONLN) && \
  make install
  
# Build ffmpeg
RUN cd /tmp/build/ffmpeg-${FFMPEG_VERSION} && \
  ./configure \
          --prefix=/app/ffmpeg \
	  --enable-cuda \
	  --enable-cuvid \
	  --enable-nvenc \
	  --enable-nonfree \
	  --enable-libnpp \
	  --extra-cflags=-I/usr/local/cuda/include  \
	  --extra-ldflags=-L/usr/local/cuda/lib64 \
	  --disable-debug \
	  --disable-doc \
	  --disable-ffplay \
	  --extra-libs="-lpthread -lm" && \
	make -j $(getconf _NPROCESSORS_ONLN) && \
	make install
	
# Copy stats.xsl file to nginx html directory and cleaning build files
RUN cp /tmp/build/nginx-rtmp-module-${NGINX_RTMP_MODULE_VERSION}/stat.xsl /app/nginx/html/stat.xsl && \
	rm -rf /tmp/build

##### Building the final image #####
FROM nvidia/cuda:11.1-runtime-ubuntu20.04

ENV  NVIDIA_DRIVER_VERSION=455
ENV  NVIDIA_VISIBLE_DEVICES all
ENV  NVIDIA_DRIVER_CAPABILITIES compute,video,utility

#Setup nvidia-patch
COPY patch.sh docker-entrypoint.sh /app/
RUN mkdir -p /patched-lib && \
  chmod +x /app/patch.sh /app/docker-entrypoint.sh && \
  ln -s /app/patch.sh /usr/local/bin/patch.sh

# Install dependencies
RUN apt-get update && \
	apt-get install -y --no-install-recommends \
		ca-certificates openssl libpcre3-dev \
		librtmp1 libtheora0 libnvidia-decode-$NVIDIA_DRIVER_VERSION libnvidia-encode-$NVIDIA_DRIVER_VERSION && \
    rm -rf /var/lib/apt/lists/*

# Copy files from build stage to final stage	
COPY --from=builder /app /app
COPY --from=builder /etc/nginx /etc/nginx
COPY --from=builder /var/log/nginx /var/log/nginx
COPY --from=builder /var/lock /var/lock
COPY --from=builder /var/run/nginx /var/run/nginx

# Forward logs to Docker
RUN ln -sf /dev/stdout /var/log/nginx/access.log && \
    ln -sf /dev/stderr /var/log/nginx/error.log

# Copy  nginx config file to container
COPY conf/nginx-nvenc.conf /etc/nginx/nginx.conf

# Copy  html players to container
COPY players /app/nginx/html/players

EXPOSE 1935
EXPOSE 8080

ENTRYPOINT ["/app/docker-entrypoint.sh"]
CMD ["/app/nginx/sbin/nginx", "-g", "daemon off;"]
