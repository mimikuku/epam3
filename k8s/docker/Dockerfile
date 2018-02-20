FROM jenkins/jenkins:latest
COPY plugins.txt /usr/share/jenkins/ref/
ARG PLUGINS_LIST
ARG USER_JENKINS=jenkins
ARG USER_ROOT=root
ARG GROUP_DOCKER=docker
ARG GROUP_DOCKER_GID=1001
RUN echo ${PLUGINS_LIST}
RUN [ -z "$PLUGINS_LIST" ] && /usr/local/bin/install-plugins.sh $(echo ${PLUGINS_LIST} | xargs) || \
    /usr/local/bin/install-plugins.sh $(cat /usr/share/jenkins/ref/plugins.txt)
USER ${USER_ROOT}

RUN apt-get update && apt-get install maven docker sudo -y
# Create docker group and add user jenkins to the group
RUN addgroup --gid ${GROUP_DOCKER_GID} ${GROUP_DOCKER} \
&& addgroup ${USER_JENKINS} ${GROUP_DOCKER}

# Add user jenkins to list of sudoers
RUN echo "jenkins ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# Switch to user jenkins
USER ${USER_JENKINS}

COPY docker-socket.sh /usr/bin

ENTRYPOINT ["/usr/bin/docker-socket.sh"]

