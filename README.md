# BookMyShow Real-Time Data Pipeline on Azure

<img width="916" height="610" alt="diagram-export-13-11-2025-10_40_36-PM" src="https://github.com/user-attachments/assets/2b13aded-0593-4f06-9a6c-e6fb5964f09b" />

## Overview
This project demonstrates the design and implementation of a **real-time streaming data pipeline** using **Azure cloud services**.  
It simulates the data flow of an online ticketing platform similar to **BookMyShow**, where user bookings and payments are captured, processed, and analyzed in near real-time.

The pipeline generates **mock booking and payment events**, streams them to **Azure Event Hubs**, processes and enriches the data using **Azure Stream Analytics**, and stores the final structured data into **Azure Synapse Analytics** for analytical and reporting purposes.

---

## Architecture Overview

### 1. Data Generation
- The Python scripts `mock_bookings.py` and `mock_payments.py` generate synthetic event data resembling real-world transactions.
- Each script produces **nested JSON structures** containing customer details, event information, booking timestamps, and payment metadata.
- Data is continuously streamed to two separate **Azure Event Hubs** — one for bookings and one for payments.
- Both Event Hubs are configured with **two partitions** to support scalability and parallel message processing.

### 2. Streaming and Transformation
- **Azure Stream Analytics** consumes events from both Event Hubs.
- The Stream Analytics job performs:
  - **Flattening of nested JSON data**
  - **Derivation of event attributes** such as event category, day of week, and booking hour
  - **Real-time joins** between booking and payment streams using a **2-minute sliding window**
- Complete transformation logic is defined in [`stream_analytics_job_query.sql`](./stream_analytics_job_query.sql).

### 3. Data Storage and Analytics
- The processed and joined records are stored in **Azure Synapse Analytics**.
- The schema for the output table is defined in [`synapse_create_table.sql`](./synapse_create_table.sql).
- The resulting table acts as a **fact table** supporting analytical queries to understand booking patterns, payment trends, and event performance.

---

## Technologies and Tools

| Component | Purpose |
|------------|----------|
| **Python** | Generate mock booking and payment data streams |
| **Azure Event Hubs** | Real-time ingestion layer for bookings and payments |
| **Azure Stream Analytics** | Real-time transformation, enrichment, and stream joining |
| **Azure Synapse Analytics** | Storage and analytics layer for processed data |

---

## Stream Analytics Logic Summary

The **Stream Analytics Job** performs continuous transformations on live event data streams.

Key operations include:
- Flattening nested booking and payment events into a relational structure.
- Deriving **event categories** based on event name patterns:
  - Concert → Music  
  - Play → Theater  
  - Movie → Cinema  
  - Others → Other
- Extracting **time-based attributes** such as booking day of the week and hour.
- Performing a **real-time join** between booking and payment events on `order_id` within a **2-minute time window**.

This ensures bookings and payments for the same order are associated in near real-time.

---

## Synapse Table Schema

The final output from **Stream Analytics** is stored in **Azure Synapse Analytics** in the following structure:

CREATE TABLE bookymyshow.bookings_fact (
    order_id NVARCHAR(50) NOT NULL,
    booking_time DATETIME2,
    customer_id NVARCHAR(50),
    customer_name NVARCHAR(100),
    customer_email NVARCHAR(100),
    event_id NVARCHAR(50),
    event_name NVARCHAR(100),
    event_location NVARCHAR(200),
    seat_number NVARCHAR(10),
    seat_price FLOAT,
    event_category NVARCHAR(50),
    booking_day_of_week NVARCHAR(20),
    booking_hour INT,
    payment_id NVARCHAR(50),
    payment_time DATETIME2,
    amount FLOAT,
    payment_method NVARCHAR(50),
    payment_type NVARCHAR(50),
    booking_event_time DATETIME2,
    payment_event_time DATETIME2
);

This table acts as the central fact table for downstream analytics and reporting in Synapse.

## Key Highlights
- End-to-end real-time data pipeline built entirely on Azure services
- Demonstrates streaming ingestion, real-time transformation, and data warehousing integration
- Implements scalable architecture using partitioned Azure Event Hubs
- Provides a realistic simulation of a production-grade event-driven data pipeline
- Enables analytical insights from real-time event data using Azure Synapse Analytics

## Future Enhancements
- Add Azure Data Factory for orchestration and batch processing
- Integrate Power BI for real-time dashboards and visualization
- Implement data quality and validation layers before ingestion into Synapse
- Incorporate Azure Data Lake Storage for raw data archival and long-term retention


