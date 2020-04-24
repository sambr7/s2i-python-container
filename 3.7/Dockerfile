# Centos - Python 3.7.7
FROM centos/s2i-base-centos7

EXPOSE 8080

ENV PYTHON_VERSION=3.7 \
    PATH=$HOME/.local/bin/:$PATH:/opt/mssql-tools/bin\
    PYTHONUNBUFFERED=1 \
    PYTHONIOENCODING=UTF-8 \
    LC_ALL=en_US.UTF-8 \
    LANG=en_US.UTF-8 \
    PIP_NO_CACHE_DIR=off


ENV SUMMARY="Platform for building and running Python $PYTHON_VERSION applications" \
    DESCRIPTION="Python $PYTHON_VERSION available as container is a base platform for \
building and running various Python $PYTHON_VERSION applications and frameworks. \
Python is an easy to learn, powerful programming language. It has efficient high-level \
data structures and a simple but effective approach to object-oriented programming. \
Python's elegant syntax and dynamic typing, together with its interpreted nature, \
make it an ideal language for scripting and rapid application development in many areas \
on most platforms."

LABEL summary="$SUMMARY" \
    description="$DESCRIPTION" \
    io.k8s.description="$DESCRIPTION" \
    io.k8s.display-name="Python 3.7.7" \
    io.openshift.expose-services="8080:http" \
    io.openshift.tags="builder,python,python37,python-37,rh-python37" \
    com.redhat.component="python37-container" \
    name="centos/python-37-centos7" \
    version="1" \
    usage="s2i build https://github.com/sclorg/s2i-python-container.git --context-dir=3.7/test/setup-test-app/ centos/python-37-centos7 python-sample-app" \
    maintainer="SoftwareCollections.org <sclorg@redhat.com>"

RUN INSTALL_PKGS="nss_wrapper \
    httpd24 httpd24-httpd-devel httpd24-mod_ssl httpd24-mod_auth_kerb httpd24-mod_ldap \
    httpd24-mod_session atlas-devel gcc-gfortran libtool-ltdl enchant" && \
    yum install -y centos-release-scl && \
    yum install -y epel-release && \
    yum -y --setopt=tsflags=nodocs install --enablerepo=centosplus $INSTALL_PKGS --nogpgcheck && \
    rpm -V $INSTALL_PKGS && \
    curl https://packages.microsoft.com/config/rhel/7/prod.repo > /etc/yum.repos.d/mssql-release.repo && \
    yum remove unixODBC-utf16 unixODBC-utf16-devel && \
    ACCEPT_EULA=Y yum install msodbcsql17 -y --nogpgcheck && \
    ACCEPT_EULA=Y yum install mssql-tools -y --nogpgcheck && \
    wget http://mirror.centos.org/centos/7/os/x86_64/Packages/libffi-devel-3.0.13-18.el7.x86_64.rpm && \
    chmod 777 libffi-devel-3.0.13-18.el7.x86_64.rpm && rpm -ivh libffi-devel-3.0.13-18.el7.x86_64.rpm && \
    wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz && tar xvf Python-3.7.7.tgz && \
    cd Python-3.7*/ && ./configure --enable-optimizations && make altinstall && \
    rm -rf /opt/app-root/src/libffi-devel-3.0.13-18.el7.x86_64.rpm && \
    rm -rf /opt/app-root/src/Python-3.7.* && \
    rpm -e --nodeps centos-logos && \
    yum -y clean all --enablerepo='*'

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH.
COPY ./s2i/bin/ $STI_SCRIPTS_PATH

# Copy extra files to the image.
COPY ./root/ /

# - Create a Python virtual environment for use by any application to avoid
#   potential conflicts with Python packages preinstalled in the main Python
#   installation.
# - In order to drop the root user, we have to make some directories world
#   writable as OpenShift default security model is to run the container
#   under random UID.

COPY ./s2i/custom/pip.conf ${APP_ROOT}/.pip/pip.conf
ENV PIP_CONFIG_FILE=${APP_ROOT}/.pip/pip.conf

RUN python3.7 -m venv ${APP_ROOT} && \
    chown -R 1001:0 ${APP_ROOT} && \
    fix-permissions ${APP_ROOT} -P && \
    rpm-file-permissions

USER 1001

# Set the default CMD to print the usage of the language image.
CMD $STI_SCRIPTS_PATH/usage