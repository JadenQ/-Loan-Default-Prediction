## Task04_建模与调参 modeling and tuning

尝试使用LightGBM、Xgboost和Random Forest三种树模型进行预测和集成

#### 1.划分数据集

```python
X_data = train_data[feature_columns]
Y_data = train_data['isDefault']
X_test = test_data[feature_columns]
```

```python
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score
#data 划分
x_train, x_val, y_train, y_val = train_test_split(X_data, Y_data, test_size = 0.2)
#x_train, x_val, y_train, y_val = train_test_split(X_data, np.log(Y_data), test_size = 0.2)
```

#### 2.打包模型使用的函数

```python
import xgboost as xgb
def build_model_xgb(x_train, y_train):
    model = xgb.XGBRegressor(n_estimators = 150, learning_rate = 0.1, gama = 0, max_depth = 7)
    model.fit(x_train, y_train)
    return model

import lightgbm as lgb
def build_model_lgb(x_train, y_train):
    model = lgb.LGBMRegressor(n_estimator = 150, num_leaves = 127, learning_rate = 0.1)
    model.fit(x_train, y_train)
    return model

from sklearn.ensemble import RandomForestRegressor
def build_model_rf(x_train, y_train):
    model = RandomForestRegressor(n_estimators = 150, max_depth = 6, max_features = 'sqrt', criterion = 'mae')
    model.fit(x_train, y_train)
    return model

```

#### 3.使用xgboost进行预测

```python
model_xgb = build_model_xgb(x_train, y_train)
val_xgb = model_xgb.predict(x_val)
AUC_xgb = roc_auc_score(y_val, val_xgb)
print('xgboost validation AUC', AUC_xgb)
```

```python
model_xgb2 = build_model_xgb(X_data, Y_data)
result_xgb = model_xgb2.predict(X_test)
```

#### 4.使用LightGBM预测

```python
model_lgb = build_model_lgb(x_train, y_train)
val_lgb = model_lgb.predict(x_val)
AUC_lgb = roc_auc_score(y_val, val_lgb)
print('lightGBM validation AUC', AUC_lgb)
```

> lightGBM validation AUC 0.7327397721650026

```python
model_lgb2 = build_model_lgb(X_data, Y_data)
result_lgb = model_lgb2.predict(X_test)
```

#### 5.利用ensemble将LightGBM和xgboost模型集成

利用AUC进行加权：

```python
import matplotlib.pyplot as plt
AUC_sum = AUC_xgb + AUC_lgb
predict_y = (1-AUC_lgb/AUC_sum) * result_lgb + (1-AUC_xgb/AUC_sum) * result_xgb
plt.hist(predict_y)
plt.show()
```

#### 6.输出结果

```python
# 输出结果
sub = pd.DataFrame()
sub['id'] = test_data.id
sub['isDefault'] = predict_y
#sub['price'] = np.exp(predict_y)
sub.to_csv('./result.csv', index = False)
```

目前的上榜结果为0.7303，截止2020.9.24晚上排名第88名。