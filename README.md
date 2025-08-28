# Fruit-Prices-By-Form-in-The-United-States
The main agenda of this project will be to compare the prices across different states of fruits at retail stores.

This is the link to the dataset: https://www.ers.usda.gov/data-products/fruit-and-vegetable-prices.aspx

#Below is the full code

import pandas as pd
import matplotlib.pyplot as plt
from scipy.stats import chi2_contingency

# Data Cleaning
# Read the CSV file
input_file = r"C:\Users\mutan\OneDrive\Desktop\Fruit-Prices-2022.csv"
df = pd.read_csv(input_file)

# Filter out rows containing 'per pint' as entry
df = df[~df['RetailPriceUnit'].str.contains('per pint', case=False)]

# Check what the column names are
print(df.columns)

# Save the filtered dataframe to a new CSV file
df.to_csv("Fruit_Prices_Per_Pounds.csv", index=False)

# Rename RetailPrice column and CupEquivalentSize column to add their unit of measurement
df.rename(columns={
    'RetailPrice': 'RetailPrice-PerPound',
    'CupEquivalentSize': 'CupEquivalentSize-PerPound'
}, inplace=True)

# Check the currently existing columns
print(df.columns)

# Drop all columns that only contain the unit of measurement
columns_to_drop = ['RetailPriceUnit', 'CupEquivalentUnit']
df.drop(columns=columns_to_drop, inplace=True)

# Drop the Yield column as it is unnecessary for analysis
df.drop(columns=['Yield'], inplace=True)

# Save the filtered dataframe to a new CSV file
df.to_csv("Fruit_Prices_Resulting_Dataset.csv", index=False)

# Transformation
# Rename CupEquivalentSize-PerPound to CupEquivalentSize-InPounds
df.rename(columns={'CupEquivalentSize-PerPound': 'CupEquivalentSize-InPounds'}, inplace=True)

# Add an extra column to show the CupEquivalentPrice per pound
df['CupEquivalentPrice-PerPound'] = df['CupEquivalentPrice'] / df['CupEquivalentSize-InPounds']

# Add an extra column to show the ratio between CupEquivalentPrice-PerPound and RetailPrice-PerPound
df['Ratio RetailPrice/CupEquivalent'] = df['RetailPrice-PerPound'] / df['CupEquivalentPrice-PerPound']

# Round up all the numerical data to two decimal places
numerical_columns = [
    'RetailPrice-PerPound',
    'CupEquivalentSize-InPounds',
    'CupEquivalentPrice',
    'CupEquivalentPrice-PerPound',
    'Ratio RetailPrice/CupEquivalent'
]
df[numerical_columns] = df[numerical_columns].apply(lambda x: round(x, 2))

# Save the transformed dataframe to a new CSV file
df.to_csv("Fruit_Prices_Transformed_Dataset.csv", index=False)

# Check column names
print(df.columns)

# Discretization (Equal Width Binning)
df = pd.read_csv("Fruit_Prices_Transformed_Dataset.csv")

# Define the number of bins
num_bins = 5

# Perform equal width binning on numerical columns
for column in df.columns:
    if df[column].dtype in ['int64', 'float64']:
        bins = pd.cut(df[column], bins=num_bins, labels=False)
        df[column + '_bin'] = bins

# Save the discretized data into a new CSV file
df.to_csv("Fruit_Prices_Discretized_Dataset.csv", index=False)

# Chi-Square Test
df = pd.read_csv("Fruit_Prices_Discretized_Dataset.csv")

# Enter the two categorical variables of interest
contingency_table = pd.crosstab(df['RetailPrice-PerPound'], df['CupEquivalentPrice-PerPound'])

# Perform chi-square test
chi2, p, dof, expected = chi2_contingency(contingency_table)

# Output the results
print("Chi-square statistic:", chi2)
print("P-value:", p)
print("Degrees of freedom:", dof)
print("Expected frequencies:\n", expected)

# Visualization
# Select the column to visualize
column_to_plot = 'RetailPrice-PerPound'

# Create histogram
plt.figure(figsize=(10, 6))
plt.hist(df[column_to_plot], bins=10, color='skyblue', edgecolor='black')
plt.title('Histogram of Fruit Prices')
plt.xlabel('Price')
plt.ylabel('Frequency')
plt.grid(True)
plt.show()
