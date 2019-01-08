---
title: "Install Agensgraph On Centos7 "
date: 2019-01-08 17:09:28 -0400
categories: agensgraph centos
---
Agensgraph is graph database management system based on the PostgreSQL.
It supports Vertex, Edge(Node, Relationship) with flexible data type (JSON, properties)
You can download it from source code or pre-built packages. And easily install with following steps. But this is sample cluster setting.So you need more settings about production level.   

anyway. Let's start.
### Installation
#### Install essential libraries (root)
```bash
yum install -y gcc glibc glib-common readline readline-devel zlib zlib-devel flex bison git

```
```
# yum install -y gcc glibc glib-common readline readline-devel zlib zlib-devel flex bison git
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
 * base: mirror.kakao.com
 * epel: mirror.premi.st
 * extras: mirror.kakao.com
 * ius: hkg.mirror.rackspace.com
 * updates: mirror.kakao.com
No package glib-common available.
Package readline-6.2-10.el7.x86_64 already installed and latest version
Package readline-devel-6.2-10.el7.x86_64 already installed and latest version
Resolving Dependencies
--> Running transaction check
---> Package bison.x86_64 0:3.0.4-1.el7 will be updated
---> Package bison.x86_64 0:3.0.4-2.el7 will be an update
---> Package flex.x86_64 0:2.5.37-3.el7 will be updated
---> Package flex.x86_64 0:2.5.37-6.el7 will be an update
---> Package gcc.x86_64 0:4.8.5-16.el7_4.2 will be updated

```
#### User add (root)
```
adduser agens
passwd agens 
(type some password)
```
#### Download source code (agens)
```bash
su - agens 
git clone https://github.com/bitnine-oss/agensgraph.git
```
```
$ git clone https://github.com/bitnine-oss/agensgraph.git
Cloning into 'agensgraph'...
remote: Enumerating objects: 21518, done.
remote: Total 21518 (delta 0), reused 0 (delta 0), pack-reused 21518
Receiving objects: 100% (21518/21518), 39.74 MiB | 4.81 MiB/s, done.
Resolving deltas: 100% (14640/14640), done.
Checking out files: 100% (5091/5091), done.
$ 
```

#### configure & build
```
cd agensgraph
./configure --prefix=/home/agens/agensgraph2
make install
make install-world
```
```
$ ./configure --prefix=/home/agens/agensgraph2
checking build system type... x86_64-pc-linux-gnu
checking host system type... x86_64-pc-linux-gnu
checking which template to use... linux
checking whether NLS is wanted... no
checking for default port number... 5432
checking for block size... 8kB
checking for segment size... 1GB
checking for WAL block size... 8kB
checking for WAL segment size... 16MB
checking for gcc... gcc
checking whether the C compiler works... yes
checking for C compiler default output file name... a.out
checking for suffix of executables... 
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking whether gcc supports -Wdeclaration-after-statement... yes
...... skip .........
config.status: linking src/backend/port/posix_sema.c to src/backend/port/pg_sema.c
config.status: linking src/backend/port/sysv_shmem.c to src/backend/port/pg_shmem.c
config.status: linking src/backend/port/dynloader/linux.h to src/include/dynloader.h
config.status: linking src/include/port/linux.h to src/include/pg_config_os.h
config.status: linking src/makefiles/Makefile.linux to src/Makefile.port
$ 


$ make install
make -C src install
make[1]: Entering directory `/home/agens/agensgraph/src'
make -C common install
make[2]: Entering directory `/home/agens/agensgraph/src/common'
make -C ../backend submake-errcodes
make[3]: Entering directory `/home/agens/agensgraph/src/backend'
make -C utils errcodes.h
make[4]: Entering directory `/home/agens/agensgraph/src/backend/utils'
'/bin/perl' ./generate-errcodes.pl ../../../src/backend/utils/errcodes.txt > errcodes.h
make[4]: Leaving directory `/home/agens/agensgraph/src/backend/utils'
prereqdir=`cd 'utils/' >/dev/null && pwd` && \
...... skip .........
make[1]: Entering directory `/home/agens/agensgraph/config'
/bin/mkdir -p '/home/agens/agengraph2/lib/postgresql/pgxs/config'
/bin/install -c -m 755 ./install-sh '/home/agens/agengraph2/lib/postgresql/pgxs/config/install-sh'
/bin/install -c -m 755 ./missing '/home/agens/agengraph2/lib/postgresql/pgxs/config/missing'
make[1]: Leaving directory `/home/agens/agensgraph/config'
PostgreSQL installation complete.
$ 

