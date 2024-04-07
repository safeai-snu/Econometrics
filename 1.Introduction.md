# Ch 1.1. Intro 실습 코드

### 필요한 패키지 불러오기

```
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import seaborn as sns
from statsmodels.tsa.holtwinters import ExponentialSmoothing
```
### 데이터 불러오기
```
austa = pd.read_csv('https://raw.githubusercontent.com/safeai-snu/Econometrics/main/dataset/Ch.1/aus_visit.csv',index_col=0)
austa['Month'] = pd.to_datetime(austa['Month'])
```

### 학습 데이터 분리
```
austa_up_to_2010 = austa[austa['Month'] <= '2010-12-31']
```

### 모델 fitting
```
model = ExponentialSmoothing(austa_up_to_2010['Visitors'], seasonal='add', seasonal_periods=12).fit()
```

### 시뮬레이션
```
forecast_start_date = '2011-01-01'
forecast_steps = 48  
num_simulations = 100
prediction = model.forecast(steps=forecast_steps)

np.random.seed(1967)
simulated_paths = pd.DataFrame()
residual_std = model.resid.std()

for i in range(num_simulations):
    new_residuals = np.random.normal(0, residual_std, size=forecast_steps)
    simulated_forecast = prediction + new_residuals
    simulated_df = pd.DataFrame({
        'Month': pd.date_range(start=forecast_start_date, periods=forecast_steps, freq='M'),
        'Visitors': simulated_forecast,
        'Simulation': i + 1
    })
    simulated_paths = pd.concat([simulated_paths, simulated_df], ignore_index=True)
```
    
### 시각화
```
plt.figure(figsize=(12, 8))
austa_filtered = austa[(austa['Month'] >= '2000-01-01') & (austa['Month'] <= '2010-12-31')]
actual_post_2010 = austa[(austa['Month'] > '2010-12-31')& (austa['Month'] <= '2016-12-31')]
sns.lineplot(data=austa_filtered, x='Month', y='Visitors', label='Historical Data (2000-2010)', color='blue')
sns.lineplot(data=actual_post_2010, x='Month', y='Visitors', label='Actual Data (Post 2010)', color='green')

for i in range(1, num_simulations+1):
    sim_data = simulated_paths[simulated_paths['Simulation'] == i]
    sns.lineplot(data=sim_data, x='Month', y='Visitors', color='gray', alpha=0.1)
    
plt.title("Total Visitors to Australia (2000-2010): Historical and Simulated Post 2010")
plt.xlabel("Month")
plt.ylabel("Visitors(Thousands)")
plt.legend()
plt.show()
```

![호주 방문자수](https://github.com/safeai-snu/Econometrics/blob/main/images/Ch.1/aus_visitor.png?raw=true)
