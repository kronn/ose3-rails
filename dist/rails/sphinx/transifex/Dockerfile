FROM centos/ruby-22-centos7
      
      
      

# ABOUT
# A Rails image to be used with OpenShift S2I.

USER root

ADD /contrib/bin/install-sphinx.sh $STI_SCRIPTS_PATH
RUN scl enable rh-ruby22 $STI_SCRIPTS_PATH/install-sphinx.sh 
USER root

# SLOW STUFF
# Slow operations, kept at top of the Dockerfile so they're cached for most changes.

# Install Apache httpd24.
RUN yum update -y && \
    INSTALL_PKGS="httpd httpd-devel apr-devel apr-util-devel sqlite3 ImageMagick-devel" && \
    yum install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    yum clean all -y

# Install passenger.
RUN /bin/bash -c "gem install passenger --no-ri --no-rdoc && \
    export PATH=$PATH:/opt/rh/rh-ruby22/root/usr/local/bin && \
    passenger-install-apache2-module --auto --languages ruby && \
    passenger-config validate-install " # bogus comment to invalidate cache

# CONFIGURATION

### Apache
# Disable digest_module.
RUN sed -i "s/LoadModule auth_digest_module/#LoadModule auth_digest_module/" /etc/httpd/conf.modules.d/00-base.conf
ENV APACHE_RUN_USER 1001
ENV APACHE_PID_FILE /opt/app-root/httpd.pid
# TODO Why do we do this? I suspect it's because of the OpenShift PID hack for apache.
RUN mkdir -p /opt/app-root/httpd/pid

### Passenger
# Add a symlink to the installed passenger gem
# so we do not depend on the minor version installed.
RUN ln -s /opt/rh/rh-ruby22/root/usr/local/share/gems/gems/passenger-5.* /opt/passenger

### Rails
ENV RAILS_ENV=production \
  RAILS_ROOT=/opt/app-root/src

### Add configuration files.
ADD /contrib/bin $STI_SCRIPTS_PATH
ADD /contrib/etc/httpd /etc/httpd
ADD /contrib/usr/local/lib /usr/local/lib

# PERMISSIONS

# TODO Why do we do this? Check with the fix-permissions call in $STI_SCRIPTS_PATH/assemble.
RUN chgrp -R 0 ./ && \
    chmod -R g+rw ./ && \
    find ./ -type d -exec chmod g+x {} + && \
    chown -R 1001:0 ./

# TODO Why do we do this? Check with the fix-permissions call in $STI_SCRIPTS_PATH/assemble.
RUN chmod -R a+rwX /opt/app-root/httpd/pid && \
    chmod +x $STI_SCRIPTS_PATH/run-httpd.sh

# Workaround for base image: Do not install gems from development and test environments
# See https://github.com/sclorg/rhscl-dockerfiles/issues/26
ENV BUNDLE_WITHOUT=development:test
USER root

ADD /contrib/bin/install-transifex.sh $STI_SCRIPTS_PATH
RUN scl enable rh-ruby22 $STI_SCRIPTS_PATH/install-transifex.sh 

# Add S2I scripts.
USER root

RUN mv $STI_SCRIPTS_PATH/assemble $STI_SCRIPTS_PATH/assemble_base
RUN mv $STI_SCRIPTS_PATH/run $STI_SCRIPTS_PATH/run_base
ADD /s2i/* $STI_SCRIPTS_PATH/

RUN chmod ugo+x $STI_SCRIPTS_PATH/*

USER 1001

USER 1001

# ENTRYPOINT

CMD $STI_SCRIPTS_PATH/run-httpd.sh