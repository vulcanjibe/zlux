FROM debian:buster-slim



ARG SSH_KEY_PRIVATE
ARG SSH_KEY_PUBLIC
ENV SSH_KEY_PRIVATE $SSH_KEY_PRIVATE

RUN echo $SSH_KEY_PRIVATE


USER root

############
# Oracle JDK
############

# Preparation

ENV JAVA_VERSION 8u261
ENV JAVA_BUILD 15
ENV JAVA_HOME /etc/jdk-${JAVA_VERSION}-b${JAVA_BUILD}

# Installation

RUN apt-get update \
    && apt-get install -y wget \
    && cd /tmp \
    && wget --no-check-certificate --header "Cookie: oraclelicense=accept-securebackup-cookie" https://javadl.oracle.com/webapps/download/GetFile/1.8.0_261-b12/a4634525489241b9a9e1aa73d9e118e6/linux-i586/jdk-${JAVA_VERSION}-linux-x64.tar.gz -O jdk-${JAVA_VERSION}-linux-x64.tar.gz \
       && mkdir jdk-${JAVA_VERSION}-linux-x64 \
    && tar -zxvf jdk-${JAVA_VERSION}-linux-x64.tar.gz --directory jdk-${JAVA_VERSION}-linux-x64 --strip-components=1 \
    && mv jdk-${JAVA_VERSION}-linux-x64 ${JAVA_HOME} \
    && rm jdk-${JAVA_VERSION}-linux-x64.tar.gz \
    && rm -rf ${JAVA_HOME}/*src.zip \
           ${JAVA_HOME}/lib/missioncontrol \
           ${JAVA_HOME}/lib/visualvm \
           ${JAVA_HOME}/lib/*javafx* \
           ${JAVA_HOME}/jre/lib/plugin.jar \
           ${JAVA_HOME}/jre/lib/ext/jfxrt.jar \
           ${JAVA_HOME}/jre/bin/javaws \
           ${JAVA_HOME}/jre/lib/javaws.jar \
           ${JAVA_HOME}/jre/lib/desktop \
           ${JAVA_HOME}/jre/plugin \
           ${JAVA_HOME}/jre/lib/deploy* \
           ${JAVA_HOME}/jre/lib/*javafx* \
           ${JAVA_HOME}/jre/lib/*jfx* \
           ${JAVA_HOME}/jre/lib/amd64/libdecora_sse.so \
           ${JAVA_HOME}/jre/lib/amd64/libprism_*.so \
           ${JAVA_HOME}/jre/lib/amd64/libfxplugins.so \
           ${JAVA_HOME}/jre/lib/amd64/libglass.so \
           ${JAVA_HOME}/jre/lib/amd64/libgstreamer-lite.so \
           ${JAVA_HOME}/jre/lib/amd64/libjavafx*.so \
           ${JAVA_HOME}/jre/lib/amd64/libjfx*.so

ENV PATH ${PATH}:${JAVA_HOME}/bin

# Cleanup

RUN unset JAVA_VERSION

#########
# Testing
#########

RUN env
RUN java -version




#####
# Ant
#####

# Preparation

ENV ANT_VERSION 1.10.9
ENV ANT_HOME /etc/ant-${ANT_VERSION}

# Installation

RUN cd /tmp \
    && apt-get install -y wget \
    && wget http://www.us.apache.org/dist/ant/binaries/apache-ant-${ANT_VERSION}-bin.tar.gz \
    && mkdir ant-${ANT_VERSION} \
    && tar -zxvf apache-ant-${ANT_VERSION}-bin.tar.gz --directory ant-${ANT_VERSION} --strip-components=1 \
    && mv ant-${ANT_VERSION} ${ANT_HOME} \
    && rm apache-ant-${ANT_VERSION}-bin.tar.gz \
    && rm -rf ${ANT_HOME}/manual \
    && unset ANT_VERSION
ENV PATH ${PATH}:${ANT_HOME}/bin

#############
# Ant Contrib
#############

# Preparation

ENV ANT_CONTRIB_VERSION 1.0b3

# Installation

RUN cd /tmp \
    && apt-get install -y wget \
    && wget https://deac-ams.dl.sourceforge.net/project/ant-contrib/ant-contrib/1.0b3/ant-contrib-1.0b3-bin.tar.gz \
    && mkdir ant-contrib-${ANT_CONTRIB_VERSION} \
    && tar -zxvf ant-contrib-${ANT_CONTRIB_VERSION}-bin.tar.gz --directory ant-contrib-${ANT_CONTRIB_VERSION} --strip-components=1 \
    && cp ant-contrib-${ANT_CONTRIB_VERSION}/ant-contrib-${ANT_CONTRIB_VERSION}.jar ${ANT_HOME}/lib \
    && rm -rf ant-contrib-${ANT_CONTRIB_VERSION} \
    && rm ant-contrib-${ANT_CONTRIB_VERSION}-bin.tar.gz \
    && unset ANT_CONTRIB_VERSION

#########
# Testing
#########

RUN env
RUN java -version
RUN javac -version
RUN ant -version

############
# Installation nodejs, npm, git, python2
###########

RUN apt-get update \
    && apt-get upgrade \
    && apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates \
    && curl -sL https://deb.nodesource.com/setup_12.x | bash - \
    && apt-get install -y nodejs \
    && apt-get install -y gcc g++ make \
    && apt-get install -y git \
    && apt-get install -y python \
    && mkdir /home/zowe \
    && cd /home/zowe

#############
# CLONE du git ZOWE/ZLUX
#############

# Make ssh dir

RUN mkdir -m 700 /root/.ssh \
    && touch -m 600 /root/.ssh/known_hosts \
    && ssh-keyscan github.com > /root/.ssh/known_hosts
 
 # encodage cat /root/.ssh/id_rsa | openssl base64 | tr -d '\n'
RUN touch -m 600 /root/.ssh/id_rsa \
    && echo "$SSH_KEY_PRIVATE" |  openssl base64 -A -d > /root/.ssh/id_rsa \
    && chmod 400 /root/.ssh/id_rsa
 
RUN touch -m 600 /root/.ssh/id_rsa.pub \
    && echo ${SSH_KEY_PUBLIC} > /root/.ssh/id_rsa.pub \
    && chmod 400 /root/.ssh/id_rsa.pub




#COPY .ssh/id_rsa /root/.ssh
#COPY .ssh/id_rsa.pub /root/.ssh


RUN cd /home/zowe \
    && git clone --recursive --quiet git@github.com:zowe/zlux.git \
    && cd zlux \
    && git submodule foreach "git checkout master"       \
    && cd zlux-build

################
# Suppression des clés SSH
################
RUN rm /root/.ssh/id_rsa.pub \
    && rm /root/.ssh/id_rsa

############
# BUILD
###########

RUN cd /home/zowe/zlux/zlux-build \
    && ./build.sh 

EXPOSE 8544
RUN chmod a+x /home/zowe/zlux/zlux-app-server/bin/app-server.sh

WORKDIR "/home/zowe/zlux/zlux-app-server/bin"
ENTRYPOINT "./app-server.sh"

