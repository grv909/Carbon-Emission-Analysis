# Carbon Footprint Analysis Project

![Carbon Footprint](pollution.jpg)

Photo by Maxim Tolchinskiy on Unsplash

## Introduction

When factoring heat generation required for the manufacturing and transportation of products, Greenhouse gas emissions attributable to products, from food to sneakers to appliances, make up more than 75% of global emissions. -The Carbon Catalogue

Our data, which is publicly available on nature.com, contains product carbon footprints (PCFs) for various companies. PCFs are the greenhouse gas emissions attributable to a given product, measured in CO2 (carbon dioxide equivalent).

This project analyzes the data stored in a PostgreSQL database, specifically in the `product_emissions` table. The table includes information about PCFs by product and the stage of production where these emissions occurred.

## Database Schema

The primary table in the database is `product_emissions`, with the following columns:

- `id`: VARCHAR
- `year`: INT
- `product_name`: VARCHAR
- `company`: VARCHAR
- `country`: VARCHAR
- `industry_group`: VARCHAR
- `weight_kg`: NUMERIC
- `carbon_footprint_pcf`: NUMERIC
- `upstream_percent_total_pcf`: VARCHAR
- `operations_percent_total_pcf`: VARCHAR
- `downstream_percent_total_pcf`: VARCHAR

## Query Examples

### 1. Number of Unique Companies and Total Carbon Footprint by Industry Group (Most Recent Year)

```sql
SELECT
  industry_group,
  COUNT(DISTINCT company) AS num_companies,
  ROUND(SUM(carbon_footprint_pcf), 1) AS total_industry_footprint
FROM
  product_emissions
WHERE
  year IN (SELECT MAX(year) FROM public.product_emissions)
GROUP BY
  industry_group
ORDER BY
  total_industry_footprint DESC;
