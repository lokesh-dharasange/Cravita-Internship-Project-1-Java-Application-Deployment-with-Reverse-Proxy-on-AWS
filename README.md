# Cravita Internship – Project 1: Java Application Deployment with Reverse Proxy on AWS

## 📌 Project Overview

This project demonstrates the deployment of a Java Student Registration Web Application on AWS using two Linux EC2 instances, Apache Tomcat, Nginx Reverse Proxy, and Amazon RDS MySQL.

The Java application is deployed as a WAR file on the backend EC2 instance and is accessed publicly through the Nginx Reverse Proxy.

---

## 🏗️ AWS Architecture Diagram

![AWS Architecture Diagram](screenshots/aws-architecture-diagram.png)

### Architecture Flow

```text
                    ┌─────────────────────┐
                    │    User / Browser   │
                    └──────────┬──────────┘
                               │
                         HTTP :80
                               │
                               ▼
              ┌─────────────────────────────┐
              │     Reverse Proxy EC2      │
              │          Nginx             │
              │                            │
              │ Public IP: 3.85.90.108    │
              │ Private IP: 172.31.92.122 │
              │ Port: 80                   │
              └──────────────┬──────────────┘
                             │
                       Private IP :8080
                             │
                             ▼
              ┌─────────────────────────────┐
              │        Backend EC2         │
              │      Apache Tomcat 9       │
              │                            │
              │ student.war                │
              │                            │
              │ Public IP: 54.91.220.81    │
              │ Private IP: 172.31.20.55  │
              │ Port: 8080                 │
              └──────────────┬──────────────┘
                             │
                         JDBC :3306
                             │
                             ▼
              ┌─────────────────────────────┐
              │       Amazon RDS MySQL     │
              │                            │
              │ Database: studentdb        │
              │ Table: students            │
              │ Port: 3306                 │
              └─────────────────────────────┘

      Direct Backend Access :8080
                    │
                    X
                 BLOCKED
```

---

## 🔄 Application Flow

1. User opens the public application URL.
2. Request reaches the Nginx Reverse Proxy EC2 on HTTP Port 80.
3. Nginx forwards the request to the Backend EC2 using its private IP on Port 8080.
4. Apache Tomcat processes the Java application.
5. The Java application connects to Amazon RDS MySQL using JDBC.
6. Student data is stored and retrieved from the `students` table.
7. Response is returned to the user through Nginx.

---

## 🎯 Project Objectives

- Deploy a Java WAR-based web application on AWS.
- Configure Apache Tomcat as the application server.
- Integrate the application with Amazon RDS MySQL.
- Configure MySQL Connector/J.
- Configure Nginx as a Reverse Proxy.
- Use private IP communication between Reverse Proxy and Backend EC2.
- Restrict direct public access to the backend application.
- Provide public access through the Reverse Proxy.

---

## ☁️ AWS Infrastructure

### Backend EC2

| Property | Value |
|---|---|
| Instance Name | backend-server |
| OS | Ubuntu |
| Instance Type | t3.micro |
| Public IP | 54.91.220.81 |
| Private IP | 172.31.20.55 |
| Application Server | Apache Tomcat 9 |
| Application Port | 8080 |

### Reverse Proxy EC2

| Property | Value |
|---|---|
| Instance Name | reverse-proxy |
| OS | Ubuntu |
| Instance Type | t3.micro |
| Public IP | 3.85.90.108 |
| Private IP | 172.31.92.122 |
| Web Server | Nginx |
| Public Port | 80 |

---

## ☕ Java Application

Application:

`student.war`

Deployment location:

`/opt/tomcat/webapps/student.war`

After deployment:

`/opt/tomcat/webapps/student/`

Application Context:

`/student`

The application is a Java Servlet/JSP based Student Registration Web Application.

---

## 🐱 Apache Tomcat

Tomcat installation directory:

`/opt/tomcat`

Tomcat runs on:

`Port 8080`

Start Tomcat:

```bash
cd /opt/tomcat
sudo ./bin/startup.sh
```

Verify Tomcat:

```bash
ps aux | grep tomcat
```

---

## 🗄️ Amazon RDS MySQL

| Property | Value |
|---|---|
| Engine | MySQL |
| Database | studentdb |
| Username | admin |
| Port | 3306 |
| Endpoint | student-db.c41mk4mean95.us-east-1.rds.amazonaws.com |

---

## 📋 Database Table

The application uses the `students` table.

DDL:

```sql
CREATE TABLE students (
    student_id INT PRIMARY KEY AUTO_INCREMENT,
    student_name VARCHAR(100) NOT NULL,
    student_addr VARCHAR(255) NOT NULL,
    student_age VARCHAR(20) NOT NULL,
    student_qual VARCHAR(100) NOT NULL,
    student_percent VARCHAR(20) NOT NULL,
    student_year_passed VARCHAR(20) NOT NULL
);
```

---

## 🔍 Database Verification

Select the database:

```sql
USE studentdb;
```

Check student records:

```sql
SELECT * FROM students;
```

The application successfully stores and retrieves student registration data from Amazon RDS MySQL.

---

## 🔌 MySQL Connector/J

MySQL Connector/J is used by Tomcat to connect the Java application with Amazon RDS MySQL.

Connector file:

`mysql-connector-j-26.7.0.jar`

Location:

`/opt/tomcat/lib/mysql-connector-j-26.7.0.jar`

Verify:

