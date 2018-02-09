---
title: Search Partition (Problem)
menu:
  main:
      parent: Search Partition
      identifier: spproblem
      weight: 401
---

## Problem Definition

Customer is building a messaging system - quite like iMessage. The requirements are these

Allow for 500 million users.

Each user can have thousands of messages in hundreds of separate conversations. Each message is small as it only contain text. 
 
There is a requirement to allow for searching over messages, its expected that these will be less that 1% of the queries

* Search for all messages in a conversation between 2 dates and contains a particular word. The SLA for this is 100ms.

* Search for all messages in all conversations between 2 dates and contains a particular word. The SLA for this is 500ms. 

Note: more search requirements will come in the future when they start to see what users want to search with. 

* The customer would like at least 1TB per node. 
* They also want to have strong consistency on all searches.
* They have considered reading all the messages into memory in the app but are concerned both with the amount of network and ram resources this will consume as well as the resulting GC to clean up after the search operations. Doing all the search in the app code also makes it harder to test so they would like to be able to run it on a command line or notebook. 

