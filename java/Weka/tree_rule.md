# 决策树

## 简介

决策树在许多实际应用，因为它们速度快，并且能够产生非常准确的可理解输出。下面介绍如何学习稳健、通用的决策树，包括处理数值属性和缺失值，以及如何修剪树中没有得到足够数据支持的部分。下面的讨论基于经典的 C4.5 算法，同时介绍 CART 决策树如何使用交叉验证来实现更稳健的修剪策略。

后面部分介绍规则学习（rule learning）。如果实现合理，规则学习具有决策树学习的优点，虽然运行成本略高，但是通常会得到更简洁的分类模型。规则学习在实践中没有决策树受欢迎。生成简洁准确的规则集比学习简洁准确的决策树更棘手。实现该目标通常有两种策略：基于规则集合的全局优化；另一种是从部分生成或修剪的决策树中提取规则。

## 1. 决策树

C4.5 算法基于分治算法生成决策树，它需要以多种方式进行扩展才能用于实际问题：

- 首先是如何处理数字属性以及缺失值
- 如何修剪决策树，因为基于分治算法构建的树在训练集上通常表现良好，但通常过拟合
- 如何将决策树转换为分类规则，并以 C4.5 算法为例
- CART 系统中实现的修剪策略，用于学习分类和回归树

### 数字属性

对数字属性，可以进行 binary-split。以如下数据集为例：

| Outlook  | Temperature | Humidity | Windy | Play |
| -------- | ----------- | -------- | ----- | ---- |
| Sunny    | 85          | 85       | FALSE | No   |
| Sunny    | 80          | 90       | TRUE  | No   |
| Overcast | 83          | 86       | FALSE | Yes  |
| Rainy    | 70          | 96       | FALSE | Yes  |
| Rainy    | 68          | 80       | FALSE | Yes  |
| Rainy    | 65          | 70       | TRUE  | No   |
| Overcast | 64          | 65       | TRUE  | Yes  |
| Sunny    | 72          | 95       | FALSE | No   |
| Sunny    | 69          | 70       | FALSE | Yes  |
| Rainy    | 75          | 80       | FALSE | Yes  |
| Sunny    | 75          | 70       | TRUE  | Yes  |
| Overcast | 72          | 90       | TRUE  | Yes  |
| Overcast | 81          | 75       | FALSE | Yes  |
| Rainy    | 71          | 91       | TRUE  | No   |

其中 temperature 为数字属性。合并重复值，有 12 个值：

| 64   | 65   | 68   | 69   | 70   | 71   | 72   | 75   | 80   | 81   | 83   | 85   |
| ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- | ---- |
| Yes  | No   | Yes  | Yes  | Yes  | No   | No   | Yes  | No   | Yes  | Yes  | No   |
|      |      |      |      |      |      | Yes  | Yes  |      |      |      |      |

12 个值，有 11 个可能的分割位置（breakpoint），如果不允许 breakpoint 将同一类的项目分开，则有 8 个 breakpoints。每个 breakpoint 的信息增益可以按常规方式计算。例如，温度 < 71.5 有 4 个 yes 和 2 个 no，而 > 71.5 有 5 个 yes 和 3 个 no。因此该测试温度的信息值为：
$$
\text{Info}([4,2],[5,3])=\frac{6}{14}\times \text{info}([4,2])+\frac{8}{14}\times\text{info}([5,3])=0.939 \text{bits}
$$
