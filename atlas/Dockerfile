ARG VERSION=2.2.0

FROM ubuntu:jammy AS build-atlas
ARG VERSION
RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y install apt-utils \
    && apt-get -y install \
      maven \
      wget \
      patch \
      git \
      openjdk-8-jdk-headless \
      unzip

ENV MAVEN_OPTS="-Xms2g -Xmx2g"
ENV JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
WORKDIR /tmp
RUN wget https://dlcdn.apache.org/atlas/${VERSION}/apache-atlas-${VERSION}-sources.tar.gz \
    && tar -xzf apache-atlas-${VERSION}-sources.tar.gz \
    && rm apache-atlas-${VERSION}-sources.tar.gz \
    && cd apache-atlas-sources-${VERSION} \
    && mvn clean -DskipTests package -Pdist,embedded-hbase-solr
RUN mv /tmp/apache-atlas-sources-${VERSION}/distro/target/apache-atlas-${VERSION}-server.tar.gz /tmp/apache-atlas-server.tar.gz
RUN tar -xzf /tmp/apache-atlas-server.tar.gz -C /opt \
    && rm -Rf /tmp/apache-atlas-server.tar.gz

COPY atlas_start.py.patch atlas_config.py.patch /opt/apache-atlas-${VERSION}/bin/

RUN cd /opt/apache-atlas-${VERSION}/bin \
    && patch -b -f < atlas_start.py.patch \
    && patch -b -f < atlas_config.py.patch

FROM ubuntu:jammy AS deploy
ARG VERSION
COPY --from=build-atlas /opt/apache-atlas-${VERSION} /opt/apache-atlas-${VERSION}

ENV JAVA_HOME="/usr/lib/jvm/java-8-openjdk-amd64"
ENV MANAGE_LOCAL_HBASE=true
ENV MANAGE_LOCAL_SOLR=true
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get -y upgrade \
    && apt-get -y install software-properties-common \
    && add-apt-repository -y ppa:deadsnakes/ppa \
    && apt-get -y install \
      openjdk-8-jdk-headless \
      python3.9 \
    && ln -s /usr/bin/python3 /usr/bin/python

COPY conf/hbase-site.xml.template /opt/apache-atlas-${VERSION}/conf/hbase/hbase-site.xml.template
COPY conf/atlas-env.sh /opt/apache-atlas-${VERSION}/conf/atlas-env.sh

RUN mv /opt/apache-atlas-${VERSION} /opt/apache-atlas \
    && /opt/apache-atlas/bin/atlas_start.py -setup || true

CMD [ "/opt/apache-atlas/bin/atlas_start.py" ]
