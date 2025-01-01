**Train Data Fetching and Analysis**

**Overview**

This project utilizes the Finnish Transport Agency's open API to fetch train data for a specific train (train number 4) over the course of November 2024. The project performs two main tasks:
1. Fetching and normalizing train data for the month of November 2024.
2. Calculating the average actual arrival time at the final destination for train number 4 in November 2024.

The script uses the `requests` library to fetch data from the API and `pandas` to normalize and analyze the data.

**Requirements**

To run the script, ensure that you have the following installed:
- Python 3.x
- pandas 
- requests 

**Script Explanation**

**Functions in the Script:**

1. **`fetch_train_data(date, train_number)`**:
   - Fetches the data for a specific train (`train_number`) and date (`date`) using the Digitraffic API.
   - Returns the data in JSON format if the request is successful, else returns `None`.

2. **`monthly_data_train(train_number, start_date, end_date)`**:
   - Fetches the data for the entire month of November 2024 (from `start_date` to `end_date`) for a given train number.
   - Iterates through each day of the month and calls the `fetch_train_data` function to collect data for all the days.
   
3. **`normalize_data_train(json_data)`**:
   - Normalizes the fetched JSON data by extracting relevant information like `train_number`, `departure_date`, `station`, `type`, `scheduled_time`, `actual_time`, and `difference_in_minutes`.
   - Converts the extracted data into a `pandas` DataFrame for easier manipulation.

4. **Average Arrival Time Calculation**:
   - Once the data is normalized, the script calculates the average actual arrival time at the final destination by converting the `actual_time` column to a `datetime` object and calculating the mean arrival time.

**Code**

```python
import requests
import csv
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

api_link="https://rata.digitraffic.fi/api/v1/trains/{DATE}/{TRAIN_NUMBER}"

**# **Task 1****

def fetch_train_data(date, train_number):
    url = api_link.format(DATE=date, TRAIN_NUMBER=train_number)
    response = requests.get(url)
    if response.status_code == 200:
        return response.json()
    else:
        print(f"Failed to fetch data for {date} (Status Code: {response.status_code})")
        return None

fetch_train_data("2024-11-02", 4)  # Test the function with train_number=4 and date=2024-11-02

def monthly_data_train(train_number, start_date, end_date):
    all_train_data = []
    current_date = start_date
    while current_date <= end_date:
        date_str = current_date.strftime("%Y-%m-%d")
        print(f"Fetching data for {date_str}...")
        daily_data = fetch_train_data(date_str, train_number)
        if daily_data:
            all_train_data.extend(daily_data)
        current_date += timedelta(days=1)
    return all_train_data

data = monthly_data_train(4, datetime(2024, 11, 1), datetime(2024, 11, 30))  # Fetching data for train 4 from 2024-11-01 to 2024-11-30

def normalize_data_train(json_data):
    records = []
    for train in json_data:
        train_number = train.get("trainNumber")
        departure_date = train.get("departureDate")
        for stop in train.get("timeTableRows", []):
            record = {
                "train_number": train_number,
                "departure_date": departure_date,
                "station": stop.get("stationShortCode"),
                "type": stop.get("type"),
                "scheduled_time": stop.get("scheduledTime"),
                "actual_time": stop.get("actualTime"),
                "difference_in_minutes": stop.get("differenceInMinutes"),
            }
            records.append(record)
    return pd.DataFrame(records)

df = normalize_data_train(data)  # Normalizing the fetched data

df.head()
df.info()

output_csv = "normalized_train_data_november_2024.csv"
df.to_csv(output_csv, index=False)  # Exporting the normalized data to a CSV file

**# **Task 2****

df['actual_time'] = pd.to_datetime(df['actual_time'])  # Convert 'actual_time' to datetime
final_destination_times = df.groupby(['train_number', 'departure_date'])['actual_time'].last()  # Getting the last arrival time at the final station
average_arrival_time = final_destination_times.mean()  # Calculating the average arrival time

print(f"The average actual arrival time at the final destination in November 2024 is: {average_arrival_time}")
``` 

**How to Run the Script**

1. **Download the script**: Save the above code into a Python file (e.g., `train_data_analysis.py`).
   
2. **Install the required Python packages**: if re
   To install the necessary libraries (`requests` and `pandas`), run the following command in your terminal or command prompt:
   ```bash
   pip install requests pandas
   ```

3. **Run the script**:
   After ensuring that the required packages are installed, run the script using:
   ```bash
   python train_data_analysis.py
   ```

4. **Output**:
   The script will:
   - Fetch train data for train number 4 from November 1st to November 30th, 2024.
   - Normalize the data into a CSV file named `normalized_train_data_november_2024.csv`.
   - Print the average actual arrival time at the final destination for train number 4.

**Expected Output**

1. A CSV file (`normalized_train_data_november_2024.csv`) will be generated with the following columns:
   - `train_number`
   - `departure_date`
   - `station`
   - `type` (e.g., departure, arrival)
   - `scheduled_time`
   - `actual_time`
   - `difference_in_minutes`

2. The average actual arrival time for the final destination of the train will be printed to the console.

