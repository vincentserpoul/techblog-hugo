+++
date = "2016-12-27T00:44:00+08:00"
title = "Golang and Oracle"
description = "How to use Oracle in golang"
tags = [ "golang", "oracle", "linux", "sql" ]
topics = [ "golang", "oracle", "linux", "sql" ]
slug = "golang-playwithsql-oracle"
+++

## Which library

To this day, the most up to date library seems to be [rana/ora](https://github.com/rana/ora)

## How to install (linux)

Download [Oracle Instant Client for linux x64](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html): both packages Basic and SDK
Unzip each of them in the same folder /opt/oracle

```powershell
mkdir -p /opt/oracle
cd /opt/oracle
unzip ~/Downloads/instantclient-basic-linux.x64-12.1.0.2.0.zip
unzip ~/Downloads/instantclient-sdk-linux.x64-12.1.0.2.0.zip
cd /opt/oracle/instantclient_12_1
```

Follow the instructions from Oracle:

```powershell
ln -s libclntsh.so.12.1 libclntsh.so
ln -s libocci.so.12.1 libocci.so
```

Add the necessary env variables and paths:

```powershell
# Oracle
export LD_LIBRARY_PATH=/opt/oracle/instantclient_12_1:/opt/oracle/instantclient_12_1/sdk/include
export PKG_CONFIG_PATH=/opt/oracle
export PATH=/opt/oracle/instantclient_12_1:$PATH
export ORACLE_HOME=/opt/oracle/instantclient_12_1:/opt/oracle/instantclient_12_1/sdk/include
```

copy from the go package ./contrib/oci8.pc to /opt/oracle and modify its content to:

```
version=12.1

includedir=/opt/oracle/instantclient_12_1/sdk/include
libdir=/opt/oracle/instantclient_12_1

Name: oci8
Description: Oracle database engine
Version: ${version}
Libs: -L${libdir} -lclntsh
Libs.private:
Cflags: -I${includedir}
```

then simply

```powershell
go install ./...
```

You should be all good!