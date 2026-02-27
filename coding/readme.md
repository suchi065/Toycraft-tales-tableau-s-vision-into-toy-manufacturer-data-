# ================================================
# ToyCraft Tales: Tableau's Vision into Toy Manufacturer Data
# Complete End-to-End Implementation
# ================================================

# Import Libraries
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import sqlite3
from sklearn.linear_model import LinearRegression

# --------------------------------
# 1. Load Dataset
# --------------------------------
df = pd.read_csv("toycraft_sales_data.csv")

# --------------------------------
# 2. Data Cleaning & Transformation
# --------------------------------
df['Sales_Date'] = pd.to_datetime(df['Sales_Date'])
df.dropna(inplace=True)

df['Year'] = df['Sales_Date'].dt.year
df['Month'] = df['Sales_Date'].dt.month

df['Revenue'] = df['Selling_Price'] * df['Units_Sold']
df['Profit'] = (df['Selling_Price'] - df['Production_Cost']) * df['Units_Sold']
df['Profit_Margin_%'] = (df['Profit'] / df['Revenue']) * 100

# --------------------------------
# 3. Store Data in SQL Database
# --------------------------------
connection = sqlite3.connect("toycraft.db")
df.to_sql("toy_sales", connection, if_exists="replace", index=False)

cursor = connection.cursor()

# --------------------------------
# 4. KPI Calculations using SQL
# --------------------------------

total_revenue = cursor.execute("""
SELECT SUM(Selling_Price * Units_Sold) FROM toy_sales
""").fetchone()[0]

total_profit = cursor.execute("""
SELECT SUM((Selling_Price - Production_Cost) * Units_Sold) FROM toy_sales
""").fetchone()[0]

profit_margin = cursor.execute("""
SELECT (SUM((Selling_Price - Production_Cost) * Units_Sold) /
        SUM(Selling_Price * Units_Sold)) * 100
FROM toy_sales
""").fetchone()[0]

print("Total Revenue:", total_revenue)
print("Total Profit:", total_profit)
print("Profit Margin %:", profit_margin)

# --------------------------------
# 5. Category-wise Revenue
# --------------------------------
category_revenue = pd.read_sql("""
SELECT Category, SUM(Selling_Price * Units_Sold) AS Revenue
FROM toy_sales
GROUP BY Category
ORDER BY Revenue DESC
""", connection)

print("\nCategory Performance:\n", category_revenue)

# --------------------------------
# 6. Region-wise Revenue
# --------------------------------
region_revenue = pd.read_sql("""
SELECT Region, SUM(Selling_Price * Units_Sold) AS Revenue
FROM toy_sales
GROUP BY Region
ORDER BY Revenue DESC
""", connection)

print("\nRegion Performance:\n", region_revenue)

# --------------------------------
# 7. Monthly Revenue Trend
# --------------------------------
monthly_sales = df.groupby(['Year', 'Month'])['Revenue'].sum()

plt.figure()
monthly_sales.plot()
plt.title("Monthly Revenue Trend")
plt.xlabel("Year-Month")
plt.ylabel("Revenue")
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

# --------------------------------
# 8. Sales Forecast using Linear Regression
# --------------------------------
monthly_data = df.groupby('Month')['Revenue'].sum().reset_index()

X = monthly_data[['Month']]
y = monthly_data['Revenue']

model = LinearRegression()
model.fit(X, y)

future_months = pd.DataFrame({'Month': [13, 14, 15]})
predictions = model.predict(future_months)

print("\nFuture Revenue Predictions:")
for i, value in enumerate(predictions):
    print(f"Month {13+i}: {value}")

# --------------------------------
# 9. Export Clean Data for Tableau
# --------------------------------
df.to_csv("toycraft_cleaned_data.csv", index=False)

print("\nData exported successfully for Tableau dashboard usage!")

# --------------------------------
# 10. Close Database Connection
# --------------------------------
connection.close()

print("\nProject Execution Completed Successfully!")
