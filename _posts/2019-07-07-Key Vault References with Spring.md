---
title: "Azure Key Vault References with Spring Apps"
tags:
    - Azure
    - Java
    - App Service
---

Azure Key Vault provides a centralized service for managing secrets and certificates with full control over access policies and auditing capabilities. This article will show how to wire up a Spring Boot application on App Service to read a database username, password, and URL from Key Vault. Using Key Vault references requires little to zero code changes, but will require some configuration acrobatics.

> See [the previous article]({{ site.baseurl }}{% post_url 2019-07-07-Data Sources with Spring %}) for instructions on setting up the Postgres Server and deploying the app to App Service.

## 