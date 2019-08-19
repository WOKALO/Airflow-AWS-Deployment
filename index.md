## Airflow AWS Deployment
Airflow is a platform to programmatically author, schedule and monitor workflows.
Main components will be installed and configured are
- Webserver
- Scheduler
- Workers
- Shared File System
- Queue
- Metadb

### EC2 Configuration
The configuration below is for a test environment burndown. Only a free micro EC2 instance initiated and the drawbacks are memory limitation which cannot start airflow webserver, scheduler, and worker services in the same time. 

#### Launch EC2 Instance
Airflow does not support running over windows server. Here, an Linux Instance should be initiated for setup.
Step 1: Select Launch Instance
Pick up Ubuntu server: 18.04LTS (Pre-installed python3.6)
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%201.png)

Step 2: Choose an instance type
Free t2 Micro for testing environment. Normally, a moderate will be required for running large scaled data pipeline jobs. Click on Configure Instance Details.
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%202.png)


Step 3: Configure Instance Details
Auto-Assign Public IP = Enabled to access through public network. Pick subnet if this instance need to be assign to same subnet work with other instances.
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%203.png)


Step 4: Add Storage
It is free to increase the instance storage to 20 GB.
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%204.png)

Step 5: Add Tags
Name for each resources type for easy management.
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%205.png)

Step 6: Configure Security Group
To allow console access and internet access to the airflow webserver. SSH access using My IP address – console access. Customer TCP Rule with port range 8080 –airflow UI access through public IP. 
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%206.png)

Step 7: preview and lunch through windows
Create a private key file which is required to obtain the password used to login into the EC2 instance. And without the file, the instance won’t be able to be accessed.
![](https://github.com/WOKALO/Airflow-AWS-Deployment/blob/master/Images/Step%207%20Download%20Key%20Pairs.png)


EC Instance Access PuTTY
PuTTY and PuTTYGen will be used to access EC2 instance via private key file on downloaded above through windows OS. Both are open source free SSH client. More details, see the guide https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/putty.html.

## Airflow Installation
This section includes
- Replace SQLite with Postgres to use real database and scale up airflow history execution logging.
- Airflow installation include basic configuration changes.
- Replace local executor with Celery Executor to allow parallel execution of tasks.
- Replace direct access with RBAC to allow role based access control.
- Additional settings to optimize the airflow.

### Postgres SQL Installation
Update existing software package list on the instance.
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%201.png)

Setup Postgres as backend database.
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%202.png)

Create metadata database for airflow backend
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%203.png)

Check database connection info and modify the configuration and the port is 5432.
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%204.png)
![](https://raw.githubusercontent.com/WOKALO/Airflow-AWS-Deployment/master/Images/Psql%20Step%205.png)

Open pg_hba.conf with the path showing above using vi (text editor) and change the ipv4 address to 0.0.0.0/0 and the ipv4 connection method from md5 (password) to trust which allow no password access Postgres database (About vi).
Quick vi notes: i - insert and change, esc - temp commit changes, :w - save to file, :q - quit
step 6

Configure the postgresql.conf file to open the listen address to all ip addresses, listen_addresses = '*'
Quick vi notes: /search for search the characters to locate the configuration.
step 7


Start Postgres SQL Service
Step 8

And any time modify the connection information, reload the postgresql service required: sudo service postgresql reload (reload postgres settings without restart the service).

### Airflow Installation



You can use the [editor on GitHub](https://github.com/WOKALO/aad.io/edit/master/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/WOKALO/aad.io/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
