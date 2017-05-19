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
    curl --silent --location https://rpm.nodesource.com/setup_6.x | bash - && \
    INSTALL_PKGS="nodejs ruby200 ruby200-ruby-devel ruby200-rubygems libpng-devel" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
    yum clean all -y && \
    rm -f ./nginx_signing.key

# forward request and error logs to docker log collector
# Using /proc/self/fd/1 and /proc/self/fd/2 instead of /dev/stdout and /dev/stderr
# to bypass https://github.com/docker/docker/issues/6880
RUN ln -sf /proc/self/fd/1 /var/log/nginx/access.log && \
    ln -sf /proc/self/fd/2 /var/log/nginx/error.log && \
    chmod -R 777 /var/log/nginx /var/cache/nginx/ && \
    chmod -R 777 /etc/nginx/* && \
    # Acquire ownership of /var/log/nginx to bypass https://github.com/docker/docker/issues/6880
    chown -R 1001:0 /var/log/nginx && \
    # Make /var/run world-writeable to allow nginx to create /var/run/nginx.pid
    chmod -R 777 /var/run && \
    # Take ownership of the nginx documents directory
    chown -R 1001:0 /usr/share/nginx/html && \
    mkdir -p /var/www/frontend && \
    chmod -R 777 /var/www/frontend

# Copy the S2I scripts from the specific language image to /usr/local/sti
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

COPY ./contrib/ /opt/app-root

# Installation
RUN mkdir -p /opt/s2i/destination && \
    npm install npm -g && \
    npm install -g bower && \
    npm install -g grunt-cli && \
    npm cache clean

# Create nginx group and user, set file ownership to that user.
RUN chmod -R 777 /opt/s2i/destination

# Open port
EXPOSE 8080 8443

USER 1001

CMD $STI_SCRIPTS_PATH/usage
