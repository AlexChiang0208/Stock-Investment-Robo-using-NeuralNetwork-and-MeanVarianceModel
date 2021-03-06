# Stock-Investment-Robot-using-NeuralNetwork-and-MeanVarianceModel

How to select stocks and invest is a big question that investors will face. Wealth management services provided by financial institutions usually have higher limits. The prevalence of Robo Advisor can provide to general public the professional wealth management and asset allocation service with low threshold.

This research uses Behavior Finance Indicators, Deep Neural Networks, and Optimization Algorithms to construct a complete Stock Robo Advisor, which includes three stages - stock selection, optimal asset allocation, and static rebalancing in three months. The performance of this Stock Robo Advisor is presented in excel and pictures.

The model constructed through the data from 2004 to 2015. It can easily assist investors in making decisions and finding the appropriate stock allocation. For future stock selection strategies, you can continue to use this research as a reference.

###### Here are following details:

## 1. Performance

Calculate the average performance in 59 different entry times: 

#### IRR 17.5%
#### Sigma 19.8%
#### Sharpe Ratio 0.87

![GITHUB](output/portfolio.png)

## 2. Model Description

#### Stage 1: Select N stocks with the highest expected returns from Deep Neural Network

#### Stage 2: Use Markowitzs' MV model to get best investment portfolio

#### Stage 3: Static rebalance in three months (60 days)

* Data: Top 150 listed stocks in TWII, excluding early unlisted and suspended trading stocks.

* X_features: Momentum of returns in Behavior Finance Indicators - mean, sigma, skewness, kurtosis. (Choose better features by finding lower p-value in Linear Regression testing.)

* y_predict: Stock returns three month later



## 3. Code Description

There are five parts about my code. When you download it, you can see different parts which separate from #%% mark.

(1) Features Engineering   

(2) Modeling (Training, evaluation and store it)

(3) Make a portfolio with Markowitz's MV model

(4) Let's investing: Get today's stock id with percentage

(5) Simulate performance of 59 start day

Part 4 and 5 are similar. So after learning part 1 ~ 3, you can easily make your own robot.

Here put some important code for explaining:

#### Split into training and testing data
I spilt the data from 2016. One is used for making model; the other is used for making portfolio.
```python
Data_train_X = data[:'2015-12-31'].drop('predict60D', axis=1)
Data_train_y = data[:'2015-12-31'][['predict60D']]
Data_test_X = data['2016-1-4':].drop('predict60D', axis=1)
Data_test_y = data['2016-1-4':][['predict60D']]
```

#### Features scaling: StandardScaler or MinMaxScaler
Both tarin test data need to use the scaler from train one. By the way, you can also try to scale y_predict, but I didn't.
```python
scaler = MinMaxScaler()
scaler_trainX = scaler.fit(Data_train_X)

Data_train_X_scaled = scaler_trainX.transform(Data_train_X)
Data_train_X_scaled = pd.DataFrame(Data_train_X_scaled, 
                                        index=Data_train_X.index, 
                                        columns=Data_train_X.columns)

Data_test_X_scaled = scaler_trainX.transform(Data_test_X)
Data_test_X_scaled = pd.DataFrame(Data_test_X_scaled, 
                                  index=Data_test_X.index, 
                                  columns=Data_test_X.columns)
```

#### Modeling
This part I spend lots of time for testing. Because I'm still not a master of deep learning, I try method of exhaustion about the layers, nodes, activation function etc., and then find a best one. You must can find the hyperparameters better than mine.
```python
model = Sequential()
model.add(layers.Dense(128, activation='relu',input_dim=(16)))
model.add(layers.Dense(112, activation='relu'))
model.add(layers.Dense(86, activation='relu'))
model.add(layers.Dense(64, activation='relu'))
model.add(layers.Dense(48, activation='relu'))
model.add(layers.Dense(32, activation='relu'))
model.add(layers.Dense(1))
    
# Change an optimizers' learning rate
adam = keras.optimizers.Adam(learning_rate=0.0001)
model.compile(optimizer=adam, loss='mean_squared_error')
    
# Train 300 times
# Training data split 20% to validate it
history = model.fit(Data_train_X_scaled, Data_train_y, 
          epochs=300, batch_size=30, validation_split=0.2, shuffle=False)
```

#### Markowitz's Mean Variable Model
You can change *#bound* for each one's maximal percentage to find the best weights in all assets.
```python
def Portfolio_volatility(weights, mean_returns, cov_matrix):
    returns = np.sum(mean_returns * weights)
    std = np.sqrt(np.dot(weights.T, np.dot(cov_matrix, weights)))    
    return std

def min_variance(mean_returns, cov_matrix):
    num_assets = len(mean_returns)
    args = (mean_returns, cov_matrix)
    constraints = ({'type':'eq', 'fun': lambda x: np.sum(x) - 1})
    bound = (0,0.1) ## Max percentage
    bounds = tuple(bound for asset in range(num_assets))
    
    result = sco.minimize(Portfolio_volatility, num_assets*[1/num_assets], args=args, method='SLSQP', bounds=bounds, constraints=constraints)
    return result
```


## 4. DIY Optimization
I will keep optimizing this model since it was not the best one. There are many ways to do this job, such as changing the input features(Technical Indicators, Financial Indicators, etc.), changing the Neural Network(LSTM), using for-loop or other method to find the best hyperparameters in this model.


## 5. Reminder

(1) To save storage space in GitHub, I only put the first photo in a file. (Solved!! Using Git LFS)

(2) Remember to change the file address and create input, output file when you’re running this code.

(3) You can try to modify maximum of percentage and numbers of stock.

(4) Since portfolio are exchanged every three months, transaction costs are not considered.

(5) When selecting current stocks, you need to remove the missing information. (Sometimes the open source from yfinance has missing data.)

