FROM debian
MAINTAINER He Bai <bai.he@outlook.com>

ENV ARCH amd64
ENV DIST wheezy
ENV MIRROR http://ftp.nl.debian.org
ENV DEFAULT stable
ENV CLOUD_CONFIG_URL http://localhost/cloud-config.yml

RUN apt-get -q update
RUN apt-get -qy install dnsmasq wget iptables curl
RUN wget --no-check-certificate https://raw.github.com/jpetazzo/pipework/master/pipework
RUN chmod +x pipework
RUN mkdir -p /tftp/stable
RUN mkdir -p /tftp/beta
RUN mkdir -p /tftp/alpha

WORKDIR /tftp

ADD http://stable.release.core-os.net/amd64-usr/current/coreos_production_pxe.vmlinuz /tftp/stable/
ADD http://stable.release.core-os.net/amd64-usr/current/coreos_production_pxe_image.cpio.gz /tftp/stable/

ADD http://beta.release.core-os.net/amd64-usr/current/coreos_production_pxe.vmlinuz /tftp/beta/
ADD http://beta.release.core-os.net/amd64-usr/current/coreos_production_pxe_image.cpio.gz /tftp/beta/

ADD http://alpha.release.core-os.net/amd64-usr/current/coreos_production_pxe.vmlinuz /tftp/alpha/
ADD http://alpha.release.core-os.net/amd64-usr/current/coreos_production_pxe_image.cpio.gz /tftp/alpha/

RUN mkdir pxelinux.cfg
ADD boot.msg /tftp/pxelinux.cfg/boot.msg

RUN mkdir /www
ADD cloud-config.yml /www/cloud-config.yml

RUN curl https://npmjs.org/install.sh | sh
RUN npm install http-server -g

CMD \
    if [ -z "$DHCP_RANGE" ]; then echo "DHCP_RANGE is not set!"; exit 1; fi &&\
    if [ -z "$BOOT_SERVER" ]; then echo "BOOT_SERVER is not set!"; exit 1; fi &&\
    http-server /www > /dev/null & \
    echo "default ${DEFAULT}" > pxelinux.cfg/default && \
    printf "prompt 1\ntimeout 15\n\ndisplay boot.msg\n" >> pxelinux.cfg/default && \
    LABEL=stable && \
    printf "\nlabel ${LABEL}\n\tkernel ${LABEL}/coreos_production_pxe.vmlinuz\nappend initrd=${LABEL}/coreos_production_pxe_image.cpio.gz cloud-config-url=${CLOUD_CONFIG_URL}" >> pxelinux.cfg/default && \
    LABEL=beta && \
    printf "\nlabel ${LABEL}\n\tkernel ${LABEL}/coreos_production_pxe.vmlinuz\nappend initrd=${LABEL}/coreos_production_pxe_image.cpio.gz cloud-config-url=${CLOUD_CONFIG_URL}" >> pxelinux.cfg/default && \
    LABEL=alpha && \
    printf "\nlabel ${LABEL}\n\tkernel ${LABEL}/coreos_production_pxe.vmlinuz\nappend initrd=${LABEL}/coreos_production_pxe_image.cpio.gz cloud-config-url=${CLOUD_CONFIG_URL}" >> pxelinux.cfg/default && \
    echo Setting up iptables... &&\
    iptables -t nat -A POSTROUTING -j MASQUERADE &&\
    echo Waiting for pipework to give us the eth1 interface... &&\
    /pipework --wait &&\
    echo Starting DHCP+TFTP server...&&\
    dnsmasq --interface=eth1 \
    	    --dhcp-range=$DHCP_RANGE,1h \
	    --dhcp-boot=pxelinux.0,pxeserver,$BOOT_SERVER \
	    --pxe-service=x86PC,"Install Linux",pxelinux \
	    --enable-tftp --tftp-root=/tftp/ --no-daemon

