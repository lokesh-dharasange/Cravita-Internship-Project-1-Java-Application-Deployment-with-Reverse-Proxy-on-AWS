# 🚀 Cravita Internship – Project 1: Java Application Deployment with Reverse Proxy on AWS

![AWS](https://img.shields.io/badge/AWS-Cloud-orange)
![Java](https://img.shields.io/badge/Java-Servlet-red)
![Tomcat](https://img.shields.io/badge/Apache-Tomcat-yellow)
![Nginx](https://img.shields.io/badge/Nginx-Reverse%20Proxy-green)
![MySQL](https://img.shields.io/badge/MySQL-RDS-blue)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu-orange)

## 📌 Project Overview

This project demonstrates the deployment of a Java Student Registration Web Application on AWS using two Linux EC2 instances, Apache Tomcat, Nginx Reverse Proxy, and Amazon RDS MySQL.

The Java application is deployed as a WAR file on the backend EC2 instance and is accessed publicly through the Nginx Reverse Proxy.

The backend application is not directly accessible from the Internet. The backend Tomcat Port 8080 is restricted to the Reverse Proxy Security Group.

---

# 🏗️ AWS Architecture Diagram

![AWS Architecture Diagram](screenshots/aws-architecture-diagram.png)

### Architecture Flow

**User Browser → Nginx Reverse Proxy → Backend EC2 / Tomcat → Amazon RDS MySQL**

- User accesses the application through HTTP Port 80.
- Nginx receives the request on the Reverse Proxy EC2.
- Nginx forwards the request to the Backend EC2 using its Private IP.
- Apache Tomcat processes the Java application on Port 8080.
- Java application connects to Amazon RDS MySQL using JDBC on Port 3306.
- Student registration data is stored in the `students` table.
- Response is returned to the user through Nginx.
- Direct public access to Backend Tomcat Port 8080 is blocked.

---

# 🎯 Project Objectives

- Deploy a Java WAR-based web application on AWS.
- Configure Apache Tomcat as the application server.
- Integrate the application with Amazon RDS MySQL.
- Configure MySQL Connector/J.
- Configure Nginx as a Reverse Proxy.
- Use private IP communication between Reverse Proxy and Backend EC2.
- Restrict direct public access to the backend application.
- Provide public application access through the Reverse Proxy.

---

# ☁️ AWS Infrastructure

## 1. Backend EC2

| Property | Value |
|---|---|
| Instance Name | `backend-server` |
| OS | Ubuntu |
| Instance Type | `t3.micro` |
| Public IP | `54.91.220.81` |
| Private IP | `172.31.20.55` |
| Application Server | Apache Tomcat 9 |
| Application Port | `8080` |

## 2. Reverse Proxy EC2

| Property | Value |
|---|---|
| Instance Name | `reverse-proxy` |
| OS | Ubuntu |
| Instance Type | `t3.micro` |
| Public IP | `3.85.90.108` |
| Private IP | `172.31.92.122` |
| Web Server | Nginx |
| Public Port | `80` |

---

# ☕ Java Student Registration Application

The application is a Java Servlet/JSP based Student Registration Web Application.

### WAR File

`student.war`

### Tomcat Deployment Location

`/opt/tomcat/webapps/student.war`

### Extracted Application

`/opt/tomcat/webapps/student/`

### Application Context

`/student`

---

# 🐱 Apache Tomcat Configuration

Tomcat is installed at:

`/opt/tomcat`

Tomcat runs on:

`Port 8080`

### Start Tomcat

`cd /opt/tomcat`

`sudo ./bin/startup.sh`

### Verify Tomcat

`ps aux | grep tomcat`

---

# 🗄️ Amazon RDS MySQL

The Java application uses Amazon RDS MySQL for persistent student data storage.

| Property | Value |
|---|---|
| Engine | MySQL |
| Database | `studentdb` |
| Username | `admin` |
| Port | `3306` |
| Endpoint | `student-db.c41mk4mean95.us-east-1.rds.amazonaws.com` |

---

# 📋 Database Schema

The application uses the `students` table.

SQL:

    CREATE TABLE students (
        student_id INT PRIMARY KEY AUTO_INCREMENT,
        student_name VARCHAR(100) NOT NULL,
        student_addr VARCHAR(255) NOT NULL,
        student_age VARCHAR(20) NOT NULL,
        student_qual VARCHAR(100) NOT NULL,
        student_percent VARCHAR(20) NOT NULL,
        student_year_passed VARCHAR(20) NOT NULL
    );

---

# 🔍 Database Verification

Select the database:

`USE studentdb;`

Check records:

`SELECT * FROM students;`

The application successfully stores and retrieves student registration data from Amazon RDS MySQL.

### Sample Data

| ID | Name | Address | Age | Qualification | Percentage | Year Passed |
|---|---|---|---:|---|---|---:|
| 1 | Lokesh | Pune | 22 | BCA | 8.46 | 2025 |
| 2 | John | Pune | 22 | BCA | 8.01 | 2025 |

---

# 🔌 MySQL Connector/J

MySQL Connector/J is used to connect the Java application with Amazon RDS MySQL.

### Connector

`mysql-connector-j-26.7.0.jar`

### Location

`/opt/tomcat/lib/mysql-connector-j-26.7.0.jar`

### Verify Connector

`ls -lh /opt/tomcat/lib/mysql-connector-j-26.7.0.jar`

---

# 🔗 Tomcat JNDI DataSource

Tomcat JNDI DataSource is configured using:

`jdbc/TestDB`

### Configuration File

`/opt/tomcat/conf/context.xml`

### DataSource Details

- Driver: `com.mysql.cj.jdbc.Driver`
- Database: `studentdb`
- Username: `admin`
- JNDI Name: `jdbc/TestDB`
- Database Port: `3306`

The database password is intentionally not included in this repository.

---

# 🌐 Nginx Reverse Proxy

Nginx is installed on the Reverse Proxy EC2 instance.

### Configuration File

`/etc/nginx/sites-available/student`

### Nginx Configuration

    server {
        listen 80;
        server_name _;

        location /student/ {
            proxy_pass http://172.31.20.55:8080/student/;

            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }

The important point is that Nginx forwards traffic to the Backend EC2 using its **Private IP**:

`172.31.20.55:8080`

---

# 🔗 Enable Nginx Configuration

`sudo ln -s /etc/nginx/sites-available/student /etc/nginx/sites-enabled/student`

---

# ✅ Nginx Configuration Test

Run:

`sudo nginx -t`

Expected result:

`syntax is ok`

`test is successful`

Check Nginx:

`sudo systemctl status nginx`

Expected status:

`active (running)`

---

# 🔐 AWS Security Group Configuration

## Reverse Proxy EC2 Security Group

| Type | Port | Source |
|---|---:|---|
| SSH | 22 | My IP |
| HTTP | 80 | `0.0.0.0/0` |

## Backend EC2 Security Group

| Type | Port | Source |
|---|---:|---|
| SSH | 22 | My IP |
| HTTP | 80 | `0.0.0.0/0` |
| HTTPS | 443 | `0.0.0.0/0` |
| Custom TCP | 8080 | Reverse Proxy Security Group |

The important security rule is:

**Backend Port 8080 → Only Reverse Proxy Security Group**

Therefore, direct Internet access to the backend Tomcat application is blocked.

---

# 🛡️ Security Flow

**Internet**

↓ `HTTP :80`

**Nginx Reverse Proxy**

↓ `Private IP :8080`

**Backend Apache Tomcat**

↓ `JDBC :3306`

**Amazon RDS MySQL**

Direct access:

`http://54.91.220.81:8080/student/`

Result:

`ERR_CONNECTION_TIMED_OUT`

This confirms that the backend application is not publicly accessible.

---

# 🌍 Final Application URL

## Student Registration Application

**http://3.85.90.108/student/**

## Students List

**http://3.85.90.108/student/viewStudents**

The application is accessed through the Nginx Reverse Proxy.

---

# 🧪 Testing and Validation

## 1. Tomcat Running

Command:

`ps aux | grep tomcat`

Result:

**Tomcat running successfully.**

## 2. WAR Deployment

Command:

`ls /opt/tomcat/webapps/`

Expected:

- `student.war`
- `student/`

## 3. Backend Application

Backend application runs internally on:

`172.31.20.55:8080`

## 4. Nginx Configuration

Command:

`sudo nginx -t`

Result:

**Nginx configuration successful.**

## 5. Database

Command:

`SELECT * FROM students;`

Result:

**Student records displayed successfully.**

## 6. Final Application

URL:

`http://3.85.90.108/student/`

Result:

**Student Registration Application working successfully.**

## 7. Backend Security

URL:

`http://54.91.220.81:8080/student/`

Result:

**Direct backend access blocked.**

---

# 📸 Project Screenshots

## 1. AWS EC2 Instances

![AWS EC2 Instances](screenshots/01-ec2-instances.png)

Shows both `backend-server` and `reverse-proxy` EC2 instances.

---

## 2. Tomcat Running

![Tomcat Running](screenshots/02-tomcat-running.png)

Shows Apache Tomcat running successfully on the backend EC2.

---

## 3. WAR Deployment

![WAR Deployment](screenshots/03-war-deployment.png)

Shows `student.war` deployed inside the Tomcat `webapps` directory.

---

## 4. Database Table Structure

![Database Table](screenshots/04-database-table.png)

Shows the `students` table and its required columns.

---

## 5. Database Records

![Database Records](screenshots/05-database-records.png)

Shows student registration records stored successfully in Amazon RDS MySQL.

---

## 6. MySQL Connector/J

![MySQL Connector](screenshots/06-mysql-connector.png)

Shows MySQL Connector/J installed in the Tomcat `lib` directory.

---

## 7. Nginx Reverse Proxy Configuration

![Nginx Configuration](screenshots/07-nginx-config.png)

Shows the Nginx configuration forwarding requests to the backend private IP.

---

## 8. Nginx Configuration Test

![Nginx Test](screenshots/08-nginx-test.png)

Shows successful `nginx -t` validation.

---

## 9. Final Student Registration Application

![Student Registration Application](screenshots/09-student-registration.png)

Shows the Java Student Registration Web Application accessed through the Reverse Proxy.

---

## 10. Backend Direct Access Blocked

![Backend Access Blocked](screenshots/10-backend-blocked.png)

Shows that direct public access to the backend Tomcat Port 8080 is blocked.

---

# 🛠️ Technologies Used

- Amazon EC2
- Amazon RDS MySQL
- Ubuntu Linux
- Java
- Java Servlet
- JSP
- Apache Tomcat 9
- JDBC
- MySQL Connector/J
- Nginx
- AWS Security Groups
- GitHub

---

# 📊 Project Outcome

The Java Student Registration Web Application was successfully deployed on AWS using a two-tier architecture.

### Successfully Implemented

- ✅ Two Linux EC2 instances
- ✅ Backend EC2 for Java application
- ✅ Reverse Proxy EC2 for Nginx
- ✅ Apache Tomcat 9
- ✅ `student.war` deployment
- ✅ Amazon RDS MySQL
- ✅ JDBC database connectivity
- ✅ MySQL Connector/J
- ✅ JNDI DataSource
- ✅ Nginx Reverse Proxy
- ✅ Private IP communication
- ✅ AWS Security Group restriction
- ✅ Public application access through Nginx
- ✅ Student registration
- ✅ Student data storage in MySQL
- ✅ Direct backend access blocked

---

# 🏆 Final Result

The application is successfully available through:

**http://3.85.90.108/student/**

The final request flow is:

**User → Nginx Reverse Proxy → Private Backend Tomcat → Amazon RDS MySQL**

The backend Tomcat server is protected from direct public access, while the application remains publicly accessible through the Nginx Reverse Proxy.

---

# 👨‍💻 Internship Details

**Organization:** Cravita Technology

**Internship:** AWS & DevOps Internship

**Project:** Project 1

**Project Title:** Java Application Deployment with Reverse Proxy on AWS

**Author:** Lokesh Dharasange

---

# ⭐ Key Learning

Through this project, I gained practical experience in AWS EC2 deployment, Apache Tomcat, Java WAR deployment, Amazon RDS MySQL integration, JDBC, Nginx Reverse Proxy configuration, private IP communication, and AWS Security Group based access control.
