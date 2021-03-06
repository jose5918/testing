# Docker image for running E2E tests using Argo.

FROM ubuntu:xenial
MAINTAINER Jeremy Lewi

# Never prompt the user for choices on installation/configuration of packages
ENV DEBIAN_FRONTEND noninteractive
ENV TERM linux

# gcc & python-dev are needed so we can install crcmod for gsutil
RUN set -ex \
    && apt-get update -yqq \
    && apt-get install -yqq --no-install-recommends \
        curl \
        locales \
        wget \
        ca-certificates \
        git \
        zip \
        unzip \
        gcc python-dev \
        python-setuptools \
    && apt-get clean \
    && rm -rf \
        /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/* \
        /usr/share/man \
        /usr/share/doc \
        /usr/share/doc-base

# Install go
RUN cd /tmp && \
    wget -O /tmp/go.tar.gz https://redirector.gvt1.com/edgedl/go/go1.9.2.linux-amd64.tar.gz && \
    tar -C /usr/local -xzf go.tar.gz 

# Install gcloud
ENV PATH=/usr/local/go/bin:/google-cloud-sdk/bin:/workspace:${PATH} \
    CLOUDSDK_CORE_DISABLE_PROMPTS=1

RUN wget -q https://dl.google.com/dl/cloudsdk/channels/rapid/google-cloud-sdk.tar.gz && \
    tar xzf google-cloud-sdk.tar.gz -C / && \
    rm google-cloud-sdk.tar.gz && \
    /google-cloud-sdk/install.sh \
        --disable-installation-options \
        --bash-completion=false \
        --path-update=false \
        --usage-reporting=false && \
    gcloud components install alpha beta kubectl

# Install CRCMOD for gsutil
RUN easy_install -U pip && \
    pip install -U crcmod

# Install Helm
RUN wget -O /tmp/get_helm.sh \
    https://raw.githubusercontent.com/kubernetes/helm/master/scripts/get && \
    chmod 700 /tmp/get_helm.sh && \
    /tmp/get_helm.sh && \
    rm /tmp/get_helm.sh

# Initialize helm
RUN helm init --client-only

# Install  Node.js
RUN curl -sL https://deb.nodesource.com/setup_8.x | bash - \
    && apt-get install -y nodejs

# Install yarn
RUN curl -sS http://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - \
    && echo "deb http://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list \
    && apt-get update -yqq \
    && apt-get install -yqq --no-install-recommends yarn


# Install glide
RUN cd /tmp && \
    wget -O glide-v0.13.0-linux-amd64.tar.gz \
            https://github.com/Masterminds/glide/releases/download/v0.13.0/glide-v0.13.0-linux-amd64.tar.gz && \
    tar -xvf glide-v0.13.0-linux-amd64.tar.gz && \
    mv ./linux-amd64/glide /usr/local/bin/

# Install ksonnet
RUN cd /tmp && \
    wget -O ks.tar.gz \
    https://github.com/ksonnet/ksonnet/releases/download/v0.8.0/ks_0.8.0_linux_amd64.tar.gz  && \
    tar -xvf ks.tar.gz && \
    mv ks_0.8.0_linux_amd64/ks /usr/local/bin && \    
    chmod a+x /usr/local/bin/ks


# Install various python libraries.
RUN  pip install --upgrade six pyyaml google-api-python-client \
     google-cloud-storage google-auth-httplib2 pylint kubernetes==4.0.0 mock retrying \
     jinja2

COPY checkout.sh /usr/local/bin
RUN chmod a+x /usr/local/bin/checkout.sh

# Work around for https://github.com/ksonnet/ksonnet/issues/298
ENV USER root

# Add the directory where we will checkout kubeflow/testing 
# which contains shared scripts.
ENV PYTHONPATH /src/kubeflow/testing/py