$ make install-world
make -C doc install
make[1]: Entering directory `/home/agens/agensgraph/doc'
make[1]: Nothing to be done for `install'.
make[1]: Leaving directory `/home/agens/agensgraph/doc'
make -C src install
make[1]: Entering directory `/home/agens/agensgraph/src'
make -C common install
make[2]: Entering directory `/home/agens/agensgraph/src/common'
make -C ../backend submake-errcodes
make[3]: Entering directory `/home/agens/agensgraph/src/backend'

...... skip .........
e-new-dtags  -lpgcommon -lpgport -lpthread -lz -lreadline -lrt -lcrypt -ldl -lm  -o vacuumlo
/bin/mkdir -p '/home/agens/agengraph2/bin'
/bin/install -c  vacuumlo '/home/agens/agengraph2/bin'
make[2]: Leaving directory `/home/agens/agensgraph/contrib/vacuumlo'
make[1]: Leaving directory `/home/agens/agensgraph/contrib'
PostgreSQL, contrib, and documentation installation complete.
$ 

```

#### Environment Settings
add these settings into ~/.bash_profile

```bash
export AG_HOME=/home/agens/agensgraph2/
export LD_LIBRARY_PATH=$AG_HOME/lib:$LD_LIBRARY_PATH
export PATH=$AG_HOME/bin:$PATH
export AGDATA=/home/agens/db_cluster

```

#### Test Binary
```bash
agens --version
```
```
$ agens --version
agens (AgensGraph) 1.4devel
$ 
```

#### initdb & start cluster
```bash
initdb 
ag_ctl start
createdb
```

```
$ initdb         
The files belonging to this database system will be owned by user "agens".
This user must also own the server process.

The database cluster will be initialized with locale "en_US.UTF-8".
The default database encoding has accordingly been set to "UTF8".
The default text search configuration will be set to "english".

Data page checksums are disabled.

creating directory /home/agens/db_cluster ... ok
creating subdirectories ... ok
selecting default max_connections ... 100
selecting default shared_buffers ... 128MB
selecting dynamic shared memory implementation ... posix
creating configuration files ... ok
running bootstrap script ... ok
performing post-bootstrap initialization ... ok
syncing data to disk ... ok

WARNING: enabling "trust" authentication for local connections
You can change this by editing pg_hba.conf or using the option -A, or
--auth-local and --auth-host, the next time you run initdb.

Success. You can now start the database server using:

    ag_ctl -D /home/agens/db_cluster -l logfile start

$ ag_ctl start
waiting for server to start.... done
server started
$
$ createdb
$ 
```


### Create your own graph data.
#### Using agens cli
```bash
agens
```
```
$ agens
agens (AgensGraph 1.4devel, based on PostgreSQL 10.4)
Type "help" for help.

agens=# 
```

#### Create graph data using Cypher
```sql
create graph test_graph;
set graph_path = test_graph;
CREATE (tom:Person {id:1, name :'Tom'})-[:knows {since : 2018}]->(sam: Person {id:2, name :'Sam'})
MATCH (a)-[e]->(b)
RETURN * 
;
```
```
agens=# create graph test_graph;     
CREATE GRAPH
agens=# 
agens=# set graph_path = test_graph;
SET
agens=# CREATE (tom:Person {id:1, name :'Tom'})-[:knows {since : 2018}]->(sam: Person {id:2, name :'Sam'});
GRAPH WRITE (INSERT VERTEX 2, INSERT EDGE 1)
agens=# 
agens=# MATCH (a)-[e]->(b)
agens-# RETURN * 
agens-# ;
                  a                  |                 e                  |                  b                  
-------------------------------------+------------------------------------+-------------------------------------
 person[3.1]{"id": 1, "name": "Tom"} | knows[4.1][3.1,3.2]{"since": 2018} | person[3.2]{"id": 2, "name": "Sam"}
(1 row)

agens=# 
```
