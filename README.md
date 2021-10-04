# TP_Highload
Highload service example, nation wide taxi service MVP

Data taken from yandex research from 2012, 2015 and their blogs from 2016-2020, some data is calculated manually.

Disclaimer: This is meant to be an educational project in order to get knowledge in fields of analytics, system design and extending my knowledge of python (and yes I am aware that C/C++/Rust even GO would be preferable, however I am famillar with those already :)).

## Audience

- Russian federation: Large cities (over 1 million population) and expanding into smaller citiest afterwards
- Active users: 15 million. With more than 10 rides a month, 40 million accounting semi-regular users.
- Age: 18-60
- Additional revenue streams: Deliveries
- Drivers: 100 thousand. Including part time drivers

## MVP

- User signup and sign in
- Driver signup and sign in
- Ordering a cab as a user
- Recieving orders as a driver
- Cancelling order as a user
- Finishing ride as a driver

## Traffic calculation

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
