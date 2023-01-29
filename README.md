# data-engineering-zoomcamp
2023 Data Engineering zoomcamp

Docker and SQL
Notes I used for preparing the videos: link

Commands
All the commands from the video

Downloading the data

wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz 
Note: now the CSV data is stored in the csv_backup folder, not trip+date like previously

Running Postgres with Docker
Windows
Running Postgres on Windows (note the full path)

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v c:/Users/alexe/git/data-engineering-zoomcamp/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
If you have the following error:

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v e:/zoomcamp/data_engineer/week_1_fundamentals/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data  \
  -p 5432:5432 \
  postgres:13

docker: Error response from daemon: invalid mode: \Program Files\Git\var\lib\postgresql\data.
See 'docker run --help'.
Change the mounting path. Replace it with the following:

-v /e/zoomcamp/...:/var/lib/postgresql/data
Linux and MacOS
docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v $(pwd)/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  postgres:13
If you see that ny_taxi_postgres_data is empty after running the container, try these:

Deleting the folder and running Docker again (Docker will re-create the folder)
Adjust the permissions of the folder by running sudo chmod a+rwx ny_taxi_postgres_data
CLI for Postgres
Installing pgcli

pip install pgcli
If you have problems installing pgcli with the command above, try this:

conda install -c conda-forge pgcli
pip install -U mycli
Using pgcli to connect to Postgres

pgcli -h localhost -p 5432 -u root -d ny_taxi
NY Trips Dataset
Dataset:

https://www1.nyc.gov/site/tlc/about/tlc-trip-record-data.page
https://www1.nyc.gov/assets/tlc/downloads/pdf/data_dictionary_trip_records_yellow.pdf
According to the TLC data website, from 05/13/2022, the data will be in .parquet format instead of .csv The website has provided a useful link with sample steps to read .parquet file and convert it to Pandas data frame.

You can use the csv backup located here, https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz, to follow along with the video.

$ aws s3 ls s3://nyc-tlc
                           PRE csv_backup/
                           PRE misc/
                           PRE trip data/
pgAdmin
Running pgAdmin

docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  dpage/pgadmin4
Running Postgres and pgAdmin together
Create a network

docker network create pg-network
Run Postgres (change the path)

docker run -it \
  -e POSTGRES_USER="root" \
  -e POSTGRES_PASSWORD="root" \
  -e POSTGRES_DB="ny_taxi" \
  -v c:/Users/alexe/git/data-engineering-zoomcamp/week_1_basics_n_setup/2_docker_sql/ny_taxi_postgres_data:/var/lib/postgresql/data \
  -p 5432:5432 \
  --network=pg-network \
  --name pg-database \
  postgres:13
Run pgAdmin

docker run -it \
  -e PGADMIN_DEFAULT_EMAIL="admin@admin.com" \
  -e PGADMIN_DEFAULT_PASSWORD="root" \
  -p 8080:80 \
  --network=pg-network \
  --name pgadmin-2 \
  dpage/pgadmin4
Data ingestion
Running locally

URL="https://github.com/DataTalksClub/nyc-tlc-data/releases/download/yellow/yellow_tripdata_2021-01.csv.gz"

python ingest_data.py \
  --user=root \
  --password=root \
  --host=localhost \
  --port=5432 \
  --db=ny_taxi \
  --table_name=yellow_taxi_trips \
  --url=${URL}
Build the image

docker build -t taxi_ingest:v001 .
On Linux you may have a problem building it:

error checking context: 'can't stat '/home/name/data_engineering/ny_taxi_postgres_data''.
You can solve it with .dockerignore:

Create a folder data
Move ny_taxi_postgres_data to data (you might need to use sudo for that)
Map -v $(pwd)/data/ny_taxi_postgres_data:/var/lib/postgresql/data
Create a file .dockerignore and add data there
Check this video (the middle) for more details
Run the script with Docker

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
Docker-Compose
Run it:

docker-compose up
Run in detached mode:

docker-compose up -d
Shutting it down:

docker-compose down
Note: to make pgAdmin configuration persistent, create a folder data_pgadmin. Change its permission via

sudo chown 5050:5050 data_pgadmin
and mount it to the /var/lib/pgadmin folder:

services:
  pgadmin:
    image: dpage/pgadmin4
    volumes:
      - ./data_pgadmin:/var/lib/pgadmin
    ...
SQL
Coming soon!
