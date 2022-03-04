FROM ubuntu:latest

LABEL maintainer="alex.zaicef@gmail.com"

# Basic env variables
ENV DEBIAN_FRONTEND noninteractive
ENV DEBIAN_NONINTERACTIVE_SEEN=true
ENV container docker

# INSTALL ESSENTIAL COMPONENTS
RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		build-essential \
		software-properties-common \
		gpg-agent \
		init \
		systemd-container \
		net-tools \
		sudo \
		curl \
		ca-certificates \
		locales \
		openssh-client \
		iputils-ping \
		apt-transport-https \
		gnupg-agent \
		inotify-tools \
		libpq-dev \
		libxmlsec1-dev;

# SET LOCALE
ENV LANG en_US.UTF-8
RUN set -eux; \
	locale-gen en_US.UTF-8; \
	update-locale LANG=en_US.UTF-8;

# INSTALL DEV TOOLS
RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends \
		vim \
		zip \
		tree \
		groff \
		bash-completion \
		less \
		git \
		jq \
		netcat \
		sed \
		wget \
		bsdmainutils \
		unzip \
		plantuml \
		graphviz \
		kafkacat;

# CREATE DEV USER
RUN set -eux; \
	useradd --create-home -s /bin/bash dev; \
	printf "\n[[ \"\$PWD\" == \"/\" ]] && cd\n" >> /home/dev/.bashrc; \
	ln -s /home/dev/.profile /home/dev/.bash_profile; \
	echo -n 'dev:dev' | chpasswd; \
	echo 'dev ALL=NOPASSWD: ALL' > /etc/sudoers.d/dev; \
	mkdir -p /home/dev/.ssh; \
	chown -R dev:dev /home/dev/.ssh;

COPY files/vimrc /home/dev/.vimrc

# INSTALL SSH SERVER
RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends openssh-server; \
	mkdir -p /var/run/sshd; \
	chmod 0755 /var/run/sshd; \
	echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config; \
	sed -i 's/AcceptEnv/# AcceptEnv/' /etc/ssh/sshd_config;
	
# INSTALL PSQL (PostgreSQL client)
ENV POSTGRES_SERVER pg-dev
ENV POSTGRES_USER postgres
ENV POSTGRES_DB dev
RUN set -eux; \
	echo "deb http://apt/postgresql.org/pub/repos/apt/ $(lsb_release -cs)-pgdg main" > /etc/apt/sources.list.d/postgres.list; \
	apt-get update; \
	apt-get install -y --no-install-recommends postgresql-client-12;
RUN echo "export POSTGRES_SERVER=${POSTGRES_SERVER}" >> /etc/profile.d/pg.sh
RUN echo "export POSTGRES_USER=${POSTGRES_USER}" >> /etc/profile.d/pg.sh
RUN echo "export POSTGRES_DB=${POSTGRES_DB}" >> /etc/profile.d/pg.sh

# INSTALL REDIS CLI
RUN set -eux; \
	apt-get update; \
	apt-get install -y --no-install-recommends redis-tools;

# INSTALL MONGO CLI
RUN set -eux; \
    apt-get update; \
    apt-get install -y --no-install-recommends mongocli;
	
# INSTALL GO AND TOOLING
ENV GOLANG_VERSION 1.16
RUN set -eux; \
	cd /tmp; \
	wget https://go.dev/dl/go${GOLANG_VERSION}.linux-amd64.tar.gz; \
	tar -xzvf go${GOLANG_VERSION}.linux-amd64.tar.gz -C /usr/local; \
	rm -rf go${GOLANG_VERSION}.linux-amd64.tar.gz;

ENV ROOT_GOROOT /usr/local/go
ENV ROOT_GOBIN /usr/local/go/bin
ENV ROOT_GOPRIVATE github.com/alexZaicev
ENV PATH "PATH=$PATH:$ROOT_GOBIN"

RUN go env -w GOBIN=${ROOT_GOBIN} GOROOT=${ROOT_GOROOT} GOPRIVATE=${ROOT_GOPRIVATE}
RUN su - dev -c "/usr/local/go/bin/go env -w GOPRIVATE=${ROOT_GOPRIVATE}"
RUN set -eux; \
	echo "export GOROOT=${ROOT_GOROOT}" > /etc/profile.d/go.sh; \
	echo 'export PATH=$PATH:'$ROOT_GOBIN:/home/dev/go/bin >> /etc/profile.d/go.sh;

RUN curl -aSfl https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOROOT)/bin v1.44.2
COPY files/golangci.yml /home/dev/.golangci.yml

ENV ROOT_MOCKERY_VERSION v2.10.0
ENV ROOT_WIRE_VERSION v0.5.0
ENV ROOT_GOPLANTUML_VERSION v1.5.2
ENV ROOT_GOIMPORTS_VERSION v0.1.9
ENV ROOT_GCI_VERSION v0.3.1

RUN go install github.com/vektra/mockery/v2@${ROOT_MOCKERY_VERSION}
RUN go install github.com/google/wire/cmd/wire@${ROOT_WIRE_VERSION}
RUN go install github.com/jfeliu007/goplantuml/cmd/goplantuml@${ROOT_GOPLANTUML_VERSION}
RUN go install golang.org/x/tools/cmd/goimports@${ROOT_GOIMPORTS_VERSION}
RUN go install github.com/daixiang0/gci@${ROOT_GCI_VERSION}

# INSTALL TRIVY
ENV TRIVY_VERSION 0.23.0
RUN wget -nv https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-64bit.deb
RUN set -eux; \
	apt-get update; \
	apt install ./trivy_${TRIVY_VERSION}_Linux-64bit.deb;

# SET ENV TYPE
ENV ENV_TYPE dev-env
RUN echo "export ENV_TYPE=${ENV_TYPE}" >> /etc/profile.d/env.sh

RUN echo "/home/dev/scripts/add_key" >> /home/dev/.profile
RUN echo "/home/dev/scripts/add_gitconfig" >> /home/dev/.profile
      
CMD ["sbin/init"]
