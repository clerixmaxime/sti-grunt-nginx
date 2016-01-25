FROM openshift/base-centos7

LABEL io.k8s.description="Platform for building and running Grunt frontends on nginx" \
      io.k8s.display-name="nginx" \
      io.openshift.expose-services="80:http" \
      io.openshift.tags="builder,nginx" \
      io.openshift.s2i.destination="/opt/s2i/destination"

ENV NGINX_VERSION 1.9.2-1.el7.ngx

ADD nginx.repo /etc/yum.repos.d/nginx.repo

# Install nginx
RUN curl -sO http://nginx.org/keys/nginx_signing.key && \
    rpm --import ./nginx_signing.key && \
    yum install -y nginx-${NGINX_VERSION} centos-release-scl && \
    INSTALL_PKGS="nodejs010 ruby200 ruby200-ruby-devel ruby200-rubygems libpng-devel" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y && \
    rm -f ./nginx_signing.key

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/nginx/access.log
RUN ln -sf /dev/stderr /var/log/nginx/error.log

# Copy the S2I scripts from the specific language image to /usr/local/sti
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

COPY ./contrib/ /opt/app-root

# Installation
RUN $PROMPT_COMMAND && \
    mkdir -p /opt/s2i/destination && \
    npm install npm -g && \
    npm install -g bower && \
    npm install -g grunt-cli && \
    gem install sass && \
    gem install compass && \
    npm cache clean

# Create nginx group and user, set file ownership to that user.
RUN chmod -R go+rw /opt/s2i/destination

# Open port
EXPOSE 80 443

USER 1001

CMD $STI_SCRIPTS_PATH/usage
