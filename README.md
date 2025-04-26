- Design a notification system
  - Support email sending
  - A Notification is defined by a content (i.e. JSON), a list of recipients (1..n) and an interval (0, 30min, 1day)
    - The interval defines the time-range when a message will be sent as a slice of a day
      - Sending a notification between 1:00AM and 1:30AM with a 30min interval will generate one email and send it at 1:30AM
      - Sending a notification at 1:05AM with a 24h (1440 min) interval will generate one email and send it at midnight
  - The service is design for internal usage (i.e., message producers are Netskope services)
  - Notifications should be grouped by recipient and interval
    - We want to send all notifications received for a same recipient, for a same interval in a time period (30min or 1 day) at the same time in the same email

Ex:
At 1:05pm:
1. n1 = (r: (a,b), i:0) i=interval
2. n2 = (r: (a,b), i:30)
3. n3 = (r: (b), i:30)
4. n4 = (r: (a), i:1440)

At 1:06pm:
5. n5 = (r: (b), i:30)

- At 1:05pm:
  - Email 1: a, n1
  - Email 2: b, n1

At 1:29 am
6. n6 = (r: (b), i:30)

- At 1:30pm:
  - Email 3: a, [n2]
  - Email 4: b, [n2, n3, n5]

- At 12:00am:
  - Email 5: a, n4

requirement:
group by time intreval
group by user

question:
would u just send to queue? without web service?
how do u do scale out with worker
why don't use grpc
u use hashTable in redis, what if 2 service udpate a key in once time

Soloution:

30 min for a interval
24hr 48 intervals

design 48 queue
n2 a, n2 b -> queue (1:30)
worker deque(group emails by user) -> storage


HA
Saclibility
Optimize 48 redis

api design
[Post]  /api/v1/email
n1 (a,b) reply timeout, retry can't send one more
Id = client ip + timestamp + recipients

how would u scale worker

interval userId email status
paritionKey: interval
cluster key: all column
