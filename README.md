# Iowa Liquor Sales - End-to-End Data Warehousing & BI Solution

## 📌 Project Overview
This project demonstrates an end-to-end Data Warehousing and Business Intelligence (DWBI) pipeline designed for the **Iowa Liquor Sales** dataset. Built using the **Kimball Dimensional Modeling** methodology, the solution extracts raw transactional data, cleanses and transforms it through a staging layer, and loads it into a Star Schema Data Warehouse (`IowaLiquor_DW`) using SQL Server Integration Services (SSIS).

---

## 🏗️ Architecture & Data Flow
1. **Source Layer:** Raw transactional sales data and transaction completion updates (`Completed_Transactions.csv`).
2. **Staging Layer:** Intermediate database tables used for data profiling, deduplication, and cleansing.
3. **ETL Layer (SSIS):** Automated packages handling dimension loading, historical tracking, surrogate key lookups, and fact table updates.
4. **Data Warehouse Layer (`IowaLiquor_DW`):** A Star Schema optimized for analytical querying and BI reporting.

---

## ⭐ Data Warehouse Design (Star Schema)
The warehouse is structured around a central Fact table surrounded by three Dimension tables:

* **`FactSales` (Accumulating Snapshot Fact Table):**
  * Captures core sales metrics: `BottlesSold`, `SaleDollars`, and `VolumeSoldLiters`.
  * Tracks transaction lifecycle milestones using `accm_txn_create_time` and `accm_txn_complete_time`.
  * Dynamically calculates processing duration in hours (`txn_process_time_hours`).
* **`DimStore` (Slowly Changing Dimension - Type 2):**
  * Tracks store details (`StoreName`, `Address`, `City`, `ZipCode`, `County`).
  * Preserves historical changes over time using `StartDate` and `EndDate` attributes without overwriting past sales records.
* **`DimVendor`:** Stores unique liquor vendor details indexed by a surrogate key (`VendorSK`).
* **`DimItem`:** Stores product catalog details, categories, and bottle volumes indexed by `ItemSK`.

---

## ⚙️ ETL Pipeline Packages (SSIS)
The ETL workflow is modularized into dedicated `.dtsx` packages:

1. **`Load_Dimensions.dtsx`:**
   * Extracts distinct vendor and item records into `DimVendor` and `DimItem`.
   * Implements **SCD Type 2** logic for `DimStore` using Derived Columns and OLE DB Commands to expire old records and insert updated store attributes.
   * Populates `FactSales` by performing Surrogate Key lookups (`StoreSK`, `VendorSK`, `ItemSK`) against the dimension tables.
2. **`Update_FactSales.dtsx`:**
   * Reads late-arriving transaction completion timestamps from `Completed_Transactions.csv` via a Flat File Source.
   * Updates `accm_txn_complete_time` in `FactSales` matching on `InvoiceNumber`.
   * Executes a T-SQL task (`DATEDIFF`) to compute `txn_process_time_hours` for completed transactions.

---

## 🛠️ Tech Stack
* **Database Engine:** Microsoft SQL Server (SQLEXPRESS)
* **ETL Tool:** SQL Server Integration Services (SSIS) / Visual Studio
* **Data Modeling:** Kimball Dimensional Modeling (Star Schema, SCD Type 2, Accumulating Snapshot)
* **Languages:** T-SQL, XML (SSIS Packages)

---

## 🚀 How to Run the Project
1. **Database Setup:** Execute the DDL scripts in SQL Server Management Studio (SSMS) to create the Staging database and `IowaLiquor_DW` tables.
2. **Connection Managers:** Open the SSIS solution in Visual Studio and update the OLE DB Connection Manager (`DESKTOP-KF7LCJ4\SQLEXPRESS.IowaLiquor_DW`) to match your local SQL Server instance.
3. **Flat File Path:** Update the Flat File Connection Manager path in `Update_FactSales.dtsx` to point to the local directory of `Completed_Transactions.csv`.
4. **Execution Order:**
   * Run the Staging load package.
   * Execute `Load_Dimensions.dtsx` to populate dimensions and initial fact records.
   * Execute `Update_FactSales.dtsx` to update completion timestamps and calculate processing hours.
