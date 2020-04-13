FROM openjdk:11-jdk

ARG THEIA_USER=theia
ARG THEIA_HOME=/home/$THEIA_USER
ARG WORKSPACE_DIR=/home/workspace

ARG HOST_SSH_DIR=%userprofile%/.ssh

ARG MAVEN_VERSION=3.6.3
ARG MAVEN_SHA=c35a1803a6e70a126e80b2b3ae33eed961f83ed74d18fcd16909b2d44d7dada3203f1ffe726c17ef8dcca2dcaa9fca676987befeadc9b9f759967a8cb77181c0
ARG MAVEN_BASE_URL=https://downloads.apache.org/maven/maven-3/$MAVEN_VERSION/binaries/

# update system
RUN apt-get update

# Install essential build and dev tools
RUN apt-get install -y build-essential git && \
    # test
    make --version && \
    git --version


# Install Maven
RUN mkdir -p /usr/share/maven /usr/share/maven/ref && \
    curl -fsSL -o /tmp/apache-maven.tar.gz $MAVEN_BASE_URL/apache-maven-$MAVEN_VERSION-bin.tar.gz && \
    echo "$MAVEN_SHA  /tmp/apache-maven.tar.gz" | sha512sum -c - && \
    tar -xzf /tmp/apache-maven.tar.gz -C /usr/share/maven --strip-components=1 && \
    rm -f /tmp/apache-maven.tar.gz && \
    ln -s /usr/share/maven/bin/mvn /usr/bin/mvn && \
    # test
    mvn --version

ENV MAVEN_HOME /usr/share/maven

# Install Node
ENV NODE_VERSION 10.20.0
RUN curl -fsSLO --compressed "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.xz" && \
    tar -xJf "node-v${NODE_VERSION}-linux-x64.tar.xz" -C /usr/local --strip-components=1 --no-same-owner && \
    rm "node-v${NODE_VERSION}-linux-x64.tar.xz" && \
    ln -s /usr/local/bin/node /usr/local/bin/nodejs && \
    # test
    node --version && \
    npm --version

# Install yarn
RUN curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | apt-key add - && \
    echo "deb https://dl.yarnpkg.com/debian/ stable main" | tee /etc/apt/sources.list.d/yarn.list && \
    apt-get update && apt-get install -y yarn && \
    rm -rf /var/lib/apt/lists/* && \
    yarn --version

# create theia user
RUN adduser --disabled-password --gecos '' ${THEIA_USER}
RUN chmod g+rw /home && \
    mkdir -p ${WORKSPACE_DIR} && \
    chown -R ${THEIA_USER}:${THEIA_USER} ${THEIA_HOME} && \
    chown -R ${THEIA_USER}:${THEIA_USER} ${WORKSPACE_DIR};

WORKDIR ${THEIA_HOME}
USER ${THEIA_USER}

# Install Theia
COPY package.json ${THEIA_HOME}
RUN yarn --cache-folder ./ycache && rm -rf ./ycache && \
    NODE_OPTIONS="--max_old_space_size=4096" yarn theia build ; \
    yarn theia download:plugins

EXPOSE 3000

ENV SHELL=/bin/bash \
    THEIA_DEFAULT_PLUGINS=local-dir:${THEIA_HOME}/plugins

# ENTRYPOINT cannot resolve arguments. Copy it to an environment variable.
ENV WS_DIR=${WORKSPACE_DIR}
ENTRYPOINT yarn theia start $WS_DIR --hostname=0.0.0.0
