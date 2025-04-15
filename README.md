# Rimonim Meter - Hubitat Driver for Water Consumption Monitoring

## Overview
Rimonim Meter is a custom Hubitat device driver that fetches water consumption data from the Israeli water meter website [https://c.mtr-smart.net](https://c.mtr-smart.net), parses the results, and sends the consumption data to an InfluxDB instance. This data can then be visualized using Grafana.

## Features
- Daily or monthly consumption reports.
- Automatic and manual data fetching.
- Authentication and session token management.
- InfluxDB v2 integration using line protocol.
- Leak detection support through exposed attributes (managed via Rule Machine).
- Configurable scheduling for periodic updates.

## Prerequisites
To use this driver, you must have:
- A valid **Billing ID**, **Meter Number**, and **Password** from the water company website.
- A working **InfluxDB v2** instance (e.g., hosted locally or on a server).
- (Optional) **Grafana** dashboard configured to read from InfluxDB.

## Installation
1. In Hubitat, go to **Drivers Code** and click **New Driver**.
2. Paste the driver code.
3. Save and name the driver `Rimonim Meter`.
4. Go to **Devices** > **Add Virtual Device**.
5. Set the driver type to `Rimonim Meter`.
6. Click **Save Device**.

## Configuration
Once the device is created:
1. Click **Preferences** and fill in:
   - Water website URL (e.g., `https://c.mtr-smart.net`)
   - Billing ID
   - Meter Number
   - Password
   - InfluxDB Host and Port
   - InfluxDB Org Name, Bucket, and Token
   - Report Type (1 = Daily, 3 = Monthly)
   - From/To Date for manual runs
   - Update Frequency in hours
2. Click **Save Preferences**, then **Save Device**.
3. Click the `Initialize` command to reset cookies and start fresh.

## Manual vs Scheduled Fetching
- Use the `getConsumptionDataManually` command to fetch data for specific dates.
- Scheduled updates run every X hours and always pull the last 7 days of data.

## Exposed Attributes
These are visible in the device page and can be used in Rule Machine:
- `consumptionData` - Full string of parsed daily values.
- `lastConsumptionDate` - Most recent date received.
- `lastConsumptionValue` - Consumption value for that date.
- `lastPollDate` - Timestamp when data was last posted to InfluxDB.
- Tokens (`requestVerificationToken`, `antiforgeryCookie`, etc.)

## Leak Detection (Manual)
You can create rules in **Rule Machine** to:
- Monitor if `lastConsumptionValue` exceeds a certain threshold.
- Alert only if `lastConsumptionDate` is newer than the last alert.

## InfluxDB Integration
The driver uses the InfluxDB v2 HTTP API:
- Data is sent in line protocol with measurement `daily_consumption`
- Tags: `meter`
- Fields: `consumption`, `date`, `month_year`
- Timestamps are at midnight of each reported date (in nanoseconds)

## Example Grafana Panel
Create a Grafana dashboard using your InfluxDB bucket with a query like:
```flux
from(bucket: "your-bucket")
  |> range(start: -30d)
  |> filter(fn: (r) => r._measurement == "daily_consumption")
  |> filter(fn: (r) => r.meter == "your-meter-number")
```

## To-Do / Future Improvements
- Add support for monthly totals (`This month consumption`).
- Optionally fetch account details from `/consumer` page.
- Add configuration UI for leak threshold.
- Clean up and modularize token handling further.

## License
This driver is licensed under the [Apache 2.0 License](http://www.apache.org/licenses/LICENSE-2.0).

## Author
Developed by Amit Halperin.

