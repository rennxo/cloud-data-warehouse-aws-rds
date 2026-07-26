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
- **Security groups** → the database's security group only accepts port 3306 traffic that originates from the bastion's security group — nothing else, not even a developer's laptop IP, can reach the database directly

## 🛠️ What Was Built

| Component | Details |
|---|---|
| VPC | Used existing default VPC, added 2 new **private subnets** across 2 Availability Zones |
| Route Table | Custom `private-rt` route table with **no internet gateway route**, associated with both private subnets |
| DB Subnet Group | `interns-db-subnet-group` spanning both private subnets |
| Security Groups | `sg-interns-db` (MySQL/3306, source = bastion SG only) and `sg-bastion` (SSH/22, source = EC2 Instance Connect IP range) |
| RDS Instance | MySQL 8.4, `db.t4g.micro`, **Public access: No**, Free Tier |
| Bastion Host | Amazon Linux EC2 (`t3.micro`) in a public subnet, used as the sole access path to the database |
| Database | `internsdb` with an `Interns` table |

## 🗄️ Database Schema

```sql
CREATE TABLE Interns (
    ID INT PRIMARY KEY AUTO_INCREMENT,
    Name VARCHAR(100) NOT NULL,
    Role VARCHAR(100) NOT NULL,
    Email VARCHAR(150) NOT NULL UNIQUE
);
```

- `ID` — Primary Key, auto-incrementing
- `Name`, `Role` — required fields (`NOT NULL`)
- `Email` — required and unique (`NOT NULL UNIQUE`)

## 📥 Sample Data Inserted

```sql
INSERT INTO Interns (Name, Role, Email) VALUES
('Annanya', 'Cloud Intern', 'annanya@example.com'),
('Akshaya', 'Backend Intern', 'akshaya@example.com'),
('Mugilan', 'DevOps Intern', 'mugilan@example.com'),
('Sharmila', 'Data Intern', 'sharmila@example.com');
```

## ✅ Verification

Connected to the private RDS instance from the bastion host and confirmed data persistence:

```sql
SELECT * FROM Interns;
```

*(See `/screenshots/06-select-query-result.png` for the query output.)*

## 🔒 Security Highlights

- Database has **no public IP** and is unreachable from the open internet
- Access is only possible by first authenticating into the bastion host
- Firewall rules are scoped to the **minimum necessary** — port 3306 only, from a single trusted security group
- SSH access to the bastion itself is restricted, using AWS's official EC2 Instance Connect IP range rather than an open port

## 📷 Screenshots

| # | Description |
|---|---|
| 1 | VPC subnets (public + private) |
| 2 | Private route table with no internet route |
| 3 | Security group rules (`sg-interns-db`, `sg-bastion`) |
| 4 | RDS instance — Available, private, MySQL |
| 5 | Bastion EC2 instance running |
| 6 | Successful `SELECT * FROM Interns;` query result |

## 🧰 Tools Used

- AWS RDS (MySQL)
- AWS EC2 (Bastion Host)
- AWS VPC (Subnets, Route Tables, Security Groups)
- AWS EC2 Instance Connect
- MySQL / MariaDB CLI client

## 📝 Key Learnings

- Designing a private subnet architecture and controlling internet reachability via route tables
- Security group chaining (allowing traffic based on *source security group*, not just IP)
- Troubleshooting real-world network connectivity issues (diagnosed and resolved an IPv6-only local network blocking direct SSH, by switching to browser-based EC2 Instance Connect)
- Enforcing data integrity at the schema level with `PRIMARY KEY`, `NOT NULL`, and `UNIQUE` constraints
