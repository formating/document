
**Luasql module实现odbc,mysql,sqlite,postgresql,oracle数据库连接，本文主要介绍Mac OS X 平台的编译以及至其他Linux平台编译时的相关问题**

###1. [有关lua连接数据库的相关module列表](http://lua-users.org/wiki/DatabaseAccess):

===
**Here are resources related to database access in Lua.
DBMS-generic**

>* **DBMS-generic**
* [LuaSQL](http://www.keplerproject.org/luasql/) (5.0/5.1) - ODBC, Oracle, MySQL, SQLite, JDBC and PostgreSQL.
* [LuaDBI](http://lua-users.org/wiki/DatabaseAccess) (5.1) - DB2, Oracle, MySQL, PostgreSQL, SQLite. Based on Perl's DBI, provides prepared statements, placeholders and bind parameters.
* [Lua/APR relational database drivers](http://lua-users.org/wiki/DatabaseAccess) wrap APR's interface to relational database engines like SQLite, MySQL, PostgreSQL, Oracle, ODBC and FreeTDS.
* [LuaLDAP](http://lua-users.org/wiki/DatabaseAccess) (5.0/5.1) - LDAP client: OpenLDAP [1] or ADSI [2].
* [Lua Web Tools](http://lua-users.org/wiki/DatabaseAccess) (5.1/5.2) - MySQL, SQLite3, FreeTDS
* [LuaODBC](http://lua-users.org/wiki/DatabaseAccess) (5.1/5.2) - ODBC bindig for Lua.


>* **DBMS-specific**
* [lgdbm](http://lua-users.org/wiki/DatabaseAccess) (5.0/5.1) - gdbm [3].
* [LuaBDB](http://lua-users.org/wiki/DatabaseAccess) (5.0) - Berkeley DB [4].
* [luacdb](http://lua-users.org/wiki/DatabaseAccess) (5.1) - Dan Bernstein's cdb [5].
* [LuaPgSQL](http://lua-users.org/wiki/DatabaseAccess) (5.1) - Marc Balmer's PostgreSQL binding for Lua. [6].
* [LuaSQLite](http://lua-users.org/wiki/DatabaseAccess) (5.1) - Marc Balmer's SQLite3 binding for Lua.
* [LJSQLite3](http://lua-users.org/wiki/DatabaseAccess) (LuaJIT 2.0) - Pure LuaJIT library to access SQLite3 databases.
* [lua-tinycdb](http://lua-users.org/wiki/DatabaseAccess) (5.1) - Michael Tokarev's tinycdb [7].
* [luatdb](http://lua-users.org/wiki/DatabaseAccess) (5.1) - Andrew Tridgell's tdb [8].
* [LuaISIS](http://lua-users.org/wiki/DatabaseAccess) (5.0) - some CDS/ISIS features [?].
* [LuaCodeBase](http://lua-users.org/wiki/DatabaseAccess) (5.0/5.1) - Sequiter's CodeBase [9].
* [luasqlite](http://lua-users.org/wiki/DatabaseAccess) (5.0/5.1) - SQLite 2/3.
* [Lua-Sqlite3](http://lua-users.org/wiki/DatabaseAccess) (5.0/5.1) - Sqlite 3. Two layer architecture, multiple interfaces, coroutines, iterators and much more.
* [Lua-wxSQLite3](http://lua-users.org/wiki/DatabaseAccess) (5.1) - Ulrich Telle's wxSQLite3 [10].
* [fbclient](http://lua-users.org/wiki/DatabaseAccess) (5.1) - Firebird 2.0-2.5 [11], pure-Lua binding, OOP and procedural interfaces, services API, blobs, and more.
* [redis-lua](http://lua-users.org/wiki/DatabaseAccess) (5.1) - a binding for Redis data store [12].
* [sidereal](http://lua-users.org/wiki/DatabaseAccess) (5.1) - a Lua binding for Redis, with an emphasis on non-blocking operation.
* [oluacle (written in Japanese)](https://bitbucket.org/zetamatta/oluacle/wiki/Home) (5.2) - Oracle
* See also [CsvUtils](http://lua-users.org/wiki/DatabaseAccess).

===

###2.环境准备:
+ Lua在Mac OS X 平台的安装可使用homebrew,也可自行编译:

	`brew search lua`

	`brew install lua`

	`brew link lua`

+ Download [Luasql](http://www.keplerproject.org/luasql/):

	`wget http://files.luaforge.net/releases/luasql/luasql/LuaSQL2.1.1/luasql-2.1.1.tar.gz`

+ Download [Oracle-InstantClient](http://www.oracle.com/technetwork/database/features/instant-client/index-097480.html)

	**编译luasql时需要以下几个包**:
	
	`instantclient-basic-macos.x64-11.2.0.3.0.zip`
	
	`instantclient-sdk-macos.x64-11.2.0.3.0.zip`
	
	`instantclient-sqlplus-macos.x64-11.2.0.3.0.zip`
	
	**配置完成Oracle客户端后需要手动export一下DYLD_LIBRARY_PATH**
	
	Mac OSX:
	`export DYLD_LIBRARY_PATH=/opt/oracleclient/instantclient_11_2/`
	
	Linux:
	
	`export LD_LIBRARY_PATH=$ORACLE_HOME/lib`
	
	完成Oracle客户端的配置后进行luasql的编译,不同平台编译需要更改config文件中的部分参数,以Mac OSX 10.9安装Oracle的oci8为例:
	
	
		formating$ cat config**
		
		# Driver (leave uncommented ONLY the line with the name of the driver)
		#T= mysql
		#T= odbc
		#T= postgres
		#T= sqlite
		#T= sqlite3
		T= oci8

		# Installation directories

		# Default prefix
		PREFIX = /usr/local

		# System's libraries directory (where binary libraries are installed)
		LUA_LIBDIR= $(PREFIX)/lib/lua/5.1

		# System's lua directory (where Lua libraries are installed)
		LUA_DIR= $(PREFIX)/share/lua/5.1

		# Lua includes directory
		LUA_INC= $(PREFIX)/include

		# Lua version number (first and second digits of target version)
		LUA_VERSION_NUM= 501

		# OS dependent
		#LIB_OPTION= -shared #for Linux
		LIB_OPTION= -bundle -undefined dynamic_lookup #for MacOS X

		LIBNAME= $T.so
		COMPAT_DIR= ../compat/src

		# Compilation parameters
		# Driver specific
		######## MySQL
		#DRIVER_LIBS= -L/usr/local/mysql/lib -lmysqlclient -lz
		#DRIVER_INCS= -I/usr/local/mysql/include
		######## Oracle OCI8
		DRIVER_LIBS= -L/opt/oracleclient/instantclient_11_2 -lz -lclntsh
		DRIVER_INCS= -I/opt/oracleclient/instantclient_11_2/sdk/demo -I/opt/oracleclient/instantclient_11_2/sdk/include
		######## PostgreSQL
		#DRIVER_LIBS= -L/usr/local/pgsql/lib -lpq
		#DRIVER_INCS= -I/usr/local/pgsql/include
		######## SQLite
		#DRIVER_LIBS= -lsqlite
		#DRIVER_INCS=
		######## SQLite3 
		#DRIVER_LIBS= -L/opt/local/lib -lsqlite3
		#DRIVER_INCS= -I/opt/local/include
		######## ODBC
		#DRIVER_LIBS= -L/usr/local/lib -lodbc
		#DRIVER_INCS= -DUNIXODBC -I/usr/local/include

		WARN= -Wall -Wmissing-prototypes -Wmissing-declarations -ansi -pedantic
		INCS= -I$(LUA_INC)
		CFLAGS= -O2 $(WARN) -I$(COMPAT_DIR) $(DRIVER_INCS) $(INCS) $(DEFS)
		CC= gcc

		# $Id: config,v 1.8 2007/10/27 22:55:27 carregal Exp $

	修改完以后`make`一下，如果无误的话直接`make install`就安装到lua的默认lib目录了。
	
	> Module 默认路径在:/usr/local/lib/lua/5.1/
	
	> 验证一下模块是否正常加载: 
	
		formating$ lua
		Lua 5.1.5  Copyright (C) 1994-2012 Lua.org, PUC-Rio
		> 
		> oci = require "luasql.oci8"
		
	