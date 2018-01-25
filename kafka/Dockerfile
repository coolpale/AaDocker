FROM aario/centos:web
MAINTAINER Aario <AarioAi@gmail.com>

ENV KAFKA_MINOR_VER_NUM 1.0.0
ENV KAFKA_VER kafka_2.12-${KAFKA_MINOR_VER_NUM}
ENV KAFKA_SRC_URL http://www-us.apache.org/dist/kafka/${KAFKA_MINOR_VER_NUM}/${KAFKA_VER}.tgz
ENV KAFKA_USER ${SHARED_USER}
ENV KAFKA_GROUP ${SHARED_GROUP}
ENV KAFKA_PREFIX ${PREFIX_BASE}/kafka
ENV KAFKA_CONF_DIR ${KAFKA_PREFIX}/config
ENV KAFKA_CONF ${KAFKA_CONF_DIR}/server.properties
ENV KAFKA_DATA_DIR /var/lib/kafka
ENV KAFKA_LOG_DIR /var/log/kafka


# Install Postfix
#RUN yum install -y postfix && /usr/sbin/alternatives --set mta /usr/sbin/sendmail.postfix

#RUN yum install -y mailx


RUN yum remove java-1.6.0* && yum remove java-1.7.0* && yum install -y java-1.8.0-openjdk
RUN for jre in $(ls /usr/lib/jvm | grep ^java-); do						\
		if [ -d '/usr/lib/jvm/'$jre'/jre' ]; then						\
			echo -e "\nJAVA_HOME=/usr/lib/jvm/$jre/jre" /etc/profile;	\
			source /etc/profile;										\
		fi																\
	done


WORKDIR ${SRC_DIR}
ADD ./src/* ${SRC_DIR}/


RUN if [ ! -d "${SRC_DIR}/${KAFKA_VER}" ]; then                                 \
        curl -sSL ${KAFKA_SRC_URL} -o ${SRC_DIR}/${KAFKA_VER}.tar.gz;           \
        tar -zxvf ${SRC_DIR}/${KAFKA_VER}.tar.gz;                               \
        rm -f ${SRC_DIR}/${KAFKA_VER}.tar.gz;                                   \
    fi


ENV BROKER_ID 0

RUN mkdir -p ${KAFKA_PREFIX}                                                    \
    && mv ${SRC_DIR}/${KAFKA_VER}/* ${KAFKA_PREFIX}                             \
    && sed -i "/^broker.id=.*/d" ${KAFKA_CONF}  \
    && sed -i "s/^zookeeper.connect\s*=.*/zookeeper.connect=aa_zookeeper:2181/" ${KAFKA_CONF} \
    && sed -i "s/^log.dirs\s*=.*/log.dirs=\/var\/log\/kafka/" ${KAFKA_CONF}     \
    && sed -i "/^#/d" ${KAFKA_CONF} \
    && echo -e "\nbroker.id=${BROKER_ID}" >> ${KAFKA_CONF}

COPY ./etc/kafka/* ${KAFKA_CONF_DIR}/

################ Entrypoint ########################
COPY ./script/entrypoint.sh         ${ENT_SCRIPT}
COPY ./script/autorun/*             ${AUTORUN_SCRIPT_DIR}/

RUN echo -e "KAFKA_USER='${KAFKA_USER}' \n KAFKA_GROUP='${KAFKA_GROUP}' \n KAFKA_PREFIX='${KAFKA_PREFIX}' \n KAFKA_DATA_DIR='${KAFKA_DATA_DIR}' \n KAFKA_LOG_DIR='${KAFKA_LOG_DIR}'" >> ${ENT_CONST_SCRIPT}         \
    && sed -i 's/^\s*//g' ${ENT_CONST_SCRIPT}

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
RUN yum clean all && rm -rf /var/cache/yum && rm -rf ${SRC_DIR}/*
#################################################


# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/stdout.log    		\
    && ln -sf /dev/stderr /var/log/stderr.log


# /script/entrypoint.sh redis-server /etc/redis.conf
ENTRYPOINT ["/aa_script/entrypoint.sh", "/usr/local/kafka/bin/kafka-server-start.sh", "/usr/local/kafka/config/server.properties"]