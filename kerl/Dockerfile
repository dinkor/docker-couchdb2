FROM debian:jessie-slim

# Based on https://github.com/klaemo/docker-couchdb/blob/master/2.0.0/Dockerfile
MAINTAINER Sergey Korobkov skorobkov@dinkor.net

ARG USER=couchdb
ARG DUMB_INIT_VERSION=1.2.0
ARG ERLANG_VERSION=18.3
ARG COUCHDB_REPO=https://github.com/apache/couchdb.git
ARG COUCHDB_BRANCH=master
ARG COUCHDB_DIR=/opt/couchdb
ARG CONFIGURE_OPTIONS="--disable-docs"
ARG PATH=/tmp/erlang/bin:${PATH}

# Add CouchDB user account

RUN echo 'APT::Install-Recommends "false";' > /etc/apt/apt.conf.d/build_docker_norecommends \
	&& echo 'APT::Get::AutomaticRemove "true";' >> /etc/apt/apt.conf.d/build_docker_norecommends \
	&& echo 'APT::AutoRemove::RecommendsImportant "false";' >> /etc/apt/apt.conf.d/build_docker_norecommends \
	&& echo 'APT::AutoRemove::SuggestsImportant "false";' >> /etc/apt/apt.conf.d/build_docker_norecommends \
	&& apt-get update \ 
	&& apt-get upgrade -y \ 
	&& apt-get install -y \
		apt-transport-https \
		build-essential \
		git-core \
		libicu52 \
		libicu-dev \
		libmozjs185-1.0 \
		libmozjs185-dev \
		libncurses5-dev \
		libssl1.0.0 \
		libssl-dev \
		ca-certificates \
		curl \
		openssl \

	&& echo 'deb https://deb.nodesource.com/node_4.x jessie main' > /etc/apt/sources.list.d/nodesource.list \
	&& echo 'deb-src https://deb.nodesource.com/node_4.x jessie main' >> /etc/apt/sources.list.d/nodesource.list \
	&& curl -fSL -o /tmp/nodesource.gpg.key https://deb.nodesource.com/gpgkey/nodesource.gpg.key \
	&& apt-key add /tmp/nodesource.gpg.key \
	&& apt-get update \ 
	&& apt-get install -y \
		nodejs \

	&& curl -fSL -o /tmp/kerl https://raw.githubusercontent.com/kerl/kerl/master/kerl \
	&& chmod +x /tmp/kerl \
	&& /tmp/kerl update releases \
	&& /tmp/kerl build ${ERLANG_VERSION} ${ERLANG_VERSION} \
	&& /tmp/kerl install ${ERLANG_VERSION} /tmp/erlang \
	&& /tmp/kerl cleanup all \

	&& npm install -g grunt-cli \
	&& git clone --depth 1 --single-branch -b ${COUCHDB_BRANCH} ${COUCHDB_REPO} /tmp/couchdb \
	&& cd /tmp/couchdb \
	# TODO: remove this
	&& sed -i -e 's/^\({fauxton,\s*"fauxton",\s*\){tag, "v1.1.9"}\(.*\)$/\1"2cec2e4d975446080247c53ccc8d50c6b6fcbcba"\2/' rebar.config.script \
	&& ./configure --disable-docs \
	&& make release \
	&& useradd --user-group --home-dir ${COUCHDB_DIR} ${USER} \
	&& mkdir ${COUCHDB_DIR} \
	&& mv rel/couchdb/* ${COUCHDB_DIR} \
	&& mkdir ${COUCHDB_DIR}/data ${COUCHDB_DIR}/etc/local.d ${COUCHDB_DIR}/etc/default.d \
	&& chown -R ${USER}:${USER} ${COUCHDB_DIR} \
        && curl -fSL -o /tmp/dumb-init.deb https://github.com/Yelp/dumb-init/releases/download/v${DUMB_INIT_VERSION}/dumb-init_${DUMB_INIT_VERSION}_amd64.deb \
        && dpkg -i /tmp/dumb-init.deb \
	&& npm uninstall -g grunt-cli \
	&& apt-get purge -y \
		apt-transport-https \
		build-essential \
		git-core \
		libicu-dev \
		libmozjs185-dev \
		libncurses5-dev \
		libssl-dev \
		nodejs \
	&& apt-get clean \
	&& rm -f /etc/apt/apt.conf.d/build_docker_norecommends /etc/apt/sources.list.d/nodesource.list \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -rf ~/.kerl/ \
	&& rm -rf ~/.npm/ \
	&& rm -rf /tmp/*

COPY couchdb_* ${COUCHDB_DIR}/bin/
RUN chmod +x ${COUCHDB_DIR}/bin/couchdb_*
RUN chown ${USER}:${USER} ${COUCHDB_DIR}/bin/couchdb_*

ENV COOKIE=""
# INIT_DB single | cluster | none
ENV INIT_DB=single
ENV NODE_NAME=""
ENV CLUSTER_NODE=""
ENV CLUSTER_PORT=5984
ENV BIND_ADDRESS=0.0.0.0
ENV PORT=5984
ENV ADMIN=admin
ENV PASSWORD=""

USER ${USER}
WORKDIR ${COUCHDB_DIR}
EXPOSE 5984 4369 9100
VOLUME ["${COUCHDB_DIR}/data", "${COUCHDB_DIR}/etc"]
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["bin/couchdb_start"]

