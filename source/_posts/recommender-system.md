---
title: 推荐系统笔记
tags:
  - 炼丹
  - 推荐系统
categories:
  - 笔记
abbrlink: bccd
date: 2019-10-18 9:52:43
---
等待填坑
<!-- more -->
大概分为用户建模模块、推荐对象建模模块、推荐算法模块

## 主要算法

### 协同过滤算法

最老最经典的一个算法了，大概是给用户推荐兴趣和关注点相似用户的感兴趣内容。

协同过滤有基于领域的方法(neighborhood methods)和隐语义模型(latent factor models)，基于图的随机游走算法(random walk on graph)
下面两个是基于邻域的方法：

#### 基于物品的协同过滤

（item-based collaborative filtering，ItemCF）
找到物品之间的属性关联，向用户推荐属性相似的物品
设N(i)是喜欢i物品的用户数
相似度：

$$w_{ij}=\frac{|N(i)\cap N(j)|}{|N(j)|}$$

避免热门物品干扰，改为

$$w_{ij}=\frac{|N(i)\cap N(j)|}{\sqrt{|N(j)||N(i)|}}$$

#### 基于用户的协同过滤

（User-based collaborative filtering，UserCF）
找到和用户相似的其他用户，将其他用户感兴趣的产品推荐给用户。还有一种是“购买过XXX等用户也XXX”，算是这个方法的一个变种

计算相似度，N(u)是用户正反馈的集合，N(v)是用户负反馈物品的集合
Jaccard相似度：

$$w_{u,v}= \frac{|N(u)\cap N(v)|}{|N(u)\cup N(v)|}$$

余弦相似度：

$$w_{u,v}= \frac{|N(u)\cap N(v)|}{\sqrt{|N(u)||N(v)|}}$$

## 冷启动问题

等待填坑，利用用户注册信息，人工标注等

## 大众行为/个性化推荐

等待填坑

## imdb电影评论推荐例子

这里使用[movielens数据集](http://files.grouplens.org/datasets/movielens/)，
数据集的rating.csv包括用户id，物品id，评分，评论时间。(这个csv文件第一行是userid，movieid，·····，把第一行直接删掉或者从第二行开始读取，不然会报TypeError: unsupported operand type(s) for -: 'str' and 'int'的错)
使用的算法是SVD矩阵分解。构建一个用户-物品矩阵，每一项的值是评分，这是一个极其稀疏的矩阵
我们导入数据集并分割：

```python
header = ['user_id', 'item_id', 'rating', 'timestamp']  # 用户id，物品id，评分，评论时间

df = pd.read_csv('../data/u.data', sep='\t', names=header)
n_users = df.user_id.unique().shape[0]  #用户数量
n_items = df.item_id.unique().shape[0]  #物品数量
print('Number of users = ' + str(n_users) + ' | Number of movies = ' + str(n_items))
# 数据集分割——训练集：测试集 = 3:1
train_data,test_data = train_test_split(df, test_size = 0.25)
```

建立评分矩阵
```python
train_data_matrix = np.zeros((n_users, n_items))
test_data_matrix = np.zeros((n_users, n_items))
#使用 pandas 遍历行数据
for line in train_data.itertuples():
    #训练集评分矩阵
    train_data_matrix[line[1]-1, line[2]-1] = line[3]
for line in test_data.itertuples():
    #测试集评分矩阵
    test_data_matrix[line[1]-1, line[2]-1] = line[3]
```

计算余弦相似度：

```python
user_similarity = pairwise_distances(train_data_matrix, metric = "cosine")  # 计算余弦距离
item_similarity = pairwise_distances(train_data_matrix.T, metric = "cosine")
```

基于用户和基于物品的协同过滤

```python
def predict(rating, similarity, type = 'user'):
    if type == 'user':
        # 
        mean_user_rating = rating.mean(axis = 1)    #mean函数：压缩列，对各行求均值，返回 m *1 矩阵
        # print(mean_user_rating)
        rating_diff = (rating - mean_user_rating[:,np.newaxis])
        pred = mean_user_rating[:,np.newaxis] + similarity.dot(rating_diff) / np.array([np.abs(similarity).sum(axis=1)]).T
        #dot函数：矩阵相乘；np.abs()：矩阵元素的绝对值  .T:转置
        # print('test',pred.min())
    elif type == 'item':
        pred = rating.dot(similarity) / np.array([np.abs(similarity).sum(axis=1)])
        # print('test2',pred.max())
    return pred
```

```python
item_prediction = predict(train_data_matrix, item_similarity, type = 'item')
user_prediction = predict(train_data_matrix, user_similarity, type = 'user')
print(len(item_prediction))
print(len(item_prediction[0]))
print(np.argsort(item_prediction[0]))
```
计算均方误差
```python
def rmse(prediction, ground_truth):
    # 计算均方误差
    #nonzero(a)返回数组a中值不为零的元素的下标
    #flatten()创建矩阵
    prediction = prediction[ground_truth.nonzero()].flatten()
    ground_truth = ground_truth[ground_truth.nonzero()].flatten()
    return sqrt(mean_squared_error(prediction, ground_truth))
print('User based CF RMSE: ' + str(rmse(user_prediction, test_data_matrix)))
print('Item based CF RMSE: ' + str(rmse(item_prediction, test_data_matrix)))
```

SVD分解的方法，其中k是特征值个数，指定将分解为$m*k$和$k*n$，中间的对角矩阵有k个特征值
>k: int, optional,Number of singular values and vectors to compute. Must be 1 <= k < min(A.shape)

```python
u, s, vt = svds(train_data_matrix, k=500)
s_diag_matrix = np.diag(s)
X_pred = np.dot(np.dot(u, s_diag_matrix), vt)

print('User-based CF MSE: ' + str(rmse(X_pred, test_data_matrix)))
```

参考

[基于矩阵分解的推荐算法](https://lumingdong.cn/recommendation-algorithm-based-on-matrix-decomposition.html)
[项亮的《推荐系统实践》的代码实现](https://github.com/qcymkxyc/RecSys)
