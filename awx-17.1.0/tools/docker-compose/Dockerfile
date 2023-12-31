### This file is generated from
### installer/roles/image_build/templates/Dockerfile.j2
###
### DO NOT EDIT
###

# Locations - set globally to be used across stages
ARG COLLECTION_BASE="/var/lib/awx/vendor/awx_ansible_collections"

# Build container
FROM centos:8 as builder

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

USER root

# Install build dependencies
RUN dnf -y module enable 'postgresql:12'
RUN dnf -y update && \
    dnf -y install epel-release 'dnf-command(config-manager)' && \
    dnf module -y enable 'postgresql:12' && \
    dnf config-manager --set-enabled powertools && \
    dnf -y install ansible \
    gcc \
    gcc-c++ \
    git-core \
    glibc-langpack-en \
    libcurl-devel \
    libffi-devel \
    libtool-ltdl-devel \
    make \
    nodejs \
    nss \
    openldap-devel \
    patch \
    @postgresql:12 \
    postgresql-devel \
    python3-devel \
    python3-pip \
    python3-psycopg2 \
    python3-setuptools \
    swig \
    unzip \
    xmlsec1-devel \
    xmlsec1-openssl-devel

RUN python3 -m ensurepip && pip3 install "virtualenv < 20"

# Install & build requirements
ADD Makefile /tmp/Makefile
RUN mkdir /tmp/requirements
ADD requirements/requirements_ansible.txt \
    requirements/requirements_ansible_uninstall.txt \
    requirements/requirements_ansible_git.txt \
    requirements/requirements.txt \
    requirements/requirements_tower_uninstall.txt \
    requirements/requirements_git.txt \
    requirements/collections_requirements.yml \
    /tmp/requirements/

RUN cd /tmp && make requirements_awx requirements_ansible_py3
RUN cd /tmp && make requirements_collections

ADD requirements/requirements_dev.txt /tmp/requirements
RUN cd /tmp && make requirements_awx_dev requirements_ansible_dev


# Final container(s)
FROM centos:8

ARG COLLECTION_BASE

ENV LANG en_US.UTF-8
ENV LANGUAGE en_US:en
ENV LC_ALL en_US.UTF-8

USER root

# Install runtime requirements
RUN dnf -y module enable 'postgresql:12'
RUN dnf -y update && \
    dnf -y install epel-release 'dnf-command(config-manager)' && \
    dnf module -y enable 'postgresql:12' && \
    dnf config-manager --set-enabled powertools && \
    dnf -y install acl \
    ansible \
    bubblewrap \
    git-core \
    git-lfs \
    glibc-langpack-en \
    krb5-workstation \
    libcgroup-tools \
    nginx \
    @postgresql:12 \
    python3-devel \
    python3-libselinux \
    python3-pip \
    python3-psycopg2 \
    python3-setuptools \
    rsync \
    subversion \
    sudo \
    vim-minimal \
    which \
    unzip \
    xmlsec1-openssl && \
    dnf -y install centos-release-stream && dnf -y install "rsyslog >= 8.1911.0" && dnf -y remove centos-release-stream && \
    dnf -y clean all

# Install kubectl
RUN curl -L -o /usr/bin/kubectl https://storage.googleapis.com/kubernetes-release/release/v1.17.8/bin/linux/amd64/kubectl && \
    chmod a+x /usr/bin/kubectl

RUN curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && \
    chmod 700 get_helm.sh && \
    ./get_helm.sh

# Install tini
RUN curl -L -o /usr/bin/tini https://github.com/krallin/tini/releases/download/v0.19.0/tini-amd64 && \
    chmod +x /usr/bin/tini

