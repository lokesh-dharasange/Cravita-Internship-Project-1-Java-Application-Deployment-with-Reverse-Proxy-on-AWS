# Cravita Internship – Project 1: Java Application Deployment with Reverse Proxy on AWS

## Project Overview

This project demonstrates the deployment of a Java Student Registration Web Application on AWS using a two-tier EC2 architecture with Nginx as a reverse proxy and Amazon RDS MySQL as the database.

The Java application is deployed as a WAR file on Apache Tomcat and runs internally on port `8080`. Nginx is configured on a separate EC2 instance and is the only publicly accessible entry point for the application.

---

## Project Objective

The main objectives of this project are:

- Deploy a Java WAR-based web application on AWS.
- Host the Java application using Apache Tomcat.
- Integrate the application with Amazon RDS MySQL.
- Configure MySQL Connector/J in Tomcat.
- Deploy Nginx as a reverse proxy on a separate EC2 instance.
- Allow public access only through the reverse proxy.
- Restrict direct public access to the backend Tomcat server.
- Verify student registration and database integration.

---

## Architecture

```text
                    Internet / User
                           |
                           |
                    HTTP Port 80
                           |
                           v
              +-------------------------+
              |   Reverse Proxy EC2     |
              |        Nginx            |
              |     Public IP           |
              |      3.85.90.108        |
              +-----------+-------------+
                          |
                          | Private IP
                          | Port 8080
                          v
              +-------------------------+
              |      Backend EC2        |
              |     Apache Tomcat       |
              |      Port 8080          |
              |     Private IP          |
              |    172.31.20.55         |
              +-----------+-------------+
                          |
                          | MySQL
                          | Port 3306
                          v
              +-------------------------+
              |     Amazon RDS MySQL    |
              |       studentdb         |
              +-------------------------+
