FROM debian
MAINTAINER He Bai <bai.he@outlook.com>

ENV ARCH amd64
ENV DIST wheezy
ENV MIRROR http://ftp.nl.debian.org

ENV DNET_MIN 192.168.1.101
ENV DNET_MAX 192.168.1.200
ENV DNET_MASK 255.255.255.0
ENV SERVER_IP 192.168.1.201

RUN apt-get -q update
RUN apt-get -qy install dnsmasq wget iptables
RUN wget --no-check-certificate https://raw.github.com/jpetazzo/pipework/master/pipework
RUN chmod +x pipework
RUN mkdir /tftp
WORKDIR /tftp

RUN wget $MIRROR/debian/dists/$DIST/main/installer-$ARCH/current/images/netboot/debian-installer/$ARCH/linux
RUN wget $MIRROR/debian/dists/$DIST/main/installer-$ARCH/current/images/netboot/debian-installer/$ARCH/initrd.gz
RUN wget $MIRROR/debian/dists/$DIST/main/installer-$ARCH/current/images/netboot/debian-installer/$ARCH/pxelinux.0
RUN mkdir pxelinux.cfg
RUN printf "DEFAULT linux\nKERNEL linux\nAPPEND initrd=initrd.gz\n" >pxelinux.cfg/default

CMD \
    echo Setting up iptables... &&\
    iptables -t nat -A POSTROUTING -j MASQUERADE &&\
    echo Waiting for pipework to give us the eth1 interface... &&\
    /pipework --wait &&\
    echo Starting DHCP+TFTP server...&&\
    dnsmasq --interface=eth1 \
    	    --dhcp-range=$DNET_MIN,$DNET_MAX,$DNET_MASK,1h \
	    --dhcp-boot=pxelinux.0,pxeserver,$SERVER_IP \
	    --pxe-service=x86PC,"Install Linux",pxelinux \
	    --enable-tftp --tftp-root=/tftp/ --no-daemon

