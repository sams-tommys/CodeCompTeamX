# Code Comp 2019 - Team X - Scoping
## Brief
***
* Amazon Echo/Alexa Skill interface to solve one of three problems:
  1. Purple Allies - Something that helps able-bodied people help disabled people
  2. Estate Environment - Something to assist Estates in monitoring/improving the environmental impact of HQ.
  3. Gamification of Learning - A skill that teaches something to the user through the use of a game

## Idea
***
### Base Goal:
The broad idea for Team X is to focus on Brief point 2. To create an Alexa interface that would allow Estates to monitor and/or control environmental factors of HQ.

This will comprise of an Alexa skill that can be asked questions about different locations/devices (i.e. "_What is the status of the air conditioning in B block?_") and retreieve back usefull information about the requested service (i.e. "_The air conditioning is currently ON and set to 21 Celsius._")
### Stretch Goals:
1. A stretch goal would be to also include device interactions that **_control_** environmental factors of HQ. For example, an utterance that is able to activate a remote actuator/servo that controls a valve/dial for hot water/gas etc.  
Being able to interact with hardware devices will pose a much greater challenge as the Amazon Lambda function will have to be able to POST data back to site, which may require additional infrastructure. However, this also provides a much greater use case and more valuable product for Estates.

2. Another stretch goal is the ability the poll and record all sensor data in a local database, perhaps periodically synced to the AWS database. This could then be used in order to run analytics such as ```Alexa, how much energy did I use last month?```  
This is a stretch as it is not essential to the final product, but would provide useful bulk feedback for Estates.

### Additional Features/Future Work:
1. Registering a new Device:
    * Adding a new device to the local service is something that a real user would want to do, and thus it may be a good idea to incorporate a Lambda function that can handle and orchestrate this. This will not be essential for an MVP and can therefore be left as an additional feature.
    * This may be impossible to do via Alexa, but could/should be possible through the local service. Architecting a user-friendly way to do this could be difficult.
    * If done solely through the Service, the Lambda function will have to know when the Service has been updated and somehow add the new device to the utterance repertoire. This could be included in the Lambda through an utterance such as ```"Alexa, I've added a new device to HQ"```, which can then fetch new devices from the Service. 

## Architecture
***
### Base Goal:
For the base goal, the sensors and devices to be queried will all need to be registered to a service that the Lambda function can interface with. This will need to be done so that Lambda can access HQ via a single static IP, which can then locally interface with various devices and return readings.  
N.B. For the purposes of demonstration, these sensors could be simulated on the Service itself.

Layout:

![alt text](Architecture.PNG "Example layout")

Example:


```python
Utterance: "Alexa, ask HQ what the temperature is"
                    |
                    V
Lambda: "POST request to get TEMPERATURE from SERVICE at HQ" 
                    |
                    V
Service: "GET information from SENSOR"
                    |
                    V
Sensor: "Gather data and return as response to SERVICE GET request"
                    |
                    V
Service: "Parse SENSOR response and forward to Lambda"
                    |
                    V
Lambda: "Parse SERVICE reposonse and generate Alexa response"
```

With the above example, it makes sense to have the Service hosted on something small-scale and low-power such as a Raspberry Pi. This will be accessed only through an API that the Lambda function can call, and as such should be 'invisible' to the outside world. For a real-world product, this should require authentication - however that can be left as 'future work' as it is not needed for the demonstration. 

### Stretch Goals:

1. One stretch goal is being able to control devices. This could function through much the same way. Local devices that are able to be controled will be registered with the Service, and can be interfaced with via the Lambda. An example could be:

```python
Utterance: "Alexa, water the Garden at HQ"
                    |
                    V
Lambda: "POST request to SERVICE to activate ACTUATOR at HQ"
                    |
                    V
Service: "POST instruction to ACTUATOR"
                    |
                    V
Actuator: "Perform action and respond 200 OK"
                    |
                    V
Service: "Respond 200 OK to Lambda"
                    |
                    V
Lambda: "Parse response and generate Alexa reponse"
```

2. The second stretch goal of being able to record and store sensor data in order to perform analytics on, could be much harder. The number crunching for these analytics could probably happen within the Lambda function itself, and will therefore not need to interface with the API. However, sending data from the Raspberry Pi database to the AWS database will require an autonomous communication between the RPi and Lambda. On top of this, if this ends up requiring large amounts of compute, it could increase running costs.  
If this is not possible, it should be possible to perform the analytics on the Raspberry Pi locally, and only retrieve these analytics when a specific question is asked through Lambda. 

## Breakdown
***
All goals will require an API being hosted on local infrastructure. Since the focus of this project is on being environmentally friendly, something low-power like a Raspberry Pi makes sense.  
For the sensors, something even lower power and more specific would make sense, such as an Arduino Nano 33 IoT. However, for the purposes of demonstration, the sensors could be integrated into, or simulated on, the Raspberry Pi.

This API will need to be well documented such that every team member knows how to implement each part of the API into their section of the project. It would be a good idea to keep a document on the repository that specifically details every API endpoint.  
It is probably a good idea for each person working on a specific sensor, to generate and document the corresponding API endpoint, as they will be the ones that understand how the specific sensor works.

It makes sense to host all of the project, along with doumentation, on Git. This allows team members to have a consolidated view of the project as a whole, as well as contributing and managing versions efficiently. 

Some of the sections of the project include:  
1. Identify and construct different intents for Alexa cloud
2. Configuring the Alexa cloud for our specific requests
3. Configuring the Lambda function to be able to handle specific request and perform API calls
4. Configuring the API such that it can interface with the Lambda funcion and respond appropriately.
5. Configure Raspberry Pi as a server, to have a given IP/Hostname such that it can be called from Lambda 
6. Configuring the Sensor hardware such that they can interface with the Raspberry Pi and send/record data appropriately. 
7. Designing the AWS Database such that it can store appropriate requests to be sent depending on the Alexa input
8. Designing the Local Database such that is can store sensor data in an appropriate way to run analytics over 
