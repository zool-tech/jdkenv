FROM debian:buster-slim

MAINTAINER zooltech@qq.com

ENV LANG C.UTF-8
ENV JAVA_HOME /usr/local/openjdk-8
ENV PATH $JAVA_HOME/bin:$PATH

# java install params
ARG JAVA_URL_VERSION=8u232b09
ARG JAVA_BASE_URL=https://github.com/AdoptOpenJDK/openjdk8-upstream-binaries/releases/download/jdk8u232-b09/OpenJDK8U-jdk_

RUN set -eux;\
	dpkgArch="$(dpkg --print-architecture)";\
	case "$dpkgArch" in \
		amd64) upstreamArch='x64' ;;\
		arm64) upstreamArch='aarch64' ;;\
		*) echo >&2 "error: unsupported architecture: $dpkgArch" ;;\
	esac;\
	apt-get update && apt-get install -y --no-install-recommends busybox ca-certificates p11-kit && rm -rf /var/lib/apt/lists/*;\
# busybox soft link
	for cmdStr in free ip less nc netstat nslookup ping ps top tracerout vi watch wget;do ln -s busybox /bin/${cmdStr};done;\
# download openjdk
	wget -O openjdk.tgz "${JAVA_BASE_URL}${upstreamArch}_linux_${JAVA_URL_VERSION}.tar.gz" -q;\
	mkdir -p "$JAVA_HOME";\
	tar --extract --file openjdk.tgz --directory "$JAVA_HOME" --strip-components 1 --no-same-owner;\
	rm openjdk.tgz* && rm "$JAVA_HOME/src.zip" && rm -rf "$JAVA_HOME/demo" && rm -rf "$JAVA_HOME/sample";\
# update "cacerts" bundle to use Debian's CA certificate
	mkdir -p /etc/ca-certificates/update.d;\
	{\
		echo '#!/usr/bin/env bash';\
		echo 'set -Eeuo pipefail';\
		echo 'if ! [ -d "$JAVA_HOME" ]; then echo >&2 "error: missing JAVA_HOME environment variable"; exit 1; fi';\
# 8-jdk uses "$JAVA_HOME/jre/lib/security/cacerts" and 8-jre and 11+ uses "$JAVA_HOME/lib/security/cacerts" directly (no "jre" directory)
		echo 'cacertsFile=; for f in "$JAVA_HOME/lib/security/cacerts" "$JAVA_HOME/jre/lib/security/cacerts"; do if [ -e "$f" ]; then cacertsFile="$f"; break; fi; done';\
		echo 'if [ -z "$cacertsFile" ] || ! [ -f "$cacertsFile" ]; then echo >&2 "error: failed to find cacerts file in $JAVA_HOME"; exit 1; fi';\
		echo 'trust extract --overwrite --format=java-cacerts --filter=ca-anchors --purpose=server-auth "$cacertsFile"';\
	} > /etc/ca-certificates/update.d/docker-openjdk;\
	chmod +x /etc/ca-certificates/update.d/docker-openjdk;\
	/etc/ca-certificates/update.d/docker-openjdk;\
# https://github.com/docker-library/openjdk/issues/331#issuecomment-498834472
	find "$JAVA_HOME/lib" -name '*.so' -exec dirname '{}' ';' | sort -u > /etc/ld.so.conf.d/docker-openjdk.conf;\
	ldconfig;\
#shell config
	echo "alias ls='ls --color=auto'\nalias ll='ls -lA'" > /root/.bashrc;\
# basic smoke test
	java -version
