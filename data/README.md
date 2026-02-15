# Data Sources

All data comes from **data.gov.my** — Malaysia's official open data portal.
Licensed under CC BY 4.0.

## Datasets Used

| Dataset | URL Pattern |
|---|---|
| Vehicle Registration Transactions (Cars) | `https://storage.data.gov.my/transportation/cars_{YEAR}.parquet` |

## Schema (Bronze — raw from source)

| Column | Type | Description |
|---|---|---|
| date_reg | date | Registration date |
| type | string | Vehicle type |
| maker | string | Manufacturer name |
| model | string | Vehicle model |
| colour | string | Vehicle colour |
| fuel | string | Fuel type (petrol, diesel, electric, etc.) |
| state | string | Registration state |

## Notes
- Parquet files are split by year (2000–2026)
- Data is updated monthly by JPJ (Road Transport Department)
- Do not commit raw data files to this repo — all data is read directly from source
