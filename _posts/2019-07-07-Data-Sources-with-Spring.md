---
title: "Data Sources for Spring Apps"
toc: true
toc_sticky: true
excerpt: "How to spin up a Postgres Server on Azure, deploy a Spring Boot app to App Service, and connect the two together."
header:
    teaser: "assets/img/data-graph-header.jpg"
    og_image: "assets/img/data-graph-header.jpg"
    overlay_image: "assets/img/data-graph-header.jpg"
    overlay_filter: 0.4
teaser: "assets/img/data-graph-header.jpg"
tags:
    - Azure
    - Java
    - App Service
---

This article shows how to spin up a Postgres Server on Azure, deploy a Spring Boot app to App Service, and how to connect the app to the database. The [next article]({{ site.baseurl }}{% post_url 2019-07-07-Key-Vault-References-with-Spring %}) shows how to reconfigure this app to use Key Vault references.

> To do this tutorial, you will need [Maven 3](https://maven.apache.org/download.cgi), [Java 8](https://www.azul.com/downloads/zulu/), and the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest) installed locally.

## Get the code

The repository for this tutorial has a few Java tutorials for App Service. Once the project is cloned, `cd` into the directory for `data-sources`. This tutorial starts from the code in the `initial/` directory. If you get stuck, take a peak at the `/complete/` directory, which has the completed code.

```bash
git clone https://github.com/Azure-Samples/java-on-app-service.git
cd java-on-app-service/data-sources/initial/
```

## Run Locally

Our Spring Boot application uses Spring Data JPA to map our Java objects to relational objects in the PostgreSQL database. Under the hood, Spring Data JPA uses Hibernate, an implementation of the JPA specification. This means we use Spring's annotations and interfaces instead of  explicitly writing SQL commands in our application. This project is configured with an in-memory H2 database for local development and testing. To run the app locally, run the following commands from the `initial/` directory.

```bash
mvn clean package
java -jar target/app.jar
```

Open a browser to `http://localhost:8080/` and you will see a simple link to create a new product. Try creating a new product with an example description, price, and image URL. You can see the SQL commands in the terminal. If you kill the app and restart it, the contents will be lost because H2 is an in-memory database. You can see all the items in the database by browsing to `http://localhost:8080/product/list`

> When we deploy to App Service, the application will instead listen on port 80.

## Provision the Server

You can provision a Postgres database through the Portal by going to Create a Resource and searching for "Azure Database for PostgreSQL". In the following screen enter your desired server name, admin username, and password.

Alternatively, you can use the following Azure CLI command to create the database by replacing the placeholder values. To get a full list of all the Azure locations (aka "regions"), run `az account list-locations`. Don't forget your username and password!

```bash
az postgres server create
    -n <desired-name> \
    -g <resource-group> \
    --sku-name B_Gen5_1 \
    -u <username> \
    -p <password> \
    -l centralus
```

### Configure the Server

Right now the server requires SSL to connect, does not allow access to Azure services, and does not allow access from our client machine. In the portal, navigate to the "Connection Security" portion of your PostgreSQL Server and update the following:

- "Allow access to Azure services" should be set to **on**.
- Click "Add client IP" (near the Save and Discard buttons).
- Finally set "Enforce SSL connection" to **disabled**.

Don't forget to click the save button!

### Create a Database

We have created a PostgreSQL Sever, and now we need to create a database in the server. You can run the following psql commands from your local machine, or from the [Azure Cloud Shell](https://azure.microsoft.com/en-us/features/cloud-shell/).

1. Run the following psql command to connect to an Azure Database for PostgreSQL database:

    ```bash
    psql --host=<servername> \
        --port=<port> \
        --username=<username>@<servername> \
        --dbname=<dbname>
    ```

    For example, the following command connects to the default database called "postgres" on a PostgreSQL server named "mydemoserver.postgres.database.azure.com". Enter the `<server_admin_password>` you chose when prompted for the password.

    ```bash
    psql --host=mydemoserver.postgres.database.azure.com \
        --port=5432 \
        --username=myadmin@mydemoserver \
        --dbname=postgres
    ```

2. Once you are connected to the server, create a blank database at the prompt:

    ```sql
    CREATE DATABASE postgres_demo;
    ```

## Configure the App

To get this application working with our database, we just need to configure Spring Data JPA via the `application.properties` file. This project already has a second Maven profile configured for use with Postgres. This allows us to build our app with a different configuration for when we deploy to App Service.

### Connection Strings

It is bad practice to put connection strings in source control, so we will put this information in environment variables and inject them into our application at build time.

1. First, create the following environment variables with the corresponding values.
    - `POSTGRES_URL`: The full URL of the PostgreSQL database: `<servername>.postgres.database.azure.com:5432/postgres_demo?sslmode=require`
    - `POSTGRES_USERNAME`: The username you used appended with `@<servername>`. For example: `username@mydemoserver`
    - `POSTGRES_PASSWORD`: The password you used when provisioning the PostgreSQL server

    See these instructions to set environment variables on [Windows](https://www.techjunkie.com/environment-variables-windows-10/) and [Mac](http://osxdaily.com/2015/07/28/set-enviornment-variables-mac-os-x/).

1. Paste the following snippet into the `production` profile of the `pom.xml`. We will use this profile to build our application before deploying to App Service.

    ```xml
    <spring.datasource.url>${POSTGRES_URL}</spring.datasource.url>
    <spring.datasource.username>${POSTGRES_USERNAME}</spring.datasource.username>
    <spring.datasource.password>${POSTGRES_PASSWORD}</spring.datasource.password>
    ```

## Build and Deploy

We will now build our app and deploy it to App Service.

1. Replace the resource group and application name in the App Service Maven plugin with your own values. (You can also change the location property to a region closer to you.)

    ```xml
    ...
    <resourceGroup>YOUR RESOURCE GROUP</resourceGroup>
    <appName>YOUR APP NAME</appName>
    ...
    ```

1. Build your app with the "production" profile and deploy it with the command: `mvn clean package -Pproduction`. If you navigate to `target/classes/application.properties` you should see the Postgres username, password, and URL.

1. Finally, deploy the application with `mvn azure-webapp:deploy`. In the future, you can chain these commands like this...

    ```bash
    mvn clean package -Pproduction azure-webapp:deploy
    ```

Browse to your application and you will see the same app now running on App Service using Azure PostgreSQL! You can confirm the application is using Postgres by querying the database with psql or pgadmin, or by restarting the app and confirming the entries are still shown.m

## Next steps

The [next article]({{ site.baseurl }}{% post_url 2019-07-07-Key-Vault-References-with-Spring %}) shows how to store your connection strings in Key Vault, thus providing easy secret management and rotation! In this example we disabled SSL, but for a production environment we should install the Postgres certificate on App Service so we can connect securely. I plan to cover this in a future article, in the meantime please see this documentation on [installing the Postgres certificate on your local machine](https://docs.microsoft.com/en-us/azure/postgresql/concepts-ssl-connection-security#applications-that-require-certificate-verification-for-ssl-connectivity).

## Helpful Links

- [Getting Started with Azure PostgreSQL](https://docs.microsoft.com/en-us/azure/postgresql/tutorial-design-database-using-azure-cli)
- [Full list of Spring Boot Starters](https://github.com/spring-projects/spring-boot/tree/master/spring-boot-project/spring-boot-starters)
- [Relationship between Spring Data JPA, JPA, and Hibernate](https://thoughts-on-java.org/what-is-spring-data-jpa-and-why-should-you-use-it/)
