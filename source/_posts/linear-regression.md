---
title: 线性回归：从公式到梯度下降
categories:
  - 机器学习
tags:
  - 机器学习
  - 线性回归
  - 梯度下降
mathjax: true
description: 线性回归是机器学习的"Hello World"，本文从公式推导到 Python 实现再到梯度下降，帮你一次性吃透。
abbrlink: '2034'
date: 2024-05-20 10:00:00
---

## 一、什么是线性回归

线性回归（Linear Regression）试图学到一个**线性模型**来预测实值输出：

$$
\hat{y} = w^\top x + b
$$

其中 $w$ 是权重向量，$b$ 是偏置项。给定数据集 $D = \{(x_i, y_i)\}_{i=1}^{m}$，我们希望找到最优的 $w, b$。

## 二、损失函数

最常用的损失函数是**均方误差（MSE）**：

$$
L(w, b) = \frac{1}{m} \sum_{i=1}^{m} \left( \hat{y}_i - y_i \right)^2
       = \frac{1}{m} \sum_{i=1}^{m} \left( w^\top x_i + b - y_i \right)^2
$$

目标：$\displaystyle \min_{w, b} L(w, b)$。

## 三、两种求解方式

### 3.1 正规方程（闭式解）

对 $w, b$ 求偏导并令其为零，可得解析解（把 $b$ 吸收进 $w$ 后）：

$$
w^* = (X^\top X)^{-1} X^\top y
$$

- ✅ 优点：一步到位，无需调参
- ❌ 缺点：$X^\top X$ 求逆复杂度 $O(n^3)$，特征多时不可接受

### 3.2 梯度下降（迭代式）

参数沿负梯度方向更新：

$$
w \leftarrow w - \eta \frac{\partial L}{\partial w}, \quad
b \leftarrow b - \eta \frac{\partial L}{\partial b}
$$

其中 $\eta$ 是学习率。常见变体：

| 方法 | 特点 |
| --- | --- |
| BGD | 每轮用全部样本更新，稳定但慢 |
| SGD | 每条样本更新一次，快但震荡 |
| Mini-batch GD | 折中，工程上最常用 |

## 四、Python 实现

```python
import numpy as np

def linear_regression_gd(X, y, lr=0.01, epochs=1000):
    m, n = X.shape
    w = np.zeros(n)
    b = 0.0
    for _ in range(epochs):
        y_pred = X @ w + b
        grad_w = (2 / m) * X.T @ (y_pred - y)
        grad_b = (2 / m) * np.sum(y_pred - y)
        w -= lr * grad_w
        b -= lr * grad_b
    return w, b

# 用 sklearn 验证
from sklearn.linear_model import LinearRegression
from sklearn.datasets import make_regression
X, y = make_regression(n_samples=200, n_features=1, noise=15, random_state=0)
w, b = linear_regression_gd(X, y, lr=0.05, epochs=2000)
print(f"w={w[0]:.3f}, b={b:.3f}")
print(f"sklearn: w={LinearRegression().fit(X, y).coef_[0]:.3f}, "
      f"b={LinearRegression().fit(X, y).intercept_:.3f}")
```

## 五、常见坑

1. **特征量纲差异大** → 一定要先 `StandardScaler`。
2. **学习率过大** → 损失震荡甚至发散。
3. **特征共线性** → $X^\top X$ 接近奇异，考虑岭回归。
4. **欠拟合 vs 过拟合** → 多项式回归 + 正则化是良药。

## 六、扩展阅读

- 《统计学习方法》- 李航，第 1 章
- CS229 Lecture 1 (Andrew Ng)
- scikit-learn: [LinearRegression](https://scikit-learn.org/stable/modules/linear_model.html#ordinary-least-squares)

---

下一篇将介绍 **多项式回归与岭回归**，敬请期待。
