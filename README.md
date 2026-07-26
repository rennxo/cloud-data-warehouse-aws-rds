# ☁️ Project 3: The Data Warehouse — Private Cloud Database on AWS

**Cloud Computing Internship | Project 3**

## 📌 Scenario

An e-commerce company managing customer data through Excel sheets needed a robust, scalable, and secure cloud database as their user base grew. This project simulates that migration by provisioning a **private, secure MySQL database on AWS RDS**.

## 🎯 Objective

Set up a managed cloud database system that:
- Runs inside a **private subnet** (not exposed to the public internet)
- Is protected by a **firewall locked strictly to port 3306** (MySQL)
- Stores intern records in a properly constrained relational table
- Is only reachable through a secure, controlled access path (bastion host)

## 🏗️ Architecture

**Internet → Internet Gateway → VPC**

**Public Subnet**
- Bastion EC2 instance (`sg-bastion`)
- Only reachable via AWS EC2 Instance Connect

⬇ port 3306 only ⬇

**Private Subnet (2 Availability Zones)**
- RDS MySQL instance (`sg-interns-db`)
- No route to the internet
- Only accepts traffic from `sg-bastion`

**Key points:**
- **Public subnet** → hosts a bastion EC2 instance, reachable only via AWS EC2 Instance Connect
- **Private subnets (2 AZs)** → host the RDS MySQL instance, with **no route to the internet**
-
