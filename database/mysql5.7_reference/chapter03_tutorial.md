# Chapter 3 Tutorial

如何使用`mysql`Client端程序创建和使用简单数据库？

- `mysql`交互式程序连接到MySQL服务器，运行查询并查看结果；
- `mysql`在批处理模式下使用：将查询事先放在文件中，然后告诉`mysql`执行文件的内容。

要查看`mysql`提供的帮助信息，请使用`--help`选项调用它：

```shell
shell> mysql --help
```

## Connecting to and Disconnecting from the Server

```shell
# 本地登陆可忽略主机：mysql -u user -p
# 匿名(未命名)用户身份连接：mysql
shell> mysql -h host -u user -p
Enter password: ********
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 25338 to server version: 5.7.32-standard

Type 'help;' or '\h' for help. Type '\c' to clear the buffer.

# 在Unix上，可通过Control+D断开连接
mysql> QUIT
Bye
```

The following table shows each of the prompts you may see and summarizes what they mean about the state that mysql is in:

```text
mysql>	Ready for new query
->	Waiting for next line of multiple-line query
'>	Waiting for next line, waiting for completion of a string that began with a single quote (')
">	Waiting for next line, waiting for completion of a string that began with a double quote (")
`>	Waiting for next line, waiting for completion of an identifier that began with a backtick (`)
/*>	Waiting for next line, waiting for completion of a comment that began with /*
```

## Creating and Using a Database

```shell
# To find out what databases currently exist on the server:
mysql> SHOW DATABASES;
+----------+
| Database |
+----------+
| mysql    |
| test     |
| tmp      |
+----------+
```

The `mysql` database describes user access privileges. The `test` database often is available as a workspace for users to try things out. If the `test` database exists, try to access it:

```shell
# USE, like QUIT, does not require a semicolon. 
# The USE statement is special in another way, too: it must be given on a single line.
mysql> USE test
Database changed
```

Suppose that you want to call yours `menagerie`(your own database). The administrator needs to execute a statement like this:

```shell
mysql> GRANT ALL ON menagerie.* TO 'your_mysql_name'@'your_client_host';
```

### Creating and Selecting a Database

```shell
# To create a database:
mysql> CREATE DATABASE menagerie;
# Under Unix, database names are case-sensitive (unlike SQL keywords)
```

Your database needs to be created only once, but you must select it for use each time you begin a mysql session. You can do this by issuing a `USE` statement as shown in the example. Alternatively, you can select the database on the command line when you invoke mysql. Just specify its name after any connection parameters that you might need to provide. For example:

```shell
# menagerie in the command just shown is not your password.
# If you want to supply your password on the command line after the -p option,
# you must do so with no intervening space (for example, as -ppassword, not as -p password).
shell> mysql -h host -u user -p menagerie
Enter password: ********
```

Also, you can see at any time which database is currently selected using `SELECT DATABASE()`.

### Creating a Table

The harder part is deciding what the structure of your database should be: what tables you need and what columns should be in each of them.

Use a `CREATE TABLE` statement to specify the layout of your table:

```shell
# VARCHAR is a good choice for the name, owner, and species columns because the column values vary in length.
# The lengths in those column definitions need not all be the same, and need not be 20.
mysql> CREATE TABLE pet (name VARCHAR(20), owner VARCHAR(20),
       species VARCHAR(20), sex CHAR(1), birth DATE, death DATE);
```

To verify that your table was created the way you expected, use a `DESCRIBE` statement:

```shell
mysql> DESCRIBE pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
```

### Loading Data into a Table

After creating your table, you need to populate it. The `LOAD DATA` and `INSERT` statements are useful for this.

Suppose that your pet records can be described as shown here:

| name     | owner  | species | sex  | birth      | death      |
| -------- | ------ | ------- | ---- | ---------- | ---------- |
| Fluffy   | Harold | cat     | f    | 1993-02-04 |            |
| Claws    | Gwen   | cat     | m    | 1994-03-17 |            |
| Buffy    | Harold | dog     | f    | 1989-05-13 |            |
| Fang     | Benny  | dog     | m    | 1990-08-27 |            |
| Bowser   | Diane  | dog     | m    | 1979-08-31 | 1995-07-29 |
| Chirpy   | Gwen   | bird    | f    | 1998-09-11 |            |
| Whistler | Gwen   | bird    |      | 1997-12-09 |            |
| Slim     | Benny  | snake   | m    | 1996-04-29 |            |

An easy way to populate it is to create a text file containing a row for each of your animals. You could create a text file `pet.txt` containing one record per line, with values separated by tabs, and given in the order in which the columns were listed in the `CREATE TABLE` statement. For missing values (such as unknown sexes or death dates for animals that are still living), you can use `NULL` values. To represent these in your text file, use `\N` (backslash, capital-N). For example, the record for Whistler the bird would look like this (where the whitespace between values is a single tab character):

```text
Whistler        Gwen    bird    \N      1997-12-09      \N
```

To load the text file `pet.txt` into the pet table, use this statement:

```shell
# On Windows, you would likely want to use LINES TERMINATED BY '\r\n'.
# On an Apple machine running macOS, you would likely want to use LINES TERMINATED BY '\r'.
mysql> LOAD DATA LOCAL INFILE '/path/pet.txt' INTO TABLE pet;
```

When you want to add new records one at a time, the `INSERT` statement is useful:

```shell
mysql> INSERT INTO pet
       VALUES ('Puffball','Diane','hamster','f','1999-03-30',NULL);
```

### Retrieving Information from a Table

The `SELECT` statement is used to pull information from a table. The general form of the statement is:

```shell
SELECT what_to_select
FROM which_table
WHERE conditions_to_satisfy;
```

- `what_to_select` indicates what you want to see. This can be a list of columns, or `*` to indicate “all columns.”
- `which_table` indicates the table from which you want to retrieve data.
- The `WHERE` clause is optional. If it is present, `conditions_to_satisfy` specifies one or more conditions that rows must satisfy to qualify for retrieval.

## Getting Information About Databases and Tables

```shell
# To find out which database is currently selected:
mysql> SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| menagerie  |
+------------+

# To find out what tables the default database contains:
mysql> SHOW TABLES;
+---------------------+
| Tables_in_menagerie |
+---------------------+
| event               |
| pet                 |
+---------------------+

# To find out about the structure of a table:
mysql> DESCRIBE pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
# DESC is a short form of DESCRIBE.
```

## Using mysql in Batch Mode

In the previous sections, you used `mysql` interactively to enter statements and view the results. You can also run `mysql` in batch mode. To do this, put the statements you want to run in a file, then tell `mysql` to read its input from the file:

```shell
shell> mysql < batch-file

# shell> mysql -h host -u user -p < batch-file
# Enter password: ********
```

When you use `mysql` this way, you are creating a script file, then executing the script.

If you want the script to continue even if some of the statements in it produce errors, you should use the `--force` command-line option.

