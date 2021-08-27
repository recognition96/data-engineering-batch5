# Airflow 2.0 Installation:

These instructions are about installing Airflow on a Ubuntu server (not using Docker). We will be using Python 3. Python 2 is sunsetting.


## Airflow Python Module Installation

#### First upgrade apt and install python3 pip

```
sudo apt-get update
sudo apt-get install -y python3-pip
```

#### Next install Airflow and other Python modules we need

```
sudo apt-get install postgresql-server-dev-all
sudo apt-get install postgresql-common
sudo pip3 install apache-airflow
sudo pip3 install apache-airflow-providers-postgres[amazon]
sudo pip3 install cryptography psycopg2-binary boto3 botocore 
```

## airflow:airflow Account Creation

Create a dedicated group and account for Airflow. airflow account will have /var/lib/airflow as its home directory.

```
sudo groupadd airflow
sudo useradd -s /bin/bash airflow -g airflow -d /var/lib/airflow -m
```

## Local Postgres Installation to store Airflow related info (DAGs, Tasks, Variables, Connections and so on)

By default, Airflow will be launced with a SQLite database which is a single thread. Will change this to use a more performant database such as Postgres later.

#### Install Postgres server

```
sudo apt-get install -y postgresql postgresql-contrib
```

#### Next create a user and a database to be used by Airflow to store its data
```
$ sudo su postgres
$ psql
psql (10.12 (Ubuntu 10.12-0ubuntu0.18.04.1))
Type "help" for help.

postgres=# CREATE USER airflow PASSWORD 'airflow';
CREATE ROLE
postgres=# CREATE DATABASE airflow;
CREATE DATABASE
postgres=# \q
$ exit
```

#### Restart Postgres

Move back to Ubuntu account by running "exit"

```
sudo service postgresql restart
```


## Initial Airflow Initialization

#### First install Airflow with the default configuration and will change some configuration

```
sudo su airflow
$ cd ~/
$ mkdir dags
$ AIRFLOW_HOME=/var/lib/airflow airflow db init
$ ls /var/lib/airflow
airflow.cfg  airflow.db  dags   logs  unittests.cfg
```

#### Now edit /var/lib/airflow/airflow.cfg to do the following 3 things:

 * change the "executor" to LocalExecutor from SequentialExecutor
 * change the db connection string ("sql_alchemy_conn") to point to the local Postgres installed above or a new one provided to you
   * If you are going to use a locally installed Postgres DB here (like explained above), use airflow for ID, PASSWORD and DATABASE and use localhost for HOST
 * change Loadexample setting to False
 
```
[core]
...
executor = LocalExecutor
...
sql_alchemy_conn = postgresql+psycopg2://airflow:airflow@localhost:5432/airflow
...
load_examples = False
```

#### Reinitialize Airflow DB

```
AIRFLOW_HOME=/var/lib/airflow airflow db init
```


## Start Airflow Webserver and Scheduler

To start up airflow scheduler and webserver as background services, follow the instructions here. Do this as <b>ubuntu</b> account (<b>not airflow</b>). If you get "[sudo] password for airflow: " error, you are still using airflow as your account. Exit so that you can use "ubuntu" account.


#### Create two files:

sudo vi /etc/systemd/system/airflow-webserver.service

```
[Unit]
Description=Airflow webserver
After=network.target

[Service]
Environment=AIRFLOW_HOME=/var/lib/airflow
User=airflow
Group=airflow
Type=simple
ExecStart=/usr/local/bin/airflow webserver -p 8080
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

sudo vi /etc/systemd/system/airflow-scheduler.service

```
[Unit]
Description=Airflow scheduler
After=network.target

[Service]
Environment=AIRFLOW_HOME=/var/lib/airflow
User=airflow
Group=airflow
Type=simple
ExecStart=/usr/local/bin/airflow scheduler
Restart=on-failure
RestartSec=10s

[Install]
WantedBy=multi-user.target
```

#### Run them as startup scripts. 

```
sudo systemctl daemon-reload
sudo systemctl enable airflow-webserver
sudo systemctl enable airflow-scheduler
```

Start them:

```
sudo systemctl start airflow-webserver
sudo systemctl start airflow-scheduler
```

To check the status of the services, run as follow:

```
sudo systemctl status airflow-webserver
sudo systemctl status airflow-scheduler
```


## Dag Installation from this repo:

The last step is to copy the files under keeyong/data-engineering repo's dags_v2 folder to /var/lib/airflow/dags. Don't forget to add "-r" option in the "cp" command:

```
sudo su airflow
cd ~/
git clone https://github.com/keeyong/data-engineering-batch4.git
cp -r data-engineering-batch4/dags/dags_v2/* dags
```

Visit your Airflow Web UI and we should see the DAGs from the repo. Some will have errors displayed and you need to add some variables and connections according to the slides 23 to 25 and 30 of "Airflow Deep-dive" preso.


## Set AIRFLOW_HOME environment variable in ~/.bashrc (/var/lib/airflow/.bashrc)

Once you switch to *airflow* user (sudo su airflow), execute the bashrc file first,
```
airflow@ip-172-31-54-137:~$ vi ~/.bashrc
```
and append the following lines to the end of the file:
```
AIRFLOW_HOME=/var/lib/airflow
export AIRFLOW_HOME

cd ~/
```

Log out of airflow user and log in again (sudo su airflow). Now you will start from the home directory. Also if you print out AIRFLOW_HOME environment variable, you will see it is set properly.
```
ubuntu@ip-172-31-54-137:~$ sudo su airflow
airflow@ip-172-31-54-137:~/$ pwd
/var/lib/airflow
airflow@ip-172-31-54-137:~/$ echo $AIRFLOW_HOME
/var/lib/airflow
```


## Add authentication to Airflow Webserver

Run the following command line (make sure to change admin to something else):

```
airflow users  create --role Admin --username admin --email admin --firstname admin --lastname admin --password admin
```
