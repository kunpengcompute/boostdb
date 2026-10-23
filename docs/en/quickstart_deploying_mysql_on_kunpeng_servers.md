# Quick Start: MySQL Porting

<!-- md-trans-meta sourceCommit=0329bc1cbb5916e473f54b25de21737377ed72cb translatedAt=2026-10-16T05:58:07.145Z pushedAt=2026-10-20T06:44:24.527Z nl=2 -->

This document aims to help developers quickly complete the deployment and basic configuration of MySQL on Kunpeng Arm servers. It covers key steps such as obtaining software packages, configuring the compilation environment, performing the installation, and verifying the installation. It also provides quick-start scenarios for initializing and running a MySQL database and for creating a database and executing basic SQL operations, making it suitable for developers who need to quickly set up a test environment or a production foundation.

## Component Introduction

MySQL is an open-source relational database management system (RDBMS) and one of the most popular relational databases in the industry. It is widely used, especially in web applications, and is developed in C/C++. MySQL stores data in different tables instead of placing all data in a single large repository, which makes it fast and highly flexible. It is small in size, has a low total cost of ownership, and is open-source. Generally, small- and medium-sized websites choose MySQL as their website database. SQL, the language used by MySQL, is the most widely used standardized language for accessing databases.

This document uses MySQL 8.0.25 as an example to guide users in quickly completing the installation, initialization, and first use of MySQL on Kunpeng servers (AArch64 architecture) through source code compilation. This document also applies to MySQL 5.7.27, MySQL 8.0.17, and other MySQL 8.0.*x* versions.

