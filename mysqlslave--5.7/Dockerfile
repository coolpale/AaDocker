FROM aario/mysql:5.7
MAINTAINER Aario <AarioAi@gmail.com>





##################### my.cnf #################################
#ENV MYSQL_PORT 3306
ENV SERVER_ID 2

#RUN sed -Ei "s/^[\s;]*port\s*=\s*[0-9]+\s*$/port=${MYSQL_PORT}/g" /etc/my.cnf
RUN sed -i '/^#/d' /etc/my.cnf      \
    && sed -Ei "s/^[\s;]*server\-id\s*=\s*[0-9]+\s*$/server-id=${SERVER_ID}/g" /etc/my.cnf

#########################################################


################ Entrypoint ########################

COPY ./script/autorun/*             ${AUTORUN_SCRIPT_DIR}/

RUN rm -rf /tmp/spool && mkdir /tmp/spool
COPY ./spool/* /tmp/spool
RUN if [ -f "/tmp/spool/crontab" ]; then            \
        if [ -f "/etc/crontab" ]; then              \
            yes | cp /etc/crontab /etc/crontab-cp1; \
            cat /tmp/spool/crontab >> /etc/crontab; \
        else                                        \
            mv /tmp/spool/crontab /etc/crontab;     \
        fi;                                         \
        sort -k2n "/etc/crontab" | sed '$!N; /^\(.*\)\n\1$/!P; D' > "/tmp/crontab";    \
        yes | mv "/tmp/crontab" "/etc/crontab";     \
        yes | cp /etc/crontab /etc/crontab-cp2;     \
        rm -rf /tmp/spool;                          \
    fi
RUN yum clean all  && rm -rf /var/cache/yum && rm -rf ${SRC_DIR}/*