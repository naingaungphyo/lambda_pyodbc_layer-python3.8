# lambda_pyodbc_layer-python3.8
pyodbc lambda layer for python3.8 runtime - built with Amazon Linux 2!

The following guide is based on this [gist](https://gist.github.com/diriver63/b72a954fa0da4851d89e5086aa13c6e8)!

## run docker container
docker run -it --rm --entrypoint bash -e ODBCINI=/opt/odbc.ini -e ODBCSYSINI=/opt/ lambci/lambda:build-python3.8

ref:https://hub.docker.com/r/lambci/lambda/tags

## download unixODBC
curl ftp://ftp.unixodbc.org/pub/unixODBC/unixODBC-2.3.9.tar.gz -O
tar xzvf unixODBC-2.3.9.tar.gz
cd unixODBC-2.3.9

ref:http://www.unixodbc.org/download.html

## install unixODBC
./configure --sysconfdir=/opt --disable-gui --disable-drivers --enable-iconv --with-iconv-char-enc=UTF8 --with-iconv-ucode-enc=UTF16LE --prefix=/opt
make
make install
cd ..
rm -rf unixODBC-2.3.9 unixODBC-2.3.9.tar.gz

## downlad and install ODBC driver for MSSQL 17
curl https://packages.microsoft.com/config/rhel/6/prod.repo > /etc/yum.repos.d/mssql-release.repo

yum install e2fsprogs.x86_64 0:1.43.5-2.43.amzn1 fuse-libs.x86_64 0:2.9.4-1.18.amzn1 libss.x86_64 0:1.43.5-2.43.amzn1
ACCEPT_EULA=Y yum install msodbcsql17
export CFLAGS="-I/opt/include"
export LDFLAGS="-L/opt/lib"

cd /opt
cp -r /opt/microsoft/msodbcsql17/ .
rm -rf /opt/microsoft/


## install pyodbc for use with python.
mkdir /opt/python/
cd /opt/python/
pip install pyodbc -t .

## edit for removing "microsoft" folder
cat <<EOF > odbcinst.ini
[ODBC Driver 17 for SQL Server]
Description=Microsoft ODBC Driver 17 for SQL Server
Driver=/opt/msodbcsql17/lib64/libmsodbcsql-17.6.so.1.1
UsageCount=1
EOF

cat <<EOF > odbc.ini
[ODBC Driver 17 for SQL Server]
Driver = ODBC Driver 17 for SQL Server
Description = My ODBC Driver 17 for SQL Server
Trace = No
EOF

# package the content in a zip file to use as a lambda layer
cd /opt
zip -r9 ~/pyodbc-layer.zip .

# copy zip file from docker container
docker cp <containerId>:/opt/pyodbc-layer.zip /home/ec2-user/
