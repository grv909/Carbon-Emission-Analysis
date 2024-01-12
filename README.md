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

2. Carbon Intensity Across Different Industry Groups

sql

SELECT
  industry_group,
  ROUND(SUM(carbon_footprint_pcf) / SUM(weight_kg), 4) AS carbon_intensity
FROM
  product_emissions
WHERE
  year = (SELECT MAX(year) FROM product_emissions)
GROUP BY
  industry_group
ORDER BY
  carbon_intensity DESC;

3. Companies Contributing Significantly to Total Carbon Footprint by Industry Group

sql

WITH CompanyEmissionProfiles AS (
  -- Subquery to calculate company-level emissions and proportions
  SELECT
    industry_group,
    company,
    SUM(carbon_footprint_pcf) AS total_company_emissions,
    SUM(carbon_footprint_pcf) / SUM(SUM(carbon_footprint_pcf)) OVER (PARTITION BY industry_group) AS proportion_to_total
  FROM
    product_emissions
  WHERE
    year = (SELECT MAX(year) FROM product_emissions)
  GROUP BY
    industry_group, company
)

SELECT
  industry_group,
  company,
  total_company_emissions,
  ROUND(proportion_to_total * 100, 2) AS proportion_to_total_percentage
FROM
  CompanyEmissionProfiles
ORDER BY
  industry_group, proportion_to_total DESC;

4. Changes in Carbon Footprints for Each Industry Group Over Time

sql

SELECT
  industry_group,
  year,
  AVG(carbon_footprint_pcf) AS avg_carbon_footprint
FROM
  product_emissions
GROUP BY
  industry_group, year
ORDER BY
  industry_group, year;

Key Findings

    Variations in carbon intensity across industry groups.
    Identification of top contributors and their proportional impact.
    Temporal analysis revealing trends and changes over time.

Recommendations
Capital Goods Industry Group

    Issue: High carbon footprint, primarily from Daikin Industries.
    Recommendation: Invest in energy-efficient and low-emission technologies, such as natural refrigerants and renewable energy sources.

Commercial & Professional Services Industry Group

    Issue: Relatively low carbon footprint but optimization potential for Toppan Printing.
    Recommendation: Reduce emissions by using more recycled materials and optimizing printing processes.

Food, Beverage & Tobacco Industry Group

    Issue: Moderate carbon footprint with potential improvements for Ajinomoto.
    Recommendation: Lower emissions by sourcing locally, using organic ingredients, and reducing packaging waste.

Materials Industry Group

    Issue: Highest carbon footprint, mainly from Mitsubishi Gas Chemical.
    Recommendation: Adopt circular economy practices, reuse/recycle materials, and use biodegradable alternatives.
