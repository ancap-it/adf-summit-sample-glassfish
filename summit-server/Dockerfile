FROM centos:7

# INstall dependencies
RUN yum -y update
RUN yum install -y unzip
RUN yum install -y wget

RUN yum -y install findutils && \
    yum -y install java-1.8.0-openjdk && \
    yum -y clean all

RUN java -version

ADD lib/oracle-instantclient19.6-basic-19.6.0.0.0-1.x86_64.rpm /oracle-instantclient19.6-basic-19.6.0.0.0-1.x86_64.rpm
RUN yum localinstall -y /oracle-instantclient19.6-basic-19.6.0.0.0-1.x86_64.rpm --nogpgcheck
ENV ORACLE_HOME="/usr/lib/oracle/19.6/client64"
ENV PATH="${ORACLE_HOME}/bin:${PATH}"

RUN adrci -version

RUN cd /opt

RUN wget http://download.oracle.com/glassfish/4.1/release/glassfish-4.1.zip
RUN unzip glassfish-4.1.zip -d /opt/
# Set environment variables and default password for user 'admin'
ENV GLASSFISH_HOME=/opt/glassfish4 \
    PATH=$PATH:/opt/glassfish4/bin \
    PASSWORD=glassfish

COPY webapps/SummitADF_application.ear /SummitADF_application.ear

#COYP ADF STUFF
COPY adf-essentials.zip /tempadf/
RUN unzip -j /tempadf/adf-essentials.zip -d /opt/glassfish4/glassfish/domains/domain1/lib
RUN wget -O /opt/glassfish4/glassfish/modules/javax.faces.jar https://repo.maven.apache.org/maven2/org/glassfish/javax.faces/2.2.8-11/javax.faces-2.2.8-11.jar
RUN wget -O /opt/glassfish4/glassfish/modules/javax.el.jar https://repo1.maven.org/maven2/org/glassfish/javax.el/3.0.1-b08/javax.el-3.0.1-b08.jar
COPY lib/com.oracle.webservices.fmw.client.jar /opt/glassfish4/glassfish/domains/domain1/lib/

RUN echo "AS_ADMIN_PASSWORD=" > /tmp/glassfishpwd && \
    echo "AS_ADMIN_NEWPASSWORD=${PASSWORD}" >> /tmp/glassfishpwd  && \
    echo "--- Enable DAS, change admin password, and secure admin access ---" && \
    asadmin --user=admin --passwordfile=/tmp/glassfishpwd change-admin-password --domain_name domain1 && \
    asadmin start-domain && \
    echo "AS_ADMIN_PASSWORD=${PASSWORD}" > /tmp/glassfishpwd && \
    asadmin --user=admin --passwordfile=/tmp/glassfishpwd enable-secure-admin && \
    asadmin create-jdbc-connection-pool --user=admin --passwordfile=/tmp/glassfishpwd \
       --restype javax.sql.DataSource \
       --datasourceclassname oracle.jdbc.pool.OracleDataSource \
       --property user=c##summit_adf:password=summit_adf:url="jdbc\:oracle\:thin\:@summit-db\:1521\:xe" kriPool &&\
    asadmin create-jdbc-resource --user=admin --passwordfile=/tmp/glassfishpwd \
       --connectionpoolid kriPool jdbc/local_xeDS && \
    asadmin --user=admin --passwordfile=/tmp/glassfishpwd \
        create-jvm-options -XX\\:MaxPermSize=512m:-Doracle.mds.cache=simple && \
    asadmin --user=admin --passwordfile=/tmp/glassfishpwd  deploy /SummitADF_application.ear && \
    asadmin stop-domain # && \
    rm /tmp/glassfishpwd

# Ports being exposed
EXPOSE 4848 8080 8181

# Start asadmin console and the domain
CMD ["asadmin", "start-domain", "-v"]