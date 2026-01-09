+++
title = "Getting Statistics Canada Data Analysis Ready"
description = "A 10 minute walkthrough Statistics Canada data."
outputs = ["Reveal"]
[logo]
src = "logo.svg"
alt = "Data for Canada"
width = "8%"
[reveal_hugo]
margin = 0.2
highlight_theme = "color-brewer"
+++

## Modernizing Access to Statistics Canada Data 📊


A five minute walkthrough.

Presented on July 11, 2025

---

### Current State
- Vast public data exists. Difficult to access and analyze.
  - 12,207 ["tables"](https://www150.statcan.gc.ca/n1/en/type/data?p=0-data/tables#tables), with 7,919 are available via the [Web Data Service (WDS)](https://www.statcan.gc.ca/en/developers/wds).
  - 284 "Profiles of a community or region". Some examples include:
    - 2021 Census of Population.
    - National Address Register (NAR).
- Thousands of CSVs (>3TBs) and other file formats.
- Many datasets are trapped behind [archived web pages](https://www12.statcan.gc.ca/english/census96/data/profiles/Rp-eng.cfm?LANG=E&APATH=3&DETAIL=0&DIM=0&FL=A&FREE=0&GC=0&GID=0&GK=0&GRP=1&PID=35544&PRID=0&PTYPE=3&S=0&SHOWALL=0&SUB=0&Temporal=1996&THEME=34&VID=0&VNAMEE=&VNAMEF=) and legacy file formats (ex. [ARC/INFO](https://www12.statcan.gc.ca/census-recensement/2011/geo/bound-limit/bound-limit-2001-eng.cfm), [MapInfo](https://www12.statcan.gc.ca/census-recensement/2011/geo/bound-limit/bound-limit-2001-eng.cfm), [IVT](https://www12.statcan.gc.ca/datasets/Index-eng.cfm?Temporal=2021)).

---

### Use Case: A Basic Data Task Made Difficult

Let's say that you want to visualize:
- **[One characteristic](https://1drv.ms/x/c/d42308bcd3b7a4a1/ESlrAmKqXp9BnsMXqurcB0sB9qy1r2-ZV8tCL9nCns_Mpg?e=8qIroJ)** from the [2021 Census of Population](https://www12.statcan.gc.ca/census-recensement/2021/dp-pd/prof/details/download-telecharger.cfm?Lang=E) - % of people that make $100,000 and over.
- At the [Dissemination Area (DA) level](https://www150.statcan.gc.ca/n1/pub/92-195-x/2021001/geo/da-ad/da-ad-eng.htm), the most granular dissemination geography.
- Across 13 capital cities: Whitehorse, Yellowknife, Iqaluit, Vancouver, Calgary, Regina, Winnipeg, Ottawa, Montréal, Saint John, Charlottetown, Halifax, and St. John's.

{{% note %}}
- 100% of data for this characteristic (168)
{{% /note %}}

---

### What's Required Today?

To complete this simple analysis, you would need to:
1. Download a 2.25 GB ZIP file.
2. Extract it - now 26.60 GB of CSVs.
3. Parse and filter your chosen characteristic.
4. Download an additional 97 MB ZIP file of a DA boundary shapefile.
5. Extract the ZIP file. Now you have a 171.72 MB file.
6. Link the processed CSV (step #3) to the Shapefile from step #5.

All of this just to make one map.

---

### Solution

```sql
SELECT
    geo.da_dguid,
    cop.count_total_1,
    cop.count_total_155,
    cop.count_total_168,
    CASE
        WHEN cop.count_total_168 = 0.0 THEN 0
        WHEN cop.count_total_155 = 0.0 THEN 0
        WHEN cop.count_total_168 IS NULL THEN 0
        WHEN cop.count_total_155 IS NULL THEN 0
        ELSE 
            ((cop.count_total_168/cop.count_total_155) * 100) 
    END AS percentage_over_100k,
    geo.geom
FROM
    'https://data-01.dataforcanada.org/processed/statistics_canada/census_of_population/2021/tabular/da_2021.parquet' AS cop,
    'https://data-01.dataforcanada.org/processed/statistics_canada/boundaries/2021/digital_boundary_files/da_2021.parquet' AS geo
WHERE geo.csd_dguid IN (
    '2021A00056001009', -- Whitehorse, YT
    '2021A00056106023', -- Yellowknife, NT
    '2021A00056204003', -- Iqaluit, NU
    '2021A00055915022', -- Vancouver, BC
    '2021A00054806016', -- Calgary, AB
    '2021A00054706027', -- Regina, SK
    '2021A00054611040', -- Winnipeg, MB
    '2021A00053506008', -- Ottawa, ON
    '2021A00052466023', -- Montréal, QC
    '2021A00051301006', -- Saint John, NB
    '2021A00051102075', -- Charlottetown, PE
    '2021A00051209034', -- Halifax, NS
    '2021A00051001519' -- St. John's, NL
    ) 
AND cop.da_dguid = geo.da_dguid;
```

**🚀 132.75 MB, 2.89 seconds**

Want a DGUID for your region? Use the [StatCan Geo Search Tool](https://statcan-geography.dataforcanada.org/) (2021 vintage).

{{% note %}}
- If you want to replicate this figure, just add EXPLAIN ANALYZE at the beginning of the SQL code
{{% /note %}}

---

### Query Performance Snapshot

![DuckDB Explain Analyze](duckdb_explain_analyze_cop_query.webp)

--- 

{{< slide background-video="2021_census_of_population_example_duckdb_lonboard.mp4" background-video-loop="true" background-video-muted="true" background-size="contain" >}}

{{% fragment %}}
📁 Demo notebook: [over_100k_cop_2021.ipynb](https://github.com/dataforcanada/process-statcan-data/blob/6b42a80529162d35b973bc7690f4950bd2f897ef/experiments/presentation/over_100k_cop_2021.ipynb)
{{% /fragment %}}

---

### Progress So Far
- Fully processed:
  - [2021 Census of Population](https://data-01.dataforcanada.org/processed/statistics_canada/census_of_population/2021/).
  - [2021 and 2016 Census of Agriculture](https://data-01.dataforcanada.org/processed/statistics_canada/census_of_agriculture/).
  - [2021 geographic boundaries](https://data-01.dataforcanada.org/processed/statistics_canada/boundaries/2021/) 
  - [Other util datasets](https://data-01.dataforcanada.org/processed/statistics_canada/).
- 99.91% of WDS tables processed - 7,911/7,918.
- Accessible at:
```
https://data-01.dataforcanada.org/experiments/statistics_canada/tables/{productId}/en/{productId}.parquet
```
---

### What's Next?
- Build a Dagster pipeline to auto-refresh WDS tables.
  - Hot storage via [data-01](https://data-01.dataforcanada.org/), [data-02](https://data-02.dataforcanada.org/), and [Zenodo](https://zenodo.org/). Torrent available as well that is HTTP seeded by Zenodo and [data-02](https://data-02.dataforcanada.org/).
  - Cold storage available through Zenodo.
- Process all Census of Population and Census of Agriculture to the highest detail available as far back as possible (work backwards: 2016, 2011, 2006, 2001, etc.).

---

### What's Next (Continued)
- Build [Python](https://github.com/diegoripley/stats_can_data) and R bindings for programmatic access.
- Generate vector tiles for geographies and Census data.
  - Example: [YouTube demo](https://www.youtube.com/watch?v=1C2RVh5Ditk).