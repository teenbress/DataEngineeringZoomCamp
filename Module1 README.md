### [Module 1: Containerization and Infrastructure as Code](https://github.com/teenbress/DataEngineeringZoomCamp/tree/main/01-docker-terraform)
This module will cover basics with the following:
+ Docker
+ PostgreSQL
+ Terraform
+ Google Cloud Platform
____________________________________________

### Docker 
1. Set up Docker  
   1.1 Install the Docker engine.  
   1.2 Verify the installation.
   ```
   docker version
   docker run hello world
   ```
2. Create a dockerfile, build image, run the container.    
  A Dockerfile is a script that defines the instructions for creating a Docker image. It specifies the base image, the environment, and the dependencies that'll be installed. It also copies a python pipeline file to the filesystem of the container.

   
•	Python 3.9：`docker run -it python:3.0`  
•	Some libraries  
```
	docker run -it --entrypoint=bash python:3.9 
	pip install pandas
```  
•	Simple pipeline
```py
docker build -t [test:pandas] # build an image [test:pandas] that is used to run a container 
docker run -it [test:pandas] # run a container using this image [test:pandas] with optional parameters input
docker images # check the list of the images available on the machine
docker rmi # remove an image if you want to build a new one.
docker build --no-cache -t test:pandas  # use the --no-cache option with the docker build command to force a new build and ignore the cache
exit 
```
![docker](https://github.com/teenbress/DataEngineeringZoomCamp/blob/main/images/docker.png)
3. Modify the docker file and update the python script, buid the image, run the container.   
```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v C:/DataEngineeringZoomCamp/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data  \
  -p 5432:5432 \
 postgres:13
```
The above command uses the **docker run** command to start a new container based on the **postgres:13** image. The **-it** option runs the container in interactive mode, which allows you to interact with the container's command prompt. The **-e** option is used to set environment variables inside the container. In this case, it sets the following env variables:
+ POSTGRES_USER to "root"
+ POSTGRES_PASSWORD to "root"
+ POSTGRES_DB to "ny_taxi"   

The **-v** option is used to mount a volume from the host machine to the container. In this case, it mounts the directory C://Users//teenbress//ny_taxi_postgres_data from the host machine to /var/lib/postgresql/data inside the container. This allows the data stored in the Postgres database to persist even if the container is removed.   
   
The **-p** option is used to map a container's port to a port on the host machine. In this case, it maps port 5432 inside the container to port 5432 on the host machine. This allows you to connect to the Postgres database running in the container from applications running on the host machine.  
_____________________________
### Postgres
1. Pip install pgcli. Access the database **Postgresql** inside the container.
   ```py
   pip install pgcli
   pgcli # test if installed successfully
   '''
2. Connect to the DB NY_Taxi_Data
```
   pgcli -h localhost -p 5432 -u root -d ny_taxi
```
This command connects to a Postgres server running on the localhost, on port 5432, using the username "root" and the database "ny_taxi".Once the connection is established, you will be presented with the pgcli prompt and you can start executing SQL commands.    

**-h** flag specifies the hostname or IP address of the Postgres server,   
**-p** flag specifies the port number,   
**-u** flag specifies the username,   
**-d** flag specifies the database name.   
3. Digest data with a Jupyter Notebook
4.Check if the schema got created inside postgres using pgcli. 
We ran **\dt** to see the list of tables. To see details about the table run **\d yellow_taxi_data**.   
5. pgAdmin
   + It's not convenient to use pgcli for data exploration and querying
   + run pgAdmin with Docker
```
# pgAdmin docker image
docker run -it \
  -e PGADMIN_DEFAULT_EMAIL='admin@admin.com' \
  -e PGADMIN_DEFAULT_PASSWORD='root' \
  -p 8080:80 \
  dpage/pgadmin4

# pgAdmin with Docker Network

docker network create pg-network

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v C:/DataEngineeringZoomCamp/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data  \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
 postgres:13
```
+ Build docker image `taxi-ingest：v001` and run the python script in the docker `taxi_ingest:v001`   
```
  python ingest_data.py \
--user=root  \
--password=root \
--host=localhost \
--port=5432 \
--db=ny_taxi \
--table_name=yellow_taxi_trips \
--url="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

docker build -t taxi_ingest:v001 .
# run the python script in the docker image: taxi_ingest. 
URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"
docker run -it \
  --network=pg-network \
  taxi_ingest:v001 \
    --user=root \
    --password=root \
    --host=pg-database \
    --port=5432 \
    --db=ny_taxi \
    --table_name=yellow_taxi_trips \
    --url=${URL}
  ```
![run python script with pgAdmin and docker ](https://github.com/teenbress/DataEngineeringZoomCamp/blob/main/images/docker%20with%20python%20script.png)   

6. ##### Docker Compose
 Docker Compose is a tool for defining and running multi-container Docker applications. It allows you to configure and run multiple containers, networks, and volumes in a single `YAML` file. This makes it easy to manage and configure multiple containers as part of a single application.    

Docker Compose uses a `docker-compose.yaml` file to define the services (containers), networks, and volumes that make up an application. The file is written in YAML, which is a human-readable format for specifying configuration settings.   
Some docker compose commands:
```
docker-compose up ## run the YAML file
docker-compose down ## shut down the containers
docker-compose upm-d ## start the containers in detached mode
```
When we run `docker-compose` - the two services we specified in the `YAML` file are added as part of the same network, which is given a default name by Docker (we can specify the network name but we haven't in our example). The `pg-network` should be irrelevant at this point. So we actually need to find the default network created. To do this run:
`docker network ls`. In my case, the network is `2_docker_sql_defauts`.  
We also need to replace the host with `pgdatabase` which is what we specified in the `YAML` file as well.  
So the new command we run after running with docker compose would be sth like:
```
docker run -it  \
    --network=<default network created> \
    taxi_ingest:v001 \
      --user=root \
      --password=root \
      --host=pgdatabase \
      --port=5432 \
      --db=ny_taxi \
      --table_name=yellow_taxi_data \
      --url="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

```
____________________
### Terraform
#### 1. Intro to Terraform
#### 1.2 Terraform commands overview
+ Scope - Identify the infrastructure for your project.  
+ Author - Write the configuration for your infrastructure.  
+ terraform init - Install the plugins Terraform needs to manage the infrastructure.
+ terraform fmt (optional): formats your configuration files so that the format is consistent.
+ terraform plan - Preview the changes Terraform will make to match your configuration.  
+ terraform apply - Make the planned changes.
+ terraform destroy - Remove the stack from the infrastructure.   
You may access `main.tf` from [this link](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/01-docker-terraform/1_terraform_gcp/terraform/terraform_basic/main.tf) and `variables.tf` from [this link](https://github.com/DataTalksClub/data-engineering-zoomcamp/blob/main/01-docker-terraform/1_terraform_gcp/terraform/terraform_with_variables/variables.tf). Take a look at them to understand the details of the implementation. Copy them to a new folder within your work directory so that the subfolder only contains the Terraform configuration files. 
#### 3. Set up GCP with Terraform: storage, BigQuery
[A guilder for how to set virtual environment on GCP](https://itsadityagupta.hashnode.dev/setting-up-the-development-environment-on-google-virtual-machine#heading-installing-terraform)     
+ Set `GOOGLE_APPLICATION_CREDENTIALS` to point to the user account key json file.  
  ```
  export GOOGLE_APPLICATION_CREDENTIALS=~/.gc/ny-rides.json
  ```
  
+ Now authenticate:  
```
gcloud auth activate-service-account --key-file $GOOGLE_APPLICATION_CREDENTIALS
```
+ Alternatively, you can authenticate using OAuth like shown in the video:  
```
gcloud auth application-default login
PROJECT_NAME="decamp-415001"
gcloud auth application-default set-quota-project ${PROJECT_NAME}
```



