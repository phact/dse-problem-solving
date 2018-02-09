---
title: Data Deletion (Problem)
menu:
  main:
      parent: Data Deletion
      identifier: ddproblem
      weight: 501
---

## Problem Definition

Customer wants to delete data and retrieve space. 

The application is a typical IoT use case which keeps data for a certain amount of time and then moves it to long term storage. 
The current requirement is to hold the data for 30 days. So each day - they copy day 31 to long term storage and delete. 30 days storage is estimated at 9 TB of data over 10 nodes. 

The number of days may change at any time due to requirements so the data must be moved manually and cannot use a TTL. 

The customer is currently in testing and when they delete the data (using spark), the data usage goes up instead of down. They are concerned that they won’t have enough head room and are really disappointed that they can’t delete the data and reclaim space. 

The application is 99.9999% write and data is kept more for audit purposes and data is usually queried by sensor id and date range. The SLA for reads is quite low because its driven from a UI. 
