FROM ghcr.io/linuxserver/baseimage-ubuntu:jammy

# set version label
ARG BUILD_DATE
ARG VERSION
ARG CODE_RELEASE
LABEL build_version="Linuxserver.io version:- ${VERSION} Build-date:- ${BUILD_DATE}"
LABEL maintainer="Dinindu Senanayake"

# environment settings
ARG DEBIAN_FRONTEND="noninteractive"
ENV HOME="/config"

RUN \
  echo "**** install runtime dependencies ****" && \
  apt-get update && \
  apt-get install -y \
    curl \
    dnsutils \
    git \
    jq \
    ldap-utils \
    libnss-ldapd \
    libpam-ldapd \
    libatomic1 \
    nano \
    net-tools \
    netcat \
    nodejs \
    nscd \
    nslcd \
    rsync \
    unzip \
    vim \
    wget \
    zip \
    sudo && \
  echo "**** install code-server ****" && \
  if [ -z ${CODE_RELEASE+x} ]; then \
    CODE_RELEASE=$(curl -sX GET https://api.github.com/repos/coder/code-server/releases/latest \
      | awk '/tag_name/{print $4;exit}' FS='[""]' | sed 's|^v||'); \
  fi && \
  mkdir -p /app/code-server && \
  curl -o \
    /tmp/code-server.tar.gz -L \
    "https://github.com/coder/code-server/releases/download/v${CODE_RELEASE}/code-server-${CODE_RELEASE}-linux-amd64.tar.gz" && \
  tar xf /tmp/code-server.tar.gz -C \
    /app/code-server --strip-components=1 && \
  echo "**** clean up ****" && \
  apt-get clean && \
  rm -rf \
    /config/* \
    /tmp/* \
    /var/lib/apt/lists/* \
    /var/tmp/* \
    && localedef -i en_NZ -c -f UTF-8 -A /usr/share/locale/locale.alias en_NZ.UTF-8

ENV LANG en_NZ.utf8


# install kubectl, required for running on the k8s cluster
ARG KUBECTL_VERSION=v1.28.5
RUN curl -LO https://dl.k8s.io/release/$KUBECTL_VERSION/bin/linux/amd64/kubectl \
    && mv kubectl /usr/local/bin/kubectl \
    && chmod +x /usr/local/bin/kubectl

# copy in ood k8s utils
ARG UTILS_HASH=6298fb01f7a7c66a8454e3f0fd74437a32491423
RUN git clone https://github.com/nesi/training-environment-k8s-utils.git /opt/ood-k8s-utils \
    && cd /opt/ood-k8s-utils \
    && git checkout $UTILS_HASH \
    && chmod +x /opt/ood-k8s-utils/files/* \
    && mv /opt/ood-k8s-utils/files/* /bin/ \
    && rm -rf /opt/ood-k8s-utils

# make a dummy module command to avoid warnings from ondemand job_script_content.sh
RUN echo "#!/bin/bash" > /bin/module \
    && chmod +x /bin/module


# add local files
COPY /root /

RUN ln -s /app/code-server/bin/code-server /usr/local/bin/code-server

# download the required data into the container image and make sure permission are ok
RUN wget -nv https://github.com/GenomicsAotearoa/introduction-to-shell/releases/download/2024-July/shell_data.zip \
        -O /var/lib/shell_data.zip \
    && unzip /var/lib/shell_data.zip -d /var/lib \
    && rm -f /var/lib/shell_data.zip \
    && chown -R root:root /var/lib/shell_data \
    && chmod -R o+rX /var/lib/shell_data 

# Following data is for inidigitdata intro2shell: download the required data into the container image and make sure permission are ok
RUN wget -nv https://github.com/GenomicsAotearoa/IndigiDataAotearoa-IntroToShell/releases/download/2023-Aug/sample-data.zip \
        -O /var/lib/sample-data.zip \
    && unzip /var/lib/sample-data.zip -d /var/lib \
    && rm -f /var/lib/sample-data.zip \
    && chown -R root:root /var/lib/sample-data \

# download the required data into the container image and make sure permission are ok
RUN wget -nv https://github.com/GenomicsAotearoa/shell-for-bioinformatics/releases/download/v2.0/puzzles_da.tar.gz \
        -O /var/lib/puzzles_da.tar.gz \
    && tar -xzf /var/lib/puzzles_da.tar.gz -C /var/lib \
    && rm -f /var/lib/puzzles_da.tar.gz \
    && chown -R root:root /var/lib/puzzles_da \
    && chmod -R o+rX /var/lib/puzzles_da \
    && wget -nv https://github.com/GenomicsAotearoa/shell-for-bioinformatics/releases/download/v2.0/shell4b_data.tar.gz \
        -O /var/lib/shell4b_data.tar.gz \
    && tar -xzf /var/lib/shell4b_data.tar.gz -C /var/lib \
    && rm -f /var/lib/shell4b_data.tar.gz \
    && chown -R root:root /var/lib/shell4b_data \
    && chmod -R o+rX /var/lib/shell4b_data



# ports and volumes
EXPOSE 8443