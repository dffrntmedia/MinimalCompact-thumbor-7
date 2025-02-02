FROM python:3.11

LABEL maintainer="Montro"

VOLUME /data

RUN  \
    apt-get update && \
    apt-get -y upgrade && \
    apt-get -y autoremove && \
    apt-get install -y -q \
        git \
        curl \
        ca-certificates \
        libjpeg-turbo-progs \
        graphicsmagick \
        libgraphicsmagick++3 \
        libgraphicsmagick++1-dev \
        libgraphicsmagick-q16-3 \
        libmagickwand-dev \
        zlib1g-dev \
        libboost-python-dev \
        libmemcached-dev \
        gifsicle \
        ffmpeg && \
    apt-get clean && \
    update-ca-certificates

ENV HOME /app
ENV SHELL bash
ENV WORKON_HOME /app
WORKDIR /app

RUN pip install thumbor==7.7.4
COPY requirements.txt /app/requirements.txt
RUN pip install --trusted-host None --no-cache-dir -r /app/requirements.txt
RUN pip install --no-dependencies tc-core==0.5

RUN pip install thumbor_request_modifier==2.0.4
RUN pip install tc_redis==2.5.0
RUN pip install tc_prometheus==2.0.0

COPY conf/thumbor.conf.tpl /app/thumbor.conf.tpl

ARG SIMD_LEVEL

RUN PILLOW_VERSION=$(python -c 'import PIL; print(PIL.__version__)') ; \
    if [ "$SIMD_LEVEL" ]; then \
      pip uninstall -y pillow || true && \
      CC="cc -m$SIMD_LEVEL" pip install --no-cache-dir -U --force-reinstall --no-binary=:all: "pillow-SIMD<=${PILLOW_VERSION}.post99" \
      # --global-option="build_ext" --global-option="--debug" \
      --global-option="build_ext" --global-option="--enable-lcms" \
      --global-option="build_ext" --global-option="--enable-zlib" \
      --global-option="build_ext" --global-option="--enable-jpeg" \
      --global-option="build_ext" --global-option="--enable-tiff" ; \
    fi ;

COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]

CMD ["thumbor", "--port=8000", "-l ${LOG_LEVEL:-debug}", "--processes=${THUMBOR_NUM_PROCESSES:-1}"]

# Thumbor is on 8000, Prometheus is on 8001
EXPOSE 8000 8001
