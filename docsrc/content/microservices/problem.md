---
title: Microservices (Problem)
menu:
  main:
      parent: Microservices
      identifier: msproblem
      weight: 301
---

## Problem Definition

A company is creating a new MicroServices architecture and DSE will be the platfrom for some of the cross cutting services that are needed.

DSE will be responsible for 

* Event/Command sourcing
* Access Token
* Log Aggregation
* Application Metrics 
* Audit logging 
* Distributed tracing
* Exception tracking

A major concern for the company is how to do distributed transactions over mulitple services. 
E.g. 
Payment service puts either debits an account or credits and account. So an internal payment takes 2 service calls that are independent of each other. How can be use DSE to 

  1. Ensure that we understand the state of an internal payment transaction.
  2. Allow an admin user to replay a transaction or part of a transaction if there had been problem with a payment.
