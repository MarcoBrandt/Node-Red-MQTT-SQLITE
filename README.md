# Node-Red-MQTT-SQLITE v 1.0.0
Simple Node-Red program to read from a MQTT input ja store it in a database. This is a part of a school project that I worked on, but you can easily use it for other purposes. You can use it as a cheap burglar alarm, a bluetooth id scanner or just use it like we did and try to estimate human traffic.

Install Node-red: https://nodered.org/#get-started

Install new nodes to your Node-Red pallette: 
- Dashboard (node-red-dashboard) 
- Dashboard table (node-red-ui-table) 
- SQlite (node-red-node-sqlite)

Import the flows.json to your Node-Red and check the numbered notes in the flow screen.

First you need a source for your MQTT, I used a ESP32 that sends ID data and separates each ID with a new line. Then the ID data is stored in the SQlite database along with the date, time and MQTT topic (this comes from the MQTT broker). To start the database you need to press the inject node CREATE TABLE, so the prosess can start. This is a workaround to create the table and file, might be worth to fix in the future.

![alt text](https://github.com/MarcoBrandt/Node-Red-MQTT-SQLITE/blob/main/images/Screenshot%202020-12-12%20at%2014.29.16.png)

Function node code for SQlite insertion:
``` 
var out = "INSERT INTO BTdata (date, time, topic, data)"
var today = new Date()
var time = today.toLocaleTimeString();
var date = today.getDate()+'.'+(today.getMonth()+1)+'.'+today.getFullYear();

out = out + "VALUES ('" + date + "','" + time + "','" 
out = out + msg.topic + "','" + msg.payload + "');"
    
msg.topic=out;

return msg;
```

After the data is stored in the database it can be read from multiple http end-points:
![alt text](https://github.com/MarcoBrandt/Node-Red-MQTT-SQLITE/blob/main/images/Screenshot%202020-12-12%20at%2014.31.16.png)


Endpoint | What does it do?
------------ | -------------
/data | Fetches all the data, JSON dump
/data/all/count | Counts all IDs from the database
/data/all/unique | Fetches all data from devices with Unique ID
/data/all/uniquedata |  Fetches data in date, time and data format from devices with Unique ID
/data/all/unique/count | Counts all Unique IDs
/data/today |  Fetches all data from today
/data/today/count | Counts all IDs from today
/data/today/unique | Fetches all data from today that has an Unique ID
/data/today/uniquedata | Fetches data from today in date, time and data format from devices with Unique ID 
/data/today/unique/count | Counts all Unique IDs from today
/data/week | Fetches all data from last week
/data/week/count | Counts all IDs from last week
/data/week/unique | Fetches all data from last week that has an Unique ID
/data/week/uniquedata | Fetches data from last week in date, time and data format from devices with Unique ID
/data/week/unique/count | Counts all Unique IDs from last week
/data/livefeed | Gets the latest scan information

## Dashboard

Dashboard nodes with timed injections. Live feed updates every 5 seconds, the rest updates once per minutes. For this to work you need Dashboard and Dashboard tables installed on your Node-Red.

![alt text](https://github.com/MarcoBrandt/Node-Red-MQTT-SQLITE/blob/main/images/Screenshot%202020-12-12%20at%2014.32.05.png)

Go to the address: localhost:1880/ui/#/0

![alt text](https://github.com/MarcoBrandt/Node-Red-MQTT-SQLITE/blob/main/images/Screenshot%202020-12-12%20at%2014.42.50.png)


