# YouTube Data Harvesting and Warehousing using SQL and Streamlit
## Overview
The YouTube Data Harvesting and Warehousing project simplifies the extraction of YouTube channel information using Google APIs. With the aid of Streamlit, users can conveniently access and utilize this data in a user-friendly interface.The project further involves storing the extracted data in a MySQL database for efficient storage. This data is then utilized for analysis and displayed in a Streamlit web application using Pandas DataFrame, enhancing the usability and accessibility of the information.
## Table of Contents
- [Key Technologies and Skills](#key-technologies-and-skills)
- [Installation](#installation)
- [Usage](#usage)
- [Features](#features)
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
```Setup the Streamlit app```: Streamlit is a user-friendly web development tool that simplifies the process of creating intuitive interfaces. With Streamlit, you can easily design a straightforward UI where users can input a channel ID and quickly access all relevant details in a simple manner.

```Connect to Google API```: To fetch all the required data, you'll require the YouTube API. Utilize the Google API client library for Python to send requests to the API and retrieve the necessary information.

```Store data in MySql```: The collected data is stored in a MySQL database. Employ the MySQL Connector package to establish a connection with the MySQL localhost server. 

```SQLAlchemy and PyMySQL```: These 2 will facilitate the creation of a temporary connection to the MySQL database, enabling bulk insertion of data.

```Data Analysis```: Using SQL queries, the retrieved data has been analyzed and visualized in Streamlit through Pandas DataFrame.

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
- The parameters for initializing the ```YouTube API client``` are set, and a connection to the ```MySQL``` server is established using ```mysql.connector```. Subsequently, a database is created, and three different tables are created within the database. **Note: Replace your API key in ```api_key```**
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
- In Tab1 of the Streamlit web application, users can input the ```Channel ID```. ```channel_part_data``` helps to fetch all the data of the particular channel.If users choose to migrate the data to ```MySQL```, all the data will be stored in the MySQL database using ```SQLAlchemy```.
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
def channel_part_data(a):
    request = youtube.channels().list(
        part="snippet,contentDetails,statistics",
        id=a
    )
    response = request.execute()

    data = {
        "channel_id": a,
        "channel_name" : response['items'][0]['snippet']['title'],
        "channel_description": response['items'][0]['snippet']['description'],
        "channel_subscriber_Count": response['items'][0]['statistics']['subscriberCount'],
        "channel_view_count": response['items'][0]['statistics']['viewCount'],
        "channel_total_video": response['items'][0]['statistics']['videoCount']
    }

    request = youtube.search().list(
        part="id,snippet",
        channelId=a,
        order="date",
        maxResults=100 
    )
    response = request.execute()

    video_id = []
    for i in range(len(response["items"])):
        vid_conf = tuple(response["items"][i]["id"].keys())
        if vid_conf[1] == "videoId":
            v_id = response["items"][i]["id"]["videoId"]
            video_id.append(v_id)
    
    data1 = {"c_id": [],"id": [],"name": [],"description": [],"publish_date": [],"view": [],"likes": [],"favorite": [],"comment": [],"duration": [],"thumbnail": []}

    for i in video_id:
        request = youtube.videos().list(
        part="id,snippet,statistics,contentDetails,status",
        id=i
        )
        response = request.execute()

        data1["c_id"].append(a)
        data1["id"].append(i)
        data1["name"].append(response["items"][0]["snippet"]["title"])
        data1["description"].append(response["items"][0]["snippet"]["description"])
        data1["publish_date"].append(response["items"][0]["snippet"]["publishedAt"])
        data1["view"].append(response["items"][0]["statistics"]["viewCount"])
        data1["likes"].append(response["items"][0]["statistics"]["likeCount"])
        data1["favorite"].append(response["items"][0]["statistics"]["favoriteCount"])
        data1["comment"].append(response["items"][0]["statistics"]["commentCount"])
        dur = response["items"][0]["contentDetails"]["duration"]
        duration_timedelta = pd.Timedelta(dur)
        total_seconds = duration_timedelta.total_seconds()
        data1["duration"].append(int(total_seconds))
        data1["thumbnail"].append(response["items"][0]["snippet"]["thumbnails"]["default"]["url"])

    data2 = {"video_id": [],"id": [],"text": [],"author": [],"publish_date": []}

    for j in video_id:
        request = youtube.commentThreads().list(
            part="snippet",
            videoId=j,
            maxResults=100
        )
        response = request.execute()

        for i in range(len(response["items"])):
            data2["video_id"].append(j)
            data2["id"].append(response["items"][i]["id"])
            data2["text"].append(response["items"][i]["snippet"]["topLevelComment"]["snippet"]["textDisplay"])
            data2["author"].append(response["items"][i]["snippet"]["topLevelComment"]["snippet"]["authorDisplayName"])
            data2["publish_date"].append(response["items"][i]["snippet"]["topLevelComment"]["snippet"]["publishedAt"])

    return data,data1,data2
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
- In Tab2 of the Streamlit web application, data related to channels will be fetched and demonstrated, mimicking the interface of YouTube. Here ```channel_general_data``` will fetch all basic details of the channels and ```channel_video_url``` will fetch the latest video url from the respective channel.
 ```bash
a = channel_general_data(channel_id)
        col1, col2 = st.columns(2)
        with col1:
            st.image(a[4])
        with col2:
            st.header(a[0])
            col3,col4,col5 = st.columns(3)
            with col3:
                st.write(f"{a[5]} .")
            with col4:
                st.write(f"{a[2]} subscribers .")
            with col5:
                st.write(f"{a[3]} videos.")
            st.write(a[1])
            if st.button('Subscribe :bell:'):
                st.toast('you will get notification regularly')
                time.sleep(1)
        b = channel_video_url(channel_id)
        st.video(b)
```
```bash
def channel_general_data(a):
    request = youtube.channels().list(
    part="snippet,statistics",
    id=a
    )
    response = request.execute()

    channel_logo = response['items'][0]['snippet']['thumbnails']['medium']['url']
    channel_name = response['items'][0]['snippet']['title']
    channel_description = response['items'][0]['snippet']['description']
    channel_subscriber_Count = response['items'][0]['statistics']['subscriberCount']
    channel_total_video = response['items'][0]['statistics']['videoCount']
    channel_user_id = response['items'][0]['snippet']['customUrl']
    return [channel_name,channel_description,channel_subscriber_Count,channel_total_video,channel_logo,channel_user_id]
```
```bash
def channel_video_url(a):
    request = youtube.search().list(
        part="id,snippet",
        channelId=a,
        order="date"
    )
    response = request.execute()

    for i in range(len(response["items"])):
        vid_conf = tuple(response["items"][i]["id"].keys())
        if vid_conf[1] == "videoId":
            video_id = response["items"][i]["id"]["videoId"]
            break
    video_url = "https://www.youtube.com/watch?v={0}".format(video_id)
    return video_url
```
- In Tab3, all the data related to that particular channel will be displayed with the help of ```Pandas Dataframe``` and user can ```download the data``` as csv with the help of download function.
```bash
option = st.selectbox('Which table do you want to display?',('channel','video','comment'))
        if option == "channel":
            df = pd.DataFrame(c,index=[1])
            st.dataframe(df)
            csv = df.to_csv(index=False)
            st.download_button(label="Download CSV", data=csv, file_name='channel.csv', mime='text/csv')
        elif option == "video":
            df1 = pd.DataFrame(d,index = [i for i in range(1,len(d["id"])+1)])
            st.dataframe(df1.iloc[:, 1:])
            csv = df1.to_csv(index=False)
            st.download_button(label="Download CSV", data=csv, file_name='video.csv', mime='text/csv')
        else:
            df2 = pd.DataFrame(e,index = [i for i in range(1,len(e["id"])+1)])
            st.dataframe(df2.iloc[:, 1:])
            csv = df2.to_csv(index=False)
            st.download_button(label="Download CSV", data=csv, file_name='comment.csv', mime='text/csv')
```
- In Tab4, Data stored in MySql database was analysed using ```Sql queries``` and displayed using ```Pandas Dataframe```.
```bash
option1 = st.selectbox('For which queries do you need to display?',("Query1","Query2","Query3","Query4","Query5","Query6","Query7","Query8","Query9","Query10"))
        if option1 == "Query1":
            st.write("What are the names of all the videos and their corresponding channels?")
            mycursor.execute("with gokul as(select channel.channel_name,video.name from channel right join video on channel.channel_id = video.c_id)\
                 select channel_name,name from gokul order by channel_name asc")
            out=mycursor.fetchall()
            df = pd.DataFrame(out,columns=['Channel', 'Video Title'],index = [i for i in range(1,len(out)+1)])
        elif option1 == "Query2":   
            st.write("Which channels have the most number of videos, and how many videos do they have?")
            mycursor.execute("select channel_name,channel_total_video from channel where channel_total_video = (SELECT MAX(channel_total_video) FROM channel)")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['Channel_name', 'total_Video'],index = [i for i in range(1,len(out)+1)])
        elif option1 == "Query3":  
            st.write("What are the top 10 most viewed videos and their respective channels?")
            mycursor.execute("with gokul as(select channel.channel_name,video.name,video.view from channel right join video on channel.channel_id = video.c_id)\
                 select channel_name,name,view from gokul order by view desc limit 10")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['Channel_name', 'Video_name','view'],index = [i for i in range(1,len(out)+1)])
        elif option1 == "Query4":  
            st.write("How many comments were made on each video, and what are theircorresponding video names?")
            mycursor.execute("select name,comment from video")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['Video_name','comments'],index = [i for i in range(1,len(out)+1)])
        elif option1 == "Query5": 
            st.write("Which videos have the highest number of likes, and what are their corresponding channel names?") 
            mycursor.execute("with gokul as(select channel.channel_name,video.name,video.likes from channel right join video on channel.channel_id = video.c_id)\
                 select channel_name,name,likes from gokul where likes = (select max(likes) from gokul)")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['channel_name','video_name','max_likes'],index = [i for i in range(1,len(out)+1)])
        elif option1 == "Query6":  
            st.write("What is the total number of likes for each video, and what are their corresponding video names?")
            mycursor.execute("select name,likes from video ")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['video_name','likes'],index = [i for i in range(1,len(out)+1)])
        elif option1 == "Query7":  
            st.write("What is the total number of views for each channel, and what are their corresponding channel names?")
            mycursor.execute("select channel_name,channel_view_count from channel")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['channel_name','total_view'],index = [i for i in range(1,len(out)+1)])
        elif option1 == "Query8":  
            st.write("What are the names of all the channels that have published videos in the year 2022?")
            mycursor.execute("WITH gokul AS (SELECT channel.channel_name,video.name, video.publish_date FROM channel JOIN video ON channel.channel_id = video.c_id)\
            SELECT channel_name,name,publish_date FROM gokul WHERE YEAR(publish_date) = 2022")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['channel_name','video_name','publish_date'],index = [i for i in range(1,len(out)+1)])
        elif option1 == "Query9":  
            st.write("What is the average duration of all videos in each channel, and what are their corresponding channel names?")
            mycursor.execute("with gokul as(select channel.channel_name,video.duration from channel right join video on channel.channel_id = video.c_id)\
                 select channel_name,avg(duration) as average_duration from gokul group by channel_name")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['channel_name','average_duration'],index = [i for i in range(1,len(out)+1)])
        else:
            st.write("Which videos have the highest number of comments, and what are their corresponding channel names?")
            mycursor.execute("with gokul as(select channel.channel_name,video.name,video.comment from channel right join video on channel.channel_id = video.c_id)\
                 select channel_name,name,comment from gokul where comment = (select max(comment) from gokul)")
            out = mycursor.fetchall()
            df = pd.DataFrame(out,columns=['channel_name','video_name','comment'],index = [i for i in range(1,len(out)+1)])
        st.dataframe(df)
```

Overall, this Python script enables us to fetch data from YouTube channels, store it in MySQL, and perform analysis on the data.

# Contact
📧 Email: [gokulakkrizhna@gmail.com](mailto:gokulakkrizhna@gmail.com)
🌐 LinkedIn: [linkedin.com/in/gokulakkrizhna-s-241562159](https://www.linkedin.com/in/gokulakkrizhna-s-241562159/)

For any further questions or inquiries, feel free to reach out. We are happy to assist you with any queries.
