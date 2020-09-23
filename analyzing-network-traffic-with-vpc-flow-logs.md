# Analyzing network traffic with VPC Flow Logs 

## Task 1. Configure a custom network with VPC Flow Logs

1. Step 1: Create a custom  vpc network   

        gcloud compute networks create vpc-net --subnet-mode=CUSTOM --description="My custom network"


2. Step 2: Create subnet in custom vpc
    
        gcloud compute newtorks subnets create vpc-subnet --region=us-central1 --range="10.1.3.0/24" --enable-flow-logs


3. Step 3: Create a firewall rule
   
        gcloud compute firewall-rules create allow-http-ssh --network=vpc-net --target-tags="http-server" --direction=INGRESS --source-ranges="0.0.0.0/0" --rules=tcp:80,tcp:22" --action=ALLOW


## Task 2. Create an Apache web server

1. Step 1: Create the web server


   
       gcloud compute instances create web-server --zone=us-central1-c  --machine-type="f1-micro"  --tags="http-server" --network=vpc-net --subnet=vpc-subnet


2. Step 2: Install the Apache server


        
	   gcloud compute ssh web-server
	
	   sudo apt-get update && sudo apt-get install apache2 -y && echo '<!doctype html><html><body><h1>Hello World!</h1></body></html>' | sudo tee /var/www/html/index.html' && exit
	
	


## Task 3. Verify that network traffic is logged

1. Step 1: Get the external IP of the server and use the curl command to generate traffic


         for ((i=1;i<=50;i++)); do curl $(gcloud compute instances describe web-server --format="value(networkInterfaces[].accessConfigs[0].natIP)"); done


2. Step 2: Access the vpc flow logs

         export MYIP=$(echo dig +short myip.opendns.com @resolver1.opendns.com) && gcloud logging read 'resource.type="gce_subnetwork" AND logName="projects/gads-2020/logs/compute.googleapis.com%2Fvpc_flows" AND jsonPayload.connection.src_ip:'${MYIP} --format=json --limit=1
         # OR
         export MYIP=$(echo dig +short myip.opendns.com @resolver1.opendns.com) && gcloud logging read 'resource.type="gce_subnetwork" AND logName="projects/gads-2020/logs/compute.googleapis.com%2Fvpc_flows" AND '\"${MYIP}\" --format=json --limit=1


## Task 4. Export the network traffic to BigQuery to further analyze the logs

1. Step 1: Create a sink from your logs to BigQuery


        
	   bq mk bq_vpcflows	

	   gcloud logging sinks create vpc-flows bigquery.googleapis.com/projects/$(gcloud info | grep project | cut -d":" -f2 | cut -d"]" -f1 | cut -d"[" -f2)/datasets/bq_vpcflows --log-filter='resource.type="gce_subnetwork" AND logName=projects/gads-2020/logs/compute.googleapis.com%2Fvpc_flows AND "217.117.10.94"'
	


2. Step 2: Repeat Step 1 of Task 3

3. Step 3: Visualize the vpc flow logs in BigQuery


    ```
      export TABLE_NAME=$(echo bq_vpcflows.$(bq query --nouse_legacy_sql --format=prettyjson 'SELECT * FROM bq_vpcflows.INFORMATION_SCHEMA.TABLES' | grep table_name | cut -d":" -f2 | cut -d"," -f1 | cut -d'"' -f2)) && \
     echo 'SELECT
        jsonPayload.src_vpc.vpc_name,
        SUM(CAST(jsonPayload.bytes_sent AS INT64)) AS bytes,
        jsonPayload.src_vpc.subnetwork_name,
        jsonPayload.connection.src_ip,
        jsonPayload.connection.src_port,
        jsonPayload.connection.dest_ip,
        jsonPayload.connection.dest_port,
        jsonPayload.connection.protocol
        FROM ' \
        ${TABLE_NAME} \
        ' GROUP BY
        jsonPayload.src_vpc.vpc_name,
        jsonPayload.src_vpc.subnetwork_name,
        jsonPayload.connection.src_ip,
        jsonPayload.connection.src_port,
        jsonPayload.connection.dest_ip,
        jsonPayload.connection.dest_port,
        jsonPayload.connection.protocol
        ORDER BY
        bytes DESC
        LIMIT
        15' | bq query --nouse_legacy_sql

   ```


4. Step 4: Analyze the VPC Flow Logs in BigQuery



   ```

    export TABLE_NAME=$(echo bq_vpcflows.$(bq query --nouse_legacy_sql --format=prettyjson 'SELECT * FROM bq_vpcflows.INFORMATION_SCHEMA.TABLES' | grep table_name | cut -d":" -f2 | cut -d"," -f1 | cut -d'"' -f2)) && \
   echo -e 'SELECT
      jsonPayload.connection.src_ip,
      jsonPayload.connection.dest_ip,
      SUM(CAST(jsonPayload.bytes_sent AS INT64)) AS bytes,
      jsonPayload.connection.dest_port,
      jsonPayload.connection.protocol
      FROM ' \
      ${TABLE_NAME} \
      ' WHERE jsonPayload.reporter = "DEST"
      GROUP BY
      jsonPayload.connection.src_ip,
      jsonPayload.connection.dest_ip,
      jsonPayload.connection.dest_port,
      jsonPayload.connection.protocol
      ORDER BY
      bytes DESC
      LIMIT
      15' | bq query --nouse_legacy_sql

  ```
