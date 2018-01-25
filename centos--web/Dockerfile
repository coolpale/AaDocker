FROM centos:7
MAINTAINER Aario <AarioAi@gmail.com>
# docker 具备自启动功能，所以不再需要supervisor

# 某些情况，需要用户地址
ENV SHARED_GROUP docker
ENV SHARED_USER docker
RUN groupadd -r "${SHARED_GROUP}"												\
	&& useradd -d "/home/${SHARED_USER}" -m -g "${SHARED_GROUP}" "${SHARED_USER}"


# System Directories
ENV SYS_CONF_DIR    /etc
#   S_P_L_DIR   Lock files
#   @warning, /var/lock direcotry is disabled in docker container
#ENV S_P_L_DIR       /var/lock
#   S_P_L_DIR     pid and sock
#ENV S_P_L_DIR         /var/run

# Spool  --> queue
ENV SPOOL_DIR       /var/spool
#ENV CRON_DIR        ${SPOOL_DIR}/cron
ENV CRONTAB_FILE    /etc/crontab
#ENV AT_DIR          ${SPOOL_DIR}/at
ENV PREFIX_BASE     /usr/local
ENV SRC_DIR         /usr/local/src
ENV BIN_DIR         /usr/local/bin
ENV SBIN_DIR        /usr/local/sbin
ENV LIB_DIR         /usr/local/lib
ENV LIB64_DIR       /usr/local/lib64
ENV LIB_EXEC_DIR    /usr/local/libexec
ENV INCLUDE_DIR     /usr/local/include
ENV CACHE_DIR       /var/cache
ENV SSL_DIR         /etc/ssl
ENV LIB_DIR         /usr/lib64

# Self-Defined Directories
#   SCRIPT_DIR         Entrypoint Scripts
#               only root user executable shell scripts
#               warning don't run these scripts by non-root user!!
#   SYNC_DIR   backup data into SYNC_DIR, then transfer the data to remote backup servers with internet
#   S_P_L_DIR  .sock, .pid, .lock 
ENV S_P_L_DIR           /aa_run
ENV SHAREDSTATE_DIR     ${S_P_L_DIR}/com
ENV SCRIPT_DIR          /aa_script
ENV AUTORUN_SCRIPT_DIR  ${SCRIPT_DIR}/autorun
ENV SYNC_DIR            /aa_sync

RUN mkdir -p "${SHAREDSTATE_DIR}" 									\
	&& chown -R ${SHARED_USER}:${SHARED_GROUP} ${S_P_L_DIR}				\
	&& chmod -R g+rxw ${S_P_L_DIR}

RUN if [ ! -d "${SCRIPT_DIR}" ]; then                               \
        mkdir -p "${SCRIPT_DIR}";                                   \
        chmod -R u+x ${SCRIPT_DIR};                                 \
    fi
    
RUN if [ ! -d "${AUTORUN_SCRIPT_DIR}" ]; then                               \
        mkdir -p "${AUTORUN_SCRIPT_DIR}";                                   \
        chmod -R u+x ${AUTORUN_SCRIPT_DIR};                                 \
    fi

RUN if [ ! -d "${SYNC_DIR}" ]; then                      \
        mkdir -p "${SYNC_DIR}";                          \
        chmod -R a+rw ${SYNC_DIR};                       \
    fi
    
    
COPY ./etc/* /etc/
ADD ./src/* ${SRC_DIR}/

RUN rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7                              \
    && yum update -y                                                                \
    && yum clean all                                                                \
    && yum install -y epel-release                                                  \
    && rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

RUN echo -e "\nexport HISTSIZE=0\n" >> /etc/profile
RUN source /etc/profile



################ Entrypoint And Clean ########################
ENV ENT_CONST_SCRIPT    ${SCRIPT_DIR}/entrypointConst.sh
ENV ENT_BASE_SCRIPT     ${SCRIPT_DIR}/entrypointBase.sh
ENV ENT_SCRIPT          ${SCRIPT_DIR}/entrypoint.sh
COPY ./script/entrypointConst.sh    ${ENT_CONST_SCRIPT}
COPY ./script/entrypointBase.sh     ${ENT_BASE_SCRIPT}
COPY ./script/entrypoint.sh         ${ENT_SCRIPT}
COPY ./script/autorun/*             ${AUTORUN_SCRIPT_DIR}/


RUN echo -e "\n SHARED_GROUP='${SHARED_GROUP}' \n SHARED_USER='${SHARED_USER}' \n SYS_CONF_DIR='${SYS_CONF_DIR}' \n SPOOL_DIR='${SPOOL_DIR}' \n S_P_L_DIR='${S_P_L_DIR}' \n PREFIX_BASE='${PREFIX_BASE}' \n CRONTAB_FILE='${CRONTAB_FILE}' \n SRC_DIR='${SRC_DIR}' \n BIN_DIR='${BIN_DIR}' \n SBIN_DIR='${SBIN_DIR}' \n LIB_DIR='${LIB_DIR}' \n LIB64_DIR='${LIB64_DIR}' \n LIB_EXEC_DIR='${LIB_EXEC_DIR}' \n INCLUDE_DIR='${INCLUDE_DIR}' \n CACHE_DIR='${CACHE_DIR}' \n PREFIX_BASE='${PREFIX_BASE}' \n SCRIPT_DIR='${SCRIPT_DIR}' \n AUTORUN_SCRIPT_DIR='${AUTORUN_SCRIPT_DIR}' \n SYNC_DIR='${SYNC_DIR}' \n SSL_DIR='${SSL_DIR}' \n ENT_CONST_SCRIPT='${ENT_CONST_SCRIPT}' \n ENT_BASE_SCRIPT='${ENT_BASE_SCRIPT}' \n ENT_SCRIPT='${ENT_SCRIPT}' \n SHAREDSTATE_DIR='${SHAREDSTATE_DIR}'" >> ${ENT_CONST_SCRIPT}               \
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
    
RUN yum clean all && rm -rf /var/cache/yum && rm -rf ${SRC_DIR}/* n