RUN python3 -m ensurepip && pip3 install "virtualenv < 20" supervisor flake8
RUN rm -rf /root/.cache && rm -rf /tmp/*

# Install OpenShift CLI
RUN cd /usr/local/bin && \
    curl -L https://github.com/openshift/origin/releases/download/v3.11.0/openshift-origin-client-tools-v3.11.0-0cbc58b-linux-64bit.tar.gz | \
    tar -xz --strip-components=1 --wildcards --no-anchored 'oc'

# Install development/test requirements
RUN dnf -y install \
    gdb \
    gtk3 \
    gettext \
    alsa-lib \
    libX11-xcb \
    libXScrnSaver \
    strace \
    vim \
    nmap-ncat \
    nodejs \
    nss \
    make \
    patch \
    socat \
    tmux \
    wget \
    diffutils \
    unzip && \
    npm install -g n && n 14.15.1 && dnf remove -y nodejs

# This package randomly fails to download.
# It is nice to have in the dev env, but not necessary.
# Add it back to the list above if the repo ever straighten up.
RUN dnf --enablerepo=debuginfo -y install python3-debuginfo || :

# Copy app from builder
COPY --from=builder /var/lib/awx /var/lib/awx

RUN openssl req -nodes -newkey rsa:2048 -keyout /etc/nginx/nginx.key -out /etc/nginx/nginx.csr \
        -subj "/C=US/ST=North Carolina/L=Durham/O=Ansible/OU=AWX Development/CN=awx.localhost" && \
    openssl x509 -req -days 365 -in /etc/nginx/nginx.csr -signkey /etc/nginx/nginx.key -out /etc/nginx/nginx.crt && \
    chmod 640 /etc/nginx/nginx.{csr,key,crt}

# Create default awx rsyslog config
ADD installer/roles/image_build/files/rsyslog.conf /var/lib/awx/rsyslog/rsyslog.conf

## File mappings
ADD tools/docker-compose/launch_awx.sh /usr/bin/launch_awx.sh
ADD tools/docker-compose/awx-manage /usr/local/bin/awx-manage
ADD tools/docker-compose/awx.egg-link /tmp/awx.egg-link
ADD tools/docker-compose/nginx.conf /etc/nginx/nginx.conf
ADD tools/docker-compose/nginx.vh.default.conf /etc/nginx/conf.d/nginx.vh.default.conf
ADD tools/docker-compose/start_tests.sh /start_tests.sh
ADD tools/docker-compose/bootstrap_development.sh /usr/bin/bootstrap_development.sh
ADD tools/docker-compose/entrypoint.sh /entrypoint.sh
ADD tools/scripts/awx-python /usr/bin/awx-python

# Pre-create things we need to access
RUN for dir in \
      /var/lib/awx \
      /var/lib/awx/rsyslog \
      /var/lib/awx/rsyslog/conf.d \
      /var/run/awx-rsyslog \
      /var/log/tower \
      /var/log/nginx \
      /var/lib/postgresql \
      /var/run/supervisor \
      /var/lib/nginx ; \
    do mkdir -m 0775 -p $dir ; chmod g+rw $dir ; chgrp root $dir ; done && \
    for file in \
      /etc/passwd ; \
    do touch $file ; chmod g+rw $file ; chgrp root $file ; done

# Adjust any remaining permissions
RUN chmod u+s /usr/bin/bwrap ; \
    chgrp -R root ${COLLECTION_BASE} ; \
    chmod -R g+rw ${COLLECTION_BASE}

RUN for dir in \
      /var/lib/awx/venv \
      /var/lib/awx/venv/awx/lib/python3.6 \
      /var/lib/awx/projects \
      /var/lib/awx/rsyslog \
      /var/run/awx-rsyslog \
      /.ansible \
      /var/lib/awx/vendor ; \
    do mkdir -m 0775 -p $dir ; chmod g+rw $dir ; chgrp root $dir ; done && \
    for file in \
      /var/run/nginx.pid \
      /var/lib/awx/venv/awx/lib/python3.6/site-packages/awx.egg-link ; \
    do touch $file ; chmod g+rw $file ; done


ENV HOME="/var/lib/awx"
ENV PATH="/usr/pgsql-10/bin:${PATH}"

EXPOSE 8043 8013 8080 22

ENTRYPOINT ["/entrypoint.sh"]
CMD ["/bin/bash"]
