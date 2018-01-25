FROM aario/centos:web
MAINTAINER Aario <AarioAi@gmail.com>

ENV LOGSTASH_VER logstash-5.2.2
ENV LOGSTASH_SRC_URL https://download.elastic.co/logstash/logstash/${LOGSTASH_VER}.tar.gz
ENV LOGSTASH_PREFIX ${PREFIX_BASE}/logstash

COPY ./etc/* /etc/
ADD ./src/* ${SRC_DIR}/

WORKDIR ${SRC_DIR}
RUN if [ ! -d "${SRC_DIR}/${LOGSTASH_VER}" ]; then                      	\
		curl -sSL ${LOGSTASH_SRC_URL} -o ${SRC_DIR}/${LOGSTASH_VER}.tar.gz;	\
		tar -zxvf ${SRC_DIR}/${LOGSTASH_VER}.tar.gz;                      	\
		rm -f ${SRC_DIR}/${LOGSTASH_VER}.tar.gz;                      	 	\
	fi
	
#	Requires:
#       Ruby
#       Java-openjdk
#       Redis
#       ElasticSearch (there's one in logstash)
RUN yum install -y java-openjdk 											\
	&& mkdir -p ${LOGSTASH_PREFIX}                                  		\         
	&& mv ${SRC_DIR}/${LOGSTASH_VER}/* ${LOGSTASH_PREFIX}                 	\
	&& ln -s ${LOGSTASH_PREFIX}/bin/logstash ${BIN_DIR}/logstash

################ Entrypoint ########################
COPY ./script/entrypoint.sh         ${ENT_SCRIPT}
COPY ./script/autorun/*             ${AUTORUN_SCRIPT_DIR}/

RUN echo -e "\n " >>${ENT_CONST_SCRIPT}       \
    && sed -i 's/^\s*//g' ${ENT_CONST_SCRIPT}

RUN rm -rf /tmp/spool && mkdir /tmp/spool
COPY ./spool/* /tmp/spool
RUN if [ -f "/tmp/spool/crontab" ]; then                    \
        if [ -f "/etc/crontab" ]; then                      \
            yes | cp /etc/crontab /etc/crontab-cp1;         \
            cat /tmp/spool/crontab >> /etc/crontab;         \
        else                                                \
            mv /tmp/spool/crontab /etc/crontab;             \
        fi;                                                 \
        sort -k2n "/etc/crontab" | sed '$!N; /^\(.*\)\n\1$/!P; D' > "/tmp/crontab";    \
        yes | mv "/tmp/crontab" "/etc/crontab";             \
        yes | cp /etc/crontab /etc/crontab-cp2;             \
        rm -rf /tmp/spool;                                  \
    fi
RUN yum clean all && rm -rf /var/cache/yum && rm -rf ${SRC_DIR}/*
#################################################

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/stdout.log    		\
    && ln -sf /dev/stderr /var/log/stderr.log