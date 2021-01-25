---
title: "A/B Testing with App Service"
excerpt: "How to do A/B testing with App Service and Application Insights"
---

Before coming to Microsoft and working on Azure App Service, I interned in data science and business analytics roles. I majored in Statistics at UC Santa Barbara and started the Data Science club. So when I started working on App Service, I noticed that the Deployment Slots feature could be used to A/B test applications. But I was surprised there wasn't a ton of concrete documentation on how to do it.

I've started working on a series of tutorials with my teammate, [Shubham Dhond](https://www.linkedin.com/in/shubham-dhond-04b6a6a4/). The tutorials below about how to A/B test your web and API applications with App Service's deployment slots. The guide uses Application Insights, but you *could* use any APM. The tutorials require some manual configuration that we eventually want to built into the App Insights SDK itself. If you find this interesting, please let us know your thoughts in the comments section on the articles.

- [Part 1: Client-side configuration](https://azure.github.io/AppService/2020/08/03/ab_testing_app_service.html)
- [Part 2: Backend configuration](https://azure.github.io/AppService/2020/08/24/ab_testing_app_service2.html)
- [Part 3: Analyzing the results](https://azure.github.io/AppService/2020/10/20/ab_testing_app_service3.html)
