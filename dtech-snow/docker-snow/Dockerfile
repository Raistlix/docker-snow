#############################################################
#    Docker file to build a Container supporting the UA     #
# This docker file has been modified so that it upgrades instead of doing a raw install of SNOW#
#############################################################

FROM    centos:centos7
MAINTAINER Discover Technologies, LLC. <shooper@discovertechnologies.com>
LABEL version="release-0.0.1" description="DTech Docker ServiceNow Container"

# Declare the default shell
SHELL ["/bin/bash", "-c"] 

# Default to UTF-8 file.encoding
ENV LC_ALL en_US.UTF-8

# Declare Build Args
ARG share
ARG account
ARG key
ARG host
ARG username
ARG pass
ARG snowfile
ENV share=$share
ENV account=$account
ENV key=$key
ENV host=$host
ENV username=$username
ENV pass=$pass
ENV snowfile=$snowfile

#  Install Yum Repo Tools
RUN \
  yum update -y && \
  yum install -y --nogpgcheck tar util-linux wget zip unzip gcc which vim curl nano zsh

# Install Tools and Supervisor
RUN \
  echo sslverify=false >> /etc/yum.conf && \
  yum install -y --nogpgcheck epel-release && \
  yum install -y python-pip htop && \
  yum clean all && \
  easy_install supervisor && \
  pip install supervisor-stdout

# Install JDK 8
RUN \
  yum install -y --nogpgcheck java-1.8*

# Configure Supervisor
RUN \
  mkdir -p /etc/supervisor/conf.d && \
  /usr/bin/echo_supervisord_conf > /etc/supervisord.conf && \
  sed -i -e "s/^nodaemon=false/nodaemon=true/" /etc/supervisord.conf && \
  echo [include] >> /etc/supervisord.conf && \
  echo 'files = /etc/supervisor/conf.d/*.conf' >> /etc/supervisord.conf
ADD docker-snow.conf /etc/supervisor/conf.d/

# Azure-CLI
RUN rpm --import https://packages.microsoft.com/keys/microsoft.asc && \
    echo -e "[azure-cli]\nname=Azure CLI\nbaseurl=https://packages.microsoft.com/yumrepos/azure-cli\nenabled=1\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" \
    > /etc/yum.repos.d/azure-cli.repo
RUN yum install -y azure-cli

RUN az storage file download --account-name $account --account-key $key --share-name $share --path $snowfile --dest /tmp/snow.zip

# # Prep ServiceNow

RUN export SERVICENOW_MOUNT=/glide
RUN export SERVICENOW_JAR_ZIP_FILE=/tmp/$snowfile
RUN export SERVICENOW_APP_NAME=sn_16001
RUN export SERVICENOW_APP_LONG_NAME="Glide DockerSnow"
RUN export SERVICENOW_INSTALL_DIR=$SERVICENOW_MOUNT/nodes/$SERVICENOW_APP_NAME
RUN export SERVICENOW_INSTALL_TIMEOUT=20
RUN export RDS_ENDPOINT_ADDRESS=$host
RUN export RDS_DB_USERNAME=$username
RUN export RDS_DB_PASSWORD=$pass
RUN export JAVA_HOME=/etc/alternatives/java_sdk
RUN export PATH=/sbin:/bin:/usr/sbin:/usr/bin
RUN export PORT=16001
RUN export JAVA_HOME


# # MORE prep

RUN yum -y install glibc* glibc.i686

# # Prep for ServiceNow install

RUN useradd servicenow

# # Upgrade ServiceNow

#RUN SNOW upgrade
RUN java -jar /tmp/snow.zip --dst-dir /glide/nodes/sn_16001 install -n sn -p 16001



# Remove ServiceNow demo data

#RUN export SERVICENOW_ZBOOT="${SERVICENOW_INSTALL_DIR}/webapps/glide/itil/WEB-INF/sys.scripts/zboot_cold_instance.js"
#RUN sed -i -e "/db_reset_successful/d" "/glide/nodes/sn_16001/webapps/glide/itil/WEB-INF/sys.scripts/"
#RUN sed -i -e "/zboot_demo_data.js/d" "/glide/nodes/sn_16001/webapps/glide/itil/WEB-INF/sys.scripts/"
#RUN sed -i -e '/}/d' "/glide/nodes/sn_16001/webapps/glide/itil/WEB-INF/sys.scripts/"
# Update ServiceNow DB properties file
#RUN mkdir /glide/nodes/sn_16001/conf
#RUN chown -R servicenow:servicenow /glide

RUN mkdir -p /glide/nodes/sn_16001/conf
RUN touch /glide/nodes/sn_16001/conf/glide.db.properties


RUN export SERVICENOW_DB_PROPERTIES="${SERVICENOW_INSTALL_DIR}/conf/glide.db.properties"
RUN echo "glide.db.user = $username" >> "/glide/nodes/sn_16001/conf/glide.db.properties"
RUN echo "glide.db.password = $pass" >> "/glide/nodes/sn_16001/conf/glide.db.properties"
RUN echo "glide.db.encrypt = true" >> "/glide/nodes/sn_16001/conf/glide.db.properties"
RUN sed -i -e "s#<DatabaseAccount>#$username#g" "/glide/nodes/sn_16001/conf/glide.db.properties"
RUN sed -i -e "s#<Password>#$pass#g" "/glide/nodes/sn_16001/conf/glide.db.properties"
RUN sed -i -e "s#localhost#$host#g" "/glide/nodes/sn_16001/conf/glide.db.properties"

# Update ServiceNow Glide properties file

RUN export SERVICENOW_GLIDE_PROPERTIES="${SERVICENOW_INSTALL_DIR}/conf/glide.properties"
RUN touch /glide/nodes/sn_16001/conf/glide.properties
RUN sed -i '/16001/a glide.db.pooler.connections = 32' "/glide/nodes/sn_16001/conf/glide.properties"
RUN sed -i '/16001/a glide.db.pooler.connections.max = 32' "/glide/nodes/sn_16001/conf/glide.properties"
RUN sed -i '/16001/a glide.sys.schedulers = 8' "/glide/nodes/sn_16001/conf/glide.properties"
RUN sed -i '/16001/a glide.monitor.url = docker-snow' "/glide/nodes/sn_16001/conf/glide.properties"
RUN sed -i '/16001/a glide.self.monitor.fast_stats = false' "/glide/nodes/sn_16001/conf/glide.properties"
RUN sed -i '/16001/a glide.self.monitor.checkin.interval = 86400000' "/glide/nodes/sn_16001/conf/glide.properties"
RUN sed -i '/16001/a glide.self.monitor.server_stats.interval = 86400000' "/glide/nodes/sn_16001/conf/glide.properties"
RUN sed -i '/16001/a glide.self.monitor.fast_server_stats.interval = 86400000' "/glide/nodes/sn_16001/conf/glide.properties"

# Add ServiceNow StartUp Script
ADD startSnow.sh wait-for /glide/nodes/sn_16001/
RUN chmod +x /glide/nodes/sn_16001/startSnow.sh && chmod +x /glide/nodes/sn_16001/wait-for

# Set JAVA_HOME
ENV JAVA_HOME /etc/alternatives/java_sdk

# Document Exposed Ports
EXPOSE 16001

# Default command
CMD ["./wait-for -q docker-mysql:3306 -- /usr/bin/supervisord"]
