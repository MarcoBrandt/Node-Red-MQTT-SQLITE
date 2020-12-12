# Node-Red-MQTT-SQLITE v 1.0.0
Simple Node-Red program to read from a MQTT input ja store it in a database.

Install Node-red: https://nodered.org/#get-started

Install new nodes to your Node-Red pallette: Dashboard, Dashboard table and SQlite.

Import the flows.json to your Node-Red and check the numbered notes in the flow screen.

First you need a source for your MQTT, I used a ESP32 that sends ID data and separates each ID with a new line. Then the ID data is stored in the SQlite database along with the date, time and MQTT topic (this comes from the MQTT broker). To start the database you need to press the inject node CREATE TABLE, so the prosess can start. This is a workaround to create the table and file, might be worth to fix in the future.

After the data is stored in the database it can be read from multiple http end-points:

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

## Dashboard

Go to the address: localhost:1880/ui/#/0
