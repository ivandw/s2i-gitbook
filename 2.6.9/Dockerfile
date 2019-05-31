FROM centos:centos7
## instala los paquetes requeridos. Necesitamos Apache, Git rsync para tirar
## l contenido estatico y varios paquetes de node

RUN rpmkeys --import file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7 && \
    PACKAGES1="git httpd rsync" && \
    yum install -y --setopt=tsflags=nodocs ${PACKAGES1} && \
    rpm -V ${PACKAGES1} && \
    yum install -y centos-release-scl-rh && \
    PACKAGES2="rh-nodejs8 rh-nodejs8-npm rh-nodejs8-nodejs-nodemon" && \
    ln -s /usr/lib/node_modules/nodemon/bin/nodemon.js /usr/bin/nodemon && \
    yum install -y --setopt=tsflags=nodocs $PACKAGES2 && \
    rpm -V ${PACKAGES2} && \
    yum clean all -y && \
    mkdir -p /opt/app-root && \
    chown 1001:0 /opt/app-root && \
#crea una cuenta no root llamada default para ser owner de los archivos de apache
    useradd -u 1001 -r -g 0 -d /opt/app-root -s /sbin/nologin \
            -c "Default Application User" default

# modfica la config de apache para escuchar en el puerto 8080 y loguear todo a
# stdout, usar el directorio donde tiramos el gitbook clonado y el home del usr que creamos antes

RUN sed -i -e "s%^Listen 80$%Listen 8080%" \
           -e "s%logs/access_log%/proc/self/fd/1%" \
           -e "s%logs/error_log%/proc/self/fd/2%" \
           -e "s%/var/www/html%/opt/app-root/src/_book%" \
           /etc/httpd/conf/httpd.conf && \
    echo "DefaultRuntimeDir /opt/app-root" >> /etc/httpd/conf/httpd.conf && \
    echo "PidFile /opt/app-root/httpd.pid" >> /etc/httpd/conf/httpd.conf

EXPOSE 8080

# Install GitBook CLI to global SCL NodeJS installation.

RUN source scl_source enable rh-nodejs8 && \
    npm install -g gitbook-cli

# Copio los scripts y seteo las etiquetas

COPY s2i /opt/app-root/s2i

LABEL io.k8s.description="S2I builder image for gitbook" \
      io.k8s.display-name="gitbook" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="gitbook" \
      io.openshift.s2i.scripts-url="image:///opt/app-root/s2i/bin"
#aseguro que puedan ejecutar los scripts
RUN chown -R 1001:0 /opt/app-root/s2i && \
    find /opt/app-root/s2i -type d -exec chmod g+ws {} \;

# Ensure container runs as non root account.

USER 1001

# setear home

ENV HOME=/opt/app-root

# instalo y updateo el GitBook package

RUN source scl_source enable rh-nodejs8 && gitbook fetch latest

# Create the 'src' directory. le seteo los permisos para que se pueda pegar archivos ( para usar rsync )

RUN mkdir /opt/app-root/src && \
find /opt/app-root -type d -exec chmod g+ws {} \;

# aseguro que la build este levantando como raiz, el directorio donde tengo los archivos estaticos

WORKDIR /opt/app-root/src

# corro usage

CMD [ "/opt/app-root/s2i/bin/usage" ]
