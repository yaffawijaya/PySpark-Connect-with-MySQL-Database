# PySpark Connect MySQL - Don't Forget to Star this Repo!!!!!

## Requirements

### Azure Virtual Machine (specification and configuration):
- **VM Specifications**: 2 vCPUs and 8 GB of RAM for PySpark job
- **Operating System**: Linux (Ubuntu Server 22.04 LTS x64 Gen2)
- **Authentication Type**: Password
- **Inbound Port**: SSH (22)

## Working Environment
Connect SSH from VM via terminal/powershell/cmd using `ssh {username}@{public_ip}` then input the password

## Installation Python 3.8 and Virtual Environment Guides

```bash
sudo apt update
sudo apt upgrade
sudo add-apt-repository ppa:deadsnakes/ppa -y
sudo apt install python3.8
python3.8 --version
```

## Create Python Virtual Environment

```bash
sudo apt-get install python3-pip
sudo pip3 install virtualenv
virtualenv {your_env_name} -p python3.8
source {your_env_name}/bin/activate
```

## Install Java

```bash
sudo apt update
sudo apt install default-jdk -y
java --version
```

## Install Apache Spark

```bash
sudo apt install curl mlocate git scala -y
wget https://dlcdn.apache.org/spark/spark-3.5.1/spark-3.5.1-bin-hadoop3.tgz
tar xvf spark-3.5.1-bin-hadoop3.tgz
```

if above command is error, then change the url in this by finding the latest release from [download page](https://spark.apache.org/downloads.html). Then try to run `wget {new_hadoop_download_link}` and do the extract as the file name like below.

(optional, and don't do it twice)
```bash
tar xvf spark-3.5.1-bin-hadoop3.tgz
```

## Configure Spark Environment

```bash
cd spark-3.5.1-bin-hadoop3 && pwd && cd ..
```
Then, copy the hadoop_path (`your/hadoop/path/spark-3.5.1-bin-hadoop3`) and save to your note

```bash
sudo nano ~/.bashrc
```

Then, paste hadoop_path into bashrc configuration:
```bash
export SPARK_HOME={hadoop_path}

export PATH=$PATH:$SPARK_HOME/bin

export SPARK_LOCAL_IP=localhost

export PYSPARK_PYTHON=/usr/bin/python3.8

export PYTHONPATH=$(ZIPS=("$SPARK_HOME"/python/lib/*.zip); IFS=:; echo "${ZIPS[*]}"):$PYTHONPATH
```

## Source the bashrc
```bash
source ~/.bashrc
```

## Test the Spark and PySpark
Test `spark-shell` installation:
```bash
spark-shell
```
Test `pyspark` installation:
```bash
pyspark
```

If no error, you good to go!

## Install mysql-connector and Depackage

```bash
wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-j_8.4.0-1ubuntu22.04_all.deb
```
```bash
sudo dpkg -i mysql-connector-j_8.4.0-1ubuntu22.04_all.deb
```

## Spark Jar Configuration Path
```bash
cd /usr/share/java/ && pwd && ls && cd ~/
```

Copy the path (`/usr/share/java/`) and copy the mysql-connector-j-{version}.jar, and save to note with this format: `/usr/share/java/mysql-connector-j-{version}.jar`. Let's say this path is mysql_connector_path.

## Install MySQL
```bash
sudo apt update
```
```bash
sudo apt upgrade
```
```bash
sudo apt install mysql-server
```
```bash
`mysqld --version`
```

## Set up mysql
```bash
sudo mysql -u root
```
```mysql
CREATE USER '{your_username}'@'localhost' IDENTIFIED BY '{your_password}';
```
```mysql
GRANT ALL PRIVILEGES ON *.* TO '{your_username}'@'localhost' WITH GRANT OPTION;
```
```mysql
quit
```

## Try to access your user just create:
```bash
mysql -u {your_username} -p
```

enter the password: `{your_password}`

## Your next objectives is: Create Database and create table with dummy data
### Create Database
```mysql
CREATE DATABASE {database_name};
```

### Use Database
```mysql
USE {database_name};
```

### Create Table

```mysql
CREATE TABLE {table_name} (
    id INT AUTO_INCREMENT PRIMARY KEY,
    column1 DATE,
    column2 FLOAT,
    column3 FLOAT,
    column4 FLOAT
);
```

### Insert Table

```sql
INSERT INTO {table_name} (column1, column2, column3, column4) VALUES
('2024-05-01', 75.2, 0.3, 60),
(make your own dummy data),
('2024-05-20', 79.1, 0.4, 63);
```

### Quit MySQL

```sql
quit
```

## Create .py Pyspark File:
```bash
nano {your_files}.py
```


## Input this script into your {your_files}.py:
```python
from pyspark.sql import SparkSession

# Create a SparkSession
spark = SparkSession.builder.appName("PySpark MySQL Connection").config("spark.jars", "{mysql_connector_path}").getOrCreate()


# Load data from MySQL using JDBC
df = spark.read.format("jdbc").option("driver","com.mysql.cj.jdbc.Driver").option("url", "jdbc:mysql://localhost:3306/{database_name}").option("query", "SELECT * FROM {table_name}").option("user", "{your_username}").option("password", "{your_password}").load()

# Show the DataFrame
df.show()

```
### Make Sure:
#### Attention:
*Important Note: `{mysql_connector_path}` is `/usr/share/java/mysql-connector-j-{version}.jar`!
#### Note:
*Note: Be careful with `{mysql_connector_path}` and `{your_files.py}`

## Command to spark submit (copy all command, and make sure your jar version and your_files.py)

```bash
spark-submit --master local[*] --jars {mysql_connector_path} --executor-cores 2 --executor-memory 4g --driver-memory 4g --driver-cores 2 --executor-memory 2g --executor-cores 1 {your_files.py}
```

### Make Sure:
#### Attention:
*Important Note: `{mysql_connector_path}` is `/usr/share/java/mysql-connector-j-{version}.jar`!
#### Note:
*Note: Be careful with `{mysql_connector_path}` and `{your_files.py}`

# And you are all set! Congratulation!

## References:
[How to Install Python 3.8 on Ubuntu 22.04 or 20.04](https://www.linuxcapable.com/install-python-3-8-on-ubuntu-linux/)
[Creation of virtual environments](https://docs.python.org/3.8/library/venv.html)
[Install Apache Spark and run PySpark in Ubuntu 22.04](https://dev.to/kinyungu_denis/to-install-apache-spark-and-run-pyspark-in-ubuntu-2204-4i79)
[How to Install MySQL on Ubuntu 22.04](https://phoenixnap.com/kb/install-mysql-ubuntu-22-04)
[Download mysql-connector](https://dev.mysql.com/downloads/connector/j/)
[PySpark Read and Write MySQL Database Table](https://sparkbyexamples.com/pyspark/pyspark-read-and-write-mysql-database-table/)
[My Friend Agung's Way of MySQL Connection Installation](https://youtu.be/oFhrE6g4pbQ?si=kh7fFBG_BGZWCJi3)
