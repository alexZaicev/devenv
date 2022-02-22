FROM ubuntu:latest

LABEL maintainer="alex.zaicef@gmail.com"

# Basic env variables
ENV DEBIAN_FRONTEND noninteractive
ENV DEBIAN_NONINTERACTIVE_SEEN=true
ENV container docker

# INSTALL ESSENTIAL COMPONENTS
RUN apt-get update && apt-get install -y --no-install-recommends \
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
	libxmlsec1-dev

# SET LOCALE
ENV LANG en_US.UTF-8
RUN locale-gen en_US.UTF-8 && update-locale LANG=en_US.UTF-8

# INSTALL DEV TOOLS
RUN apt-get update && apt-get install -y --no-install-recommends \
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
	kafkacat

# CREATE DEV USER
RUN useradd --create-home -s /bin/bash dev && \
	printf "\n[[ \"\$PWD\" == \"/\" ]] && cd\n" >> /home/dev/.bashrc && \
	ln -s /home/dev/.profile /home/dev/.bash_profile && \
	echo -n 'dev:dev' | chpasswd && \
	echo 'dev ALL=NOPASSWD: ALL' > /etc/sudoers.d/dev && \
	mkdir -p /home/dev/.ssh && \
	chown -R dev:dev /home/dev/.ssh

COPY files/vimrc /home/dev/.vimrc

# INSTALL SSH SERVER
RUN apt-get update && apt-get install -y --no-install-recommends \
	openssh-server && \
	mkdir -p /var/run/sshd && \
	chmod 0755 /var/run/sshd && \
	echo "StrictHostKeyChecking no" >> /etc/ssh/ssh_config && \
	sed -i 's/AcceptEnv/# AcceptEnv/' /etc/ssh/sshd_config
	
# INSTALL GO AND TOOLING
ENV GOLANG_VERSION 1.18rc1
RUN cd /tmp && \
	wget https://go.dev/dl/go${GOLANG_VERSION}.linux-amd64.tar.gz && \
	tar -xzvf go${GOLANG_VERSION}.linux-amd64.tar.gz -C /usr/local && \
	rm -rf go${GOLANG_VERSION}.linux-amd64.tar.gz

ENV ROOT_GOROOT /usr/local/go
ENV ROOT_GOBIN /usr/local/go/bin
ENV ROOT_GOPRIVATE github.com/alexZaicev
ENV PATH "PATH=$PATH:$ROOT_GOBIN"

RUN go env -w GOBIN=${ROOT_GOBIN} GOROOT=${ROOT_GOROOT} GOPRIVATE=${ROOT_GOPRIVATE}
RUN su - dev -c "/usr/local/go/bin/go env -w GOPRIVATE=${ROOT_GOPRIVATE}"
RUN echo "export GOROOT=${ROOT_GOROOT}" > /etc/profile.d/go.sh && \
	echo 'export PATH=$PATH:'$ROOT_GOBIN:/home/dev/go/bin >> /etc/profile.d/go.sh

RUN curl -aSfl https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b $(go env GOROOT)/bin v1.44.2
COPY files/golangci.yml /home/dev/.golangci.yml

RUN go install github.com/vektra/mockery/v2@latest
RUN go install github.com/google/wire/cmd/wire@latest
RUN go install github.com/jfeliu007/goplantuml/cmd/goplantuml@latest
RUN go install golang.org/x/tools/cmd/goimports@latest
RUN go install github.com/daixiang0/gci@latest

# INSTALL TRIVY
ENV TRIVY_VERSION 0.23.0
RUN wget -nv https://github.com/aquasecurity/trivy/releases/download/v${TRIVY_VERSION}/trivy_${TRIVY_VERSION}_Linux-64bit.deb
RUN apt-get update && apt install ./trivy_${TRIVY_VERSION}_Linux-64bit.deb	

# SET ENV TYPE
ENV ENV_TYPE dev-env
RUN echo "export ENV_TYPE=${ENV_TYPE}" >> /etc/profile.d/env.sh

RUN echo "/home/dev/scripts/add_key" >> /home/dev/.profile
RUN echo "/home/dev/scripts/add_gitconfig" >> /home/dev/.profile
      
CMD ["sbin/init"]
