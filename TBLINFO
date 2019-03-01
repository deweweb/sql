-- Description: Tabellestruktur Helper(select und print) mit Parameter:
-- ALL      > Alles anzeigen (default)
-- STRUKTUR > SELECT Struktur Abfrage
-- SELECT   > Print SELECT Abfrage
-- DROP     > Print DROP Abfrage
-- CREATE   > Print CREATE Abfrage
-- INSERT   > Print INSERT Abfrage, mit Random-Values im Columns befüllung

/* Drop the old stored porcedure (if exists). */
IF EXISTS (SELECT 1 FROM sysobjects WHERE id = object_id('TBLINFO') AND type in ('P','PC'))
	DROP PROCEDURE TBLINFO
GO

/* Create the new stored procedure */
CREATE PROCEDURE TBLINFO @tablename VARCHAR(50) = NULL, @SAVE_RESU_IN_TABLE INT = 0 , @TYP VARCHAR(10) = 'ALL', @table_view VARCHAR(10) = 'TABLE' 
AS
BEGIN

	DECLARE @name_of_entity AS int = NULL,
			@cols           AS varchar(max),
			@cols_nam       AS varchar(max),
			@query          AS varchar(max),
			@query_select   AS varchar(max),
			@query_drop     AS varchar(max),
			@query_create   AS varchar(max),
			@query_insert   AS varchar(max);
	
	
	------BITTE_TABELLE_NAME_EINGEBEN------------------------
	IF @tablename IS NULL BEGIN SET @tablename = 'TABLE_DEFAUT_NAME_THAT_EXIST' END
	---------------------------------------------------------
	
	-- Tabelle oder View erstelln
	SET @name_of_entity = (SELECT object_id FROM sys.tables WHERE name = @tablename)
	IF @name_of_entity IS NULL
	BEGIN
		SET @name_of_entity = (SELECT object_id FROM sys.views WHERE name = @tablename);
		IF @name_of_entity IS NOT NULL SET @table_view = 'VIEW'
		ELSE BEGIN 
			SET @name_of_entity = (SELECT object_id FROM sys.objects WHERE object_id = OBJECT_ID(@tablename) AND type in (N'P', N'PC'))
			IF @name_of_entity IS NOT NULL SET @table_view = 'SP'
			ELSE BEGIN
				SET @table_view = NULL 
				print 'Keine Tabelle/View/SP ist gefunden'
			END
		END
	END 
	
	
	IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N'_tmp_TBLINFO') AND type IN (N'U')) DROP TABLE _tmp_TBLINFO
	CREATE TABLE _tmp_TBLINFO (TBLINFO varchar(max) NULL)

	IF @table_view = 'SP' BEGIN
		print '-- ' + @table_view +': ' + @tablename + '' + CHAR(13);

		-- ABFRAGE: CREATE SP PARAMETER TABLE
		IF (@TYP = 'ALL' OR @TYP = 'CREATE') AND @table_view = 'SP'
		BEGIN
			SET @query_create = 'CREATE TABLE _tmp_sp_' + @tablename + ' (
			'
			SET @query_create = @query_create +
				'' SELECT @query_create = @query_create + COLUMN_NAME + ' ' + DATA_TYPE + ' ' + IS_NULLABLE  + ', '
						FROM (
						SELECT substring(name, 2, LEN(name)-1) AS COLUMN_NAME
							, parameter_id AS ORDINAL_POSITION
							, CASE WHEN system_type_id in (35, 99, 167, 175, 231, 239)
								   THEN  type_name(user_type_id) + '(' + cast(max_length AS VARCHAR(50)) + ')'
								   ELSE type_name(user_type_id)
							  END AS DATA_TYPE
							, CASE WHEN is_nullable = 1 THEN 'NULL' ELSE 'NOT NULL' END AS IS_NULLABLE
						   FROM sys.parameters where object_id = object_id('' + @tablename + '')
								) AS T
						order by ORDINAL_POSITION
			SET @query_create =  LEFT(@query_create, LEN(@query_create) - 1) + '
		)';
		------
		print '-- ABFRAGE: CREATE SP PARAMETER TABLE:
		' + @query_create + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (@query_create)
		END

		-- ABFRAGE: INSERT INTO SP PARAMETER TABLE 
		IF (@TYP = 'ALL' OR @TYP = 'INSERT') AND @table_view = 'SP'
		BEGIN
			SET @query_insert = 'INSERT _tmp_sp_' + @tablename + ' ('
			SET @query_insert = @query_insert + '' SELECT @query_insert = @query_insert + COLUMN_NAME + ', ' 
						FROM (SELECT substring(name, 2, LEN(name)-1) AS COLUMN_NAME, parameter_id AS ORDINAL_POSITION FROM sys.parameters WHERE object_id = object_id('' + @tablename + '') ) AS T
						order by ORDINAL_POSITION
			SET @query_insert =  LEFT(@query_insert, LEN(@query_insert) - 1) + ') VALUES ('
			SET @query_insert = @query_insert + '' SELECT @query_insert = @query_insert + PARAMETER_VALUE + ', ' 
						FROM (SELECT name AS PARAMETER_VALUE, parameter_id AS ORDINAL_POSITION FROM sys.parameters WHERE object_id = object_id('' + @tablename + '') ) AS T
						order by ORDINAL_POSITION
			SET @query_insert =  LEFT(@query_insert, LEN(@query_insert) - 1) + ') '
		------
		print '-- ABFRAGE: INSERT SP PARAMETER TABLE:
		' + @query_insert + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (@query_insert)
		
		END
		
		-- ABFRAGE: DROP SP PARAMETER TABLE
		IF @TYP = 'ALL' OR @TYP = 'DROP'
		BEGIN
			IF @table_view = 'SP' SET @query_drop = 'IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(_tmp_sp_N''' + @tablename + ''') AND type in (N''U'')) DROP TABLE _tmp_sp_' + @tablename
			------
		print '-- ABFRAGE: DROP ' + @table_view + ':
		' + @query_drop + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (@query_drop)
		END

		-- ABFRAGE: CREATE SP
		SET @query_create = (SELECT definition FROM sys.sql_modules WHERE object_id = @name_of_entity)
		------
		print CHAR(13) + CHAR(13) + CHAR(13) +'-- ABFRAGE: CREATE SP:
		' + @query_create + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (@query_create)

	END
	ELSE BEGIN
		-- TABELLE STRUKTUR
		IF @TYP = 'ALL' OR @TYP = 'STRUKTUR'
		BEGIN
			SET @cols     = stuff((SELECT DISTINCT ','+' isnull(cast(max(CASE WHEN ' + name + ' IS NULL THEN 1 ELSE 0 END) AS VARCHAR(50)), 0) AS ' + name + ' '
									FROM sys.all_columns
									WHERE object_id = @name_of_entity
										FOR XML path(''), type).value('.', 'nvarchar(max)'),1,1, '')
			SET @cols_nam = stuff((SELECT DISTINCT ', '+name+''
									FROM sys.all_columns
									WHERE object_id = @name_of_entity
										FOR XML path(''), type).value('.', 'nvarchar(max)'),1,1, '')
		
			-- STRUKTUR RESULT ABFRAGE
			SET @query = 'SELECT a.name_s, b.is_nullable, has_null_value, DATA_TYPE + isnull(''(''+cast(CHARACTER_MAXIMUM_LENGTH AS VARCHAR(50))+'')'', '''') AS data_typ
							FROM (SELECT name_s, has_null_value FROM (SELECT '+@cols+' FROM '+@tablename+') AS subsl UNPIVOT (has_null_value FOR name_s IN ('+@cols_nam+')) as unpvt) AS a
							INNER JOIN (select distinct COLUMN_NAME, CASE WHEN IS_NULLABLE = ''YES'' THEN 1 ELSE 0 END AS IS_NULLABLE , DATA_TYPE, CHARACTER_MAXIMUM_LENGTH FROM INFORMATION_SCHEMA.COLUMNS where TABLE_NAME = '''+@tablename+''') AS b
								ON a.name_s = b.COLUMN_NAME'
			exec (@query)
		END
		
		
		print @table_view +': ' + @tablename + '' + CHAR(13);
		print @tablename + ': COLUMNS STRUKTUR:
		' + STUFF(@cols_nam, 1, 1, '') + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (STUFF(@cols_nam, 1, 1, ''))
		
		-- ABFRAGE: SELECT FROM TABLE/VIEW
		IF @TYP = 'ALL' OR @TYP = 'SELECT'
		BEGIN
			SET @query_select = 'SELECT '
			SET @query_select = @query_select + '' SELECT @query_select = @query_select + COLUMN_NAME + ', '
						FROM (SELECT DISTINCT COLUMN_NAME, ORDINAL_POSITION FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = '' + @tablename + '') AS T
						order by ORDINAL_POSITION
			SET @query_select =  LEFT(@query_select, LEN(@query_select) - 1) + ' FROM ' + @tablename
		------
		print 'ABFRAGE: SELECT FROM ' + @table_view + ': 
		' + @query_select + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (@query_select)
		
		END
		
		
		-- ABFRAGE: DROP TABLE/VIEW
		IF @TYP = 'ALL' OR @TYP = 'DROP'
		BEGIN
			IF @table_view = 'TABLE' SET @query_drop = 'IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N''' + @tablename + ''') AND type in (N''U'')) DROP TABLE ' + @tablename
			ELSE SET @query_drop = 'IF EXISTS (SELECT * FROM sys.objects WHERE object_id = OBJECT_ID(N''' + @tablename + ''') AND type in (N''V'')) DROP TABLE ' + @tablename
		------
		print 'ABFRAGE: DROP ' + @table_view + ':
		' + @query_drop + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (@query_drop)
		
		END
		
		
		-- ABFRAGE: CREATE TABLE
		IF (@TYP = 'ALL' OR @TYP = 'CREATE') AND @table_view = 'TABLE'
		BEGIN
			SET @query_create = 'CREATE TABLE ' + @tablename + ' (
			'
			SET @query_create = @query_create + '' SELECT @query_create = @query_create + COLUMN_NAME + ' ' + DATA_TYPE + ' ' + IS_NULLABLE  + ', '
						FROM (SELECT DISTINCT COLUMN_NAME, ORDINAL_POSITION
								  , DATA_TYPE + isnull('('+cast(CHARACTER_MAXIMUM_LENGTH AS VARCHAR(50))+')', '') AS DATA_TYPE
								  , CASE WHEN IS_NULLABLE = 'YES' THEN 'NULL' ELSE 'NOT NULL' END AS IS_NULLABLE
								FROM INFORMATION_SCHEMA.COLUMNS
								WHERE TABLE_NAME = '' + @tablename + '') AS T
						order by ORDINAL_POSITION
			SET @query_create =  LEFT(@query_create, LEN(@query_create) - 1) + '
		)';
		------
		print 'ABFRAGE: CREATE TABLE:
		' + @query_create + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (@query_create)
		
		END
		
		
		-- ABFRAGE: INSERT TABLE 
		IF (@TYP = 'ALL' OR @TYP = 'INSERT') AND @table_view = 'TABLE'
		BEGIN
			SET @query_insert = 'INSERT ' + @tablename + ' ('
			SET @query_insert = @query_insert + '' SELECT @query_insert = @query_insert + COLUMN_NAME + ', ' 
						FROM (SELECT DISTINCT COLUMN_NAME, ORDINAL_POSITION FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = '' + @tablename + '') AS T
						order by ORDINAL_POSITION
			SET @query_insert =  LEFT(@query_insert, LEN(@query_insert) - 1) + ') VALUES ('
			SET @query_insert = @query_insert + '' SELECT @query_insert = @query_insert + COLUMN_VALUE + ', ' 
						FROM (SELECT DISTINCT CASE WHEN DATA_TYPE LIKE '%int%' OR DATA_TYPE IN ('bit', 'decimal', 'float', 'image', 'numeric') THEN '1'
												   WHEN DATA_TYPE = 'time' THEN '''14:45:39.000'''
												   WHEN DATA_TYPE = 'datetime' THEN '''1900-01-01 14:45:39.000'''
												   ELSE '''d''' END AS COLUMN_VALUE, ORDINAL_POSITION FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = '' + @tablename + '') AS T
						order by ORDINAL_POSITION
			SET @query_insert =  LEFT(@query_insert, LEN(@query_insert) - 1) + ') '
		------
		print 'ABFRAGE: INSERT TABLE:
		' + @query_insert + CHAR(13);
		INSERT _tmp_TBLINFO (TBLINFO) VALUES (@query_insert)

		END
	END

	IF @SAVE_RESU_IN_TABLE = 0 DROP TABLE _tmp_TBLINFO

END

GO

/* Tests
exec TBLINFO 'TABLE_NAME','STRUKTUR'

exec TBLINFO 'NAME_VIEW'
*/
