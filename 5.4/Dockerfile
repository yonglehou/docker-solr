
FROM    java:openjdk-8-jre
MAINTAINER  Martijn Koster "mak-docker@greenhills.co.uk"

# Override the solr download location with e.g.:
#   docker build -t mine --build-arg SOLR_DOWNLOAD_SERVER=http://www-eu.apache.org/dist/lucene/solr .
ARG SOLR_DOWNLOAD_SERVER

# Override the GPG keyserver with e.g.:
#   docker build -t mine --build-arg GPG_KEYSERVER=hkp://eu.pool.sks-keyservers.net .
ARG GPG_KEYSERVER

RUN apt-get update && \
  apt-get -y install lsof && \
  rm -rf /var/lib/apt/lists/*

ENV SOLR_USER solr
ENV SOLR_UID 8983

RUN groupadd -r -g $SOLR_UID $SOLR_USER && \
  useradd -r -u $SOLR_UID -g $SOLR_USER $SOLR_USER

ENV SOLR_KEY E6E21FFCDCEA14C95910EA65051A0FAF76BC6507
ENV GPG_KEYSERVER ${GPG_KEYSERVER:-hkp://ha.pool.sks-keyservers.net}
RUN gpg --keyserver "$GPG_KEYSERVER" --recv-keys "$SOLR_KEY"

ENV SOLR_VERSION 5.4.1
ENV SOLR_SHA256 3e4b4ec7bd728b49b2ebc3dbe8f3d1ef89fded4ab86b9e2f856bedd58c99f28b
ENV SOLR_URL ${SOLR_DOWNLOAD_SERVER:-https://archive.apache.org/dist/lucene/solr}/$SOLR_VERSION/solr-$SOLR_VERSION.tgz

RUN mkdir -p /opt/solr && \
  wget -nv $SOLR_URL -O /opt/solr.tgz && \
  wget -nv $SOLR_URL.asc -O /opt/solr.tgz.asc && \
  echo "$SOLR_SHA256 */opt/solr.tgz" | sha256sum -c - && \
  (>&2 ls -l /opt/solr.tgz /opt/solr.tgz.asc) && \
  gpg --batch --verify /opt/solr.tgz.asc /opt/solr.tgz && \
  tar -C /opt/solr --extract --file /opt/solr.tgz --strip-components=1 && \
  rm /opt/solr.tgz* && \
  mkdir -p /opt/solr/server/solr/lib /opt/solr/server/solr/mycores && \
  sed -i -e 's/#SOLR_PORT=8983/SOLR_PORT=8983/' /opt/solr/bin/solr.in.sh && \
  sed -i -e '/-Dsolr.clustering.enabled=true/ a SOLR_OPTS="$SOLR_OPTS -Dsun.net.inetaddr.ttl=60 -Dsun.net.inetaddr.negative.ttl=60"' /opt/solr/bin/solr.in.sh && \
  chown -R $SOLR_USER:$SOLR_USER /opt/solr && \
  mkdir /docker-entrypoint-initdb.d /opt/docker-solr/

COPY scripts /opt/docker-solr/scripts
RUN chown -R $SOLR_USER:$SOLR_USER /opt/docker-solr

ENV PATH /opt/solr/bin:/opt/docker-solr/scripts:$PATH

EXPOSE 8983
WORKDIR /opt/solr
USER $SOLR_USER

ENTRYPOINT ["docker-entrypoint.sh"]
CMD ["solr"]
