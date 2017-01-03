FROM debian:jessie-slim

# Based on https://github.com/klaemo/docker-couchdb/blob/master/2.0.0/Dockerfile
MAINTAINER Sergey Korobkov skorobkov@dinkor.net

ARG USER=couchdb
ARG DUMB_INIT_VERSION=1.2.0
ARG ERLANG_VERSION=18.3
ARG COUCHDB_REPO=https://github.com/apache/couchdb.git
ARG COUCHDB_BRANCH=master
ARG COUCHDB_DIR=/opt/couchdb

ENV PATH=/tmp/erlang/bin:${PATH}

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

        && curl -fSL -o /tmp/erlang-solutions_1.0_all.deb  https://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb \
        && dpkg -i /tmp/erlang-solutions_1.0_all.deb \
        && echo "Package: erlang*" > /etc/apt/preferences.d/erlang_version \
        && echo "Pin: version 1:${ERLANG_VERSION}*" >> /etc/apt/preferences.d/erlang_version \
        && echo "Pin-Priority: 1001" >> /etc/apt/preferences.d/erlang_version \
        && apt-get update \
        && apt-get install -y \
                erlang \

	&& npm install -g grunt-cli \
	&& git clone --depth 1 --single-branch -b ${COUCHDB_BRANCH} ${COUCHDB_REPO} /tmp/couchdb \
	&& cd /tmp/couchdb \
	# TODO: remove this
	&& sed -i -e 's/^\({fauxton,\s*"fauxton",\s*\){tag, "v1.1.9"}\(.*\)$/\1"2e770d66b2b849a60468ad390dfed57da54561a9"\2/' rebar.config.script \
	&& ./configure --disable-docs \
	&& make release \
	&& useradd --user-group --home-dir ${COUCHDB_DIR} ${USER} \
	&& mkdir ${COUCHDB_DIR} \
	&& mv rel/couchdb/* ${COUCHDB_DIR} \
	&& mkdir ${COUCHDB_DIR}/data ${COUCHDB_DIR}/etc/local.d ${COUCHDB_DIR}/etc/default.d \
	&& chown -R ${USER}:${USER} ${COUCHDB_DIR} \
	&& curl -fSL -o /tmp/dumb-init_${DUMB_INIT_VERSION}_amd64.deb https://github.com/Yelp/dumb-init/releases/download/v${DUMB_INIT_VERSION}/dumb-init_${DUMB_INIT_VERSION}_amd64.deb \
	&& dpkg -i /tmp/dumb-init_*_amd64.deb \
	&& npm uninstall -g grunt-cli \
	&& apt-get purge -y \
		apt-transport-https \
		build-essential \
		git-core \
		libicu-dev \
		libmozjs185-dev \
		libncurses5-dev \
		libssl-dev \
		curl \
		nodejs \
		erlang \
		erlang-solutions \
	&& apt-get clean \
	&& rm -f /etc/apt/apt.conf.d/build_docker_norecommends \
	&& rm -f /etc/apt/preferences.d/erlang_version \
	&& rm -rf /var/lib/apt/lists/* \
	&& rm -rf ~/.npm/ \
	&& rm -rf /tmp/*

USER ${USER}
WORKDIR ${COUCHDB_DIR}
EXPOSE 5984 4369 9100
VOLUME ["/opt/couchdb/data"]
ENTRYPOINT ["/usr/bin/dumb-init", "--"]
CMD ["/opt/couchdb/bin/couchdb"]

