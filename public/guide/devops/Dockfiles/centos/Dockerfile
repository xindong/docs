FROM centos:latest
MAINTAINER Tomasen "https://github.com/tomasen"

RUN yum -y update
RUN yum -y install epel-release
RUN yum -y install systemd
RUN yum -y install openssh-server
RUN yum -y install sudo

RUN yum clean all

RUN sed -ri 's/^PermitRootLogin\s+.*/PermitRootLogin no/' /etc/ssh/sshd_config
RUN sed -ri 's/^PasswordAuthentication\s+.*/PasswordAuthentication no/' /etc/ssh/sshd_config
RUN ssh-keygen -t rsa -f /etc/ssh/ssh_host_rsa_key -N ''

RUN useradd -ms /bin/bash -d /home/centos centos

RUN systemctl enable sshd.service

EXPOSE 22

CMD [“/usr/sbin/init”]
