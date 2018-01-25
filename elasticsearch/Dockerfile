FROM aario/centos:web
MAINTAINER Aario <AarioAi@gmail.com>

ENV ELASTICSEARCH_VER elasticsearch-5.2.2
ENV ELASTICSEARCH_SRC_URL https://artifacts.elastic.co/downloads/elasticsearch/${ELASTICSEARCH_VER}.tar.gz

ENV ELASTICSEARCH_USER ${SHARED_USER}
ENV ELASTICSEARCH_GROUP ${SHARED_GROUP}
ENV ELASTICSEARCH_PREFIX  ${PREFIX_BASE}/elasticsearch


WORKDIR ${SRC_DIR}
ADD ./src/* ${SRC_DIR}/
RUN if [ ! -d "${SRC_DIR}/${ELASTICSEARCH_VER}" ]; then                                 \
        curl -sSL ${ELASTICSEARCH_SRC_URL} -o ${SRC_DIR}/${ELASTICSEARCH_VER}.tar.gz;	\
        tar -zxvf ${SRC_DIR}/${ELASTICSEARCH_VER}.tar.gz;                               \
        rm -f ${SRC_DIR}/${ELASTICSEARCH_VER}.tar.gz;                                   \
    fi


RUN yum install -y java-openjdk 											\
	&& mkdir -p ${ELASTICSEARCH_PREFIX}                                    	\
    && mv ${SRC_DIR}/${ELASTICSEARCH_VER}/* ${ELASTICSEARCH_PREFIX}     	\
	&& chown -R ${ELASTICSEARCH_USER}:${ELASTICSEARCH_GROUP} ${ELASTICSEARCH_PREFIX} \
	&& mv ${ELASTICSEARCH_PREFIX}/config/elasticsearch.yml ${ELASTICSEARCH_PREFIX}/config/elasticsearch.yml.cp
COPY ./config/* ${ELASTICSEARCH_PREFIX}/config/


################ Entrypoint ########################
COPY ./script/entrypoint.sh         ${ENT_SCRIPT}
COPY ./script/autorun/*             ${AUTORUN_SCRIPT_DIR}/

RUN echo -e "\n ELASTICSEARCH_USER='${ELASTICSEARCH_USER}' \n ELASTICSEARCH_GROUP='${ELASTICSEARCH_GROUP}' \n ELASTICSEARCH_PREFIX='${ELASTICSEARCH_PREFIX}'" >> ${ENT_CONST_SCRIPT}         \
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
ENTRYPOINT ["/aa_script/entrypoint.sh", "/usr/local/elasticsearch/bin/elasticsearch"]

# docker run -d --restart=always --name aa_elasticsearch --privileged=true -p 9200:9200 -p 9300:9300 -v /var/lib/mnt/om/dba/elasticsearch_0:/var/lib/elasticsearch -v /var/lib/mnt/om/log/elasticsearch_0:/var/log -e ENTRYPOINT_LOG=/var/log/elasticsearch/docker_entrypoint.log -e UPDATE_REPO=0 -e TIMEZONE=Asia/Shanghai -it aario/elasticsearch