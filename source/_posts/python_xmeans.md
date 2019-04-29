title: Python - XMeans实现
date: 2019/04/29
categories:
- Program Development
tags:
- Python
---


## XMeans聚类、画图 ##

```python
# -*- coding: UTF-8 -*-


import math
from sklearn.cluster import KMeans
import numpy as np


DEBUG_PRINT = False

def cluster_by_xmeans(data, min_k=1, max_k=10, activation_bic_ratio=0.8, init="k-means++", n_init=10, max_iter=300, tol=0.0001):
    """
    :param data: list of points
    :param min_k, max_k: the min and max clusters for xmeans
    :param activation_bic_ratio: the bic threshold for k=1
    :param init, n_init, max_iter, tol: see sklearn.cluster.KMeans
    :return: KMeans object
    """

    if min_k > max_k:
        return None

    cur_k = min_k
    while cur_k < max_k:
        if DEBUG_PRINT: print("cur_k = %d" % cur_k)
        new_k = 0

        kmeans = KMeans(n_clusters=cur_k, init=init, n_init=n_init, max_iter=max_iter, tol=tol).fit(data)
        data_predicted = kmeans.predict(data)
        cluster_data_map = create_cluster_data_map(cur_k, data, data_predicted)

        # 对各个簇进行二分裂尝试
        for i in range(cur_k):
            cur_cluster_data = cluster_data_map[i]
            cur_cluster_center = kmeans.cluster_centers_[i]
            R = len(cur_cluster_data)
            M = len(cur_cluster_center)

            if R >= 2:
                if DEBUG_PRINT: print("\tcheck cluster %d of %d" % (i, cur_k))
                # 检查当前簇的点是否全相同
                equal_flag = True
                for j in range(1, R):
                    if np.linalg.norm(np.array(cur_cluster_data[j]) - np.array(cur_cluster_data[0]) != 0):
                        equal_flag = False
                # 当前簇的点全相同，不做二分裂检查，加速(此处优化是针对：sklearn.cluster.KMeans对完全相同的一组点做2means很慢的问题)
                if equal_flag:
                    if DEBUG_PRINT: print("\t\tequal flag!")
                    new_k += 1
                else:
                    child_kmeans = KMeans(n_clusters=2, init=init, n_init=n_init, max_iter=max_iter, tol=tol).fit(cur_cluster_data)
                    cur_cluster_data_predicted = child_kmeans.predict(cur_cluster_data)
                    child_cluster_data_map = create_cluster_data_map(2, cur_cluster_data, cur_cluster_data_predicted)
                    if DEBUG_PRINT: print("\t\tnot equal")

                    if len(child_cluster_data_map[0]) != 0 and len(child_cluster_data_map[1]) != 0:
                        bic1 = cal_BIC([cur_cluster_center], {0: cur_cluster_data}, R, M)
                        bic2 = cal_BIC(child_kmeans.cluster_centers_, child_cluster_data_map, R, M)
                        # 对cur_k的kmeans每个簇，尝试二分裂。
                        # 如果根据BIC，分裂成功，则一个原簇对应二个新簇。
                        # 如果根据BIC，分裂不成功，对原簇数量不产生影响，即一个簇仍对应一个簇；
                        if DEBUG_PRINT: print("\t\tbic1 = %.2f" % bic1)
                        if DEBUG_PRINT: print("\t\tbic2 = %.2f" % bic2)
                        if bic1 <= bic2:
                            new_k += 2
                            if DEBUG_PRINT:
                                print("\t\tsplit:")
                                for d in cur_cluster_data:
                                    print(d)
                        elif cur_k == 1 and bic2 / bic1 >= activation_bic_ratio:
                            new_k += 2
                            if DEBUG_PRINT:
                                print("\t\tsplit (init):")
                                for d in cur_cluster_data:
                                    print(d)
                        else:
                            new_k += 1
                            if DEBUG_PRINT:
                                print("\t\tno split")
                                for d in cur_cluster_data:
                                    print(d)
                    else:
                        # 无法进行2means，也是分裂不成功的场景
                        new_k += 1
            elif R == 1:
                if DEBUG_PRINT: print("\tno need check cluster %d of %d, count = 1" % (i, cur_k))
                if DEBUG_PRINT: print(cur_cluster_data[0])
                # 该簇只有1个元素，无法做2means，只好保留当前簇
                new_k += 1
            else:
                if DEBUG_PRINT: print("\tshould not create this cluster %d of %d, count = 0" % (i, cur_k))
                # 该簇为空，意味着cur_k-means不成功，不应聚cur_k了
                # 不应考虑本簇，此时new_k不应再自增
                if cur_k != min_k:
                    # 但是注意：若是在cur_k的迭代中，本次cur_k是由上次cur_k-1中某个簇2means决定分裂的，
                    # 故本次cur_k中该簇也会要分裂，对此场景将new_k自减以抵消影响；
                    # 若不是在cur_k的迭代中，则为最初场景cur_k == min_k，仅需不考虑本簇即可，即new_k不再自增。
                    new_k -= 1
            # 迭代到cur_k的某个簇（尝试二分裂的过程中）已经达到max_k
            if new_k >= max_k:
                new_k = max_k
                break
        # 迭代到cur_k的所有簇，经过二分裂尝试后，仍聚cur_k簇（或甚至不足cur_k簇），即已经无法再向下分裂
        if new_k <= cur_k:
            if new_k >= min_k:
                cur_k = new_k
            else:
                cur_k = min_k
            break
        else:
            cur_k = new_k

    if DEBUG_PRINT: print("cur_k = %d" % cur_k)

    return KMeans(n_clusters=cur_k, init=init, n_init=n_init, max_iter=max_iter, tol=tol).fit(data)


def create_cluster_data_map(k, data, data_predicted):
    """
    create a dict: {label: [points_of_this_label]}
    """

    cluster_data_map = {}
    for i in range(k):
        cluster_data_map[i] = []
    for i in range(len(data)):
        cluster_data_map[data_predicted[i]].append(data[i])
    return cluster_data_map


def cal_BIC(centers, labelled_data, R, M):
    """
    calculate the bayesian information criterion
    """

    K = len(centers)
    sigma2 = cal_cluster_variance(centers, labelled_data, R, K)
    if sigma2 <= 0:
        sigma_multiplier = float("-inf")
    else:
        sigma_multiplier = M * 0.5 * math.log(sigma2)
    l = 0.0

    for i in range(K):
        Rn = len(labelled_data[i])
        l += Rn * math.log(Rn) - Rn * math.log(R) - Rn * 0.5 * math.log(2.0 * math.pi) \
             - Rn * sigma_multiplier - (Rn - K) * 0.5

    bic = l - K * (K * (M + 1.0)) / 2.0 * math.log(R)

    return bic


def cal_cluster_variance(centers, labelled_data, R, K):
    """
    calculate the variance for a set of clusters
    """

    sum = 0
    for cur_label in range(K):
        cur_center = centers[cur_label]
        cur_data = labelled_data[cur_label]
        for cur_point in cur_data:
            #dist = np.linalg.norm(np.array(cur_point) - np.array(cur_center))
            dist2 = np.sum(np.square(np.array(cur_point) - np.array(cur_center)))
            sum += dist2

    v = sum * 1.0
    if R - K > 0:
        v = v / (R - K)

    return v


import colorsys
def get_colors(color_count):
    hues = np.linspace(0, 1, color_count, endpoint=False)
    for i in range(color_count):
        #hue = i * 1. / color_count
        hue = hues[i]
        rgb = [int(x * 255.0) for x in colorsys.hsv_to_rgb(hue, 1.0, 1.0)]
        yield "#{0:02x}{1:02x}{2:02x}".format(*rgb)


import matplotlib.pyplot as plt
def plot_by_clusters(cluster_data_map, savepath=None):
    plt.figure(figsize=(8, 6))

    actual_cluster_count = 0
    for c in cluster_data_map:
        if len(cluster_data_map[c]) > 0:
            actual_cluster_count += 1

    colors = get_colors(actual_cluster_count)

    for c in cluster_data_map:
        if len(cluster_data_map[c]) > 0:
            col = next(colors)
            for d in cluster_data_map[c]:
                plt.plot(range(1, len(d) + 1), d, color=col)

    plt.title("actually %d clusters" % actual_cluster_count, fontsize=16)
    plt.xlabel("time")
    plt.ylabel("kpi value")

    if savepath is not None:
        plt.savefig(savepath)
        plt.close()
    else:
        plt.show()


if __name__ == '__main__':
    import time

    """
    # 5 clusters
    data = [[3.487966, 2.617258], [3.052439, 2.939565], [2.541804, 2.855116],
            [2.993872, 2.741651], [2.645149, 2.988544], [3.275658, 2.734759],
            [3.4383, 3.126239], [3.149475, 2.774026], [2.808158, 2.987228],
            [2.609178, 2.608235], [2.89733, 2.502805], [2.599489, 3.317544],
            [3.330802, 3.329103], [2.504006, 3.13618], [2.598317, 2.838451],
            [3.468053, -3.205529], [3.473971, -2.881551], [2.915968, -3.146783],
            [3.287449, -2.818057], [3.199882, -3.082526], [3.33998, -2.892212],
            [2.652254, -3.284869], [3.062117, -3.377962], [3.439667, -2.904208],
            [3.131923, -2.860522], [2.770989, -2.585313], [2.678803, -3.25333],
            [3.20751, -2.602797], [2.548267, -3.350991], [3.488662, -2.793679],
            [-3.419303, -3.460712], [-2.66534, -3.327664], [-2.843735, -2.520843],
            [-3.256643, -2.505888], [-2.723389, -3.145575], [-2.819643, -2.775133],
            [-3.074187, -3.408583], [-3.286655, -2.857967], [-2.650737, -2.697036],
            [-3.152726, -3.137763], [-2.999019, -2.686993], [-2.688528, -2.741138],
            [-2.592045, -3.197096], [-3.160052, -2.52554], [-3.301692, -2.993403],
            [-2.946691, 3.19919], [-3.408824, 2.648383], [-3.148617, 3.378614],
            [-2.563794, 2.702014], [-3.157149, 2.622525], [-2.975722, 2.604931],
            [-2.518403, 3.039607], [-3.140483, 3.32115], [-2.619992, 2.968638],
            [-2.875432, 3.374492], [-2.555934, 2.997142], [-3.254758, 2.814866],
            [-2.947021, 3.04663], [-3.177994, 2.874514], [-3.110401, 3.071354],
            [0.11111, 0.111111], [0.11111, -0.111111], [-0.11111, 0.111111],
            [-0.11111, -0.11111], [0.234232, 0.34322], [-0.23223, 0.3323]]

    # use kmeans directly
    time1 = time.time()
    kmeans = KMeans(n_clusters=5)
    kmeans.fit(data)
    print("use kmeans directly".center(80, "-"))
    print("cur kmeans model: %s" % kmeans)
    print("cur centers: %s" % kmeans.cluster_centers_)
    print("labels of data: %s" % kmeans.predict(data))
    time2 = time.time()
    print("timecost: %0.3f" % (time2 - time1))
    print("".center(80, "-"))

    print
    print

    # use xmeans to find the right k
    time3 = time.time()
    xmeans = cluster_by_xmeans(data, n_init=20)
    print("use xmeans to find the right k".center(80, "-"))
    print("cur kmeans model: %s" % xmeans)
    print("cur centers: %s" % xmeans.cluster_centers_)
    print("labels of data: %s" % xmeans.predict(data))
    time4 = time.time()
    print("timecost: %0.3f" % (time4 - time3))
    print("".center(80, "-"))
    """


    # another test
    """
    data0 = [(1.2, 0, 0), (1, 0, 0), (1.1, 0, 0),
             (0, 0.9, 0), (0, 0.8, 0), (0, 1, 0),
             (0, 0, 1), (0, 0, 1.3), (0, 0, 1.1)]

    data1 = [(1, 0, 0), (1, 0, 0), (1, 0, 0),
             (0, 1, 0), (0, 1, 0), (0, 1, 0),
             (0, 0, 1), (0, 0, 1), (0, 0, 1)]

    data2 = [(1.2, 0, 0), (1, 0, 0), (1.1, 0, 0),
             (0, 0.9, 0), (0, 0.8, 0), (0, 1, 0),
             (0, 0, 1), (0, 0, 1.3), (0, 0, 1.1), (1, 0, 1)]
    
    time1 = time.time()
    xmeans = cluster_by_xmeans(data0)
    print("cur kmeans model: %s" % xmeans)
    print("cur centers: %s" % xmeans.cluster_centers_)
    print("labels of data: %s" % xmeans.predict(data0))
    time2 = time.time()
    print("timecost: %0.3f" % (time2 - time1))

    print
    print
    
    time3 = time.time()
    xmeans = cluster_by_xmeans(data1)
    print("cur kmeans model: %s" % xmeans)
    print("cur centers: %s" % xmeans.cluster_centers_)
    print("labels of data: %s" % xmeans.predict(data1))
    time4 = time.time()
    print("timecost: %0.3f" % (time4 - time3))

    print
    print
    
    time5 = time.time()
    xmeans = cluster_by_xmeans(data2)
    print("cur kmeans model: %s" % xmeans)
    print("cur centers: %s" % xmeans.cluster_centers_)
    print("labels of data: %s" % xmeans.predict(data2))
    time6 = time.time()
    print("timecost: %0.3f" % (time6 - time5))
    """

    """
    data1 = [
        [0.999, 1.999, 0.999, 2.999, 6.999, 7.999, 3.999, 1.999, 0.999],
        [0.999, 1.999, 0.999, 2.999, 6.999, 7.999, 3.999, 1.999, 0.999],
        [1, 2, 1, 3, 7, 8, 4, 2, 1],
        [1.001, 2.001, 1.001, 3.001, 7.001, 8.001, 4.001, 2.001, 1.001]
    ]

    data2 = [
        [2.999, 2.999, 2.999, 2.999, 2.999, 2.999, 2.999, 2.999, 2.999],
        [2.999, 2.999, 2.999, 2.999, 2.999, 2.999, 2.999, 2.999, 2.999],
        [3, 3, 3, 3, 3, 3, 3, 3, 3],
        [3.001, 3.001, 3.001, 3.001, 3.001, 3.001, 3.001, 3.001, 3.001]
    ]

    data3 = [
        [-0.001, -0.001, -0.001, -0.001, -0.001, -0.001, -0.001, -0.001, -0.001],
        [-0.001, -0.001, -0.001, -0.001, -0.001, -0.001, -0.001, -0.001, -0.001],
        [0, 0, 0, 0, 0, 0, 0, 0, 0],
        [0.001, 0.001, 0.001, 0.001, 0.001, 0.001, 0.001, 0.001, 0.001]
    ]

    # for data in [data1, data2, data3]:
    #     for d in data:
    #         for i in range(len(d)):
    #             d[i] += 100000

    time3 = time.time()
    xmeans = cluster_by_xmeans(data1, min_k=1)
    print("use xmeans to find the right k".center(80, "-"))
    print("cur kmeans model: %s" % xmeans)
    print("cur centers: %s" % xmeans.cluster_centers_)
    print("labels of data: %s" % xmeans.predict(data1))
    time4 = time.time()
    print("timecost: %0.3f" % (time4 - time3))
    print("".center(80, "-"))

    time3 = time.time()
    xmeans = cluster_by_xmeans(data2, min_k=1)
    print("use xmeans to find the right k".center(80, "-"))
    print("cur kmeans model: %s" % xmeans)
    print("cur centers: %s" % xmeans.cluster_centers_)
    print("labels of data: %s" % xmeans.predict(data2))
    time4 = time.time()
    print("timecost: %0.3f" % (time4 - time3))
    print("".center(80, "-"))

    time3 = time.time()
    xmeans = cluster_by_xmeans(data3, min_k=1)
    print("use xmeans to find the right k".center(80, "-"))
    print("cur kmeans model: %s" % xmeans)
    print("cur centers: %s" % xmeans.cluster_centers_)
    print("labels of data: %s" % xmeans.predict(data3))
    time4 = time.time()
    print("timecost: %0.3f" % (time4 - time3))
    print("".center(80, "-"))
    """

    # xmeans = cluster_by_xmeans(data, min_k=1, max_k=4)
    # cluster_data_map = create_cluster_data_map(len(xmeans.cluster_centers_), data, xmeans.predict(data))
    # plot_by_clusters(cluster_data_map)#, savepath="D://out.png")

```
