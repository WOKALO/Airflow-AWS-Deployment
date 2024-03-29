## Airflow AWS Deployment
Airflow is a platform to programmatically author, schedule and monitor workflows.</br>
Main components will be installed and configured are
- Webserver
- Scheduler
- Workers
- Shared File System
- Queue
- Metadb

### EC2 Configuration</br>
The configuration below is for a test environment burndown. Only a free micro EC2 instance initiated and the drawbacks are memory limitation which cannot start airflow webserver, scheduler, and worker services in the same time. </br>

#### Launch EC2 Instance
Airflow does not support running over windows server. Here, an Linux Instance should be initiated for setup.</br>
Step 1: Select Launch Instance</br>
Pick up Ubuntu server: 18.04LTS (Pre-installed python3.6)</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%201.png)</br>

Step 2: Choose an instance type</br>
Free t2 Micro for testing environment. Normally, a moderate will be required for running large scaled data pipeline jobs. Click on Configure Instance Details.</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%202.png)</br>


Step 3: Configure Instance Details</br>
Auto-Assign Public IP = Enabled to access through public network. Pick subnet if this instance need to be assign to same subnet work with other instances.</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%203.png)</br>


Step 4: Add Storage</br>
It is free to increase the instance storage to 20 GB.</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%204.png)</br>

Step 5: Add Tags</br>
Name for each resources type for easy management.</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%205.png)</br>

Step 6: Configure Security Group</br>
To allow console access and internet access to the airflow webserver. SSH access using My IP address – console access. Customer TCP Rule with port range 8080 –airflow UI access through public IP. </br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%206.png)</br>

Step 7: preview and lunch through windows</br>
Create a private key file which is required to obtain the password used to login into the EC2 instance. And without the file, the instance won’t be able to be accessed.</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%207%20Download%20Key%20Pairs.png)</br>


EC Instance Access PuTTY</br>
PuTTY and PuTTYGen will be used to access EC2 instance via private key file on downloaded above through windows OS. Both are open source free SSH client. More details, see the guide https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html.</br>

## Airflow Installation
This section includes
- Replace SQLite with Postgres to use real database and scale up airflow history execution logging.</br>
- Airflow installation include basic configuration changes.</br>
- Replace local executor with Celery Executor to allow parallel execution of tasks.</br>
- Replace direct access with RBAC to allow role based access control.</br>
- Additional settings to optimize the airflow.</br>

### Postgres SQL Installation
Update existing software package list on the instance.</br>
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%201.png)</br>

Setup Postgres as backend database.</br>
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%202.png)</br>

Create metadata database for airflow backend</br>
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%203.png)</br>

Check database connection info and modify the configuration and the port is 5432.</br>
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%204.png)</br>
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%205.png)</br>

Open pg_hba.conf with the path showing above using vi (text editor) and change the ipv4 address to 0.0.0.0/0 and the ipv4 connection method from md5 (password) to trust which allow no password access Postgres database (About vi).</br>
Quick vi notes: i - insert and change, esc - temp commit changes, :w - save to file, :q - quit</br>
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%206.png)</br>

Configure the postgresql.conf file to open the listen address to all ip addresses, listen_addresses = '*'</br>
Quick vi notes: /search for search the characters to locate the configuration.</br>
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%207.png)</br>

Start Postgres SQL Service </br>
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%208.png)</br>
And any time modify the connection information, reload the postgresql service required. (reload postgres settings without restart the service).</br>
```
sudo service postgresql reload
```

### Airflow Installation
Setup home directory for AIRFLOW_HOME default folder will be created as airflow. (Optional as an environment variable)</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step1.png)

Install pip3 python package installer</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step2.png)

After installing these dependencies, we can install airflow and its packages. (You can modify these packages depending on need. Celery and RabbitMQ are needed to use the Web-based GUI)</br>
```
sudo apt-get install libmysqlclient-dev (dependency for airflow[mysql] package)
sudo apt-get install libssl-dev (dependency for airflow[cryptograph] package)
sudo apt-get install libkrb5-dev (dependency for airflow[kerbero] package)
sudo apt-get install libsasl2-dev (dependency for airflow[hive] package)
sudo pip3 install apache-airflow[async,devel,celery,crypto,druid,gcp_api,jdbc,hdfs,hive,kerberos,ldap,password,postgres,qds,rabbitmq,s3,samba,slack]
```
Setup first-time configurations </br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step3.png)

Backup configurations</br>
```
cd ~/airflow
cp airflow.cfg airflow.cfg.bkp
```
Use vi to change following configurations. Replace SequentialExecutor to CeleryExecutor. </br>
```
sudo vi airflow.cfg
```
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step4.png)

Change sql_alchemy_conn to postgresql+psycopg2://ubuntu@localhost:5432/airflow.</br>
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step5.png)

Change broker_url and result_backend under [celery] section.</br>
result_backend=db+postgresql+psycopg2://ubuntu@localhost:5432/airflow

![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step6.png)

Install pyamqp connection driver.</br>
```
pip3 install pyamqp
```
Recommit airflow changes.
```
airflow initdb
```
Resolve the warning usage of psycopg2.
```
sudo pip3 install psycopg2-binary
airflow initdb
```
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step7.png)

Test airflow webserver, create multiple screen or session to monitoring the process.
```
airflow webserver
```
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step8.png)

Go to web UI, EC2 public ip:8080. Then ctrl + c to stop the service. 
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Airflow_Step9.png)

### Rabbitmq and Celery Installation
RabbitMQ is a message broker. Its job is to manage communication between multiple services by operating message queues. It provides an API for other services to publish and to subscribe to the queues. Celery is a task queue. It can distribute tasks on multiple workers by using a protocol to transfer jobs from the main application to Celery workers. It relies on a message broker to transfer the messages.
```
sudo apt-get install rabbitmq-server
```
Change the configuration file /etc/rabbitmq/rabbitmq-env.conf, and make a copy before change the file. Configuration need to be updated is NODE_IP_ADDRESS=0.0.0.0 and start the service.
```
sudo service rabbitmq-server start
```
Check celery if it is already installed by 
```
pip3 freeze | grep celery
```
### RBAC configuration
There are five roles created for Airflow by default: Admin, User, Op, Viewer, and Public. The master branch adds beta support for DAG level access for RBAC UI. Each DAG comes with two permissions: read and write.

To enable authorized access, the following settings need to be updated under airflow.cfg. 

To create a user, run:
```airflow initdb
sudo airflow create_user -r Admin -u Admin -f Admin -l Admin -e Admin@hqz.com -p Abcd1234*
```

### Additional Configurations
By default, Airflow will try to backfill jobs that it may have missed. This is not the desired behavior. 
catchup_by_default = False
