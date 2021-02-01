---
title: mysql存储过程样例
date: 2019-04-25 22:03:36
tags: ["mysql"]

---

mysql 存储过程样例

<!--more-->

```sql
CREATE DEFINER=`alertcode`@`%` PROCEDURE `logic_bak_table`(IN `bak_id` bigint(20),IN `tablename` varchar(255))
BEGIN
-- 功能说明： 根据备份表id恢复数据
-- 参数说明：bak_id          备份id
-- 					 tablename       源表名称


-- 		表如果不存在则创建备份表
			SET @createTable=CONCAT('CREATE TABLE IF NOT EXISTS ',tablename,'_bak',' LIKE ',tablename);
			PREPARE createTable FROM @createTable;
			EXECUTE createTable;
			DEALLOCATE PREPARE createTable;

--    备份数据

			SET @insertSql=concat('INSERT INTO ',tablename,'_bak',' SELECT * FROM ',tablename,' WHERE id = ',bak_id);
			SET @deleteSql=concat('DELETE FROM ',tablename,' WHERE id =',bak_id);
			PREPARE stmt FROM @insertSql;
			EXECUTE stmt;
			PREPARE delSql FROM @deleteSql;
			EXECUTE delSql;

			DEALLOCATE PREPARE stmt;
			DEALLOCATE PREPARE delSql;
END


CREATE DEFINER=`alertcode`@`%` PROCEDURE `logic_recover`(IN type TINYINT(4),IN tablename VARCHAR(255),IN databasename VARCHAR(255),IN bak_id BIGINT(20),OUT newId BIGINT(20))
BEGIN
-- 功能说明： 根据备份表id恢复数据
-- 参数说明：type            恢复策略 1
-- 					 tablename       源表名称
--           databasename    数据库名称
--           bak_id          备份id
--           newId           返回id 只有恢复策略为3时有值

-- 	定义局部变量
	DECLARE _AA,_A BIGINT DEFAULT 0;
	DECLARE _column MEDIUMTEXT;
-- 	查询id在备份表中是否存在

	SET @bakCount=concat('SELECT COUNT(*) INTO @baknum FROM ',tablename,'_bak',' WHERE id =',bak_id);
	PREPARE bakcount FROM @bakCount;
	EXECUTE bakcount;
  DEALLOCATE PREPARE bakcount;


-- 	查询id在主表中是否存在
	SET @mainCount=concat('SELECT COUNT(*) INTO @tnum FROM ',tablename,' WHERE id =',bak_id);
	PREPARE mainCount FROM @mainCount;
	EXECUTE mainCount;
  DEALLOCATE PREPARE mainCount;

-- 备份表
	SET _AA = @baknum;
-- 主表
	SET _A = @tnum;

-- 		恢复时A'表中的id永远不会出现在A表中
	IF _AA = 1 && _A = 0 then
			SET @insertSql=concat('INSERT INTO ',tablename,' SELECT * FROM ',tablename,'_bak WHERE id = ',bak_id);
			SET @deleteSql=concat('DELETE FROM ',tablename,'_bak WHERE id =',bak_id);
			PREPARE stmt FROM @insertSql;
			EXECUTE stmt;
			PREPARE delSql FROM @deleteSql;
			EXECUTE delSql;
			DEALLOCATE PREPARE stmt;
			DEALLOCATE PREPARE delSql;
			SET newId = bak_id;
	END IF;

-- 	恢复策略为1
-- 	恢复时A'表中的id主键存在于A表中,恢复时保留A表的id,删除A'表的id

	IF type=1 && _AA = 1 && _A = 1 THEN
 		  SET @deleteSqlAA=concat('DELETE FROM ',tablename,'_bak WHERE id =',bak_id);
			PREPARE deleteSqlAA FROM @deleteSqlAA;
			EXECUTE deleteSqlAA;
			DEALLOCATE PREPARE deleteSqlAA;
			SET newId = bak_id;
	END IF;



-- 	恢复策略为2
-- 	恢复时A'表中的id主键存在于A表中,恢复时保留A'表的id,删除A表的id

	IF type=2 && _AA = 1 && _A = 1 THEN
			SET @insertSql=concat('REPLACE INTO ',tablename,' SELECT * FROM ',tablename,'_bak',' WHERE id = ',bak_id);
			SET @deleteSqlAA=concat('DELETE FROM ',tablename,'_bak WHERE id =',bak_id);
			PREPARE insertSql FROM @insertSql;
			EXECUTE insertSql;
			PREPARE deleteSqlAA FROM @deleteSqlAA;
			EXECUTE deleteSqlAA;
			DEALLOCATE PREPARE insertSql;
			DEALLOCATE PREPARE deleteSqlAA;
			SET newId = bak_id;
	END IF;


-- 恢复策略为3
-- 恢复时A'表中的id主键存在于A表中，A表中的id保持不变，在A表中新增A'表的id
	IF type=3 && _AA = 1 && _A = 1 THEN

			SET @tmpsql=CONCAT('SELECT GROUP_CONCAT(COLUMN_NAME) INTO @column FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = "',tablename,'_bak"','and COLUMN_NAME !="id" and TABLE_SCHEMA = "',databasename,'"');
			PREPARE _tmpsql from @tmpsql ;
			EXECUTE _tmpsql;
			DEALLOCATE PREPARE _tmpsql;

			SET _column =@column;
			SET @selectSql=CONCAT('INSERT INTO ',tablename,' (',_column,' )',' SELECT ',_column,' FROM ',tablename,'_bak WHERE id = ',bak_id);

		  PREPARE selectSql FROM @selectSql;
			EXECUTE selectSql;
			DEALLOCATE PREPARE selectSql;
			SELECT LAST_INSERT_ID() INTO newId;
-- 			删除备份表id
		  SET @deleteSqlAA=concat('DELETE FROM ',tablename,'_bak WHERE id =',bak_id);
			PREPARE deleteSqlAA FROM @deleteSqlAA;
			EXECUTE deleteSqlAA;
			DEALLOCATE PREPARE deleteSqlAA;
		END IF;
END


CREATE DEFINER=`alertcode`@`%` PROCEDURE `create`()
BEGIN
	#Routine body goes here...
	 declare i int(11) DEFAULT 0;
	 SET autocommit=0;
	 while i<10000 do
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 insert into table_name(lending_id) values
			 (rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000),(rand()*10000);
			 set i=i+1;
			 if i % 100 =0 then
			 COMMIT;
			 end if;
		end while;
END




```
