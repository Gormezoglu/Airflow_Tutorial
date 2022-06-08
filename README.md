# Airflow & InfluxDB on Google Cloud Platform (GCP) Walkthrough
In this study, I will try to explain to set airflow jobs on Google Cloud Platform (GCP) step-by-step.


- New VM instance created for influxdb (we choose influxdb since it provides system metrics in default.)
    - name &rarr; influxdb-study
    - region &rarr; us-central1(Iowa) | Zone &rarr; us-central1-a 
    - e2-standart-2 would be sufficient
    - Under BootDisk -> 
        - operating system : Ubuntu
        - Version : Ubuntu 18.04 LTS (bionic version)
        - Boot disk type: SSD persistent disk (recommended to using Influx)
        - Disk size: 20GB (for init)
    - Firewall:
        - Allow HTTP & HTTPS traffic ( for development purpose only. In production, take in consideration of security constraints.)
    - Access scopes : Allow full access to all Cloud APIs
    - Then hit "Crate"
    - In VPC Networks, allow the belowmentioned port numbers to being allowed to use these services.  
        - Influx default port # : 8086
        
            - Crate firewall rule 
                - name: influx-db
                - logs: off -> prevent additional cost
                - Source IPv4 ranges : 0.0.0.0/0 &rarr; All IP address
                - Protocols and ports: specified protocols and ports -> tcp: 8086 (influx) 
                - hit "Create"

- New VM instance created for Airflow 
    - name &rarr; airflow-study
    - region &rarr; us-central1(Oregon) | Zone &rarr; us-west1-b 
    - machine type: e2-standart-2 
    - Under BootDisk -> 
        - operating system : Ubuntu
        - Version : Ubuntu 18.04 LTS (bionic version)
        - Boot disk type: SSD persistent disk (recommended to using Influx)
        - Disk size: 20GB (for init)
    - Firewall:
        - Allow HTTP & HTTPS traffic ( for development purpose only. In production, take in consideration of security constraints.)
    - Access scopes : Allow full access to all Cloud APIs
    - Then hit "Crate"
    - In VPC Networks, allow the belowmentioned port numbers to being allowed to use these services.  
        
        - airflow default port # : 8080
            - Crate firewall rule 
                - name: influx-db
                - logs: off -> prevent additional cost
                - Source IPv4 ranges : 0.0.0.0/0 &rarr; All IP address
                - Protocols and ports: specified protocols and ports -> tcp: 8080 (airflow)
                - hit "Create"
- In order to set fixed IP adresses of these service: VPC Network -> IP adresses -> Reserve External Static Address
    - Region must be the same where VM created.  
    - Type of these address change to *Ephemeral* to *Static*   

 
- Connect via SSH to influx-study . 
    - `apt-upgrade` to update the VM 
    - Then `sudo su` command to install these services.

- Install Influxdb 
    - using documentation to install influx on our Linux Ubuntu installed VM.
        -  `wget https://dl.influxdata.com/influxdb/releases/influxdb2-2.2.0-amd64.deb`  (to download)
        -  `sudo dpkg -i influxdb2-2.2.0-amd64.deb` (to open the file)
            - `systemctl status influxdb` (shows the status of influx service)
            - `systemctl start influxdb`  (starts the service) 

- Install Telegraf (to get system metrics as sample data) 
    - using documentation to install Telegraf on our Linux Ubuntu installed VM.
        -  `wget -qO- https://repos.influxdata.com/influxdb.key | sudo tee /etc/apt/trusted.gpg.d/influxdb.asc >/dev/null`
        -  `echo "deb https://repos.influxdata.com/debian stable main" | sudo tee /etc/apt/sources.list.d/influxdb.list`
        -  `sudo apt-get update && sudo apt-get install telegraf`
            - `systemctl status influxdb` (shows the status of influx service)
            - `systemctl start influxdb`  (starts the service)

    - to access telegraf use external IP of influxdb VM and append port number as `:8086`
        - get started
            - username: admin
            - password: password
            - Initial Org: xyzcorp
            - Initial Bucket Name: Telegraf
        - Quick Start
            - Data
                - API Tokens
                    - admin's token (to get info for telegraf.conf file)
                - 
               
        
    - Configure Telegraf to connect with our influxdb
        - `cd /etc/telegraf/`
        - `nano telegraf.conf`
            -  we edit parameters under *outputs.influxdb_v2*
                -  remove hashtags on:
                    -  urls
                        - VM's external IP instead of localhost IP (127.0.0.1)  
                    -  token
                        - admin's token   
                    -  organization 
                        - `organization = "xyzcorp"`  
                    -  bucket
                        - `bucket = "telegraf"`    
                    -  timeout
                        - `timeout = "10s"` 
                    -  user_agent
                        - `user_agent= "telegraf"`  


## Airflow stage

- Connect via SSH to airflow-study . 
    - `apt-upgrade` to update the VM 
    - Then `sudo su` command to install these services.

