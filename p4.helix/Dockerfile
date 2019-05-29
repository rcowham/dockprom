FROM centos:centos7
MAINTAINER Robert Cowham "rcowham@perforce.com"

RUN yum update -y; \
    yum install -y net-tools; \
    yum install -y perl; \
    yum install -y sudo; \
    yum install -y wget; \
    echo /usr/local/lib>> /etc/ld.so.conf; \
    echo /usr/lib64>> /etc/ld.so.conf; \
    sed -ie "s/^Defaults[ \t]*requiretty/#Defaults  requiretty/g" /etc/sudoers

# Create perforce user with UID to 1000 before p4d installation
RUN useradd --home-dir /p4 --create-home --uid 1000 perforce
RUN echo perforce:perforce | /usr/sbin/chpasswd
RUN cd /usr/local/bin && wget http://ftp.perforce.com/perforce/r18.2/bin.linux26x86_64/p4 && \
    chmod +x /usr/local/bin/p4

RUN echo 'perforce ALL=(ALL) NOPASSWD:ALL'> /tmp/perforce; \
    chmod 0440 /tmp/perforce; \
    chown root:root /tmp/perforce; \
    mv /tmp/perforce /etc/sudoers.d

RUN mkdir /hxdepots /hxmetadata /hxlogs; \
    chown -R perforce:perforce /hx*; \
    mkdir -p /hxdepots/reset; \
    cd /hxdepots/reset; \
    curl -k -s -O https://swarm.workshop.perforce.com/downloads/guest/perforce_software/helix-installer/main/src/reset_sdp.sh; \
    chmod +x reset_sdp.sh; \
    ./reset_sdp.sh -fast -no_ssl -t default


EXPOSE 3903
EXPOSE 1999

USER perforce

RUN mkdir /p4/mtail/
COPY mtail/mtail /p4/mtail/
COPY mtail/p4d_log.mtail /p4/mtail/

ENTRYPOINT \
	/p4/1/bin/p4d_1_init start && \
    /p4/mtail/mtail --logs /p4/1/logs/log --progs /p4/mtail/ 
