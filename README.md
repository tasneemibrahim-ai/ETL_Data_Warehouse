# 🏭 ETL Data Warehouse Pipeline

<div align="center">

![Data Warehouse](https://img.shields.io/badge/Project-Data%20Warehouse-blue?style=for-the-badge&logo=databricks&logoColor=white)
![SSIS](https://img.shields.io/badge/Tool-SSIS-CC2927?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)
![SQL Server](https://img.shields.io/badge/Database-SQL%20Server-CC2927?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)
![Status](https://img.shields.io/badge/Status-Completed-brightgreen?style=for-the-badge)

**A full ETL pipeline built with SSIS that extracts sales data from Excel, transforms and cleans it, and loads it into a structured star-schema data warehouse.**

</div>

---

## 📑 Table of Contents

- [📌 Project Overview](#-project-overview)
- [🎓 Course Information](#-course-information)
- [👥 Team Members](#-team-members)
- [🏗️ Architecture](#️-architecture)
- [🗃️ Data Warehouse Schema](#️-data-warehouse-schema)
- [⚙️ ETL Pipeline Details](#️-etl-pipeline-details)
  - [📦 Dimension Tables](#-dimension-tables)
  - [📊 Fact Table](#-fact-table)
- [🔧 Technologies Used](#-technologies-used)
- [🚀 How to Run](#-how-to-run)
- [📋 Prerequisites](#-prerequisites)
- [📄 Report](#-report)

---

## 📌 Project Overview

This project implements a complete **ETL (Extract, Transform, Load)** pipeline that populates an **OLAP Data Warehouse** from raw sales data stored in Excel files. The pipeline follows a classic **star schema** design, with four dimension tables surrounding a central fact table.

The main goals of this project are:
- **Extract** raw transactional data from Excel source files
- **Transform** the data by handling null values, removing duplicates, and generating surrogate keys
- **Load** clean, structured data into a SQL Server data warehouse for analytical querying

---

## 🎓 Course Information

| Field         | Details                              |
|---------------|--------------------------------------|
| **Course**    | Data Mining (AIE323)                 |
| **Faculty**   | Computer Science & Engineering (CSE) |
| **Level**     | Third Year                           |
| **Semester**  | Fall 2025                            |
| **Instructor**| Dr. Ashraf Saeed                     |
| **TA**        | Eng. Rowan                           |

---

## 👥 Team Members

| Name                     | Student ID  |
|--------------------------|-------------|
| Tasneem Ibrahim Hassan   | 22101382    |
| Rana Ehab Mahmoud Sakr   | 22100784    |
| Marc Ehab Edourd         | 22100908    |
| Mortada Mohamed Mortada  | —           |

---

## 🏗️ Architecture

The pipeline follows a standard **ETL star-schema** architecture:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         DATA SOURCES                                │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────────────────┐ │
│   │  Customers   │  │   Products   │  │  Orders / OrderDetails   │ │
│   │  (Excel)     │  │   (Excel)    │  │        (Excel)           │ │
│   └──────┬───────┘  └──────┬───────┘  └────────────┬─────────────┘ │
└──────────┼─────────────────┼──────────────────────┼──────────────┘
           │                 │                        │
           ▼                 ▼                        ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       TRANSFORMATION LAYER (SSIS)                   │
│  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐  ┌──────────┐ │
│  │ Null Split  │  │  Sort &     │  │   Script     │  │  Merge   │ │
│  │ (Null rows  │  │  Dedup      │  │  Component   │  │   Join   │ │
│  │  removed)   │  │ (duplicates)│  │ (Auto-incr   │  │(combine  │ │
│  │             │  │             │  │  surrogate   │  │ tables)  │ │
│  │             │  │             │  │   keys)      │  │          │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬───────┘  └────┬─────┘ │
└─────────┼────────────────┼────────────────┼───────────────┼───────┘
          │                │                │               │
          ▼                ▼                ▼               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     DATA WAREHOUSE (OLAP - SQL Server)              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────┐  ┌───────────┐  │
│  │ DimCustomer  │  │  DimProduct  │  │ DimDate  │  │ DimOrders │  │
│  └──────┬───────┘  └──────┬───────┘  └────┬─────┘  └─────┬─────┘  │
│         │                 │               │              │          │
│         └─────────────────┴───────────────┴──────────────┘          │
│                                   │                                  │
│                                   ▼                                  │
│                        ┌──────────────────┐                         │
│                        │  FactOrderDetails │                         │
│                        │   (Fact Table)    │                         │
│                        └──────────────────┘                         │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🗃️ Data Warehouse Schema

The warehouse uses a **Star Schema** design:

```
                    ┌─────────────────┐
                    │   DimCustomer   │
                    │─────────────────│
                    │ CustomerID (PK) │
                    │ CustomerName    │
                    │ ...             │
                    └────────┬────────┘
                             │
┌─────────────┐    ┌─────────┴──────────┐    ┌──────────────┐
│  DimProduct │    │  FactOrderDetails  │    │   DimDate    │
│─────────────│    │────────────────────│    │──────────────│
│ ProductID   ├────│ OrderDetailID (PK) ├────│ DateID (PK)  │
│ ProductName │    │ OrderID (FK)       │    │ OrderDate    │
│ ...         │    │ CustomerID (FK)    │    └──────────────┘
└─────────────┘    │ ProductID  (FK)    │
                    │ DateID (FK)        │
                    │ Quantity           │
                    │ UnitPrice          │
                    │ ...                │
                    └────────┬───────────┘
                             │
                    ┌────────┴────────┐
                    │   DimOrders     │
                    │─────────────────│
                    │ OrderID (PK)    │
                    │ CustomerID      │
                    │ ...             │
                    └─────────────────┘
```

---

## ⚙️ ETL Pipeline Details

### 📦 Dimension Tables

#### 🧑‍🤝‍🧑 DimCustomer
| Step | Action |
|------|--------|
| 1    | Create `DimCustomer` table in SQL Server (SSMS) |
| 2    | Connect Excel source file to SSIS package |
| 3    | Map Excel columns → DB columns using Data Flow |
| 4    | Apply **Conditional Split** to filter/remove NULL rows |
| 5    | Load clean records into the OLAP warehouse |

#### 📦 DimProduct
| Step | Action |
|------|--------|
| 1    | Create `DimProduct` table in SQL Server (SSMS) |
| 2    | Connect Excel source file to SSIS package |
| 3    | Map Excel columns → DB columns using Data Flow |
| 4    | Apply **Conditional Split** to filter/remove NULL rows |
| 5    | Load clean records into the OLAP warehouse |

#### 📅 DimDate
| Step | Action |
|------|--------|
| 1    | Extract `OrderDate` column from the Orders Excel source |
| 2    | Apply **Conditional Split** to remove NULL date entries |
| 3    | Apply **Sort** transformation to sort dates and remove duplicates |
| 4    | Use **Script Component** to auto-increment a surrogate `DateID` variable |
| 5    | Map `DateID` + `OrderDate` columns to the warehouse destination |

#### 🛒 DimOrders
| Step | Action |
|------|--------|
| 1    | Create `DimOrders` table with only required columns (e.g., `OrderDate`) |
| 2    | Map `CustomerID` from Excel source into the data flow |
| 3    | Apply **Conditional Split** to remove NULL rows |
| 4    | Apply **Sort** to deduplicate and order by date |
| 5    | Use **Script Component** to auto-increment `ID_Date` surrogate key |
| 6    | Load output into the `DimOrders` warehouse table |

---

### 📊 Fact Table

#### 📈 FactOrderDetails
| Step | Action |
|------|--------|
| 1    | Take two OLE DB sources: the pre-loaded `DimOrders` and `DimOrderDetails` warehouse tables |
| 2    | Apply **Sort** transformation on both sources |
| 3    | Use **Merge Join** (Inner Join) to combine orders with order details |
| 4    | Map all joined output columns to the `FactOrderDetails` destination table |

---

## 🔧 Technologies Used

| Technology | Purpose |
|---|---|
| **SQL Server Management Studio (SSMS)** | Database creation, table schema design |
| **SSIS (SQL Server Integration Services)** | ETL pipeline design and execution |
| **Visual Studio (VS)** | SSIS package development environment |
| **Microsoft Excel** | Source data files |
| **OLAP / Star Schema** | Data warehouse design methodology |

---

## 📋 Prerequisites

Before running this project, ensure you have the following installed:

- ✅ **SQL Server** (2019 or later recommended)
- ✅ **SQL Server Management Studio (SSMS)**
- ✅ **Visual Studio** with **SQL Server Data Tools (SSDT)** extension
- ✅ **Microsoft Excel** (for source data files)

---

## 🚀 How to Run

Follow these steps to set up and execute the ETL pipeline:

### Step 1 — Set Up the Database
```sql
-- Create the OLAP Data Warehouse database in SSMS
CREATE DATABASE DataWarehouse_OLAP;
USE DataWarehouse_OLAP;
```

### Step 2 — Create Dimension & Fact Tables
Run the SQL scripts to create:
- `DimCustomer`
- `DimProduct`
- `DimDate`
- `DimOrders`
- `FactOrderDetails`

### Step 3 — Configure the SSIS Package
1. Open the `.dtsx` SSIS project in **Visual Studio**
2. Update the **Connection Managers** to point to:
   - Your local SQL Server instance
   - The Excel source files on your machine
3. Verify all column mappings in the Data Flow tasks

### Step 4 — Execute the ETL Pipeline
Run the SSIS packages in the following order to respect foreign key dependencies:

```
1. DimCustomer  ──┐
2. DimProduct   ──┤
3. DimDate      ──┤──► FactOrderDetails
4. DimOrders    ──┘
```

> [!IMPORTANT]
> Always load all **Dimension tables first** before executing the Fact table pipeline to avoid foreign key constraint violations.

---

## 📄 Report

A full project report is included in this repository:

📘 [`ETL_Data_Warehouse_Pipeline_Report.docx`](./ETL_Data_Warehouse_Pipeline_Report.docx)

The report covers:
- Detailed explanation of each ETL step
- SSIS package screenshots
- Data transformation logic
- Warehouse schema diagrams

---

<div align="center">

**📚 AIE323 — Data Mining | Fall 2025 | Faculty of Computer Science & Engineering**

Made with ❤️ by the ETL Data Warehouse Team

</div>
