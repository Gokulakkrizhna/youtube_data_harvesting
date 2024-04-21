# YouTube Data Harvesting and Warehousing using SQL and Streamlit
## Overview
The YouTube Data Harvesting and Warehousing project simplifies the extraction of YouTube channel information using Google APIs. With the aid of Streamlit, users can conveniently access and utilize this data in a user-friendly interface.The project further involves storing the extracted data in a MySQL database for efficient storage. This data is then utilized for analysis and displayed in a Streamlit web application using Pandas DataFrame, enhancing the usability and accessibility of the information.
## Table of Contents
- [Key Technologies and Skills](#key-technologies-and-skills)
- [Installation](#installation)
- [Usage](#usage)
- [Features](#features)
- [Retrieving data from the YouTube API](#retrieving-data-from-the-youtube-api)
- [Migrating data to a SQL data warehouse](#migrating-data-to-a-sql-data-warehouse)
- [Data Analysis](#data-analysis)
- [Approach](#approach)
- [Contact](#contact)
# Key Technologies and Skills
- Python Scripting
- Streamlit
- Api Integration
- Pandas-Dataframe
- Database management using MySql
# Installation
To run this project, please install below python packages as prerequisites.
```bash
pip install streamlit
pip install google-api-python-client
pip install mysql-connector-python
pip install pandas
pip install pymysql sqlalchemy
```
# Usage
To use this project, Please follow the below steps.
- Install the required packages: ```pip install -r requirements.txt ```
- Run the Streamlit app: ```streamlit run p1.py```
- Access the app in your browser at ```http://localhost:8501```
# Features
- Fetch YouTube channel information like channel, video, comment information using the YouTube Data API
- Store extracted data in a MySQL database
- Analyzing the data with the help of SQL queries
- Visualize data using Pandas DataFrames
- User-friendly interface powered by Streamlit
# Approach
Setup the Streamlit app: Streamlit is a user-friendly web development tool that simplifies the process of creating intuitive interfaces. With Streamlit, you can easily design a straightforward UI where users can input a channel ID and quickly access all relevant details in a simple manner.

Connect to Google API: To fetch all the required data, you'll require the YouTube API. Utilize the Google API client library for Python to send requests to the API and retrieve the necessary information.

Store data in MySql: The collected data is stored in a MySQL database. Employ the MySQL Connector package to establish a connection with the MySQL localhost server. 

SQLAlchemy and PyMySQL: These 2 will facilitate the creation of a temporary connection to the MySQL database, enabling bulk insertion of data.

Data Analysis: Using SQL queries, the retrieved data has been analyzed and visualized in Streamlit through Pandas DataFrame.

The provided code consists of Python scripting utilizing various libraries and an API key to fetch data and store it in a MySQL database. Additionally, it incorporates a Streamlit web application to facilitate user interaction.

Here's a breakdown of what the code does:
- Importing all the neccessary libraries includes ```Streamlit``` which creates UI to interact with user and display the analysed data, ```google-api-python-client``` which helps python to connect with Youtube with the help of API key and fetch the all the details, ```Pandas``` which helps to display the analysed data in Streamlit web,```mysql-connector-python``` will create a connection between python and MySql server,
```pymysql sqlalchemy``` will create a temporary connection with MySql database for a bulk insertion,```time``` used to create toast notification in a UI in Streamlit.
```bash
import streamlit as st
import googleapiclient.discovery
import pandas as pd
import mysql.connector
from sqlalchemy import create_engine
import time
```
- The parameters for initializing the ```YouTube API client``` are set, and a connection to the ```MySQL``` server is established using ```mysql.connector```. Subsequently, a database is created, and three different tables are created within the database.
```bash
api_service_name = "youtube"
api_version = "v3"
api_key = "Enter your API key"
youtube = googleapiclient.discovery.build(api_service_name, api_version, developerKey=api_key)

mydb = mysql.connector.connect(
 host="localhost",
 user="root",
 password="",
 )
mycursor = mydb.cursor(buffered=True)

mycursor.execute("create database if not exists youtubedb")
mycursor.execute("use youtubedb")
mycursor.execute("create table if not exists channel (channel_id varchar(255) primary key,channel_name varchar(255),channel_description varchar(255),channel_subscriber_Count integer(10),channel_view_count integer(10),channel_total_video integer(10))")
mycursor.execute("create table if not exists video(c_id varchar(255),id varchar(255) primary key,name varchar(255),description text,publish_date timestamp,view integer(10),likes integer(10),favorite integer(10),comment integer(10),duration integer(10),thumbnail varchar(255),foreign key(c_id) references channel(channel_id))")
mycursor.execute("create table if not exists comment(video_id varchar(255),id varchar(255) unique,text text,author varchar(255),publish_date timestamp,foreign key(video_id) references video(id))")

streamlit_home()
```
- Four separate tabs have been implemented in the Streamlit web application to enhance user interaction and improve data visualization.
```bash
tab1, tab2, tab3, tab4= st.tabs(["Home", "Channel info", " Data Collection","Data Analysis"])
```
-In Tab1 of the Streamlit web application, users can input the ```Channel ID```. If users choose to migrate the data to ```MySQL```, all the data will be stored in the MySQL database using ```SQLAlchemy```.
```bash
st.header('YOUTUBE DATA HARVESTING AND WAREHOUSING', divider='rainbow')
        channel_id = st.text_input('CHANNEL ID', '')
        c,d,e = channel_part_data(channel_id)
        if st.button("Migrate to Mysql"):
            sql_db_val_insert(c,1)
            sql_db_val_insert(d,2)
            sql_db_val_insert(e,3)
            st.success('Data migrated to Mysql server!', icon="✅")
```
```bash
def sql_db_val_insert(a,b):
    username = 'root'
    password = ''
    host = 'localhost'
    database = 'youtubedb'

    engine = create_engine(f'mysql+pymysql://{username}:{password}@{host}/{database}')

    if b==1:
        df = pd.DataFrame(a,index = [1])
        df.to_sql(name='channel', con=engine, if_exists='append', index=False)
    elif b==2:
        df = pd.DataFrame(a,index = [i for i in range(1,len(a["id"])+1)])
        df.to_sql(name='video', con=engine, if_exists='append', index=False)
    else:
        df = pd.DataFrame(a,index = [i for i in range(1,len(a["id"])+1)])
        df.to_sql(name='comment', con=engine, if_exists='append', index=False)
    
    engine.dispose()
```



