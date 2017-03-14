+++
date = "2016-12-27T00:44:00+08:00"
title = "Golang and Oracle"
description = "How to use Oracle in golang"
tags = [ "golang", "oracle", "linux", "macosx", "sql" ]
topics = [ "golang", "oracle", "linux", "macosx", "sql" ]
slug = "golang-playwithsql-oracle"
+++

## Which library

To this day, the most up to date library seems to be [rana/ora](https://github.com/rana/ora)

## How to install (linux & macosx)

Download [Oracle Instant Client for linux x64](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html): both packages Basic and SDK
Unzip each of them in the same folder /opt/oracle

```powershell
mkdir -p /opt/oracle
cd /opt/oracle
unzip ~/Downloads/instantclient-basiclite-linux.x64-12.2.0.1.0.zip
unzip ~/Downloads/instantclient-sdk-linux.x64-12.2.0.1.0.zip
cd /opt/oracle/instantclient_12_1
```

Add the necessary env variables and paths:

```powershell
# Oracle
export LD_LIBRARY_PATH=/opt/oracle/instantclient_12_2:/opt/oracle/instantclient_12_2/sdk/include
export PKG_CONFIG_PATH=/opt/oracle
export PATH=/opt/oracle/instantclient_12_2:$PATH
export ORACLE_HOME=/opt/oracle/instantclient_12_2:/opt/oracle/instantclient_12_2/sdk/include
```

copy from the [go package ./contrib/oci8.pc](https://github.com/rana/ora/tree/v4/contrib) to /opt/oracle and modify its content to:

```
prefix=/opt/oracle/instant_client_12_2
version=12.2
build=client64

libdir=${prefix}/lib
includedir=${prefix}/sdk/include

Name: oci8
Description: Oracle database engine
Version: ${version}
Libs: -L${libdir} -lclntsh
Libs.private:
Cflags: -I${includedir}
```

## On linux

Follow the [instructions](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html#ic_x64_inst) from Oracle:

```powershell
ln -s /opt/oracle/instantclient_12_2/libclntsh.so.12.1 /opt/oracle/instantclient_12_2/libclntsh.so
ln -s /opt/oracle/instantclient_12_2/libocci.so.12.1 /opt/oracle/instantclient_12_2/libocci.so
```

## On macosx

Follow the instructions from Oracle:

```powershell
ln -s /opt/oracle/instantclient_12_2/libclntsh.dylib.12.1 /opt/oracle/instantclient_12_2/libclntsh.dylib
ln -s /opt/oracle/instantclient_12_2/libocci.dylib.12.1 /opt/oracle/instantclient_12_2/libocci.dylib
```

One more step is necessary for macosx.

You have to add your machine name in your /etc/hosts for the 127.0.0.1

```
127.0.0.1 localhost YOURMACHINENAME
```

## Install the package

```powershell
go get -u cd gopkg.in/rana/ora.v4
```

You should be all good!