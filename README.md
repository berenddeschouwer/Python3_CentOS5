# Python3.7.0_CentOS5

Python_Build
============


# Start a CentOS5 docker container
docker run -it astj/centos5-vault bash

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
wget --no-check-certificate  https://sqlite.org/2018/sqlite-autoconf-3240000.tar.gz
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
wget  https://raw.githubusercontent.com/joaompinto/Python3_CentOS5/scripts/python -O /opt/python3.7/bin
tar czvf /tmp/python3.7-centos5.tar.gz -C /opt /opt/python3.7/
exit

# From the docker host
ls -1 /tmp/python2.17_mdatapipe/* | xargs -I% -n1 docker cp % $CONTAINER_ID:/tmp
docker cp  $CONTAINER_ID:/tmp/python3.7-centos5.tar.gz .tar.gz /tmp

# ToDo
Move libffi to the ld_lib_path

