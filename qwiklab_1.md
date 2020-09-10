## LAB: Recommend Products using ML with Cloud SQL and Dataproc

## Overview:

In this lab, you populate rentals data in Cloud SQL for the rentals recommendation engine to use.

## What you learn

In this lab, you will:

    - Create Cloud SQL instance

    - Create database tables by importing .sql files from Cloud Storage

    - Populate the tables by importing .csv files from Cloud Storage

    - Allow access to Cloud SQL

    - Explore the rentals data using SQL statements from CloudShell

## Introduction

In this lab, you populate rentals data in Cloud SQL for the rentals recommendation engine to use. The recommendations engine itself will run on Dataproc using Spark ML.

## Steps:

1. Create Cloud SQL instance

    gcloud sql instances create rentals --tier=db-n1-standard-2 --region=europe-west2

2. Create tables

    1. While you wait for your instance to be created, read the below mySQL script and answer the questions that follow below

            CREATE DATABASE IF NOT EXISTS recommendation_spark;

            USE recommendation_spark;

            DROP TABLE IF EXISTS Recommendation;
            DROP TABLE IF EXISTS Rating;
            DROP TABLE IF EXISTS Accommodation;

            CREATE TABLE IF NOT EXISTS Accommodation
            (
            id varchar(255),
            title varchar(255),
            location varchar(255),
            price int,
            rooms int,
            rating float,
            type varchar(255),
            PRIMARY KEY (ID)
            );

            CREATE TABLE  IF NOT EXISTS Rating
            (
            userId varchar(255),
            accoId varchar(255),
            rating int,
            PRIMARY KEY(accoId, userId),
            FOREIGN KEY (accoId)
                REFERENCES Accommodation(id)
            );

            CREATE TABLE  IF NOT EXISTS Recommendation
            (
            userId varchar(255),
            accoId varchar(255),
            prediction float,
            PRIMARY KEY(userId, accoId),
            FOREIGN KEY (accoId)
                REFERENCES Accommodation(id)
            );

            SHOW DATABASES;

    2. In Cloud Shell, view your instance information

            gcloud sql instances describe rentals


    3. Connect to the database

        - At the Cloud Shell prompt, connect to your Cloud SQL instance:

                gcloud sql connect rentals --user=root --quiet

        - Wait for your IP Address to be whitelisted

                Whitelisting your IP for incoming connection for 5 minutes...⠹

        - Run the below command

                SHOW DATABASES;

        - Copy and paste the below SQL statement you analyzed earlier paste it into the command line

                CREATE DATABASE IF NOT EXISTS recommendation_spark;

                USE recommendation_spark;

                DROP TABLE IF EXISTS Recommendation;
                DROP TABLE IF EXISTS Rating;
                DROP TABLE IF EXISTS Accommodation;

                CREATE TABLE IF NOT EXISTS Accommodation
                (
                id varchar(255),
                title varchar(255),
                location varchar(255),
                price int,
                rooms int,
                rating float,
                type varchar(255),
                PRIMARY KEY (ID)
                );

                CREATE TABLE  IF NOT EXISTS Rating
                (
                userId varchar(255),
                accoId varchar(255),
                rating int,
                PRIMARY KEY(accoId, userId),
                FOREIGN KEY (accoId)
                    REFERENCES Accommodation(id)
                );

                CREATE TABLE  IF NOT EXISTS Recommendation
                (
                userId varchar(255),
                accoId varchar(255),
                prediction float,
                PRIMARY KEY(userId, accoId),
                FOREIGN KEY (accoId)
                    REFERENCES Accommodation(id)
                );

                SHOW DATABASES;

        - Run the following command to show our tables

                USE recommendation_spark;

                SHOW TABLES;

        - Run the following query

                SELECT * FROM Accommodation;

3. Stage Data in Google Cloud Storage

    - Open a new Cloud Shell tab and paste in the below command

            echo "Creating bucket: gs://$DEVSHELL_PROJECT_ID"
            gsutil mb gs://$DEVSHELL_PROJECT_ID

            echo "Copying data to our storage from public dataset"
            gsutil cp gs://cloud-training/bdml/v2.0/data/accommodation.csv gs://$DEVSHELL_PROJECT_ID
            gsutil cp gs://cloud-training/bdml/v2.0/data/rating.csv gs://$DEVSHELL_PROJECT_ID

            echo "Show the files in our bucket"
            gsutil ls gs://$DEVSHELL_PROJECT_ID

            echo "View some sample data"
            gsutil cat gs://$DEVSHELL_PROJECT_ID/accommodation.csv