>![NOTE icon](public_sys-resources/icon_note.gif) **NOTE:**
>
> * When using open-source software, comply with the applicable licenses.
> * You need to upgrade MySQL 8.0.16 by applying the patch on the official website. You are advised to directly use MySQL 5.7.27, MySQL 8.0.17, MySQL 8.0.25, or later versions.
> * For more information about MySQL, visit the [official MySQL website](https://www.mysql.com/).

## Quick Installation

### Environment Setup

Before performing operations, ensure that the hardware and software meet the following requirements:

* Hardware: Kunpeng server (Kunpeng 920 processor). For performance testing, use a separate drive for the data directory (one system drive and one data drive). For non-performance testing, you can create the data directory directly on the system drive.

* OS: openEuler 20.03 LTS SP1, openEuler 22.03 LTS SP1, or CentOS 7.6. If a fresh OS installation is required, choose `Minimal Install` and select `Development Tools`; otherwise, many software packages will need to be installed manually.

The verified combinations of MySQL and OS versions are as follows:

|MySQL Version|OS Version|
|---|---|
|MySQL 5.7.27/MySQL 8.0.17|CentOS 7.6/openEuler 20.03 LTS SP1|
|MySQL 8.0.25|openEuler 20.03 LTS SP1/openEuler 22.03 LTS SP1|

### Installation Procedure

#### Obtaining the Software Package

|Software Type|Required/Optional|Package Description|Package Name|How to Obtain|
|---|---|---|---|---|
|Main installation package|Required|MySQL source package (includes Boost headers; the Boost header path must be specified during compilation)|mysql-boost-8.0.25.tar.gz|[Official MySQL website](https://downloads.mysql.com/archives/community/)|
|Compilation tool|Optional|CMake ≥ 3.4.3 (required when the version bundled with the OS does not meet the requirements)|cmake-3.5.2.tar.gz|[Official CMake website](https://cmake.org/files/v3.5/cmake-3.5.2.tar.gz)|
|Compilation tool|Optional|GCC ≥ 5.3.0 (For CentOS 7.6, an upgrade is required; for openEuler, an upgrade is not required.)|gcc-7.3.0.tar.gz|[GNU mirror site](https://mirrors.tuna.tsinghua.edu.cn/gnu/gcc/gcc-7.3.0/gcc-7.3.0.tar.gz)|

If the server has Internet access, you can directly download the MySQL source package.

```bash
cd /home
wget https://downloads.mysql.com/archives/get/p/23/file/mysql-boost-8.0.25.tar.gz --no-check-certificate
```

#### Configuring the Compilation Environment

1. Create the `mysql` user group and user (at the OS level, for permission isolation).

   ```bash
    groupadd mysql
    useradd -g mysql mysql
    passwd mysql
    ```

2. Create a data directory (for non-performance-testing scenarios).

    ```bash
    mkdir -p /data/mysql
    cd /data/mysql
    mkdir data tmp run log relaylog
    chown -R mysql:mysql /data
    ```

    >![NOTE icon](public_sys-resources/icon_note.gif) **NOTE:**
    > When performing performance testing, you need to format a separate drive and mount it to the data directory. For details, see [Creating a Data Drive](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_03_0007.html).

3. Install dependencies.

    ```bash
    yum -y install bison bison-devel ncurses ncurses-devel libaio-devel openssl openssl-devel gmp gmp-devel mpfr mpfr-devel libmpc libmpc-devel libcurl-devel
    yum -y install wget tar gcc gcc-c++ git rpcgen cmake make
    yum -y install numactl numactl-devel*
    yum -y install libtirpc libtirpc-devel m4
    yum -y install zstd-devel
    ```

4. If the CMake or GCC version does not meet the requirements, upgrade it first. (The GCC bundled with openEuler meets the requirements. In this case, the GCC upgrade can be skipped.)

    * Upgrade CMake to 3.4.3 or later (this document uses 3.5.2 as an example): [Upgrading CMake](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0006.html)

    * On CentOS 7.6, upgrade GCC to 5.3.0 or later (this document uses 7.3.0 as an example): [Installing GCC](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0007.html)

#### Performing Installation

1. Decompress the source package, enter the source code directory, and create a compilation directory.

    ```bash
    cd /home
    tar -zxvf mysql-boost-8.0.25.tar.gz
    cd /home/mysql-8.0.25
    mkdir build
    ```

2. Enter the compilation directory and configure MySQL.

    ```bash
    cd build
    cmake .. -DBUILD_CONFIG=mysql_release -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql/data -DWITH_BOOST=/home/mysql-8.0.25/boost/boost_1_73_0
    ```

    Key parameter description:

    |Parameter|Description|
    |---|---|
    |DBUILD_CONFIG|Set to `mysql_release`, indicating that the compilation parameters in the official MySQL release are used for the CMake compilation command.|
    |DCMAKE_INSTALL_PREFIX|Installation path of the software. In this document, the installation path is `/usr/local/mysql`. Replace it as required.|
    |DMYSQL_DATADIR|Path where data files are stored during database creation. In this document, the path is `/data/mysql/data`.|
    |DWITH_BOOST|Path to the `boost_1_73_0` folder in the decompressed MySQL source package. In this document, the package is decompressed to the `/home` directory, so the path is `/home/mysql-8.0.25/boost/boost_1_73_0`.|

3. Compile and install MySQL.

    ```bash
    make -j 96
    make -j 96 install
    ```

    >![NOTE icon](public_sys-resources/icon_note.gif) **NOTE:**
    > The `-j 96` parameter fully utilizes multiple CPU cores to speed up compilation. The number following `-j` indicates the number of CPU cores, which can be checked using `cat /proc/cpuinfo | grep processor | wc -l`. This value should be less than or equal to the total number of CPU cores.
    > If the compilation and installation fail, run `rm -rf /home/mysql-8.0.25` to clean up the environment, decompress the MySQL source package again, and then compile and install it.

### Installation Verification

1. View the MySQL installation directory.

    ```bash
    ls /usr/local/mysql/
    ```

    The expected output is as follows:

    ```output
    bin  docs  include  lib  LICENSE  LICENSE.router  LICENSE-test  man  mysqlrouter-log-rotate  mysql-test  README  README.router  README-test  run  share  support-files  var
    ```

2. View the database version.

    ```bash
    /usr/local/mysql/bin/mysql --version
    ```

    The expected output is as follows:

    ```output
    /usr/local/mysql/bin/mysql  Ver 8.0.25 for Linux on aarch64 (Source distribution)
    ```

## Quick Start

### Scenario A: Initializing and Running the MySQL Database

#### Scenario Description

This section describes how to modify the configuration file, set the environment variable, initialize the database, start the database, and complete the first login after MySQL 8.0.25 has been compiled and installed, helping users quickly complete the first run of MySQL on a Kunpeng server.

#### Procedure

1. Modify the configuration file.

    ```bash
    rm -f /etc/my.cnf
    echo -e "[mysqld_safe]\nlog-error=/data/mysql/log/mysql.log\npid-file=/data/mysql/run/mysqld.pid\n[mysqldump]\nquick\n[mysql]\nno-auto-rehash\n[client]\ndefault-character-set=utf8\n[mysqld]\nbasedir=/usr/local/mysql\nsocket=/data/mysql/run/mysql.sock\ntmpdir=/data/mysql/tmp\ndatadir=/data/mysql/data\ndefault_authentication_plugin=mysql_native_password\nport=3306\nuser=mysql\n" > /etc/my.cnf
    ```

    ```bash
    cat /etc/my.cnf
    chown mysql:mysql /etc/my.cnf
    ```

    >![NOTE icon](public_sys-resources/icon_note.gif) **NOTE:**
    > Replace the file paths (including the software installation path `basedir`, the data path `datadir`, etc.) as required. `user=mysql` refers to the user at the OS level, that is, the user created in section 2.2.2. If performance tuning is required, see *Modifying Database Parameters* to modify the configuration parameters in `my.cnf`.

2. Configure the MySQL environment variable.

    ```bash
    echo 'export PATH=$PATH:/usr/local/mysql/bin' >> /etc/profile
    source /etc/profile
    ```

    Run the following command to confirm that the environment variable has taken effect:

    ```bash
    mysql --version
    ```

3. Initialize the database.

    ```bash
    chmod 755 /data/mysql/data/
    su - mysql
    mysqld --defaults-file=/etc/my.cnf --initialize
    ```

    The expected output (partial) is as follows:

    ```output
    [Note] [MY-010454] [Server] A temporary password is generated for root@localhost: t<7Xtw3adX>B
    [System] [MY-013170] [Server] /usr/local/mysql/bin/mysqld (mysqld 8.0.25) initializing of server has completed
    ```

    >![NOTE icon](public_sys-resources/icon_note.gif) **NOTE:**
    >
    > * After initialization, a temporary initial password is generated following `root@localhost:` in the output. Save it, as it will be used in step 5.
    > * If initialization fails and the message "--initialize specified but the data directory has files in it." is displayed, run `ls /data/mysql/data` to check the directory contents, run `rm -rf /data/mysql/data/*` to delete the data, and then re-initialize the database.

4. Start the database.

    ```bash
    /usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf &
    ```

    Check the database process and listening port.

    ```bash
    ps -ef | grep mysql
    netstat -anpt | grep 3306
    ```

    >![NOTE icon](public_sys-resources/icon_note.gif) **NOTE:**
    >
    > * If the first execution of `service mysql start` as the `root` user fails (prompting that the `mysql.log` file is missing), switch to the `mysql` user (`su - mysql`) to start the database service and generate `mysql.log`, stop the service, and then start it again as the `root` user.
    > * If the `netstat` command fails, run `yum -y install net-tools` first to install the dependency package.

5. Log in to the database and change the initial password.

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

    When prompted for a password, enter the temporary initial password generated in step 3. After a successful login, change the `root` user password and create a `root` user that can connect from other servers.

    ```sql
    alter user 'root'@'localhost' identified by '123456';
    create user 'root'@'%' identified by '123456';
    grant all privileges on *.* to 'root'@'%';
    flush privileges;
    ```

    The expected output is as follows:

    ```output
    Query OK, 0 rows affected (0.01 sec)
    ```

    >![NOTE icon](public_sys-resources/icon_note.gif) **NOTE:**
    > The users and passwords in this document are for reference only. Configure them according to the actual customer environment.  
    > **Security note**: `123456` is a weak password used only for demonstration. In production environments, you must use a strong password (12 or more characters recommended, including uppercase and lowercase letters, digits, and special characters). In addition, avoid passing plaintext passwords directly in command-line parameters (such as `mysql -uroot -p123456`) to prevent passwords from being leaked in shell command history.

6. Log in again with the new password for verification.

    Run `exit` to exit the database, and then log in again.

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

    The expected output (partial) is as follows:

    ```output
    Welcome to the MySQL monitor.  Commands end with ; or \g.
    Your MySQL connection id is 8
    Server version: 8.0.25
    ```

#### Result Description

You have successfully logged in to the database with the new password, and the `mysql>` command-line prompt appears, indicating that MySQL has completed initialization and is running normally, with the database listening on port 3306.

> **NOTE:**
> To shut down the database, use the command that corresponds to the method used to start it. For a database started with `service mysql start`, use `service mysql stop` to shut it down (you can run `service mysql restart` to restart it); for a database started with `mysqld` or `mysqld_safe`, you can run `mysqladmin -uroot -p123456 shutdown -S /data/mysql/run/mysql.sock` to shut it down.

### Scenario B: Creating a Database and Executing Basic SQL Operations

#### Scenario Description

This section uses an example of creating a test database and performing basic table creation, insertion, and query operations to help users quickly experience the basic usage of MySQL.

#### Procedure

1. Log in to the database.

    ```bash
    mysql -uroot -p -S /data/mysql/run/mysql.sock
    ```

2. Create a database and a data table.

    ```sql
    create database testdb;
    use testdb;
    create table users (id int primary key auto_increment, name varchar(32) not null, created_at timestamp default current_timestamp);
    ```

    The expected output is as follows:

    ```output
    Query OK, 1 row affected (0.01 sec)
    Database changed
    Query OK, 0 rows affected (0.02 sec)
    ```

    Description of each column attribute in the table creation statement: `id int primary key auto_increment` indicates an integer primary key that auto-increments; `name varchar(32) not null` indicates a string of up to 32 characters that cannot be null; `created_at timestamp default current_timestamp` indicates a timestamp column that defaults to the current time.

3. Insert and query data.

    ```sql
    insert into users (name) values ('kunpeng');
    select * from users;
    ```

    The expected output is as follows:

    ```output
    Query OK, 1 row affected (0.00 sec)

    +----+---------+---------------------+
    | id | name    | created_at          |
    +----+---------+---------------------+
    |  1 | kunpeng | 2026-08-27 10:00:00 |
    +----+---------+---------------------+
    1 row in set (0.00 sec)
    ```

#### Result Description

The data is successfully written and can be queried normally, indicating that the basic MySQL functions such as database creation, table creation, insertion, and querying run properly on the Kunpeng server.

## FAQs

The following frequently asked questions (FAQs) are extracted from the troubleshooting section of the porting guide. For detailed solutions, see the corresponding linked documents.

**Question 1: What should I do if "--initialize specified but the data directory has files in it." is displayed during database initialization?**

Answer: Files already exist in the data directory, causing initialization to fail. Run `ls /data/mysql/data` to check the directory contents, run `rm -rf /data/mysql/data/*` to clear the data directory, and then run the initialization command again.

**Question 2: What should I do if the first startup with `service mysql start` as the `root` user fails and reports that the `mysql.log` file is missing?**

Answer: Switch to the `mysql` user (`su - mysql`) to start the database service. The system will generate the `mysql.log` file in the `/data/mysql/log` directory. Then stop the database service (`/usr/local/mysql/bin/mysqladmin -u root -p shutdown`), and start it again as the `root` user.

**Question 3: What should I do if the `netstat` command is not found?**

Answer: The net-tools dependency package is missing. Run `yum -y install net-tools` to install it and try again.

**Question 4: What should I do if the new CMake version does not take effect after CMake is upgraded?**

Answer: See [New CMake Version Does Not Take Effect After CMake Is Upgraded](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0010.html) in the troubleshooting section.

**Question 5: What should I do if the CMake command fails during the compilation and installation of MySQL?**

Answer: See [CMake Command Execution Failure During MySQL Compilation and Installation](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0011.html) in the troubleshooting section.

## 5. More Functions

For more scenarios of using MySQL on Kunpeng servers, see the following documents:

* [MySQL Porting Guide (including compilation troubleshooting)](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0001.html)

* [New CMake Version Does Not Take Effect After CMake Is Upgraded](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0010.html)

* [CMake Command Execution Failure During MySQL Compilation and Installation](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengmysql8017_02_0011.html)

* [MySQL Best Practices of Migration: x86-to-Kunpeng migration with primary/secondary replication maintained](https://www.hikunpeng.com/document/detail/en/kunpengdbs/ecosystemEnable/MySQL/kunpengdbs_mysql_bestpractices_21_0001.html)

* [Official MySQL website](https://www.mysql.com/)

## Change History

|Version | Date | Description |
| --------| -------- | -------- |
|01| 2026-09-30 | This is the first official release. |
