FROM debian:stretch-slim
ENV DEBIAN_FRONTEND=noninteractive
#
# install neko dependencies
RUN set -eux; apt-get update; \
    apt-get install -y --no-install-recommends git pulseaudio dbus-x11 xserver-xorg-video-dummy gnupg curl ; \
    apt-get install --install-recommends -y dirmngr ; \
    apt-get install -y --no-install-recommends libcairo2 libxcb1 cmake libxrandr2 libxv1 libopus0 libvpx4; \
    apt-get -y --no-install-recommends install libgstreamer1.0-0 gstreamer1.0-dev gstreamer1.0-tools gstreamer1.0-doc; \
    apt-get -y --no-install-recommends install libgstreamer-plugins-base1.0-dev  libgstreamer-plugins-bad1.0-dev gstreamer1.0-plugins-base gstreamer1.0-plugins-bad gstreamer1.0-plugins-good; \
    apt-get -y --no-install-recommends install libxtst-dev libxrandr-dev libx11-dev gstreamer1.0-doc gstreamer1.0-tools gstreamer1.0-x gstreamer1.0-alsa gstreamer1.0-pulseaudio;


RUN apt-get update \
 && apt-get install -y --no-install-recommends ca-certificates

RUN update-ca-certificates
#
# add gst to env
#ENV PATH=/gst/local/bin:$PATH
#ENV LD_LIBRARY_PATH=/gst/local/lib:$LD_LIBRARY_PATH
#ENV PKG_CONFIG_PATH=/gst/local/lib/pkgconfig:$PKG_CONFIG_PATH

#
# copy gst
#COPY .build/gst/local /gst/local/
COPY .docker/files/deps/dbus /usr/bin/dbus
COPY .docker/files/deps/default.pa /etc/pulse/default.pa


ENV NODE_VERSION 12.16.2
RUN ARCH= && dpkgArch="$(dpkg --print-architecture)" \
    && case "${dpkgArch##*-}" in \
      amd64) ARCH='x64';; \
      ppc64el) ARCH='ppc64le';; \
      s390x) ARCH='s390x';; \
      arm64) ARCH='arm64';; \
      armhf) ARCH='armv7l';; \
      i386) ARCH='x86';; \
      *) echo "unsupported architecture"; exit 1 ;; \
    esac \
    && set -ex \
    && apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates wget \
    && curl -sL https://deb.nodesource.com/setup_12.x | bash - \
    && apt -y install nodejs \
    && apt -y  install gcc g++ make \
    && apt-mark auto '.*' > /dev/null \
    && find /usr/local -type f -executable -exec ldd '{}' ';' \
      | awk '/=>/ { print $(NF-1) }' \
      | sort -u \
      | xargs -r dpkg-query --search \
      | cut -d: -f1 \
      | sort -u \
      | xargs -r apt-mark manual \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs \
    # smoke tests
    && node --version \
    && npm --version

