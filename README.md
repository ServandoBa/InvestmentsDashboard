# InvestmentsDashboard

## Context
I have a financial friend who approached me with a specific inquiry: the feasibility of monitoring his investment portfolio in a centralized dashboard. Recognizing the potential value in such a solution, I embarked on a project to create a tailored dashboard for him. The objective is to empower him with a data-driven tool that facilitates a comprehensive view of his portfolio distribution, enabling informed decision-making in managing his investments.

<details>
<Summary> <h2> I. Business Understanding </h2> </Summary>

### 1.1 Project Objectives
The primary objective of the portfolio investments monitoring project is to provide a comprehensive and interactive dashboard that enables the user to monitor the distribution of their investment portfolio. This involves categorizing investments by multiple variables such as sector, industry, and market. Additionally, the project aims to facilitate the monitoring of individual stock performance based on various key metrics.

### 1.2 Addressed questions and porblems

In order to define and refine the project objectives, several crucial questions were addressed:

- Metrics for Monitoring: <br>
a. What specific metrics would you like monitor within your investment portfolio?
b. Do the metrics require a complex calculation beyond Yahoo Finance metrics?  

- Data collection:<br>
a. Does the broker provide a function to export your portfolio information?<br>
b. How is the investment portfolio data exported from the broker?<br>
c. Can you provide me a sample to better understand its structure?

- Future Visualization Preferences:<br>
a. Are there any specific visualization preferences or features that you would like to visualize?

### 1.3 Key Performance Indicators (KPIs)

The essential KPIs for monitoring the portfolio include:

Investment distribution:
- Amount invested distribution categorized by Sector by Industry

Individual stock performance:
- Trailing P/E
- Forward P/E
- Price-to-Book
- Book Value
- Recommendations

### 1.4 Stakeholder Identification

The primary stakeholder for this project is a financial analyst specializing in investments. He focuses on managing investments across various markets with the goal of optimizing portfolio profits.

### 1.5 Business Risks or Limitations

The primary identified risk is the project's reliance on the Yahoo Finance library in Python. The potential risk arises if the library is not regularly updated by its owners, which may impact the availability and accuracy of financial data.
</details>

<details>
<Summary> <h2> II. Data Understanding </h2> </Summary>

### 2.1 Data Collection

The data exported from the broker shows the stock symbols, market value (current price), profit or loss, among others. The format of the file is completely unfavored for data analysis due to the table is offset from the (0,0) and the table structure in unconventional, so the file needs to be manipulated to just retrieve the relevant values from the table.

### 2.2 Data Exploration

#### 2.2.1 Key Attributes

The dataset is composed of multiple columns, each representing different aspects of the investment portfolio. However, not all columns are equally relevant for the project. The following table shows the multiple columns that can be found in the table:

***TABLE HERE

#### 2.2.2 Relevant Attributes

Taking into account the previous columns, there are just few attributes that are relevant for the project. The next list breaks down each of these attributes and explains why they could be helpful for the project.

***TABLE 2 HERE

It is important to note that the data does not require a cleaning process; however, a transformation process is necessary to extract the pertinent metrics. Additionally, there is potential for enriching the dataset by incorporating information such as 'Market,' indicating the origin of the stock (Nacional, SIC, or Efectivo).

***TABLE 3 HERE

</details>

<details>
<Summary> <h2> III. Data Preparation </h2> </Summary>

### 3.1 Construct Data
To handle the original data pulled from the broker's website, we're diving into Python. The game plan here is to clean up the data mess, getting rid of the noise and keeping only the metrics that matter. The following code will be the entire process to retrive the original file and transform it into a table with relevant metrics only.

<details>
<Summary> Code </Summary>
  
```
import yfinance as yf
import pandas as pd
from datetime import date, timedelta

df = pd.read_excel('Portfolio_db\stock_portfolio2.xlsx', header=1)
df_section = df.iloc[:, 1:]
sections = df_section[df_section.iloc[:, 0] == 'Emisora/Fondo'].index
headers = dict(df_section.iloc[sections[0],:])

data_nacional1 = df_section.iloc[sections[0]:sections[1]-1,:].rename(columns=headers).drop(0)
data_nacional1['Market'] = 'Nacional'

data_SIC1 = df_section.iloc[sections[1]:sections[2]-1,:].rename(columns=headers).drop(len(data_nacional1)+2)
data_SIC1['Market'] = 'SIC'

data_cash1 = pd.DataFrame(df_section.iloc[sections[2]:,:].rename(columns=headers).drop(len(data_nacional1)+len(data_SIC1)+4).sum()).transpose()
data_cash1.at[0, 'Emisora/Fondo'] = 'Efectivo'
data_cash1['Market'] = 'Efectivo'

data_port1 = pd.concat([data_nacional1, data_SIC1, data_cash1], ignore_index=True)[['Emisora/Fondo', 'Valor mercado', 'Market', 'P / M']]
data_port1['Emisora/Fondo'] = data_port1['Emisora/Fondo'].apply(lambda x: x.replace(' *', '').strip() if isinstance(x, str) else x)
data_port1.rename(columns={'Emisora/Fondo': 'Ticker'}, inplace=True)

```
</details>

### 3.2 Enriching the Dataset

This process is one of the most extensive within Data preparation process due to the project is based on add more data from other source, which in this case is Yahoo Finance, to the original data. In this library we will retrieve fundamental information about each stock in one table and also extract historical data from each stock.  

<details>
<Summary> Code </Summary>

