# Presentation
Presentation is available [here](https://www.diegoripley.ca/files/modernizing_access_to_statistics_canada_data_july_11_2025/)

# Setup
```bash
hugo --minify --baseURL "https://www.diegoripley.ca/files/modernizing_access_to_statistics_canada_data_july_11_2025/"
cd public
rclone --progress copy . cloudflare:/diegoripley-www/files/modernizing_access_to_statistics_canada_data_july_11_2025
```
