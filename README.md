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

if above command is error, then change the url in this by finding the latest release from [download page](https://spark.apache.org/downloads.html). Then try to run `wget {new_hadoop_download_link}` and redo the extract below

```bash
tar xvf spark-3.5.1-bin-hadoop3.tgz
```

## Configure Spark Environment

```bash
cd spark-3.5.1-bin-hadoop3 && pwd && cd ..
```
Then, copy the hadoop_path and save to your note

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

copy the path (`/usr/share/java/`) and copy the mysql-connector-j-{version}.jar, and save to note with this format: `/usr/share/java/mysql-connector-j-{version}.jar`

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
GRANT ALL PRIVILEGES ON *.* TO 'your_username'@'localhost' WITH GRANT OPTION;
```
```mysql
quit
```

## Try to access your user just create:
```bash
mysql -u {your_username} -p
```

enter the password `{your_password}`

## Your next objectives is: Create Database and create table with dummy data
### Create Database
```mysql
CREATE DATABASE {database_name};
```

### User Database
```mysql
USE {database_name};
```

## Create Table

```mysql
CREATE TABLE {table_name} (
    id INT AUTO_INCREMENT PRIMARY KEY,
    date DATE,
    temperature FLOAT,
    precipitation FLOAT,
    humidity FLOAT
);
```

## Insert Table

```sql
INSERT INTO {table_name} (date, temperature, precipitation, humidity) VALUES
('2024-05-01', 75.2, 0.3, 60),
('2024-05-02', 74.8, 0.5, 62),
('2024-05-03', 76.5, 0.2, 58),
('2024-05-04', 78.3, 0.1, 55),
('2024-05-05', 77.6, 0.4, 63),
('2024-05-06', 76.9, 0.6, 59),
('2024-05-07', 74.5, 0.8, 61),
('2024-05-08', 73.2, 0.3, 57),
('2024-05-09', 72.8, 0.2, 54),
('2024-05-10', 71.4, 0.1, 56),
('2024-05-11', 70.9, 0.4, 60),
('2024-05-12', 69.8, 0.5, 58),
('2024-05-13', 71.5, 0.3, 62),
('2024-05-14', 73.2, 0.2, 59),
('2024-05-15', 74.6, 0.6, 55),
('2024-05-16', 75.3, 0.7, 57),
('2024-05-17', 76.7, 0.5, 61),
('2024-05-18', 77.2, 0.3, 58),
('2024-05-19', 78.4, 0.2, 60),
('2024-05-20', 79.1, 0.4, 63);
```

## Quit mysql

```sql
quit
```

## Create .py Pyspark File:
```bash
nano {your_files}.py
```


## Input this script into your .py:

```python
from pyspark.sql import SparkSession

# Define connection parameters
database_name = "{your_database_name}"
table_name = "{your_table_name}"
username = "{your_username}"
password = "{your_password}"
mysql_version = "{mysql_version}"  # e.g., "8.0"

# Create a SparkSession
spark = SparkSession.builder \
    .appName("PySpark MySQL Connection") \
    .config("spark.jars", f"/usr/share/java/mysql-connector-java-{mysql_version}.jar") \
    .getOrCreate()

# Define JDBC connection properties
jdbc_url = f"jdbc:mysql://localhost:3306/{database_name}"
jdbc_properties = {
    "driver": "com.mysql.cj.jdbc.Driver",
    "url": jdbc_url,
    "query": f"SELECT * FROM {table_name}",
    "user": username,
    "password": password
}

# Load data from MySQL using JDBC
df = spark.read.format("jdbc").options(**jdbc_properties).load()

# Show the DataFrame
df.show()

```


## Command to spark submit (copy all command, and make sure your jar version and your_files.py)
*Be careful with `/usr/share/java/mysql-connector-j-{version}.jar` and `{your_files.py}`
```bash
spark-submit --master local[*] --jars /usr/share/java/mysql-connector-j-{version}.jar --executor-cores 2 --executor-memory 4g --driver-memory 4g --driver-cores 2 --executor-memory 2g --executor-cores 1 {your_files.py}
```
