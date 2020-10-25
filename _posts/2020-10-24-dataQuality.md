---
layout: post
title: "Data Quality, Young and Promising?"
author: "Jiahui"
categories: blog
tags: [blog, tech]
image: fire-on-car.png
---
# Data Quality

Data has been put into the central of lots of software systems, as data driven methods growing popularity in all kinds of business.Companies are hiring tons of data scientists or data analysts (or other fancy titles doing similar work) to mine information from data and reduce cost or make profit through it. 

We now have a standard software develop process: write code, push differences, deploy, roll back if things are broken, monitoring... With the massive usage of data in many places, people are trying to build something similar for data, in a way that data also should be validated before consuming, be under constant monitoring and potential points of error should be identified as soon as possible.

Data quality is still a relatively new field, not as crowded as many problem scopes. Yet, many big tech companies have built their own solutions, and several entrepreneur s come into this market in recent years. So in this article, I'll start with a summary on the common practice conduct by those existing systems, and also take a look at start-ups streaming into the competition.

## Common Components
### Meta Data
Before doing any meaningful thing using any data, people keep asking question about questions about basic information about this ot that table, for example, the source of the data, what this data should be look like or who should I contact when the data is wrong. These kinds of questions should not be coupled with by going around the office and look for people who is responsible for that, and who is probably not at his/her table every time. 

In 2020s, we should have a more modern way to handle that, and that is the data catalog system, which is not a new thing. 

Commonly, it will provide schema, owner and description about tables across database that users can easily check on a central portal. Moreover, in many metadata system, some additional features are also added depending on different use case. [Uber Databook](https://eng.uber.com/databook/) maintains the freshness and lineage of data, [AirBnB Dataportal](https://medium.com/airbnb-engineering/democratizing-data-at-airbnb-852d76c51770) also has top user information.

Here, I would call out that a lineage graph for tracking how data is transformed in data pipelines is a popular and important part to meta data nowadays. People can dig out an clear insight of data pipelines, statistics and metrics can be generated, and potential issues are always hidden behind it.

### Data Validation
Lots of data engineers, data scientists are suffering from bad data. Where data could go wrong can be the foremost aspect to solve the issue. Data can go wrong in different stage of data processing:
- Infrastructure: when infrastructure is down, data could not be retrieve,     
- Data Pipelines: some data pipeline job can fail due to various reasons.
- Data Structure: things like schema changes.
- Values: things like null values, abnormal data, wrong format.

Regarding data in tables, there're also a lot checks we can do to dig out potential issues:
- range to data: too big? too small?
- format: if it's a date, it should not look like this
- freshness: too old?
- volumes: should be more data here?
- a lot more..

It's tough job to do well in data validation, enforcing rules for tables from your database might not be comprehensive enough and there's always seasonality in values that you have to manually change your rules across time. Some team are trying machine learning or time series model on numerical values, and often have lots of false positive and couldn't get results accurate enough. 

There are also many ways to apply validation depending on different needs. A validation step can be added to many part of the system: before deploy, before analytical work, after data pipeline job and so on. It's definitely an indispensable segment in data quality.

### Circuit Breaker
Circuit breaker, originally a concept from electrical engineering, borrowed by software system to introduce a pattern to act as a protector to prevent disaster caused by malfunction of a single service in a huge system. 

Circuit breaker is used by wrapping all the calls to a service with this circuit breaker object that monitor failures, and when the number of failures reaches a certain limit (or other triggering conditions), the circuit will be open, which means further calls will not reach the service until the problem is fixed and close the circuit.

The same pattern could be leveraged by data quality monitoring system: instead of detecting calls failures and malfunction of a service, the circuit will be open whenever data issue is detected, so that downstream service will not consume the known bad data that can lead to catastrophic ramification.

## The Competition Pool
None of the companies could refuse to follow current trend, and neither do their customers. So you can image, every product is coming with the following features:
- fast, easy, no-code onboarding process
- compatible with main-stream database/warehouses
- cloud based everything

### Databand
[Databand](https://databand.ai/) is building open-source product that manage data pipelines from orchestration, task metadata to alert. If you're using (Apache Airflow)[https://airflow.apache.org/], that should be something interesting and they're making efforts to make Airflow easier to use with several optimizations and Databand as a manager and master for your DAGs.

### Datafold
[Datafold](https://www.datafold.com/) is focus on analytical data, and their tool includes tables diff across database, validations and alerts. The live example app on their website is not so straightforward for new users like me. their main tool is a data version of code diffs: it would be easy for you to compare tables in different stages or versions, in respect of schema, statistics, primary keys... 

It's interesting that the team are trying to reduce pain in analytical work. Let's see if that fits people's need! 

### Great Expectation
[Great Expectation](https://greatexpectations.io/) is an open-source package in Python for setting expectation for setting up expectations about columns in your table like the format of strings and the range of values. It also includes a auto profiling feature for your data that automatically generates expectations and some other tools for configuring, validating and writing docs.

It's certainly a great light-weight tool that helps with working with data, making sure everything is in control, but it only works in Python/Bash for now, and it seems to be not flexible and scalable to provide a unifying solution for long-lasting large systems with lots of data, services and pipelines.

### Monte Carlo
[Monte Carlo](https://www.montecarlodata.com/), a start-up with an interesting name founded by a couple who seems to be quite experienced in data business. they bring up a concept "data downtime" as a measurement of data quality, and they are bringing solutions similar to solving "service downtime" with their observability engine that works from ETL, data warehouse to BI tools and tracks critical metrics about data leveraging machine learning. Their goal is let customers understand data, prevent data downtime, and trust data.

### Toro
ex-Uber, any SQL based, JDBC compliant database
built on top of data warehouse, runs your queries and suggests and collects metrics, with some analytic and alert functionality.

### Soda
[Soda](https://www.soda.io/) is building a product that bring data issues to your team in real time. Their unique feature is that after they find issues in your data, they would prioritize the issues and let people resolve them in order. Not clear about sources of data they work with, they don't have much information on their website. 

### Don't forget those tech giants
Big techs certainly have their homegrown versions of data quality system tailored for their business. Netflix has down versioned data, circuit breaker mechanism for data, safe deployment for data. Google has more than one teams building data quality tools that can help check behavior in data with multiple statistical models and send alerts, other teams choose to onboard to these platforms based on their needs. 

## Tiny Market or Bright Future?
There's not lot start-ups trying to take over data quality business in the market, which gives everyone an opportunity to do something. Moreover, considering nearly all the industries going onto cloud and taking more seriously about data, data quality is coming into a maturer market and is essential in decent data solution. 

However, some companies are already providing softwares that include data integration, data governance and other related stuffs in one place, I see some of them also added data quality component to their features. Similarly, some of the data validation checks can be accomplished by other existing big cloud provider or data warehouse. Even though those features are simpler and incomplete comparing to a nice general data quality solution and can never substitute them, they are satisfying needs of some of the potential customers and also have advantages like great integration and low friction to pick up.

Data workers always talk about pains in data reliability in interviews and articles, why not let softwares take care of that just like how we utilize tools for developing, for deployment or for monitoring? I believe it is a problem to be widely resolved to make their life better, Because Engineers & scientists' happiness matters!

## Reference
- [CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html)
- [Data Discovery in 2020](https://medium.com/toro-data-quality/data-discovery-in-2020-8c85eed328bb)
- [Data Management, Quality and Governance](https://medium.com/analytics-and-data/data-management-quality-and-governance-3082fff40950)
- [Crisis to Calm: Story of Data Validation @ Netflix](https://www.infoq.com/presentations/data-validation-netflix/)