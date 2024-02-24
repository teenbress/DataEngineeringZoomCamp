## Module 1 Homework_Solution

## Docker & SQL

## Question 1. Knowing docker tags


Which tag has the following text? - *Automatically remove the container when it exits* 

- `--rm`

## Question 2. Understanding docker first run 

Run docker with the python:3.9 image in an interactive mode and the entrypoint of bash.
Now check the python modules that are installed ( use ```pip list``` ). 

What is version of the package *wheel* ?

- 0.42.0
- 1.0.0
- 23.0.1
- 58.1.0
Command:
```
docker run -it python:3.9 bash
pip list
```
Command output:
```
Package    Version
---------- -------
pip        23.0.1
setuptools 58.1.0
wheel      0.42.0
```
# Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from September 2019:

```wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-09.csv.gz```

You will also need the dataset with zones:

```wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv```

Download this data and put it into Postgres (with jupyter notebooks or with a pipeline)   
   
Solution: Before solving the problem, we need to ingest the green taxi trips dataset using following commands.

```
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v C:/DataEngineeringZoomCamp/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data  \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
 postgres:13

python ingest_green_taxi.py \
    --user=root \
    --password=root \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name green_taxi_trips

python ingest_taxi_zone_lookup.py \
    --user=root \
    --password=root \
    --host=localhost \
    --port=5432 \
    --db=ny_taxi \
    --table_name zones
```
Then I started `pgcli`
```
pgcli -h localhost -p 5432 -u root -d ny_taxi
```
Run codes to ensure there's a connection to the database and check if it's working.  
- Run the command **\dt** to see a list of tables available in our database - This returned empty as we have not populated the database with data   
![db](https://github.com/teenbress/DataEngineeringZoomCamp/blob/main/images/homework1.png)   

## Question 3. Count records 

How many taxi trips were totally made on September 18th 2019?    
**15612**
```
SELECT count(1)
FROM green_taxi_trips
WHERE lpep_pickup_datetime::date = '2019-09-18'
AND lpep_dropoff_datetime::date = '2019-09-18';
```
```
+-------+
| count |
|-------|
| 15612 |
+-------+
```
## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

- 2019-09-18
- 2019-09-16   
**2019-09-26**
- 2019-09-21

```
SELECT lpep_pickup_datetime::date AS date, 
       max(trip_distance) AS largest_distance
FROM green_taxi_trips
GROUP BY 1
ORDER BY 2 DESC
LIMIT 1;
```
```
+------------+--------------+
| date       | max_distance |
|------------+--------------|
| 2019-09-26 | 341.64       |
+------------+--------------+

```
## Question 5. Three biggest pick up Boroughs

Consider lpep_pickup_datetime in '2019-09-18' and ignoring Borough has Unknown

Which were the 3 pick up Boroughs that had a sum of total_amount superior to 50000?
 
- "Brooklyn" "Manhattan" "Queens"
- "Bronx" "Brooklyn" "Manhattan"   
**"Bronx" "Manhattan" "Queens"**
- "Brooklyn" "Queens" "Staten Island"
```
SELECT pickup."Borough", SUM(taxi.total_amount) AS amount
FROM green_taxi_trips AS taxi
LEFT JOIN
zones AS pickup
ON taxi."PULocationID" = pickup."LocationID"
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3
```
```
+-----------+--------------------+
| Borough   | amount             |
|-----------+--------------------|
| Brooklyn  | 2619378.540000108  |
| Queens    | 2460386.1700005634 |
| Manhattan | 2427880.9200005606 |
+-----------+--------------------+
```
## Question 6. Largest tip

For the passengers picked up in September 2019 in the zone name Astoria which was the drop off zone that had the largest tip?
We want the name of the zone, not the id.

Note: it's not a typo, it's `tip` , not `trip`

- Central Park
- Jamaica
**JFK Airport**
- Long Island City/Queens Plaza

```
SELECT COALESCE(dropoff."Zone", 'Unknown') AS dropoff_zone,
       MAX(tip_amount) AS largest_tip
FROM green_taxi_trips AS taxi
LEFT JOIN zones AS pickup ON taxi."PULocationID" = pickup."LocationID"
LEFT JOIN zones AS dropoff ON taxi."DOLocationID" = dropoff."LocationID"
WHERE pickup."Zone" = 'Astoria'
GROUP BY 1
ORDER BY largest_tip DESC
LIMIT 1;
```
```
+--------------+-------------+
| dropoff_zone | largest_tip |
|--------------+-------------|
| JFK Airport  | 62.31       |
+--------------+-------------+
```
## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform. 
Copy the files from the course repo
[here](https://github.com/DataTalksClub/data-engineering-zoomcamp/tree/main/01-docker-terraform/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.


## Question 7. Creating Resources

After updating the main.tf and variable.tf files run:

```
terraform apply
```

Paste the output of this command into the homework submission form.



  
