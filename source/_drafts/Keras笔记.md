---
title: Keras笔记
abbrlink: d5cc
date: 2019-07-08 16:48:51
tags:
 - 炼丹
 - keras
categories:
 - 笔记&札记
---
瞎写的
<!-- more -->

## Keras函数式API

构造张量

    inputs = Input(shape=(64,))

创建层

    dense = layers.Dense(32, activation='relu')

    output_tensor = dense(input_tensor)

输出也是一个张量

## 回调函数等

 EarlyStopping中断训练, ModelCheckpoint保存模型,如
```python
callbacks_list = [
keras.callbacks.EarlyStopping(
monitor='acc',
patience=1,
),
keras.callbacks.ModelCheckpoint(
filepath='my_model.h5',
monitor='val_loss',
save_best_only=True,
)
]
```
改变学习率,ReduceLROnPlateau函数
```python
callbacks_list = [
keras.callbacks.ReduceLROnPlateau(
monitor='val_loss'
factor=0.1,
patience=10,
)
]
model.fit(x, y,
epochs=10,
batch_size=32,
callbacks=callbacks_list,
validation_data=(x_val, y_val))
```

## 参考

1. [keras官方文档](https://keras.io)
2. Python深度学习
