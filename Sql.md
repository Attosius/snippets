
<!-- TOC -->

- [SQLSERVER](#sqlserver)
    - [UPDATE FROM SELECT](#update-from-select)
    - [ALTER](#alter)
    - [IF EXISTS](#if-exists)
    - [Begin try](#begin-try)
    - [Print](#print)
    - [Create](#create)
    - [Инфа о таблице](#инфа-о-таблице)
    - [ARITHABORT Разные планы выполнения в vs и в mms](#arithabort-разные-планы-выполнения-в-vs-и-в-mms)
    - [Вызов хранимки по поиску цен](#вызов-хранимки-по-поиску-цен)
- [PG](#pg)
    - [Pg текущие подключения](#pg-текущие-подключения)
    - [Pg строки](#pg-строки)
    - [wrong timezone in postgress](#wrong-timezone-in-postgress)
    - [pg restore from sql](#pg-restore-from-sql)

<!-- /TOC -->

# SQLSERVER

## UPDATE FROM SELECT

```js
   UPDATE
    Table_A
SET
    Table_A.col1 = Table_B.col1,
    Table_A.col2 = Table_B.col2
FROM
    Some_Table AS Table_A
    INNER JOIN Other_Table AS Table_B
        ON Table_A.id = Table_B.id
WHERE
    Table_A.col3 = 'cool'
    
```

## ALTER

```sql
    ALTER TABLE table_name
	    ADD column_name uniqueidentifier NULL

    ALTER TABLE table_name
        DROP COLUMN column_name;
    
    ALTER TABLE table_name
    Alter COLUMN column_name  int;
```

## IF EXISTS


```sql
	IF (EXISTS ( SELECT *  FROM   sysobjects  WHERE  id = object_id(N'[dbo].[table_name]')))
	BEGIN
		PRINT 'DROP TABLE [dbo].[table_name]'
		DROP TABLE [dbo].[table_name]
	END

    SELECT * 
    FROM sys.indexes 
    WHERE name='index_name' 

```

## Begin try

```sql

    begin try
    begin transaction T;


    raiserror ('text! %s', 20, 101, 'text!') with LOG
    commit transaction T;

    end try
    begin catch

        rollback transaction T;
        select
            @errorMessage = ERROR_MESSAGE()
        raiserror ('Error Occured: %s', 20, 101, @errorMessage) with LOG

    end catch

```

## Print
```sql

    print formatmessage(N'Количество открытых транзакций: %i', @@TRANCOUNT);
    if(@@TRANCOUNT > 0)
    begin
        print N'Есть открытая транзакция. Откатываем...';
        rollback transaction;
        print formatmessage(N'Количество открытых транзакций: %i', @@TRANCOUNT);
    end

```


## Create table

```sql
	PRINT 'CREATE TABLE [dbo].[table_name]'
	CREATE TABLE [dbo].[table_name](
		[Id] [int] IDENTITY(1,1) NOT NULL,
		[SecurityGroupId] [int] NOT NULL,
		[FiasId] [uniqueidentifier] NOT NULL,
		[EditedOn] [DateTime] NULL,
		[EditedBy] [nvarchar](250) NULL,
		[Description] [nvarchar](512) NULL,
		[IsRemoved] [bit]  NOT NULL DEFAULT 0
	 CONSTRAINT [PK_dbo.table_name] PRIMARY KEY CLUSTERED ( [Id] ASC)
	)


	ALTER TABLE [dbo].[table_name]  WITH CHECK ADD  CONSTRAINT [FK_dbo.table_name1_dbo.table_name2] FOREIGN KEY([SecurityGroupId])
	REFERENCES [dbo].[table_name2] ([Id])
	ON DELETE CASCADE

	ALTER TABLE [dbo].[table_name] CHECK CONSTRAINT [FK_dbo.table_name1_dbo.table_name2]

```


## Инфа о таблице
 sp_help table_name



## ARITHABORT Разные планы выполнения в vs и в mms

```sql
 declare @value sql_variant

select @value = SESSIONPROPERTY('ARITHABORT') 
if @value <> 0 
begin 
	PRINT('asdf')
	USE master 
	ALTER DATABASE [table_name] SET ARITHABORT OFF WITH NO_WAIT 
	use [table_name]
end
```

## Вызов хранимки c параметрами

```sql
declare @p3 dbo.HXIDTableType
insert into @p3 values(N'02-001')

declare @p4 dbo.AccountStatuses
insert into @p4 values(N'1')
insert into @p4 values(N'3')
insert into @p4 values(N'5')

exec GetContractCartPrices
		@hxids = @p3,
		@ContractStatus=N'Основной',
		@AccountStatus=@p4,
		@CityCode=N'000000001',
		@PartnerCode=N''

```

# PG

## Pg текущие подключения

```sql
    --DROP DATABASE table_name;


    SELECT pg_terminate_backend(pg_stat_activity.pid)
    FROM pg_stat_activity
    WHERE pg_stat_activity.datname = 'table_name'
    AND pid <> pg_backend_pid();


    SELECT *
    FROM pg_stat_activity
    WHERE pg_stat_activity.datname = 'table_name'
    AND pid <> pg_backend_pid();


    select * from pg_stat_activity 



```
## Pg strings in select query

```sql 

    select 'INSERT INTO @regionsData values (' || 
            quote_literal(substring("KladrPlaincode" from 0 for 3)) || ', ' ||  quote_literal("Id") || ');',  "Id", substring("KladrPlaincode" from 0 for 3) as code, "Officalname", "KladrPlaincode"
    from "Addresses" 
    where "Level" < 2 
    order by code
```

## wrong timezone in postgress

```sql
    CREATE TABLE tz (
    "timestamp" timestamp with time zone
    )
    
    INSERT INTO tz ("timestamp") VALUES 
    ('2017-10-10 12:13:14 Europe/Moscow'), 
    ('2017-10-10 12:13:14 America/Los_Angeles'),
    ('2017-10-10 12:13:14+08')

    SELECT * FROM tz;
```

## pg restore from sql

```s
 psql -U postgres -d address_test_prod -1 -f D:\Work\SQL\pg_backups\prod\Addresses.sql
```