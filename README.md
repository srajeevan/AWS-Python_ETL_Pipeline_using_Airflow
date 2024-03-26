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
![EC2 Instance](https://github.com/srajeevan/AWS-Python_ETL_Pipeline_using_Airflow/blob/main/images/EC2_Instance.png)

We will be able to connect to airflow once we have the EC2 instance ready.
We have the port 8080 allocated for airflow.
An airflow UI is generated when the webserver is up and running with some pre-defined DAGs as shown below:

![Airflow UI](https://github.com/srajeevan/AWS-Python_ETL_Pipeline_using_Airflow/blob/main/images/airflow_UI.png)

* **Step 2** - In order for airflow to make API calls to open weather, there needs to be a connection between two services which can be done using 'connections' tab in airflow. This will allow airflow to access openweather map using HTTP operator.
![Airflow Connection Tab](https://github.com/srajeevan/AWS-Python_ETL_Pipeline_using_Airflow/blob/main/images/Airflow_Connection.png)

* **Step 3** - It is time to create our first DAG (Directed Acyclic Graph) with proper imports and dependencies. This step is subdivided into three steps each accounting for a task within our DAG. Create dag file for example weather_dag.py and add the required import and libraries:
     * **Task 1** - Tasks are written inside the DAGs using operators. Here, we use the *HTTPSensor* Operator to check if the API is callable or not. You use your API key and choose any city in which you are interested.
     * weather_dag is the dag we have created.We pass on the default_args that we defined in a dictionary as well.
       At firt we check if the weather_api end point is ready or not and this is done is the is_weather_api_ready task.
       This taks involves accessing the api endpoint by passing the API key and the city.

```
with DAG('weather_dag',
        default_args=default_args,
        schedule_interval = '@daily',
        catchup=False) as dag:


        is_weather_api_ready = HttpSensor(
        task_id = 'is_weather_api_ready',
        http_conn_id='weathermap_api',
        endpoint='/data/2.5/weather?q=Houston&appid=xxxxxxxx'    
        )
```
 * **Task 2** - This task calls the weather API and invokes a GET method to get the data in json format. We use a lambda function to convert the load into a text.
```
        extract_weather_data = SimpleHttpOperator(
        task_id = 'extract_weather_data',
        http_conn_id='weathermap_api',
        endpoint='/data/2.5/weather?q=Houston&appid=7ca17c851237e628dcc5331cbfba9566',    
        method = 'GET',
        response_filter = lambda response: json.loads(response.text),
        log_response= True
        )
```

  * **Task 3** - This task calls a python function that transforms the json format into csv file and store it in AWS S3 buckets. You can define the schedule intervals in which you want to execute your DAG, based on default_args.
```
        transform_load_weather_data = PythonOperator(
        task_id = 'transform_weather_data',
        python_callable = transform_load_data
        )
```
 After implementing above step, if we go over to Airflow UI, we can see the tasks inside DAG, the directed chain is achieved by adding tasks ordering at the end of the DAG.
 <h4 align ='center' >   is_weather_api_available >> extract_weather_data >> transform_load_weather_data </h4>

 ![Airflow DAG](https://github.com/srajeevan/AWS-Python_ETL_Pipeline_using_Airflow/blob/main/images/Airflow_Dag.png)

 * **Step 4** - We can write the tranform function and give appropriate permissions to airflow for using AWS S3 and proper session credentials. For every transaction, AWS creates a session window that allows a service to interact with AWS components.
```
def transform_load_data(task_instance):
    data = task_instance.xcom_pull(task_ids="extract_weather_data")
    city = data["name"]
    weather_description = data["weather"][0]['description']
    temp_farenheit = kelvin_to_fahrenheit(data["main"]["temp"])
    feels_like_farenheit= kelvin_to_fahrenheit(data["main"]["feels_like"])
    min_temp_farenheit = kelvin_to_fahrenheit(data["main"]["temp_min"])
    max_temp_farenheit = kelvin_to_fahrenheit(data["main"]["temp_max"])
    pressure = data["main"]["pressure"]
    humidity = data["main"]["humidity"]
    wind_speed = data["wind"]["speed"]
    time_of_record = datetime.utcfromtimestamp(data['dt'] + data['timezone'])
    sunrise_time = datetime.utcfromtimestamp(data['sys']['sunrise'] + data['timezone'])
    sunset_time = datetime.utcfromtimestamp(data['sys']['sunset'] + data['timezone'])

    transformed_data = {"City": city,
                        "Description": weather_description,
                        "Temperature (F)": temp_farenheit,
                        "Feels Like (F)": feels_like_farenheit,
                        "Minimun Temp (F)":min_temp_farenheit,
                        "Maximum Temp (F)": max_temp_farenheit,
                        "Pressure": pressure,
                        "Humidty": humidity,
                        "Wind Speed": wind_speed,
                        "Time of Record": time_of_record,
                        "Sunrise (Local Time)":sunrise_time,
                        "Sunset (Local Time)": sunset_time                        
                        }
    transformed_data_list = [transformed_data]
    df_data = pd.DataFrame(transformed_data_list)
    aws_credentials = {"key": "xxxxxx", "secret": "xxxxxx", "token": "xxxx"}

    now = datetime.now()
    dt_string = now.strftime("%d%m%Y%H%M%S")
    dt_string = 'current_weather_data_houston_' + dt_string
    df_data.to_csv(f"s3://houstonweatherdata/{dt_string}.csv", index=False, storage_options=aws_credentials)
```

* **Step 5** - Now, we can see that, we have csv files stored in AWS S3 buckets using data pipeline that we just created.

 ![CSV files in S3 Location](https://github.com/srajeevan/AWS-Python_ETL_Pipeline_using_Airflow/blob/main/images/s3_bucket.png)
