`Luasql module实现odbc,mysql,sqlite,postgresql,oracle数据库连接，本文主要介绍Mac OS X 平台的编译以及至其他Linux平台编译时的相关问题`

1. [有关lua连接数据库的相关module列表](http://lua-users.org/wiki/DatabaseAccess):

**Here are resources related to database access in Lua.
DBMS-generic**

>* **DBMS-generic**
* [LuaSQL](http://www.keplerproject.org/luasql/) (5.0/5.1) - ODBC, Oracle, MySQL, SQLite, JDBC and PostgreSQL.
* [LuaDBI] (5.1) - DB2, Oracle, MySQL, PostgreSQL, SQLite. Based on Perl's DBI, provides prepared statements, placeholders and bind parameters.
* [Lua/APR relational database drivers] wrap APR's interface to relational database engines like SQLite, MySQL, PostgreSQL, Oracle, ODBC and FreeTDS.
* [LuaLDAP] (5.0/5.1) - LDAP client: OpenLDAP [1] or ADSI [2].
* [Lua Web Tools] (5.1/5.2) - MySQL, SQLite3, FreeTDS
* [LuaODBC] (5.1/5.2) - ODBC bindig for Lua.

===
>* **DBMS-specific**
* [lgdbm] (5.0/5.1) - gdbm [3].
* [LuaBDB] (5.0) - Berkeley DB [4].
* [luacdb] (5.1) - Dan Bernstein's cdb [5].
* [LuaPgSQL] (5.1) - Marc Balmer's PostgreSQL binding for Lua. [6].
* [LuaSQLite] (5.1) - Marc Balmer's SQLite3 binding for Lua.
* [LJSQLite3] (LuaJIT 2.0) - Pure LuaJIT library to access SQLite3 databases.
* [lua-tinycdb] (5.1) - Michael Tokarev's tinycdb [7].
* [luatdb] (5.1) - Andrew Tridgell's tdb [8].
* [LuaISIS] (5.0) - some CDS/ISIS features [?].
* [LuaCodeBase] (5.0/5.1) - Sequiter's CodeBase [9].
* [luasqlite] (5.0/5.1) - SQLite 2/3.
* [Lua-Sqlite3] (5.0/5.1) - Sqlite 3. Two layer architecture, multiple interfaces, coroutines, iterators and much more.
* [Lua-wxSQLite3] (5.1) - Ulrich Telle's wxSQLite3 [10].
* [fbclient] (5.1) - Firebird 2.0-2.5 [11], pure-Lua binding, OOP and procedural interfaces, services API, blobs, and more.
* [redis-lua] (5.1) - a binding for Redis data store [12].
* [sidereal] (5.1) - a Lua binding for Redis, with an emphasis on non-blocking operation.
* [oluacle (written in Japanese)](https://bitbucket.org/zetamatta/oluacle/wiki/Home) (5.2) - Oracle
* See also CsvUtils.