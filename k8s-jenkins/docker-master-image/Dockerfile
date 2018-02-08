FROM jenkins/jenkins:latest
USER root
RUN apt-get update && apt-get -y install sudo
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
USER jenkins
COPY plugins.txt /usr/share/jenkins/ref/
ARG PLUGINS_LIST
RUN echo ${PLUGINS_LIST}
RUN [ -z "$PLUGINS_LIST" ] && /usr/local/bin/install-plugins.sh $(echo ${PLUGINS_LIST} | xargs) || \
    /usr/local/bin/install-plugins.sh $(cat /usr/share/jenkins/ref/plugins.txt)
