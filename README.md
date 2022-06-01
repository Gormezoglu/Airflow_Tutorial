# Airflow_Tutorial
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
                -API Tokens
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



 
