title: sklearn logistic regression详细注释版
date: 2019/10/28
categories:
- Program Development
tags:
- Python
---


```python
# from sklearn import datasets
#
# iris = datasets.load_iris()
# x = iris.data
# y = iris.target

import pickle
import numpy as np
import pandas as pd
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler, label_binarize
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import confusion_matrix, classification_report, roc_curve, auc

if __name__ == '__main__':
    data = pd.read_csv('iris.data', header=None).values
    # multi-dim features
    X = data[0:100, [0, 2]]
    y = data[0:100, 4]
    y = label_binarize(y, classes=np.unique(y)).ravel()


    # ----- 划分训练集、测试集 -----
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.25, random_state=0)


    # ----- 数据标准化 -----
    sc = StandardScaler()
    sc.fit(X_train)
    X_train_std = sc.transform(X_train)
    # 测试集数据也由训练集的均值、方差来标准化
    X_test_std = sc.transform(X_test)


    # ----- 模型训练 -----
    # LR也可以处理多分类，自动使用one vs others
    lr = LogisticRegression(penalty='l2', solver='liblinear')
    lr.fit(X_train_std, y_train)


    # ----- 模型保存 -----
    with open('lr.pkl', 'wb') as fd:
        pickle.dump(lr, fd)


    # ----- 模型读取 -----
    fd = open('lr.pkl', 'rb')
    lr2 = pickle.load(fd)
    fd.close()


    # ----- 模型推理 -----
    y_test_pred = lr2.predict(X_test_std)
    y_test_proba = lr2.predict_proba(X_test_std)


    # ----- 模型评价 -----
    print('results =')
    for a in zip(y_test_proba, y_test_pred, y_test):
        print(a)

    # ===== acc =====
    print('acc = {}'.format(lr2.score(X_test_std, y_test)))

    # ===== confusion_matrix =====
    # known to be in group i but predicted to be in group j
    print('confusion_matrix = \n{}'.format(confusion_matrix(y_test, y_test_pred)))

    # ===== classification_report =====
    print('classification_report = \n{}'.format(classification_report(y_test, y_test_pred)))

    # ===== cross validation =====
    # 交叉验证并不是一个指标，多次拆分给定数据集，获取模型平均性能。（用来评价一个已训好模型的泛化性能的）
    # scoring参数指定计算的指标，例如scoring='roc_auc'，默认值为模型默认指标
    print('cross_val_score = \n{}'.format(cross_val_score(lr2, X_test_std, y_test, cv=5)))

    # ===== grid search (with cross validation) =====
    # 网格搜索同样并不是一个指标，用于模型训练（选择），往往使用了交叉验证技术
    hyper_paras = dict(penalty=['l1', 'l2'], C=np.logspace(0, 4, 10))
    clf = GridSearchCV(LogisticRegression(solver='liblinear'), hyper_paras, cv=5, verbose=0)
    best_model = clf.fit(X_train_std, y_train)
    print('Best Penalty= \n{}'.format(best_model.best_estimator_.get_params()['penalty']))
    print('Best C: = \n{}'.format(best_model.best_estimator_.get_params()['C']))

    # # ===== roc =====
    # # only for label==1
    # fpr, tpr, _ = roc_curve(y_test, y_test_proba[:, 1])
    # roc_auc = auc(fpr, tpr)
    # # plot
    # import matplotlib.pyplot as plt
    # plt.title('Receiver Operating Characteristic')
    # plt.plot(fpr, tpr, 'b', label='AUC = %0.2f' % roc_auc)
    # plt.legend(loc='lower right')
    # plt.plot([0, 1], [0, 1], 'r--')
    # plt.xlim([0, 1])
    # plt.ylim([0, 1])
    # plt.ylabel('True Positive Rate')
    # plt.xlabel('False Positive Rate')
    # plt.show()

```