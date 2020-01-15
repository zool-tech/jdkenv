#OPENJDK-8
FROM openjdk:8-slim

# File Author / Maintainer
MAINTAINER yinxb@mochasoft.com.cn

RUN set -eux;\
	apt-get update && apt-get install -y --no-install-recommends busybox \
	&& rm -rf /var/lib/apt/lists/*;\
	for cmdStr in free ip less nc netstat nslookup ping top tracerout vi watch wget;do ln -s /bin/busybox /bin/${cmdStr};done;\
	#shell配置
	{\
		echo "alias ls='ls --color=auto'";\
		echo "alias ll='ls -l'";\
	} >> /root/.bashrc
