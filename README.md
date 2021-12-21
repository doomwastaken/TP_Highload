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
  For average user within 15 000 000 users we expect to do 10 rides, on 12 "active" hours, so 15 000 000 / ( 3 * 12 * 3600 ) = 115 rps, with extra checks and accounting for spikes, we expect 115 * 2 * 2  = 460 rps (for big events, celebrations and weather conditions)

Drivers:
  Stats are negligable compared to users, given 100 000 active drivers, however we expect much heavier traffic, so we expect another 150 rps for monitoring drivers and additional services.
  
Total of 600 rps.

### Traffic:
Overall traffic can be calculated by stats above, however we will have to account for time of day and will be calculating at peak hours.

### Calculation method 2:
If we consider that most people use taxi services to go to and from work, we may expect much higher density of orders, time calculations and order cancellations, aswell as higher amount of working taxi drivers. My assumption would be atleast 4x density of orders and as much as 10x increase in traffic during 7:30am-9:30am and 4:30pm and 7:00pm hours, to a total of 6000 rps for that scenario 

## 4. Logical schema
![schema](https://i.imgur.com/wPecXKn.png)

## 5. Physical schema

We will be splitting user / app data in following ways:
- client, user, order data will be stored in Postgres
- location data, coordinates will be stored in Postgres + PostGIS
- session data for drivers and clients will be stored in Redis

Postgres was chosen due to reliability and ease of data replication, due to value of user data. Redis chosen for session data for perfomance reasons.

We expect each shard to be able to hold ~15k RPS, which means that we require following amount of shards per region:
- Moscow 1 shard
- Saint Petersburg 1 shard
- Central Russia 1 shard
- South Russia 1 shard

As you could see data center were chose based on geographical principle.

To ensure stability and fault resistance we will be keeping 2 replicas per database, in addition we will be storing hystorical data for data analysis and following goverment regulations. Scheme for replication will be following:
* Writing all data to master
* Copying data from master to 2 replicas per master in real time
* Reading data only from replicas

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
![project](https://i.imgur.com/BeB61G1.png)

## 8. Hardware
With 6000 rps for our 15 000 000 potential users and 4 shards, I will assume that third to half of that activity will fall to Moscow region.
Out of 3000 RPS, our split per update is: (location is 9b, longitude 4b + latitude 4b + satelites 1b)
* 10 shares - calculating route - ID 4b + start location 9b + end location 9b + duration 4b + price 4b + time of taxi arrival 4b  + driver id 4b + name 4b * 32 + car number 4b * 9 = 202b + additional debug data + session ids etc - 300b
* 1 share - order taxi = id + location + end location + time + class = 32b
* 4 shares - check taxi location = id + time + location = 16b
* 1 share - rating the driver = id + driver_id + rating = 12b
* 1 share - other operations, such as cancel or changing destination = 600b (due to review text or new order calculation

We assume driver traffic to be much lower and we will increase total traffic by 20% to accomodate for frequent location updates. Total amount of shares - 26 + 20% ~30 or 100rps per share, so we will be multiplying packet size by 100.

Total traffic would be 3 000 000b + 32 000b + 64 000b + 12 000b + 600 000b = 3.7mb/s at peak, however we will need to account for protocols, packet loss, resending packets and additional info that we could be gathering from an app, it would be safe to assume 10mb/s or 864gb (100mb * 3600 * 24) a day, which can be easily managed through a standard 100gb lan at a datacenter. With upto double the amount to entire country traffic, however due to geographical position, size and different timezones that traffic will be spread throughout a day. 

1) Hardware for load balancer.
Load for the balancer is X requests per second. Based on [perfomance tests](https://www.nginx.com/wp-content/uploads/2014/07/NGINX-SSL-Performance.pdf), Nginx with 8 cores can hold about 2k requests with SSL termination. So with 64 cores we can expect load of 16k RPS. In order to hold X requests per second we will need Y servers. For reliability reasons we will double the amount to Z. Drive is for the sole purpose of logs, however these days I would assume there would be some UDP based logger

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 4         | 4          | 100(logs) |   1       |

2) For message broker we will require. [Info](https://engineering.linkedin.com/kafka/benchmarking-apache-kafka-2-million-writes-second-three-cheap-machines)

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 8         | 16         | 500       |   1       |

3) Geolocation service

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 8         | 16         | 100       |   1       |

4) Orders and active orders

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 4         | 8          | 100       |  1        |

5) Backend / gateway service

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 8         | 8          | 100       |  1        |

6) Database with driver data

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 4         | 16         | 5000      | 2         |

7) Database with legacy location data / user location data (previous locations, destinations)

| CPU, Cores|  RAM, Gb   |  SSD, Gb  | Servers   |
|-----------| -----------|-----------|-----------|
| 4         | 4          | 5000      | 4         |

We are talking total of 10 machines, with total of 56 cores, we can safely duplicate that per shard without increasing our maintenance cost.
