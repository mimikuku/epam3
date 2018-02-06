FROM jenkins
#USER root
#RUN echo "deb http://apt.dockerproject.org/repo debian-jessie main" \
#        > /etc/apt/sources.list.d/docker.list \
#    && apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 \
#        --recv-keys 58118E89F3A912897C070ADBF76221572C52609D \
#    && apt-get update \
#    && apt-get install -y apt-transport-https \
#    && apt-get install -y sudo \
#    && apt-get install -y docker-engine \
#    && rm -rf /var/lib/apt/lists/*
#RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
#USER jenkins
COPY plugins.txt /usr/share/jenkins/ref/
ARG PLUGINS_LIST
RUN echo ${PLUGINS_LIST}
RUN [ -z "$PLUGINS_LIST" ] && /usr/local/bin/install-plugins.sh $(echo ${PLUGINS_LIST} | xargs) || \
    /usr/local/bin/install-plugins.sh $(cat /usr/share/jenkins/ref/plugins.txt)
