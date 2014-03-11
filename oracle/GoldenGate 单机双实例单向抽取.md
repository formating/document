GoldenGate 单机双实例单向抽取
=======

**检查状态**:
    
    1.检查数据库是否处于归档状态，是否强制日志，是否使用了附加日志
        SQL> select log_mode,supplemental_log_data_min,force_logging from v$database;  
    2.**更改数据库为以上状态**:
        (1).更改归档日志路径,开启归档:
        SQL> alter system set log_archive_dest_1=‘LOCATION=/oradata/hxdbdg/arch1’;
        SQL> alter database archive log;
        
        (2).更改数据库到强制日志模式:
          SQL> alter database force logging;

        (3).更改数据库的日志模式为附加日志模式:
          SQL> alter database add supplemental log data;

2.建立用户:

        SQL>   create user ogg identified by ogg;
               grant execute on utl_file to ogg;
               grant dba,connect,resource to ogg;
               
               create user oggt identified by oggt;
               grant execute on utl_file to oggt;
               grant dba,connect,resource to oggt;
 
3.禁用recycle bin:

    SQL> alter system set recyclebin= OFF scope=both;
    (Oracle 11G需要重启才可以禁用回收站)

4.完成后进行Goldengate基础配置:

    (1).配置LIB文件路径.
         vi ~/.bash_profile
         export LD_LIBRARY_PATH=$ORACLE_HOME/lib:/lib/:/usr/lib:/usr/local/lib:$TUXDIR/lib:/oradata/ggs/

     (2).GGSCI (standby-220206) 2> create subdirs   (两个节点都需要建立)

     (3).在selinux开启的环境下可能无法执行ggsci需要进行某些so文件的授权:
      chcon -t texrel_shlib_t  /local/oracle/OraHome1/lib/libnnz10.so

####源端:

     编辑全局配置，指定具体goldengate的具体schema以及checkpointtable,两边均需要添加.
     node1 >  edit params ./global
     ggschema ogg
     checkpointtable ogg.checkpoint

配置manager端口:
    
    node1 > view params mgr
    port 7809

在源端登陆:

    node1 > dblogin userid ogg,password ogg

建立checkpint table:
     
     node 1 > add checkpointtable .ogg.checkpoint

建立抽取组 ext1:
    
     node1 > add extract ext1 ,tranlog ,begin now

指定源端读取日志后trail文件的位置:
    
     node1 > add exttrail /oradata/ggs/dirdat/k1,extract ext1

 编辑抽取配置文件:
     
     node1 > view params ext1

     extract ext1
     setenv(ORACLE_SID=hxdbdg)
     userid ogg,password ogg
     exttrail /oradata/ggs/dirdat/k1
     table test1.*;

查看具体抽取进程信息:
     
     node1 > info extract *
     node1 > info extract * ,detail
     node1 > info extract *,showch

添加捕获进程组:(需要抓取的trail文件位置)
     
     node1 > add extract dpump ,exttrailsource /oradata/ggs/dirdat/k1

添加目标trail文件目录位置(若在同一机器上需要指定不同的目录):

     node1 > add rmttrail /oradata/ggt/dirdata/k1,extract dpump
     
编辑捕获进:
    #(本机双实例测试的情况，指定的目标端口不同，在这还可以设置字符集 setenv(NLS_LANG=AMERICAN_AMERICA.ZHS16GBK))
     
     node1 >  editor params dpump
     extract dpump
     userid ogg,password ogg
     rmthost 127.0.0.1,mgrport 8809
     rmttrail /oradata/ggt/dirdat/k1
     table test1.*;

启动抽取进程以及捕获进程:
     
     node1 > start extract ext1
     node1 > start dpump 

查看抽取进程以及mgr等配置的信息

     node1 > info all
     Program          Status           Group   Lag at Chkpt   Time Since Chkpt

     MANAGER    RUNNING                                          
     EXTRACT     RUNNING     DPUMP    00:00:00      00:00:02   
     EXTRACT     RUNNING     EXT1        00:00:00      00:00:06    

日志信息:

     [root@standby-220206 ~]# tail -f /oradata/ggs/ggserr.log

####目标端:

     在目标节点配置复制进程组:
     
     配置全局设置:
     node2 > edit param ./globals
     ggschema oggt
     checkpointtable oggt.checkpoint

配置管理端:
  
     node2 > edit param mgr
     port 8809

 
指定源端推送来的trail文件位置进行读取复制，并且配置checkpoint表):

     node2 > add replicat rep1,exttrail /oradata/ggt/dirdat/k1,checkpointtable oggt.checkpoint
    
编辑复制进程配置:

     node2 > edit params rep1
     replicat rep1
     setenv(ORACLE_SID=hxdbstd)
     userid oggt,password oggt
     handlecollisions
     assumetargetdefs
     reperror default,discard
     discardfile /oradata/ggt/dirrpt/rpl1.dsc,append,megabytes 10
     map test1.*,target test1.*;


至此单向复制已经完成了，现在可以进行数据初始化或者直接进行复制:

     node2 > start rep1
     在使用expdp flashback_scn号的方式时候的指定从哪个SCN号复制开始.
     node2 > start rep1 aftercsn 1234567
     
     node2 > info all 
     Program     Status      Group       Lag at Chkpt  Time Since Chkpt
     MANAGER     RUNNING                                          
     REPLICAT    RUNNING     REP1        00:00:00      00:00:09


在使用与配置Goldengate之前需先将已有数据初始化至目标库,初始化方式有几种:

    1.使用Goldengate直接进行数据初始化.
    2.使用Rman进行数据初始化.
    3.使用expdp/impdp进行数据初始化.(基于SCN号的expdp).
    * 获取当前scn
        SQL> select current_scn from v$database;
    
     expdp user/password full=Y flashback_scn=1234567;
     impdp user/password dumpdir=dumpdir dumpfile=expdat.dmp table_existe_action=replace;

**可以完成数据初始化后再开始配置也可以先配置好goldengate后在进行数据的导入**.





         