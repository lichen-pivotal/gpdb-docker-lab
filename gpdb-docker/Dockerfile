#
#  Dockerfile for a GPDB SNE Sandbox Base Image
#

FROM centos:6.7
MAINTAINER lichen@pivotal.io

COPY * /tmp/
RUN echo root:changeme | chpasswd \
        && yum install -y unzip which tar more util-linux-ng passwd openssh-clients openssh-server perl ed m4; yum clean all \
        && echo $(grep $(hostname) /etc/hosts | cut -f1) mdw >> /etc/hosts \
        && unzip /tmp/greenplum-db*.zip -d /tmp/ \
        && rm /tmp/greenplum-db*.zip \
        && sed -i s/"more << EOF"/"cat << EOF"/g /tmp/greenplum-db*.bin \
        && echo -e "yes\n\nyes\nyes\n" | /tmp/greenplum-db*.bin \
        && rm /tmp/greenplum-db*.bin \
        && cat /tmp/sysctl.conf.add >> /etc/sysctl.conf \
        && cat /tmp/limits.conf.add >> /etc/security/limits.conf \
        && rm -f /tmp/*.add \
        && echo "mdw" > /tmp/gpdb-hosts \
        && chmod 777 /tmp/gpinitsystem_singlenode \
        && hostname > ~/orig_hostname \
        && mv /tmp/run.sh /usr/local/bin/run.sh \
        && chmod +x /usr/local/bin/run.sh \
        && /usr/sbin/groupadd gpadmin \
        && /usr/sbin/useradd gpadmin -g gpadmin -G wheel \
        && echo "changeme"|passwd --stdin gpadmin \
        && echo "gpadmin        ALL=(ALL)       NOPASSWD: ALL" >> /etc/sudoers \
        && mv /tmp/bash_profile /home/gpadmin/.bash_profile \
        && chown -R gpadmin: /home/gpadmin \
        && mkdir -p /data/master /data/primary /data/mirror \
        && chown -R gpadmin: /data \
        && chown -R gpadmin: /usr/local/green* \
        && service sshd start \
        && su gpadmin -l -c "source /usr/local/greenplum-db/greenplum_path.sh;gpssh-exkeys -f /tmp/gpdb-hosts"  \
        && su gpadmin -l -c "source /usr/local/greenplum-db/greenplum_path.sh;gpinitsystem -a -c  /tmp/gpinitsystem_singlenode -h /tmp/gpdb-hosts --max_connections=20; exit 0 " \
        && su gpadmin -l -c "export MASTER_DATA_DIRECTORY=/data/master/gpseg-1;source /usr/local/greenplum-db/greenplum_path.sh;psql -d template1 -c \"alter user gpadmin password 'changeme'\"; createdb gpadmin;  exit 0"

EXPOSE 5432 22

#VOLUME /data
# Set the default command to run when starting the container

CMD echo $(grep $(hostname) /etc/hosts | cut -f1) mdw  >> /etc/hosts \
        && echo "127.0.0.1 $(cat ~/orig_hostname)" >> /etc/hosts \
        && service sshd start \
#       && sysctl -p \
        && su gpadmin -l -c "/usr/local/bin/run.sh" \
        && /bin/bash
