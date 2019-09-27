title: 多模型融合方法
date: 2019/09/27
categories:
- Machine Learning
tags:
- Model Futsion
---

## 融合方式

各个classifier好而不同

- summing
- voting (futhermore, use the correct classification ratios as weights)
- cascading
  - give the solved record final result, pass the unsolved record to the next classifier
    - all classifiers for one class, eg: rule based method (cbr, association rule mining, ...) + ...
    - one classifier for each class
- stacking: previous model outputs as latter model inputs （多模型结果做特征）
  - lr: linar regression, logistic regression
- multi stages: process design for steps focusing on different problems
  - predicting a feature
  - clustering source data
    - 同质化，去除noise、unrepresentative，数据集更representative、更纯净
      - a two-stage clustering method consisting of a hierarchical model, such as Ward’s minimum variance method (or self-organizing map), followed by a non-hierarchical method, such as K-means method, can provide a better solution. The key is that hierarchical methods can determine the candidate number of clusters and starting point of each cluster, and non-hierarchical methods can provide better clustering result with the specified information.
	  - 直接删除isolated clusters、进一步处理inconsistent clusters
    - different models、as a feature、switch labels
      - switch labels（面向分类）: For the applicants in the inconsistent cluster, the good credit applicants would have more of an opportunity to delay future payments than other good credit applicants, and the bad credit applicants would have more of an opportunity to repay the delayed payments than the other bad credit applicants. To indicate this behavioral trend in the built credit scoring model, cluster-2 was further divided into cluster-21 and cluster-22 according to their original credit status.
