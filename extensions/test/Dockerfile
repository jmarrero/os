FROM registry.ci.openshift.org/rhcos-devel/rhel-coreos:4.11
#We want to test it's only using the extensions repo
RUN rm -rf /etc/yum.repos.d/*
ADD ext.repo /etc/yum.repos.d/extention.repo
#Install usbguard as provided by the repo
RUN rpm-ostree install usbguard
RUN usbguard
