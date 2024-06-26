# Reference: https://github.com/FRRouting/frr/blob/master/docker/ubuntu20-ci/Dockerfile
FROM ubuntu:20.04

ARG DEBIAN_FRONTEND=noninteractive
ENV APT_KEY_DONT_WARN_ON_DANGEROUS_USAGE=DontWarn
# Update Ubuntu Software repository
RUN apt update && \
    apt-get install -y \
      git autoconf automake libtool make libreadline-dev texinfo \
      pkg-config libpam0g-dev libjson-c-dev bison flex python3-pip \
      libc-ares-dev python3-dev python3-sphinx \
      install-info build-essential libsnmp-dev perl \
      libcap-dev python2 libelf-dev libprotobuf-c-dev protobuf-c-compiler \
      sudo gdb curl iputils-ping time vim pppoe pppoeconf ethtool iptables \
      lua5.3 liblua5.3-dev iperf3 openvswitch-switch openvswitch-common \
      net-tools iproute2 wget traceroute termshark && \
      curl https://bootstrap.pypa.io/pip/2.7/get-pip.py --output /tmp/get-pip.py && \
      python2 /tmp/get-pip.py && \
      rm -f  /tmp/get-pip.py && \
      python3 -m pip install wheel && \
      python3 -m pip install pytest && \
      python3 -m pip install pytest-xdist && \
      python3 -m pip install "scapy>=2.4.2" && \
      python3 -m pip install xmltodict && \
      python2 -m pip install 'exabgp<4.0.0'

RUN apt update && apt -y install tcpdump \
 && mv /usr/sbin/tcpdump /usr/bin/tcpdump

RUN groupadd -r -g 92 frr && \
      groupadd -r -g 85 frrvty && \
      adduser --system --ingroup frr --home /home/frr \
              --gecos "FRR suite" --shell /bin/bash frr && \
      usermod -a -G frrvty frr && \
      useradd -d /var/run/exabgp/ -s /bin/false exabgp && \
      echo 'frr ALL = NOPASSWD: ALL' | tee /etc/sudoers.d/frr && \
      mkdir -p /home/frr && chown frr.frr /home/frr

#for libyang 2
RUN apt-get install -y cmake libpcre2-dev

WORKDIR /temp
RUN wget https://dl.google.com/go/go1.19.3.linux-amd64.tar.gz \
 && tar -C /usr/local -xzf go1.19.3.linux-amd64.tar.gz \
 && echo "export PATH=$PATH:/usr/local/go/bin" >> ~/.bashrc

# Reference: https://github.com/slankdev/Dockerfiles/tree/master/ubuntu/18.04
# Scripts to enable/disable seg6 router
ADD ./enable_seg6_router.py /usr/bin/enable_seg6_router.py
ADD ./disable_seg6_router.py /usr/bin/disable_seg6_router.py
RUN chmod +x /usr/bin/enable_seg6_router.py
RUN chmod +x /usr/bin/disable_seg6_router.py

USER frr:frr

# Build and install libyang2
# libyang2 >= 2.1.128
RUN cd && pwd && ls -al && \
    git clone https://github.com/CESNET/libyang.git && \
    cd libyang && \
    git checkout v2.1.128 && \
    mkdir build && cd build && \
    cmake -DCMAKE_INSTALL_PREFIX:PATH=/usr \
          -DCMAKE_BUILD_TYPE:String="Release" .. && \
    make -j $(nproc) && \
    sudo make install

RUN git clone -b yykzm-draft-spring-srv6-mpls-interworking-service-iw https://github.com/y-kzm/frr.git /home/frr/frr && chown frr:frr /home/frr/frr

RUN cd && ls -al && ls -al frr

RUN cd ~/frr && \
    ./bootstrap.sh && \
    ./configure \
       --prefix=/usr \
       --localstatedir=/var/run/frr \
       --sbindir=/usr/lib/frr \
       --sysconfdir=/etc/frr \
       --enable-vtysh \
       --enable-pimd \
       --enable-sharpd \
       --enable-multipath=64 \
       --enable-user=frr \
       --enable-group=frr \
       --enable-vty-group=frrvty \
       --enable-snmp=agentx \
       --enable-scripting \
       --with-pkg-extra-version=-my-manual-build && \
    make -j $(nproc) && \
    sudo make install

RUN cd ~/frr && \
    sudo install -m 775 -o frr -g frr -d /var/log/frr && \
    sudo install -m 775 -o frr -g frrvty -d /etc/frr && \
    sudo install -m 640 -o frr -g frrvty tools/etc/frr/vtysh.conf /etc/frr/vtysh.conf && \
    sudo install -m 640 -o frr -g frr tools/etc/frr/frr.conf /etc/frr/frr.conf && \
    sudo install -m 640 -o frr -g frr tools/etc/frr/daemons.conf /etc/frr/daemons.conf && \
    sudo install -m 640 -o frr -g frr tools/etc/frr/daemons /etc/frr/daemons 

RUN cd ~/frr && make check || true

COPY ./docker-start /usr/sbin/docker-start

USER root
WORKDIR /root

CMD ["/usr/sbin/docker-start"]
