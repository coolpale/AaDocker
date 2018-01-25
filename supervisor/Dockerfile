FROM aario/centos:web
MAINTAINER Aario <AarioAi@gmail.com>

RUN yum install -y supervisor

################ Entrypoint ########################
COPY ./script/entrypoint.sh         ${ENT_SCRIPT}
COPY ./script/autorun/*             ${AUTORUN_SCRIPT_DIR}/

RUN echo -e "" >>${ENT_CONST_SCRIPT}       \
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
RUN yum clean all && rm -rf /var/cache/yum
#################################################

# forward request and error logs to docker log collector
RUN ln -sf /dev/stdout /var/log/stdout.log    		\
    && ln -sf /dev/stderr /var/log/stderr.log
	
	
COPY ./etc/* /etc/
COPY ./etc/supervisord.d/* /etc/supervisord.d/
COPY ./bin/* /usr/local/bin/
	
RUN chown -R ${SHARED_USER}:${SHARED_GROUP} "/var/log/supervisor" 		\
	&& chmod -R u+rwx "/var/log/supervisor"		\
	&& chown -R ${SHARED_USER}:${SHARED_GROUP} "/etc/supervisord.d" 		\
	&& chown ${SHARED_USER}:${SHARED_GROUP} "/etc/supervisord.conf"		\
	&& chown -R ${SHARED_USER}:${SHARED_GROUP} "/usr/local/bin"
	

USER ${SHARED_USER}
ENTRYPOINT ["supervisord", "-c", "/etc/supervisord.conf"]

# docker run -d --restart=always --name aa_supervisor --privileged=true -p 8888:8888 -v /var/lib/mnt/om/dba/supervisor_0:/var/lib/data -v /var/lib/mnt/om/log/supervisor_0:/var/log -e ENTRYPOINT_LOG=/var/log/supervisor/docker_entrypoint.log -e UPDATE_REPO=0 -e TIMEZONE=Asia/Shanghai -it aario/supervisor