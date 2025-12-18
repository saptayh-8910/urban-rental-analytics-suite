# CityStays: Hospitality Revenue & Operations Suite

This project is a comprehensive business intelligence suite designed to manage a portfolio of short-term rental properties in a major metropolitan market. It integrates **Revenue Management**, **Marketing Campaign Tracking**, and **Regulatory Compliance** into a unified Looker Studio dashboard.

## Business Value
This dashboard solves three critical business problems for urban property managers:
1.  **Regulatory Risk:** Automatically tracks rental nights to ensure no property exceeds the strict **Annual Regulatory Cap** (e.g., 90-180 day limits common in major cities).
2.  **Revenue Optimization:** Monitors **RevPAR** (Revenue Per Available Room) and **ADR** (Average Daily Rate) to optimize pricing strategies across channels (Airbnb vs. Booking.com vs. Direct).
3.  **Campaign ROI:** Tracks the performance of seasonal promotional codes (e.g., `SEASONAL2025`) to measure gross sales lift and coupon usage.

## Architecture

```mermaid
graph LR
    A[Google Sheets<br/>(PMS Data Export)] -->|Data Blending| B(Looker Studio<br/>Data Source)
    B -->|Calculated Fields<br/>& Parameters| C{Business Logic<br/>Layer}
    C -->|Report A| D[Revenue Dashboard<br/>(ADR, RevPAR)]
    C -->|Report B| E[Operations Dashboard<br/>(Occupancy, Cleaning)]
    C -->|Report C| F[Compliance Dashboard<br/>(Regulatory Limits)]
    
    style C fill:#f9f,stroke:#333,stroke-width:2px
```

## Key Logic & Definitions
1. Regulatory Compliance (Annual Night Cap)
- Many cities enforce strict limits on how many days a property can be rented short-term per year. Exceeding this limit results in severe fines.
- The Problem: Managing "Actual Stays" + "Future Bookings" manually across multiple OTA platforms is error-prone.
- The Logic: I implemented a tracking system that aggregates nights sold per fiscal year.
- The Alert: The dashboard calculates Remaining Compliant Nights and flags properties approaching the limit, forcing a strategic switch to "Monthly Rentals" (30+ days) which are often exempt from caps.

2. Revenue KPIs
- Standard hospitality metrics were calculated to assess financial health:
- ADR (Average Daily Rate): Total Billed / Nights Sold
- RevPAR (Revenue Per Available Room): Total Billed / Total Available Nights (Crucial for measuring inventory efficiency).
- Channel Mix: Comparison of margins between OTAs (Online Travel Agencies) vs. Direct Booking (which typically saves ~15% in platform fees).

3. Campaign Tracking
- Scope: Tracking the impact of seasonal discount codes.
- Metric: Gross Sales attribution specifically tied to promo code usage vs. organic baseline sales.

## Tools Used
- Data Source: Google Sheets (Aggregated Property Management System logs)
- Visualization: Looker Studio
- Techniques: Data Blending, RegEx for Channel Grouping, Calculated Fields.

## Acknowledgments

Data represents anonymized operations for a property management group managing 50+ units.

## KPI Logic & Calculated Fields

### 1. Occupancy Rate
Defined as the percentage of available nights that were actually sold.
```sql
SUM(Nights Stay) / SUM(Days in Month)
```

### 2. Regulatory Remaining Nights (Compliance)
Logic to ensure compliance with the Annual Short-Term Rental Cap (e.g., 180 days).

-- "Annual_Limit" is a parameter set to 180 (or 90 for other regions)
Annual_Limit - (SUM(Nights_Used_Current_Fiscal) + SUM(Future_Bookings))

### 3. Channel Grouping (CASE Logic)
Used to group various booking sources into clean categories for analysis.

CASE
  WHEN Source REGEXP 'airbnb' THEN 'Airbnb Official'
  WHEN Source REGEXP 'booking' THEN 'Booking.com'
  WHEN Source = 'google' THEN 'Google Hotel Ads'
  ELSE 'Direct Booking'
END

### 4. Revenue Per Available Room (RevPAR)
A standard metric to measure revenue generation efficiency.
```sql
Total_Revenue / Total_Inventory_Nights
```