4. Loading Data from Google Cloud Storage into Cloud SQL tables

        gcloud sql import sql rentals gs://$DEVSHELL_PROJECT_ID/accommodation.csv \
                            --database=recommendation_spark

        gcloud sql import sql rentals gs://$DEVSHELL_PROJECT_ID/rating.csv \
                            --database=recommendation_spark

5. Explore Cloud SQL data

    - Query the ratings data:

            USE recommendation_spark;

            SELECT * FROM Rating
            LIMIT 15;

    - Use a SQL aggregation function to count the number of rows in the table

            SELECT COUNT(*) AS num_ratings
            FROM Rating;


## LAB: Generating housing recommendations with Machine Learning using Cloud Dataproc

In this lab, you carry out recommendations machine learning using Dataproc.

## What you learn

In this lab, you will:

    - Launch Dataproc

    - Run SparkML jobs using Dataproc

## Introduction
In this lab, you use Dataproc to train the recommendations machine learning model based on users' previous ratings. You then apply that model to create a list of recommendations for every user in the database.

In this lab, you will:

    - Launch Dataproc

    - Train and apply ML model written in PySpark to create product recommendations

    - Explore inserted rows in Cloud SQL


## Steps

1. Launch Dataproc

    - Create a cluster

        gcloud dataproc clusters create rentals --region=us-central1
        ...
        Waiting for cluster creation operation...done.
        Created [... rentals]

    - Copy and paste the below bash script into your Cloud Shell (optionally change CLUSTER, ZONE, NWORKERS if necessary before running)

            echo "Authorizing Cloud Dataproc to connect with Cloud SQL"
            CLUSTER=rentals
            CLOUDSQL=rentals
            ZONE=us-central1-a
            NWORKERS=2

            machines="$CLUSTER-m"
            for w in `seq 0 $(($NWORKERS - 1))`; do
            machines="$machines $CLUSTER-w-$w"
            done

            echo "Machines to authorize: $machines in $ZONE ... finding their IP addresses"
            ips=""
            for machine in $machines; do
                IP_ADDRESS=$(gcloud compute instances describe $machine --zone=$ZONE --format='value(networkInterfaces.accessConfigs[].natIP)' | sed "s/\['//g" | sed "s/'\]//g" )/32
                echo "IP address of $machine is $IP_ADDRESS"
                if [ -z  $ips ]; then
                ips=$IP_ADDRESS
                else
                ips="$ips,$IP_ADDRESS"
                fi
            done

            echo "Authorizing [$ips] to access cloudsql=$CLOUDSQL"
            gcloud sql instances patch $CLOUDSQL --authorized-networks $ips

    - Hit enter then, when prompted, type Y, then enter again to continue .Wait for the patching to complete. You will see

            Patching Cloud SQL instance...done.

2. Run ML model

    - Copy over the model code by executing the below in Cloud Shell

            gsutil cp gs://cloud-training/bdml/v2.0/model/train_and_apply.py train_and_apply.py
            nano train_and_apply.py

    - In train_and_apply.py, find line 30: CLOUDSQL_INSTANCE_IP and paste your Cloud SQL IP address

            # MAKE EDITS HERE
            CLOUDSQL_INSTANCE_IP = '<paste-your-cloud-sql-ip-here>'   # <---- CHANGE (database server IP)
            CLOUDSQL_DB_NAME = 'recommendation_spark' # <--- leave as-is
            CLOUDSQL_USER = 'root'  # <--- leave as-is
            CLOUDSQL_PWD  = '<type-your-cloud-sql-password-here>'  # <---- CHANGE

    - In your cloud shell, then
            
            Press Ctrl + X then Y and ENTER

3. Run your ML job on Dataproc

        gcloud dataproc jobs submit pyspark \
        --cluster=rentals \
        --region=us-central1 \
        other dataproc-flags \
        -- job-args

4. Explore inserted rows with SQL

    - Open your sql instance in clou shell and paste this command
    
        USE recommendation_spark;

        SELECT COUNT(*) AS count FROM Recommendation;

    - Find the recommendations for a user:

            SELECT
                r.userid,
                r.accoid,
                r.prediction,
                a.title,
                a.location,
                a.price,
                a.rooms,
                a.rating,
                a.type
            FROM Recommendation as r
            JOIN Accommodation as a
            ON r.accoid = a.id
            WHERE r.userid = 10;
        



        


    


    