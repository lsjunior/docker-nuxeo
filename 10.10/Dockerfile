# vim:set ft=dockerfile:
FROM       eclipse-temurin:8u345-b01-jdk-jammy
MAINTAINER Nuxeo <packagers@nuxeo.com>


# Add needed convert tools
RUN apt-get update && apt-get install -y --no-install-recommends \
    perl \
    locales \
    pwgen \
    imagemagick \
    ffmpeg2theora \
    poppler-utils \
    libwpd-tools \
    exiftool \
    ghostscript \
    libreoffice \
    ffmpeg \
    x264 \
    unzip

# UFRaw
#RUN apt-get update && apt-get install -y software-properties-common && add-apt-repository ppa:crass/ufraw

# Remove setuid/setgid binaries from images for security
RUN find / -perm 6000 -type f -exec chmod a-s {} \; || true

ENV NUXEO_USER nuxeo
ENV NUXEO_HOME /opt/nuxeo/server
ENV HOME /opt/nuxeo/server
ARG NUXEO_VERSION=10.10-HF64
#ARG NUXEO_DIST_URL=http://community.nuxeo.com/static/releases/nuxeo-10.10/nuxeo-server-10.10-tomcat.zip
#ARG NUXEO_MD5=90ef2ac005020e880b6277510800c30c
ARG NUXEO_ZIP_FILE=nuxeo-server-tomcat-10.10-HF72.zip
ARG NUXEO_MD5=1593d0d339ddb6b60f12ffc54abab563


# Create Nuxeo user
RUN useradd -m -d /home/$NUXEO_USER -u 1000 -s /bin/bash $NUXEO_USER

# Add distribution
#RUN curl -fsSL "${NUXEO_DIST_URL}" -o /tmp/nuxeo-distribution-tomcat.zip \
#    && if [ $NUXEO_VERSION != "master" ]; then echo "$NUXEO_MD5 /tmp/nuxeo-distribution-tomcat.zip" | md5sum -c -; fi \
COPY $NUXEO_ZIP_FILE /tmp/nuxeo-distribution-tomcat.zip
RUN if [ $NUXEO_VERSION != "master" ]; then echo "$NUXEO_MD5 /tmp/nuxeo-distribution-tomcat.zip" | md5sum -c -; fi \
    && mkdir -p /tmp/nuxeo-distribution $(dirname $NUXEO_HOME) \
    && unzip -q -d /tmp/nuxeo-distribution /tmp/nuxeo-distribution-tomcat.zip \
    && DISTDIR=$(/bin/ls /tmp/nuxeo-distribution | head -n 1) \
    && mv /tmp/nuxeo-distribution/$DISTDIR $NUXEO_HOME \
    && sed -i -e "s/^org.nuxeo.distribution.package.*/org.nuxeo.distribution.package=docker/" $NUXEO_HOME/templates/common/config/distribution.properties \
    && rm -rf /tmp/nuxeo-distribution* \
    && chmod +x $NUXEO_HOME/bin/*ctl $NUXEO_HOME/bin/*.sh \
    # For arbitrary user in the admin group
    && chmod g+rwX $NUXEO_HOME/bin/*ctl $NUXEO_HOME/bin/*.sh \
    && $NUXEO_HOME/bin/nuxeoctl mp-init \
    && chown -R 1000:0 $NUXEO_HOME && chmod -R g+rwX $NUXEO_HOME

COPY docker-template $NUXEO_HOME/templates/docker
COPY nuxeo.conf /etc/nuxeo/nuxeo.conf.template
ENV NUXEO_CONF /etc/nuxeo/nuxeo.conf

# Protecting writeable directories to support arbitrary User IDs for OpenShift
# https://docs.openshift.com/container-platform/3.4/creating_images/guidelines.html
RUN chown -R 1000:0 /etc/nuxeo && chmod g+rwX /etc/nuxeo && rm -f $NUXEO_HOME/bin/nuxeo.conf \
    && mkdir -p /var/lib/nuxeo/data \
    && chown -R 1000:0 /var/lib/nuxeo/data && chmod -R g+rwX /var/lib/nuxeo/data \
    && mkdir -p /var/log/nuxeo \
    && chown -R 1000:0 /var/log/nuxeo && chmod -R g+rwX /var/log/nuxeo \
    && mkdir -p /var/run/nuxeo \
    && chown -R 1000:0 /var/run/nuxeo && chmod -R g+rwX /var/run/nuxeo \
    && mkdir -p /docker-entrypoint-initnuxeo.d \
    && chown -R 1000:0 /docker-entrypoint-initnuxeo.d && chmod -R g+rwX /docker-entrypoint-initnuxeo.d  \
    && chmod g=u /etc/passwd

ENV PATH $NUXEO_HOME/bin:$PATH

WORKDIR $NUXEO_HOME
COPY docker-entrypoint.sh /
ENTRYPOINT ["/docker-entrypoint.sh"]
EXPOSE 8080
CMD ["nuxeoctl","console"]

USER 1000
