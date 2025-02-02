---
title: "Generating Synthetic Stock Data"
weight: 4
chapter: false
pre: " <b> 2.4 </b> "
---

#### Generating Sample Stock Data  

We will create a CSV file containing stock price data for a fictional company, **FAKECO**, for demonstration purposes.  

```python
import pandas as pd
import numpy as np
from datetime import datetime, timedelta

def make_synthetic_stock_data(filename):
    # Define the start and end dates
    start_date = datetime(2023, 6, 27)
    end_date = datetime(2024, 6, 27)

    # Create a date range
    date_range = pd.date_range(start_date, end_date, freq='D')

    # Initialize lists to store the data
    symbol = []
    dates = []
    open_prices = []
    high_prices = []
    low_prices = []
    close_prices = []
    adj_close_prices = []
    volumes = []

    # Set the initial stock price
    initial_price = 100.0

    # Generate realistic stock prices
    for date in date_range:
        symbol.append('FAKECO')
        dates.append(date)
        open_price = np.round(initial_price + np.random.uniform(-1, 1), 2)
        high_price = np.round(open_price + np.random.uniform(0, 5), 2)
        low_price = np.round(open_price - np.random.uniform(0, 5), 2)
        close_price = np.round(np.random.uniform(low_price, high_price), 2)
        adj_close_price = close_price
        volume = np.random.randint(1000, 10000000)

        open_prices.append(open_price)
        high_prices.append(high_price)
        low_prices.append(low_price)
        close_prices.append(close_price)
        adj_close_prices.append(adj_close_price)
        volumes.append(volume)

        initial_price = close_price

    # Create a DataFrame
    data = {
        'Symbol': symbol,
        'Date': dates,
        'Open': open_prices,
        'High': high_prices,
        'Low': low_prices,
        'Close': close_prices,
        'Adj Close': adj_close_prices,
        'Volume': volumes
    }

    stock_data = pd.DataFrame(data)

    # Save the DataFrame
    stock_data.to_csv(filename, index=False)
```

```python
# Ensure the output directory exists
import os
if not os.path.exists('output'):
    os.makedirs('output')

stock_file = os.path.join('output', 'FAKECO.csv')
if not os.path.exists(stock_file):
    make_synthetic_stock_data(stock_file)
```

#### 1. Generating Synthetic Stock Data for FAKECO  
- Uses the `pandas`, `numpy`, and `datetime` libraries to generate stock data for the fictional company **FAKECO**.  
- The dataset includes **trading date, opening price, highest price, lowest price, closing price, adjusted close price, and trading volume**.  
- Time range: **June 27, 2023 – June 27, 2024**.  

#### 2. Creating Randomized Stock Price Variations  
- **Initial price**: `100.0 USD`.  
- Each day, the **opening, highest, lowest, and closing prices** are generated randomly to simulate real market fluctuations.  
- **Trading volume**: randomly selected between **1,000 and 10,000,000**.  
- The data is stored in a **pandas DataFrame**.  

#### 3. Saving Data to a CSV File  
- The CSV file will be created in the `output/FAKECO.csv` directory.  
- If the directory does not exist, it will be created automatically.  
- If the file does not exist, the program will generate new data and save it.  

![stock-data](/images/2-prerequisites/2.4-generating-synthetic-stock-data/image.png)  