#
# add golang
ENV GOLANG_VERSION 1.14.2
RUN set -eux; \
	dpkgArch="$(dpkg --print-architecture)"; \
	case "${dpkgArch##*-}" in \
		amd64) goRelArch='linux-amd64'; goRelSha256='6272d6e940ecb71ea5636ddb5fab3933e087c1356173c61f4a803895e947ebb3' ;; \
		armhf) goRelArch='linux-armv6l'; goRelSha256='eb4550ba741506c2a4057ea4d3a5ad7ed5a887de67c7232f1e4795464361c83c' ;; \
		arm64) goRelArch='linux-arm64'; goRelSha256='bb6d22fe5806352c3d0826676654e09b6e41eb1af52e8d506d3fa85adf7f8d88' ;; \
		i386) goRelArch='linux-386'; goRelSha256='cab5f51e6ffb616c6ee963c3d0650ca4e3c4108307c44f2baf233fcb8ff098f6' ;; \
		ppc64el) goRelArch='linux-ppc64le'; goRelSha256='48c22268c81ced9084a43bbe2c1596d3e636b5560b30a32434a7f15e561de160' ;; \
		s390x) goRelArch='linux-s390x'; goRelSha256='501cc919648c9d85b901963303c5061ea6814c80f0d35fda9e62980d3ff58cf4' ;; \
		*) goRelArch='src'; goRelSha256='98de84e69726a66da7b4e58eac41b99cbe274d7e8906eeb8a5b7eb0aadee7f7c'; \
			echo >&2; echo >&2 "warning: current architecture ($dpkgArch) does not have a corresponding Go binary release; will be building from source"; echo >&2 ;; \
	esac; \
	url="https://golang.org/dl/go${GOLANG_VERSION}.${goRelArch}.tar.gz"; \
	wget -O go.tgz "$url"; \
	echo "${goRelSha256} *go.tgz" | sha256sum -c -; \
	tar -C /usr/local -xzf go.tgz; \
	rm go.tgz; \
	if [ "$goRelArch" = 'src' ]; then \
		echo >&2; \
		echo >&2 'error: UNIMPLEMENTED'; \
		echo >&2 'TODO install golang-any from jessie-backports for GOROOT_BOOTSTRAP (and uninstall after build)'; \
		echo >&2; \
		exit 1; \
	fi; \
	export PATH="/usr/local/go/bin:$PATH"; \
	go version

ENV GOPATH /go
ENV PATH $GOPATH/bin:/usr/local/go/bin:$PATH
RUN mkdir -p "$GOPATH/src" "$GOPATH/bin" && chmod -R 777 "$GOPATH"

#
# install Go tools w/module support
RUN mkdir -p /tmp/gotools \
    && cd /tmp/gotools \
    && GOPATH=/tmp/gotools GO111MODULE=on go get -v golang.org/x/tools/gopls@latest 2>&1 \
    && GOPATH=/tmp/gotools GO111MODULE=on go get -v \
        honnef.co/go/tools/...@latest \
        golang.org/x/tools/cmd/gorename@latest \
        golang.org/x/tools/cmd/goimports@latest \
        golang.org/x/tools/cmd/guru@latest \
        golang.org/x/lint/golint@latest \
        github.com/mdempsky/gocode@latest \
        github.com/cweill/gotests/...@latest \
        github.com/haya14busa/goplay/cmd/goplay@latest \
        github.com/sqs/goreturns@latest \
        github.com/josharian/impl@latest \
        github.com/davidrjenni/reftools/cmd/fillstruct@latest \
        github.com/uudashr/gopkgs/v2/cmd/gopkgs@latest  \
        github.com/ramya-rao-a/go-outline@latest  \
        github.com/acroca/go-symbols@latest  \
        github.com/godoctor/godoctor@latest  \
        github.com/rogpeppe/godef@latest  \
        github.com/zmb3/gogetdoc@latest \
        github.com/fatih/gomodifytags@latest  \
        github.com/mgechev/revive@latest  \
        github.com/go-delve/delve/cmd/dlv@latest 2>&1 \
    #
    # build Go tools w/o module support
    && GOPATH=/tmp/gotools go get -v github.com/alecthomas/gometalinter 2>&1 \
    #
    # build gocode-gomod
    && GOPATH=/tmp/gotools go get -x -d github.com/stamblerre/gocode 2>&1 \
    && GOPATH=/tmp/gotools go build -o gocode-gomod github.com/stamblerre/gocode \
    #
    # install Go tools
    && mv /tmp/gotools/bin/* /usr/local/bin/ \
    && mv gocode-gomod /usr/local/bin/ \
    #
    # install golangci-lint
    && curl -sSfL https://raw.githubusercontent.com/golangci/golangci-lint/master/install.sh | sh -s -- -b /usr/local/bin 2>&1 \
    #
    # setup paths
    && mkdir -p "$GOPATH/src" "$GOPATH/bin" "$GOPATH/pkg/mod" \
    && chmod -R 777 "$GOPATH"

#
# turn on go modules
ENV GO111MODULE=on


