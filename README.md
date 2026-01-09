# Presentation
Presentation is available [here](https://data-01.diegoripley.ca/modernizing_access_to_statistics_canada_data_july_11_2025/)

# Setup
```bash
hugo --minify
cd public
rclone copy --progress . cloudflare:diegoripley-data-01/modernizing_access_to_statistics_canada_data_july_11_2025/
```
