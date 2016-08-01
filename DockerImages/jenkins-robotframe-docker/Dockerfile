FROM ubuntu:latest
MAINTAINER John Lin <knives1003@gmail.com>
#Install Robot Framework

RUN apt-get update
RUN apt-get install -y git curl unzip libssl-dev python-pip python-dev gcc  phantomjs libnss3-dev 
#
#Install Google chromium
RUN apt-get install -y 
RUN apt-get install -y chromium-browser xvfb 



# Install selenium2 Chrome Driver
RUN curl -o chromedriver_linux64.zip http://chromedriver.storage.googleapis.com/2.20/chromedriver_linux64.zip 
RUN unzip chromedriver_linux64.zip -d .
RUN chmod +x ./chromedriver
RUN mv -f ./chromedriver /usr/local/share/chromedriver

#  Change the directory to /usr/bin/chromedriver
RUN ln -s /usr/local/share/chromedriver /usr/local/bin/chromedriver
RUN ln -s /usr/local/share/chromedriver /usr/bin/chromedriver
RUN pip install --upgrade setuptools
RUN pip install --upgrade pip
RUN pip install robotframework 
RUN pip install pycrypto
RUN pip install robotframework-sshlibrary 
RUN pip install robotframework-selenium2library



#Install Chinese font
RUN apt-get install fonts-wqy-zenhei

RUN mkdir /robot
RUN mkdir /testing

ENV DISPLAY=:1.0
ENV ROBOT_TESTS=/testing/






# Install Java
ENV DEBIAN_FRONTEND noninteractive
ENV INITRD No
RUN apt-get update && \
    apt-get install --no-install-recommends -y openjdk-8-jre-headless  openjdk-8-jdk
# Install Jenkins

ENV JENKINS_HOME /var/jenkins_home
ENV JENKINS_SLAVE_AGENT_PORT 50000

ARG user=jenkins
ARG group=jenkins
ARG uid=1000
ARG gid=1000

# Jenkins is run with user `jenkins`, uid = 1000
# If you bind mount a volume from the host or a data container, 
# ensure you use the same uid
RUN groupadd -g ${gid} ${group} \
    && useradd -d "$JENKINS_HOME" -u ${uid} -g ${gid} -m -s /bin/bash ${user}

# Jenkins home directory is a volume, so configuration and build history 
# can be persisted and survive image upgrades
VOLUME /var/jenkins_home

# `/usr/share/jenkins/ref/` contains all reference configuration we want 
# to set on a fresh new installation. Use it to bundle additional plugins 
# or config file with your custom jenkins Docker image.
RUN mkdir -p /usr/share/jenkins/ref/init.groovy.d

ENV TINI_VERSION 0.9.0
ENV TINI_SHA fa23d1e20732501c3bb8eeeca423c89ac80ed452

# Use tini as subreaper in Docker container to adopt zombie processes 
RUN curl -fsSL https://github.com/krallin/tini/releases/download/v${TINI_VERSION}/tini-static -o /bin/tini && chmod +x /bin/tini \
  && echo "$TINI_SHA  /bin/tini" | sha1sum -c -

COPY init.groovy /usr/share/jenkins/ref/init.groovy.d/tcp-slave-agent-port.groovy

ARG JENKINS_VERSION
ENV JENKINS_VERSION ${JENKINS_VERSION:-2.7.1}
ARG JENKINS_SHA
ENV JENKINS_SHA ${JENKINS_SHA:-12d820574c8f586f7d441986dd53bcfe72b95453}


# could use ADD but this one does not check Last-Modified header 
# see https://github.com/docker/docker/issues/8331
RUN curl -fsSL http://repo.jenkins-ci.org/public/org/jenkins-ci/main/jenkins-war/${JENKINS_VERSION}/jenkins-war-${JENKINS_VERSION}.war -o /usr/share/jenkins/jenkins.war \
  && echo "$JENKINS_SHA  /usr/share/jenkins/jenkins.war" | sha1sum -c -

ENV JENKINS_UC https://updates.jenkins.io
RUN chown -R ${user} "$JENKINS_HOME" /usr/share/jenkins/ref

# for main web interface:
EXPOSE 8080

# will be used by attached slave agents:
EXPOSE 50000

ENV COPY_REFERENCE_FILE_LOG $JENKINS_HOME/copy_reference_file.log

USER ${user}

#COPY Scripts 
COPY shell_scripts/jenkins.sh /usr/local/bin/jenkins.sh
COPY shell_scripts/plugins.sh /usr/local/bin/plugins.sh
COPY shell_scripts/install_jenkins_plugin_with_dependency.sh /usr/local/bin/install_jenkins_plugin_with_dependency.sh

# Copy pre-install plugins list 
# Then you can Use /usr/local/bin/install_jenkins_plugin_with_dependency.sh /usr/share/jenkins/ref/plugins.txt 
# to Pre-install jenkins plugins

COPY plugins.txt /usr/share/jenkins/ref/plugins.txt

# Plugins list Format :
# Every row like :{pluginId}:{version} or {pluginId} 
# If you don't assign version , the default verison is latest

USER root

#RUN  /usr/local/bin/install_jenkins_plugin_with_dependency.sh /usr/share/jenkins/ref/plugins.txt
ENTRYPOINT ["/bin/tini", "--", "/usr/local/bin/jenkins.sh"]












