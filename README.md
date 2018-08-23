# Python3.7.0_CentOS5

Python_Build
============


# Start a CentOS5 docker container
docker run -it astj/centos5-vault bash

# Install all the packages needed by the build process
#yum install -y  epel-release
yum install -y unzip wget \
    gcc zlib-devel python-setuptools readline-devel make bzip2-devel perl
#libffi-devel


# Install our custom libffi
wget ftp://sourceware.org/pub/libffi/libffi-3.2.1.tar.gz
tar xzvf libffi*
cd libfii*
./configure --prefix=/opt/python37
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
./configure --prefix=/opt/python37
make install
cd ..

# Get Python from a mirror which is not strict about the SSL version requirement
wget https://npm.taobao.org/mirrors/python/3.7.0/Python-3.7.0.tgz

# Built Python using the local SQLite and SSL versions
export CC=/usr/bin/gcc44
tar xzvf Python-*
cd Python-*

# Patch to use our local ssl version
#sed -i "s/#SSL=/SSL=/"  Modules/Setup.dist
#sed -i "s/#_ssl/_ssl/"  Modules/Setup.dist
#sed -i "s/#.*-DUSE_SSL/-DUSE_SSL/"  Modules/Setup.dist
#sed -i "s/#.*-L\$(SSL)/-L\$(SSL)/" Modules/Setup.dist

LD_RUN_PATH=/opt/mdatapipe-$MDP_VERSION/sqlite3/lib/ ./configure LDFLAGS="-L/opt/mdatapipe-$MDP_VERSION/sqlite3/lib/" CPPFLAGS="-I/opt/mdatapipe-$MDP_VERSION/sqlite3/include" -prefix=/opt/mdatapipe-$MDP_VERSION/python-3.7.0
LD_RUN_PATH=/opt/mdatapipe-$MDP_VERSION/sqlite3/lib/ make -j4
make altinstall

/opt/mdatapipe-$MDP_VERSION/python-3.7.0/bin/pip3.7  install https://github.com/mdatapipe/mdatapipe/archive/master.zip
find /opt/mdatapipe-$MDP_VERSION/ -name "*.pyc" -delete
find /opt/mdatapipe-$MDP_VERSION/ -name "*.pyo" -delete
tar czvf /tmp/mdatapipe-$MDP_VERSION.tar.gz -C /opt mdatapipe-$MDP_VERSION/
exit

# From the docker host
ls -1 /tmp/python2.17_mdatapipe/* | xargs -I% -n1 docker cp % $CONTAINER_ID:/tmp
docker cp  $CONTAINER_ID:/tmp/python27_mdatapipe.tar.gz /tmp

# ToDo
Move libffi to the ld_lib_path

