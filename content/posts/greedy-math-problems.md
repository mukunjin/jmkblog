---
title: 用贪心做选压？
date: 2026-04-25
categories: 计算机
---
## 先说题目
数列 a_k = k²，其中 k = 1, 2, 3, …, 125。
在其中每个一个数前任意加上＋号或－号，并将所得的代数和的绝对值记为结果，称这种操作为“H运算”。那么，对于上述数列而言，是否存在某种“H运算”使得数列的运算结果满足最小值为1？
> 请注意，这是简化后的题目，且部分描述做了改动。

## 发生了什么
我自己在补习班做到这个题的。我们老师说这个题很难很超纲，他自己都做不来，说如果我感兴趣的话，去找学校老师问一下。我去学校问了老师，我数学老师不会，她就去问了年级上的另一个数学老师，他也不会。我数学老师就把我叫到办公室，对我说中考绝对不会考到这个题，不要给自己压力，而且确实这个题是两年前的老题了。拿到班上去问那些同学，他们也不会，做出来的答案全都是错的。

## 转折点
我有一个同学把我这个题抄了一遍，晚上带回家去问他认识的一个大学生，然后那个大学生又去问了一个研究生，研究生就写了一个Python脚本来算，最后发现确实满足。具体怎么算的我不知道，我同学给我发了一张研究生的Pycharm运行结果的照片，然后给我复制了一遍研究生的解答回复（看不懂）
![python-solution.webp](https://img.mukunjin.com/python-solution.webp)
欸我就在想，反正这个题人力也算不出来了，不如就用电脑做着玩玩吧，然后就开始思考怎么解决。
>我怀疑研究生是用AI写的脚本，毕竟写这样的一个小玩意儿，谁还会写注释啊？（照片里面能看到）。

## 具体实现
首先这个平方和求和很简单，直接可以算出来这个数列的和等于658875，那么，要想满足最后结果的最小值为1的话，就一定要是所有正数的和为329438，所以现在就是要凑出这个数。
然后其实我当时第一反应就是想到去试一下贪心，无限的逼近这个数，然后最后能不能把它凑出来。
>这里我必须说一点，我不是数竞或信竞生，只是恰巧略懂一点皮毛。

## 代码
其实挺简单的，查查文档这些十分钟就写好了
![my-cpp-solution.webp.png](https://img.mukunjin.com/my-cpp-solution.webp.webp)

源码：

```cpp
#include <iostream>
#include <vector>
using namespace std;

int main() {
    const int N = 125;
    vector<long long> squares(N);
    
    for (int i = 0; i < N; i++) {
        squares[i] = (long long)(i+1) * (i+1);
    }
    
    long long S = 0;
    for (long long x : squares) S += x;
    cout << "S_125 = " << S << endl;
    
    long long T = (S + 1) / 2;
    cout << "T = " << T << endl;
    
    long long remaining = T;
    vector<long long> chosen;
    
    for (int i = N-1; i >= 0; i--) {
        if (squares[i] <= remaining) {
            chosen.push_back(squares[i]);
            remaining -= squares[i];
        }
    }
    
    if (remaining == 0) {
        cout << "\nOK:" << T << " :" << endl;
        for (long long x : chosen) cout << x << " ";
        cout << endl;
    } else {
        cout << "\nNO" << remaining << ",LOL" << endl;
    }
    
    return 0;
}
```

静待编译完成...
![calculated-results.webp.png](https://img.mukunjin.com/calculated-results.webp.webp)

我自己也没想到最后居然真的凑出来了！
Yeeeeeeeeees!

## 写在最后
然后我就录了个屏，发给那个同学嘲讽他做不出来（哈哈我们关系不错，平时就是喜欢相互开玩笑，各位不要误解了），然后今天上午去学校上自习，他看到我的第一句话就是：**“装逼遭雷劈。”**

*上述内容难免有误，还请指正。*