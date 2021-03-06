XCACHE

## Introduction
xcache is based on ccache. It can store the outputs of compile into remote servers.

It consists of several components:

- xcache

>A new in-memory database which has high efficiency. All the outputs of compile will be stored in it.

- xcache_client

>It's just the ccache with the capability to communicate with the xcache.

- xcache_monitor

>The monitor of the xcache.

## How to build

 `cd server && make -j`

 `cd client && ./autogen.sh && ./configure && make -j`

>xcache and xcache_monitor under the directory of server

>xcache_client under the the directory of server

## How to use:
### Single Mode
- Setup the xcache on remote server which is 127.0.0.1. We will run:

 `xcache -d -c -p 20190 -m 1`

 >The parameter -d means to run as a daemon.
 
 >The parameter -c means to generate a core file when xcache crashed.
 
 >The parameter -m 1 means to set the original memory of the database to 1G.You can change the value by yourself.
 
 >The parameter -p 20190 means to set the port to 20190. You can change the value you like.
 
 >You can run xcache -h to see another parameters available.
 
- Setup the xcache_client, Please run any one of the following commands.

 `export CCACHE_REMOTE_CONF="--SERVER=127.0.0.1:20190 10"`

  >The value 10 is the xcache_core, please see the distribute cluster mode below.
  
- Run the ccache to compile

 `xcache_client gcc test.c -c -o test.o`

### Distributed Cluster Mode
- Setup the xcache on remote server which is 127.0.0.1. We will run two databases.

 `xcache -d -c -p 20190 -m 1`

 `xcache -d -c -p 20191 -m 1`

 >The parameter -d means to run as a daemon.
 
 >The parameter -c means to generate a core file when xcache crashed.
 
 >The parameter -m 1 means to set the original memory of the database to 1G.You can change the value by yourself.
 
 >The parameter -p 20190 means to set the port to 20190. You can change the value you like.
 
 >You can run xcache -h to see another parameters available.

- Setup the proxy on remote server which is 127.0.0.1. Please run: 

 `xcache -d -c -p 6666 -P`

 >The parameter -P means to change the xcache to a proxy which has a high priority.

- Tell the proxy about the information of the xcache. Please run :

 `xcache_monitor -S 127.0.0.1:6666["--SERVER=127.0.0.1:20190 10 --SERVER=127.0.0.1:20191 10"]`

  >The format of command is like: 
  
  `xcache_monitor -S proxy_ip:proxy_port["--SERVER=xcache_ip:xcache_port xcache_core"]`
 
  >The xcache_core means the power of the database. The one which has bigger xcache_core would store more data than others.
  
- Setup the xcache_client.

 If you don’t use the proxy, please run:
 
 `export CCACHE_REMOTE_CONF="--SERVER=127.0.0.1:20190 10 --SERVER=127.0.0.1:20191 10"` 

 If you use the proxy, please run one of the following commands:
 
 `export CCACHE_REMOTE_CONF="--PROXY=127.0.0.1:6666"`

 ``` export CCACHE_REMOTE_CONF=`xcache_monitor -G 127.0.0.1:6666 | tail -1 | grep "SERVER="` ``` 
    
- Run the ccache to compile

 `xcache_client gcc test.c -c -o test.o`