```
# Get stock data
tickers = [ticker for ticker in data_port1['Ticker']]

# Create function to retrieve information from Yahoo Finance and store it into a dictionary as output
def GetData(symbol):
    stock = yf.Ticker(symbol)

    try:
        # Get information and store it into variables
        industry = stock.info.get("industry", None)
        sector = stock.info.get("sector", None)
        trailingPE = round(float(stock.info["trailingPE"]),1) if "trailingPE" in stock.info and stock.info["trailingPE"] is not None else None
        forwardPE = round(float(stock.info["forwardPE"]),1) if "forwardPE" in stock.info and stock.info["forwardPE"] is not None else None
        bookValue = round(float(stock.info["bookValue"]),1) if "bookValue" in stock.info and stock.info["bookValue"] is not None else None
        priceToBook = round(float(stock.info["priceToBook"]),1) if "priceToBook" in stock.info and stock.info["priceToBook"] is not None else None
        recommendationKey = stock.info.get("recommendationKey", None)
        targetHighPrice = round(float(stock.info["targetHighPrice"]),3) if "targetHighPrice" in stock.info and stock.info["targetHighPrice"] is not None else 0
        targetMeanPrice = round(float(stock.info["targetMeanPrice"]),3) if "targetMeanPrice" in stock.info and stock.info["targetMeanPrice"] is not None else 0
        targetLowPrice = round(float(stock.info["targetLowPrice"]),3) if "targetLowPrice" in stock.info and stock.info["targetLowPrice"] is not None else 0
        
        #Create dictionary based on previous variables and return it
        stock_info = {
            'symbol': symbol,
            'industry': industry,
            'sector': sector,
            'trailing_PE': trailingPE,
            'forward_PE': forwardPE,
            'book_Value': bookValue,
            'price_To_Book': priceToBook,
            'recommendation_Key': recommendationKey,
            'target_High_Price': targetHighPrice,
            'target_Mean_Price': targetMeanPrice,
            'target_Low_Price': targetLowPrice
            }
            
        return stock_info
    
    except:
        # In case the stock cannot be found in yahoo library, return the same structure but with Null values
        stock_info = {
            'symbol': symbol,
            'industry': None,
            'sector': None,
            'trailing_PE': None,
            'forward_PE': None,
            'book_Value': None,
            'price_To_Book': None,
            'recommendation_Key': None,
            'target_High_Price': 0,
            'target_Mean_Price': 0,
            'target_Low_Price': 0
            }
        return stock_info

#apply function to each value in Ticker column from data_port1 table 
data = pd.DataFrame(data_port1['Ticker'].apply(GetData).tolist())

#Merge the previous table with the fundamental information of each stock with the original dataset to consider Valor mercado, Market and P / M columns
final_data = data.merge(data_port1, left_on = 'symbol', right_on ='Ticker', how='right').drop(columns='Ticker')

#Get stock daily data

today = date.today()

endd = today.strftime("%Y-%m-%d")
startd = today - timedelta(days=365)
startdd = startd.strftime("%Y-%m-%d")


# Create function to retrieve data from each day of the last twelve months from yahoo finance 
def GetDataHistoric(symbol):

    try:
        h_data = yf.download(symbol, start=startdd, end=endd)[['High', 'Low', 'Volume']]
        h_data = h_data.copy()
        h_data.loc[:, 'Mean'] = (h_data['High'] + h_data['Low']) / 2
        h_data_stacked = h_data[['Mean', 'Volume']].reset_index()
        h_data_stacked['Ticker'] = symbol
        h_data_stacked.loc[:, 'Volume'] = h_data_stacked['Volume'].astype(int)
        return h_data_stacked

    except Exception as e:
        print(f"Error for {symbol}: {e}")

#Retrieve ticker historical data
historical_data = pd.DataFrame(columns=['Date', 'Mean', 'Volume','Ticker'])

#Apply function to Ticker column to get Historical Data and add it into historical_data
for item in data_port1['Ticker']:
    try:
        historical_data = pd.concat([historical_data, GetDataHistoric(item)], ignore_index=True)
    except Exception as e:
        print(f"Error for {item}: {e}")

final_daily_data = pd.DataFrame(historical_data)
```
</details>

The data preparation process will be facilitated using the Python script feature in Power BI. This automation streamlines the entire procedure, ensuring the seamless retrieval of the necessary tables for the creation of the dashboard.

</details>


<details>
<Summary> <h2> IV. Data modeling </h2> </Summary>

### 4.1 Data connection

In the previous section, the Python script will be executed through Power BI. This feature will help automate the transformation process, combining and enriching tables before data deployment. After this process, two tables will be created: final_data, storing fundamental information, and final_daily_data, storing the last twelve months of each stock. These tables will be the main sources for the dashboard.  

### 4.2 Dashboard Design

Due to the client's specific needs, the visualizations are divided into two sections. The first section displays general portfolio distribution, presenting Amount Money distribution by Sector in a treemap. This visualization effectively represents distribution considering subgroups. I took the initiative and decided to add a pie graph to show the portfolio distribution by Market, this visualization will show how much money are in cash, national investments (Mexico) and SIC. The second section focuses on individual stock monitoring, showcasing performance metrics such as Trailing PE, Forward PE, Book Value, Book-to-Price, Buy/Sell recommendations by Yahoo Finance, and the Min/Mean/Max target value and I also took the initiative to show the Last twelve months of the market value stock with its volume by day.

SECTION 1 TABLE HERE / SECTION 2 TABLE HERE

### 4.3 Dashboard Creation

Now, the exciting part, the dashboard creation. Considering the previous information, the structure takes into account both general portfolio distribution and individual stock monitoring.


Section 1 Explanation here


Section 2 Explanation here


### 4.4 Iterative development

There was recurrent interaction with the client to identify any areas for improvement or additional information. The only changes applied to the visualization were to the background design, as the client preferred a minimalist design over a striking one. Based on Scrum, this section is crucial for understanding the client's needs quickly and making changes based on what is built as soon as possible to identify opportunities for improvement.    

</details>

