# TP_Highload
Highload service example, nation wide taxi service MVP

Data taken from yandex research from 2012, 2015 and their blogs from 2016-2021, some data is calculated manually.

Disclaimer: This is meant to be an educational project in order to get knowledge in fields of analytics, system design and extending my knowledge of python (and yes I am aware that C/C++/Rust even GO would be preferable, however I am famillar with those already :)).

## 1. Audience

- Russian federation: Large cities (over 1 million population) and expanding into smaller citiest afterwards
- Active users: 15 million. With more than 10 rides a month, 40 million accounting semi-regular users.
- Age: 18-65 (working population)
- Additional revenue streams: Deliveries
- Drivers: 100 thousand. Including part time drivers

## 2. MVP

- User signup and sign in
- Driver signup and sign in
- Ordering a cab as a user
- Recieving orders as a driver
- Cancelling order as a user
- Finishing ride as a driver

## 3. Traffic calculation

### According to research, taxi market equals:

Average user using taxi service atleast 10 times a month.
### Scenario for user app use (in a day):
* Calculating route: 2 times
* Order taxi: 1 time
* Check taxi location: 2 times
* Rating driver: 0.5 times
* Cancelling order: 0.05 times
* Changing destination: 0.05 times

### Scenario for driver app use (in a day):
* Shift start : 1.5 times
* Receiving orders: 12-24 times ~18 times, considering third of them being cancelled
* Updating driver coordinatest: 1 time per 60 seconds, average shift 6 hours ~360 times
* Shift end: 1.5 times

As a result we have following statistics:

### RPS depending on request type
Users:
* Calculating route ~ 
* Order taxi ~ 
* Check taxi location ~ 
* Rating driver ~ 
* Cancelling order ~ 
* Changing destination ~ 
Drivers:
* Shift start ~ 
* Receiving orders ~ 
* Updating driver coordinatest ~ 
* Shift end ~ 

### Traffic:
Overall traffic can be calculated by stats above, however we will have to account for time of day and will be calculating at peak hours.

## 4. Logical schema
![schema](https://i.imgur.com/wPecXKn.png)

## 5. Physical schema

We will be splitting user / app data in following ways:
- client, user, order data will be stored in Postgres
- location data, coordinates will be stored in Postgres + PostGIS
- session data for drivers and clients will be stored in Redis

Postgres was chosen due to reliability and ease of data replication, due to value of user data. Redis chosen for session data for perfomance reasons.

We expect each shard to be able to hold ~15k RPS, which means that we require following amount of shards per region:
- Moscow 6 shards
- Saint Petersburg 3 shards
- Western Russia 3 shards
- Central Russia 2 shards
- Eastern Russia 1 shard
- South Russia 2 shards
- Northern Russia 1 shard

As you could see data center were chose based on geographical principle.

To ensure stability and fault resistance we will be keeping 2 replicas per database, in addition we will be storing hystorical data for data analysis and following goverment regulations. Scheme for replication is master - replica. 

## 6. Technologies
Will be going with "default" stack of technologies

### Backend
Microservices on Golang, gRPC for connection with services, data from location services can be cached and pushed via subscriptions to minimize amount of API calls

### Frontend
React + Redux + Redux Saga + Material UI
React Native can be considered for rendering on mobile devices, however above combo can be one of the most efficient combos for delivering the MvP

### QA
Golang allows us easily cover code with Unit tests. 
For frontend storybook + selenium + snapshot + screenshot autotests can be implemented. 
We believe in automation so in order to see the stability of system, we will require some sort of "mock" service for gRPC to act as a middleware (for testing only!) in order to implement rare scenarios/delays based on client location. Load testing and other necessities. We expect spikes in service use during events, harsh weather conditions etc.
As a part of QA, CI/CD / documentation services will be implemented. P.S. based on amount of data here, you could guess that I love QA <3

## 7. Project scheme

## 8. Hardware
1) Hardware for load balancer.
Load for the balancer is X requests per second. Based on [perfomance tests](https://www.nginx.com/wp-content/uploads/2014/07/NGINX-SSL-Performance.pdf), Nginx with 8 cores can hold about 2k requests with SSL termination. So with 64 cores we can expect load of 16k RPS. In order to hold X requests per second we will need Y servers. For reliability reasons we will double the amount to Z. 

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 32        | 16         |           |           |

2) For message broker we will require. [Info](https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines)

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 16        | 32         | 500       |           |

3) Geolocation service

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 32        | 32         | 500       |           |

4) Orders and active orders

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 32        | 32         | 500       |           |

5) Backend / gateway service

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 32        | 32         | 500       |           |

6) Database with driver data

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 32        | 32         | 500       |           |

7) Database with legacy location data / user location data (previous locations, destinations)

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 32        | 32         | 5000      |           |