- Check if there are any docker instance on VM : `docker ps` 
- Go root folder: `cd /`
- make directory for Airflow: `mkdir airflow`
- enter airflow directory: `cd airflow`
- Install Docker Engine: `https://docs.docker.com/engine/install/ubuntu/`
    - to root folder: `cd /`
    - install docker engine:
    ` $ sudo apt-get install \
    ca-certificates \
    curl \
    gnupg \
    lsb-release`
    -  

    - GPG key :  `sudo mkdir -p /etc/apt/keyrings`
 `curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg` ( sudo mkdir -p /etc/apt/keyrings kısmından emin değilim)!!
 
    - to set up repository : `echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`
 
     - Install Docker Engine : `sudo apt-get update` ` sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin`
     - Check for successfull installation : `docker ps`
     - Check installed docker version : `docker version`
     - List the available repos : `apt-cache madison docker-ce` (result will be entered in version_string)
     - Install indicated version :  *sudo apt-get install docker-ce=<VERSION_STRING> docker-ce-cli=<VERSION_STRING> containerd.io docker-compose-plugin*
        - `sudo apt-get install docker-ce=5:20.10.16~3-0~ubuntu-jammy docker-ce-cli=5:20.10.16~3-0~ubuntu-jammy containerd.io docker-compose-plugin`
    - Check docker compose version : docker compose version
    - Go to airflow folder under root folder : `cd /` `cd airflow/`
    - (info) To download any file from VM to local PC, you can use download option at upper right corner. you can write */airflow/docker-compose.yaml* in the dialog box to download docker-compose file.
    - go to airflow folder : `cd airflow/` 
    - create yaml file via `nano docker-compose.yaml`
    - copy from **docker-compose.yaml.txt**
        - we need PostgreSQL to keep DAG's metadata  
        - since we define `AIRFLOW_EXECUTER=LocalExecutor` , any airflow-worker nodes fail since it just run on master thanks to *LocalExecutor* notion
        - save and exit
    - make directory for DAGs : `mkdir dags`
    - if any plugins is used   : `mkdir dags plugins`
    - if any scripts are used, new folder is created under dags folder : `cd dags/`  `mkdir scripts`
- Install Airflow :  
    -  use yaml file to install Airflow : `docker-compose up -d` (-d is for detached run)
    -  check downloaded images and status of containers: `docker ps`  (`docker ps -a` for all containers, which are included failed containers)
    -  We need to define username and password for Airflow. we can do on airflow_airflow-scheduler1
        - execute with `docker exec -it -u 0 542 bash`  to access the container. (542 is the first 3 digit of Container ID of airflow scheduler)
        - ![image](https://user-images.githubusercontent.com/46892583/171581001-b1aabbe2-bcc7-41c9-a4d6-69a7685b5f2f.png)
        - to create credentials : `airflow users create --username admin --firstname melih --lastname melih --role Admin --password admin --email admin@airflow`
        - to exit from container : `exit`
  
    -  access to UI : `<airflow VM IP>:8080`
        -  username : admin
        -  password : admin
    
    -  to activate the changes that we made on telegraf.conf file we restart the service : `systemctl restart influxdb`
    -  to see the activities of Influxdb : `journalctl -fu influxdb`
    -  restart telegraf as well: `systemctl restart telegraf`
    -  check status of telegraf : `systemctl status telegraf`
    
    
    -  Get into dags file : `cd airlow/dags/`
    -  create an influxdb dag file : `nano influxdb_dag.py`  (we define functions in this file and indicate dependencies at the bottom line of the file.)
    -  paste all from Airflow-configuration.txt
        - we import *InfluxDBOperator* to connect the airflow with influxdb
        - we import *BashOperator* to execute commands in a Bash shell.
        - we need to use influxdb VM IP address on tokens, buckets, and URLs in the functions.
        - we need to adapt query which is written in flux syntax properly. (watch out for `r["host"] = <VMname>`) 
        - to write the output on BigQuery, need to edit `project_id = <bigquery project id>`
        - indicate the table_id where the data is written on `table_id = <tableIndicatedInFunctions>`
    - We need to import the libraries which are taken place on `influx_dag.py` file. we do this operation in airflow-scheduler1 container. (you can find commands in *Airflow Study&HelperFunctions.txt*)
         ```
         sudo pip3 install virtualenv
        virtualenv -p python3 (target folder)
        
        $ sudo su -
        # .  /opt/bitnami/airflow/venv/bin/activate
        (venv) # pip3 install influxdb
        ...
        (venv) # pip3 install influxdb-client
        
        (venv) # pip3 install apache-airflow-providers-influxdb
        
        (venv) # pip3 install pandas_gbq

        python3 -m venv --upgrade ```
        
    - then `exit` from the virtual environment.
    - We do the configurations on scheduler container.
    - execute with `docker exec -it -u 0 542 bash`  to access the airflow-scheduler container. (542 is the starting 3 digits of scheduler container's ID) (we can find out Container ID via `docker ps`command.
    - Reach UI of Airflow
    - Since we installed the required libraries, the warning on upper banner must be dissappear.
    - `cd airflow/dags/scripts` and `nano command.sh` to copy commands in command.sh file
    - to make command.sh executable we need to `chmod +x command.sh`
    - 




<details>
<summary>Additional Info</summary>
`
- To define username and password:  `airflow users create  --username admin --firstname melih --lastname gor --role Admin --password admin --email admin@airflow` command.
- exit from the container `exit`


## Create a DAG

- Refresh services to allow changes on configurations : `systemctl restart influxdb` 
- Check the status of restarted service : `systemctl status influxdb`
- to see the activities of Influxdb : `journalctl -fu influxdb`
- restart telegraf as well: `systemctl restart telegraf`
- check status of telegraf : `systemctl status telegraf`
- in "airflow/dags" folder, create a file as `nano influxdb_dag.py` and paste the codes inside of *"Airflow-configuration.txt"* file.
`
</details>


    

 
 
 

 
