# AWS-Python_ETL_Pipeline_using_Airflow
This project demonstrates how to build and automate an ETL pipeline written in Python and schedule it using open source Apache Airflow orchestration tool on AWS EC2 instance.


# Project Goals 

1. Data Ingestion - Create a data ingestion pipeline to extract data from OpenWeather API.
2. Data Storage - Create a data storage repository using AWS S3 buckets.
3. Data Transformation - Create ETL job to extract the data, do simple transformations and load the clean data using Airflow.
4. Data Pipeline - Create a data pipeline written in Python that extracts data from API calls and store it in AWS S3 buckets.
5. Pipeline Automation - Create scheduling service using Apace Airflow to trigger the data pipeline and automate the process.

# Architecture

![Architecture_Diagram](https://github.com/srajeevan/AWS-Python_ETL_Pipeline_using_Airflow/assets/16627503/45e4047a-2d7b-4134-9b9f-d2ef31dba318)


# Objective
In this project we build and automate an ETL pipeline that extracts data from OpenWeathermap API and trasform it using Pandas library and then stores/load the transformed data into AWS S3 bucket.
This whole process is orchestrated using Apachae Airflow.

# OpenWeathermap API
The data extraction is done by calling the GET method of the OpenWeathermap API endpoint.
This will give the weatherdata in json format which we trasform using Pandas.
Here is API endpoint look like.
```python
https://api.openweathermap.org/data/2.5/weather?q={city name}&appid={API key}
```
where there are two parameters
1. q - City name: Takes the name of any city in the world, the name goes in lowercase
2. API key - It can be obtained after creating an account on Openweathermap.org. The API key is unique to each account, make sure you do not share it.

# Tools Used

1. **AWS EC2** - Amazon Elastic Compute Cloud (Amazon EC2) is a web service that provides secure, resizable compute capacity in the cloud. It is a virtual environment where users run their own applications while the system is completely managed by Amazon.
2. **AWS S3** - Amazon Simple Storage Service (Amazon S3) is a storage service that offers high scalability, data availability, security, and performance. It stores the data as objects within buckets and is readily available for integration with thousands of applications.
3. **Apache Airflow** - Apachee Airflow is an open source orchestration or a workflow tool that allows one to programatically author, maintain, monitor and schedule workflows.

# Implementation

**Step 1** <br/>
AWS EC2 instance is deployed first as the first step.All the coding and computation is done in this VM.
We install all the necessary dependencies in this VM and set up a python virtual environment as well.
Here are baash commands that we run in the EC2 instance.

```sudo apt install python3-pip``` - to install pip <br/>
```sudo apt install python3.10-venv``` - to install python 3 virtual environment <br/>
```python3 -m venv airflow_venv``` - creating the virtual environment airflow_venev <br/>
```source airflow_venv/bin/activate``` - activiating the venv <br/>
```sudo pip install pandas``` - install pandas library <br/>
```sudo pip install s3fs``` - install s3fs to interact with AWS S3 <br/>
```sudo pip install apache-airflow``` - install apache airflow <br/>
```airflow standalone``` - runs airflow <br/>

