FROM quay.io/centos/centos:stream8 as rpm-ostree
RUN sed -i -e 's,enabled=0,enabled=1,' /etc/yum.repos.d/CentOS-Stream-PowerTools.repo

RUN yum -y group install "Development Tools"
RUN yum update -y

#Clone rpm-ostree
RUN git clone https://github.com/jmarrero/rpm-ostree.git  
WORKDIR rpm-ostree
RUN git checkout rhel8-revert-method
RUN ./ci/installdeps.sh
RUN PATH=$PATH:/rpm-ostree/target/cxxbridge/bin/
RUN git submodule update --init
RUN env CFLAGS='-ggdb -Og' CXXFLAGS='-ggdb -Og' ./autogen.sh --prefix=/usr --libdir=/usr/lib64 --sysconfdir=/etc
RUN make
RUN pwd

## Downloads the extensions given the extensions.yaml
ARG RHCOS_VERSION=latest
FROM registry.ci.openshift.org/rhcos-devel/rhel-coreos:latest as os

COPY --from=rpm-ostree /rpm-ostree/target/debug/rpm-ostree /usr/bin/rpm-ostree

#Expects repos from redhat-coreos, like when building rhcos locally.
RUN git clone  https://github.com/openshift/os.git
WORKDIR os
RUN git clone https://github.com/coreos/fedora-coreos-config.git fedora-coreos-config

RUN git -c http.sslVerify=false clone https://gitlab.cee.redhat.com/coreos/redhat-coreos.git 
RUN cp redhat-coreos/*.repo .
RUN cp redhat-coreos/content_sets.yaml .
RUN rpm-ostree compose extensions --rootfs=/ --output-dir=/usr/share/rpm-ostree/extensions/ {manifest,extensions}.yaml

## Creates the repo metadata for the extensions
FROM quay.io/centos/centos:stream8 as builder
COPY --from=os /usr/share/rpm-ostree/extensions/ /usr/share/rpm-ostree/extensions/
RUN dnf install -y createrepo_c golang
ADD repo-server/main.go .
RUN mkdir /build
RUN go build -o /build/webserver main.go

RUN createrepo_c /usr/share/rpm-ostree/extensions/

## Final container that has the extensions and webserver
FROM registry.access.redhat.com/ubi8/ubi:latest
COPY --from=builder /build/webserver /usr/bin/webserver
COPY --from=builder /usr/share/rpm-ostree/extensions/ /usr/share/rpm-ostree/extensions/

CMD ["./usr/bin/webserver"]
EXPOSE 9666/tcp
