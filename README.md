# Python3.7_CentOS5

Unfortunately there are still some CentOS/RHEL5 systems running around, the repository provides the instructions and a binary tar with Python 3.7.0, compiled for CentOS5.

The following source based componentes  are used:
* Docker image: astj/centos5-vault
* ftp://sourceware.org/pub/libffi/libffi-3.2.1.tar.gz
* https://www.openssl.org/source/openssl-1.0.2p.tar.gz
* https://sqlite.org/2018/sqlite-autoconf-3240000.tar.gz
* https://npm.taobao.org/mirrors/python/3.7.0/Python-3.7.0.tgz


Build Process
============
```bash

# Start a CentOS5 docker container
docker run -it themattrix/centos5-vault-i386 bash

# fix repos
cd /etc/yum.repos.d/
sed -e 's+http://vault.centos.org/5.11/+http://archive.kernel.org/centos-vault/5.11/+' -i *

# Install all the packages needed by the build process
yum install -y unzip wget patch \
    gcc zlib-devel python-setuptools readline-devel make bzip2-devel perl

# Install our custom libffi
wget ftp://sourceware.org/pub/libffi/libffi-3.2.1.tar.gz
tar xzvf libffi*
cd libfii*
./configure --prefix=/opt/python3.7
make
cd ..

# Build a recent openssl version, required for Python's ssl support
wget --no-check-certificate https://www.openssl.org/source/openssl-1.0.2p.tar.gz
tar xzvpf openssl*
cd openssl-*
./config --prefix=/usr/local/ssl --openssldir=/usr/local/ssl
sed -i.orig '/^CFLAG/s/$/ -fPIC/' Makefile
make -j4 && make install
cd ..

# Build a recent sqlite version
wget --no-check-certificate  http://sqlite.org/2018/sqlite-autoconf-3240000.tar.gz
tar xzvf sqlite*.tar.gz
cd sqlite*
./configure --prefix=/opt/python3.7
make install
cd ..

# Get Python from a mirror which is not strict about the SSL version requirement
wget https://npm.taobao.org/mirrors/python/3.7.0/Python-3.7.0.tgz

# Built Python using the local SQLite and SSL versions
tar xzvf Python-*
cd Python-*
wget --no-check-certificate https://raw.githubusercontent.com/joaompinto/Python3_CentOS5/master/python3_local_ssl.patch
patch -p0 < python3_local_ssl.patch
LD_RUN_PATH=/opt/python3.7/lib ./configure LDFLAGS="-L/opt/python3.7/lib" CPPFLAGS="-I/opt/python3.7/lib" \
    -prefix=/opt/python3.7/
wget --no-check-certificate https://raw.githubusercontent.com/joaompinto/Python3_CentOS5/master/python3_local_ffi.patch
patch -p0 < python3_local_ffi.patch
LD_RUN_PATH=/opt/python3.7/lib:/opt/python3.7/lib64 make -j4
make altinstall

find /opt/python3.7/  -name "*.pyc" -delete
find /opt/python3.7/ -name "*.pyo" -delete
wget --no-check-certificate https://raw.githubusercontent.com/joaompinto/Python3_CentOS5/master/scripts/python -O /opt/python3.7/bin/python
wget --no-check-certificate https://raw.githubusercontent.com/joaompinto/Python3_CentOS5/master/scripts/pip -O /opt/python3.7/bin/pip
chmod 755 /opt/python3.7/bin/*
tar czvf /tmp/python3.7-centos5.tar.gz -C /opt /opt/python3.7/

# From the docker host
docker cp  $CONTAINER_ID:/tmp/python3.7-centos5.tar.gz  /tmp
```
