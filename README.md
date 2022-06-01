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


- Connect via SSH to influx-study
- 





 
