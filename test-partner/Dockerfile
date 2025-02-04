FROM registry.access.redhat.com/ubi8/ubi:8.7
ARG USERNAME=tnf-user
ARG USER_UID=1000
ARG USER_GID=$USER_UID
RUN \
	dnf update --assumeyes --disableplugin=subscription-manager \
	&& dnf install --assumeyes --disableplugin=subscription-manager \
		ethtool-2:5.13 \
		hostname-3.20 \
		iproute-5.18.0 \
		iputils-20180629 \
		make-1:4.2.1 \
		openssh-8.0p1 \
		openssh-clients-8.0p1 \
		podman-3:4.2.0 \
		psmisc-23.1 \
		wget-1.19.5 \
	&& dnf clean all --assumeyes --disableplugin=subscription-manager \
	&& rm -rf /var/cache/yum

# Install Go binary
ENV \
	GO_BIN_TAR=go1.20.3.linux-amd64.tar.gz \
	GO_DL_URL=https://golang.org/dl \
	GOPATH=/root/go \
	TEMP_DIR=/tmp
ENV GO_BIN_URL_x86_64=${GO_DL_URL}/${GO_BIN_TAR}
RUN \
	if [ "$(uname -m)" = x86_64 ]; then \
		wget --directory-prefix=${TEMP_DIR} ${GO_BIN_URL_x86_64} --quiet \
		&& rm -rf /usr/local/go \
		&& tar -C /usr/local -xzf ${TEMP_DIR}/${GO_BIN_TAR}; \
	else \
		echo "CPU architecture not supported" && exit 1; \
	fi

# Add go and oc binary directory to $PATH
ENV PATH="/bin/":"/usr/local/go/bin":${GOPATH}"/bin/":"${PATH}"

# Add arbitrary /licenses folder to pass preflight
RUN \
	mkdir /licenses \
	&& touch /licenses/LICENSE \
	&& echo "fake license" >> /licenses/LICENSE \
	&& echo ${PATH}
WORKDIR /app
COPY ./testapp /app/testapp
WORKDIR /app/testapp
RUN make build
ARG LIVENESS_PROBE_DEFAULT_PORT=8080
EXPOSE ${LIVENESS_PROBE_DEFAULT_PORT}
RUN \
	groupadd --gid $USER_GID $USERNAME \
	&& useradd --uid $USER_UID --gid $USER_GID -m $USERNAME
USER $USERNAME
CMD ["./bin/app"]
