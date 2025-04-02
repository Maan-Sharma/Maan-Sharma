Final – starting from 5th column + 4

import requests
import pandas as pd
from bs4 import BeautifulSoup

# Define the Moneycontrol URL
url = "https://www.moneycontrol.com/financials/bankofbaroda/results/quarterly-results/BOB#BOB"

# Send request and parse HTML
headers = {"User-Agent": "Mozilla/5.0"}
response = requests.get(url, headers=headers)
soup = BeautifulSoup(response.text, "html.parser")

# Find the table containing quarterly results
table = soup.find("table", {"class": "mctable1"})

# Extract data from the table
data = []
for row in table.find_all("tr"):
    cols = row.find_all("td")
    cols = [ele.text.strip() for ele in cols]
    if cols:
        data.append(cols)

# Convert data to a DataFrame
df = pd.DataFrame(data)

# Set column names using the first row (headers)
df.columns = ["Metric"] + df.iloc[0, 1:].tolist()
df = df[1:]  # Remove the first row since it's now the column header

# **Get total number of columns**
num_cols = df.shape[1]

# **Identify Q1 column positions**  
# Start from 5th column (index 4) and pick every 4th column
q1_columns = [0] + list(range(4, num_cols, 4))[:5]  # Last 5 years' Q1 data

# **Extract relevant columns**
q1_data = df.iloc[:, q1_columns]

# **Extract years dynamically from column headers**
years = [df.columns[i].split()[-1] for i in q1_columns[1:]]

# **Rename columns properly**
q1_data.columns = ["Metric"] + [f"Q1 {year}" for year in years]

# **Display the extracted Q1 financials**
print(q1_data)

# **Save the data to a CSV file**
q1_data.to_csv("new", index=False)
print("\nData saved to new.csv")