```bash
ls -lh /opt/tomcat/lib/mysql-connector-j-26.7.0.jar
```

---

## 🔗 Tomcat JNDI DataSource

Tomcat JNDI DataSource is configured using the resource name:

`jdbc/TestDB`

Configuration file:

`/opt/tomcat/conf/context.xml`

The DataSource uses:

- MySQL JDBC Driver
- Amazon RDS endpoint
- Database `studentdb`
- Username `admin`
- JNDI name `jdbc/TestDB`

Database password is intentionally not included in this repository.

---

## 🌐 Nginx Reverse Proxy

Nginx is configured on the Reverse Proxy EC2 instance.

Configuration file:

`/etc/nginx/sites-available/student`

Configuration:

```nginx
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
```

The Reverse Proxy forwards requests to the backend using the backend EC2 private IP:

`172.31.20.55:8080`

---

## 🔗 Enable Nginx Configuration

```bash
sudo ln -s /etc/nginx/sites-available/student /etc/nginx/sites-enabled/student
```

---

## ✅ Nginx Configuration Test

Run:

```bash
sudo nginx -t
```

Expected result:

`syntax is ok`

`test is successful`

Check Nginx:

```bash
sudo systemctl status nginx
```

Expected status:

`active (running)`

---

## 🔐 Security Group Configuration

### Reverse Proxy EC2 Security Group

| Type | Port | Source |
|---|---:|---|
| SSH | 22 | My IP |
| HTTP | 80 | 0.0.0.0/0 |

### Backend EC2 Security Group

| Type | Port | Source |
|---|---:|---|
| SSH | 22 | My IP |
| HTTP | 80 | 0.0.0.0/0 |
| HTTPS | 443 | 0.0.0.0/0 |
| Custom TCP | 8080 | Reverse Proxy Security Group |

The backend Tomcat Port 8080 is allowed only from the Reverse Proxy Security Group.

Therefore, users cannot directly access the backend application from the Internet.

---

## 🛡️ Security Architecture

```text
Internet
    │
    │ HTTP :80
    ▼
Nginx Reverse Proxy
    │
    │ Private IP :8080
    ▼
Backend Apache Tomcat
    │
    │ JDBC :3306
    ▼
Amazon RDS MySQL
```

Direct backend access is blocked:

`http://54.91.220.81:8080/student/`

Result:

`ERR_CONNECTION_TIMED_OUT`

---

## 🌍 Final Application URL

The application is publicly accessible through the Nginx Reverse Proxy:

`http://3.85.90.108/student/`

Students List:

`http://3.85.90.108/student/viewStudents`

---

## 🧪 Testing

### 1. Tomcat Running

```bash
ps aux | grep tomcat
```

Result:

Tomcat running successfully.

### 2. WAR Deployment

```bash
ls /opt/tomcat/webapps/
```

Expected:

`student.war`

`student/`

### 3. Backend Application Test

Backend application runs internally on:

`172.31.20.55:8080`

### 4. Nginx Test

```bash
sudo nginx -t
```

Result:

Nginx configuration successful.

### 5. Database Test

```sql
SELECT * FROM students;
```

Result:

Student records displayed successfully.

### 6. Final Application Test

`http://3.85.90.108/student/`

Result:

Student Registration Application working successfully.

### 7. Direct Backend Test

`http://54.91.220.81:8080/student/`

Result:

Direct backend access blocked.

---

## 📸 Project Screenshots

The following screenshots provide evidence of the project implementation.

### AWS EC2 Instances

![EC2 Instances](screenshots/01-ec2-instances.png)

### Tomcat Running

![Tomcat Running](screenshots/02-tomcat-running.png)

### WAR Deployment

![WAR Deployment](screenshots/03-war-deployment.png)

### Database Table

![Database Table](screenshots/04-database-table.png)

### Database Records

![Database Records](screenshots/05-database-records.png)

### MySQL Connector/J

![MySQL Connector](screenshots/06-mysql-connector.png)

### Nginx Configuration

![Nginx Configuration](screenshots/07-nginx-config.png)

### Nginx Test

![Nginx Test](screenshots/08-nginx-test.png)

### Nginx Service

![Nginx Service](screenshots/09-nginx-service.png)

### Student Registration Application

![Student Registration](screenshots/10-student-registration.png)

### Students List

![Students List](screenshots/11-students-list.png)

### Backend Direct Access Blocked

![Backend Blocked](screenshots/12-backend-blocked.png)

---

## 🛠️ Technologies Used

- Amazon EC2
- Amazon RDS MySQL
- Ubuntu Linux
- Java
- Servlet
- JSP
- Apache Tomcat 9
- JDBC
- MySQL Connector/J
- Nginx
- AWS Security Groups
- GitHub

---

## 📊 Project Outcome

The Java Student Registration Web Application was successfully deployed on AWS.

The final architecture provides:

- Two Linux EC2 instances
- Nginx Reverse Proxy
- Apache Tomcat backend
- Java WAR deployment
- Amazon RDS MySQL database
- JDBC database connectivity
- MySQL Connector/J
- Private IP communication
- Security Group based backend restriction
- Public application access through Nginx
- Successful student registration
- Successful database storage and retrieval

---

## 👨‍💻 Internship Details

**Organization:** Cravita Technology

**Internship:** AWS & DevOps Internship

**Project:** Project 1

**Project Title:** Java Application Deployment with Reverse Proxy on AWS

**Author:** Lokesh Dharasange
