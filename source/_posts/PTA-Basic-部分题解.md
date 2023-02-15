---
title: PTA Basic 部分题解
mathjax: true
tags:
  - PTA
  - 上机
  - 算法
categories:
  - [刷题, PTA]
abbrlink: a74ee18
date: 2022-01-01 15:31:48
---
<!-- toc -->
<!-- more -->
# 1025. 反转链表

[题目链接](https://pintia.cn/problem-sets/994805260223102976/problems/994805296180871168)
![1025 反转链表](/assets/1025%20反转链表.png)

这个题目第一眼看上去不知道怎么处理链表结点的地址，以及不知道如何用简便的方法将链表反转过来，如果用常规的设置链表的结构体来逆置链表，会比较麻烦。而且输入的结点并不是按照链表顺序给出的，并且地址是人为设定的，因此用结构体的方法会非常麻烦，因此考虑用数组来存储链表相关地址信息和顺序关系。

并且，翻转链表可以使用algorithm头文件中的reverse函数，会非常简便。

要注意输入的结点可能是一个完整的链表，他可以从中间断开，此时我们只要找到以头结点开始的那个链表就可以了。

完整代码参考了柳婼的代码：

```cpp
#include <iostream>
#include <algorithm>

using namespace std;

int main() {
    int first, n, k, temp;
    int data[100005], next[100005], list[100005];
    memset(data, 0, sizeof(data));
    memset(list, 0, sizeof(list));
    memset(next, 0, sizeof(next));
    cin >> first >> n >> k;
    for (int i = 0; i < n; ++i) {
        cin >> temp;
        cin >> data[temp];
        cin >> next[temp];
    }
    int sum = 0;  //除去断开的链表，获得完整的链表长度
    while (first != -1) {
        list[sum++] = first;
        first = next[first];
    }
    for (int i = 0; i + k <= sum; i += k) {
        reverse(begin(list) + i, begin(list) + i + k);
    }
    for (int i = 0; i < sum - 1; i++)
        printf("%05d %d %05d\n", list[i], data[list[i]], list[i + 1]);
        //注意这里是list[i + 1],一开始写成了next[list[i]]
    printf("%05d %d -1\n", list[sum - 1], data[list[sum - 1]]);
    return 0;
}
```

# 1035. 插入与归并
[题目链接](https://pintia.cn/problem-sets/994805260223102976/problems/994805286714327040)
![插入与归并](a74ee18/1035%20插入与归并.png)

分析：先将 `i` 指向中间序列列中满⾜足从左到右是从⼩小到⼤大顺序的最后⼀一个下标，再将 `j` 指向从 `i+1` 开始，第一个不不满⾜足 `a[j] == b[j]` 的下标，如果 `j` 顺利利到达了了下标`n`，说明是插⼊入排序，再下⼀一次的序列列是 `sort(a, a+i+2);`否则说明是归并排序。归并排序就别考虑中间序列列了了，直接对原来的序列列进⾏行行模拟归并时候的归并过程，`i` 从`0` 到 `n/k`，每次一段段得 `sort(a + i * k, a + (i + 1) * k);` 最后别忘记还有最后剩余部分的 `sort(a + n / k * k, a + n);` 这样是一次归并的过程。直到有一次发现`a`的顺序和`b`的顺序相同，则再归并一次，然后退出循环

**AC代码**
```cpp
#include <iostream>
#include <cstring>
#include <algorithm>

using namespace std;

int main() {
    int a[105], b[105], n;
    cin >> n;
    memset(a, 0, sizeof(a));
    memset(b, 0, sizeof(b));
    int i = 0, j = 0;
    for (j = 0; j < n; ++j) {
        cin >> a[j];
    }
    for (j = 0; j < n; ++j) {
        cin >> b[j];
    }
    for (i = 0; i < n - 1 && b[i] <= b[i + 1]; i++);
    for (j = i + 1; j < n && a[j] == b[j]; j++);
    int pow = 2, flag = 0;  //标记当前趟是否为目标趟
    if (j == n) {
        cout << "Insertion Sort" << endl;
        sort(a, a + i + 2);
    } else {
        cout << "Merge Sort" << endl;
        while (!flag) {
            flag = 1;
            //归并
            for (i = 0; i < n / pow; i++) {
                sort(a + i * pow, a + (i + 1) * pow);
            }
            sort(a + (n / pow) * pow, a + n);
            //比较当前趟是否为目标趟
            for (i = 0; i < n; i++) {
                if (a[i] != b[i]) {
                    flag = 0;
                    break;
                }
            }
            pow *= 2;
        }
        for (i = 0; i < n / pow; i++) {
            sort(a + i * pow, a + (i + 1) * pow);
        }
        sort(a + (n / pow) * pow, a + n);
    }
    cout << a[0];
    for (i = 1; i < n; i++) {
        cout << " " << a[i];
    }
    return 0;
}
```

# 1040. 有几个PAT（25) [逻辑题]

[题目链接](https://pintia.cn/problem-sets/994805260223102976/problems/994805282389999616)
![有几个PAT](a74ee18/1040%20有几个PAT.png)

这是一道思维题，要想知道构成多少个PAT，那么遍历字符串串后对于每一A，它前⾯面的P的个数和它后⾯面的T的个数的乘积就是能构成的PAT的个数。然后把对于每一个A的结果相加即可。只需要先遍历字符串串数一数有多少个T，然后每遇到一个T，便countT–-;每遇到一个P呢， countP++;然后一遇到字⺟母A呢就countT * countP，把这个结果累加到result中～～最后输出结果就好啦~~对了了别忘记要对10000000007取余哦～～

穷举遍历显然会超时～～

```cpp
#include <iostream>

#define M 1000000007

using namespace std;

int main() {
    string str;
    cin >> str;
    int countP = 0, countA = 0, countT = 0;
    int res = 0;
    for (char c : str) {
        if (c == 'T') countT++;
    }
    for (char c : str) {
        if (c == 'T') countT--;
        if (c == 'P') countP++;
        if (c == 'A') {
            res += countP * countT % M;
            res %= M;
        }
    }
    cout << res % M << endl;
    return 0;
}
//033510010
```

# 1045. 快速排序（25）
[题目链接](https://pintia.cn/problem-sets/994805260223102976/problems/994805278589960192)
![快速排序](a74ee18/1045%20快速排序.png)

分析：先看数据规模，1e5，如果用穷举，平方之后就是1e10，肯定超时，于是要想其他方法。
对原序列sort排序，逐个比较，当当前元素没有变化并且它左边的所有值的最大值都比它小的时候就可以认为它一定是主元（证明正确性：因为无论如何当前这个数要满足左边都比他大右边都比他小，那它的排名【当前数在序列中处在第几个，也就是相对位置】一定不会变）

一开始测试点 2 段错误，后来才想到如果没有主元存在的话，v[0]非法访问，改正后发现格式错误，然后加了句换行才正确。

`AC`代码：
```cpp
#include <iostream>
#include <vector>

using namespace std;

int main() {
    int n, cnt = 0, max = 0;
    vector<int> v;
    int a[100005], b[100005];
    cin >> n;
    memset(a, 0, sizeof(a));
    memset(b, 0, sizeof(b));
    for (int i = 0; i < n; i++) {
        cin >> a[i];
    }
    memcpy(b, a, sizeof(a));
    sort(a, a + n);
    for (int i = 0; i < n; i++) {
        if (a[i] == b[i] && b[i] > max) {
            cnt++;
            v.push_back(b[i]);
        }
        if (b[i] > max) {
            max = b[i];
        }
    }
    cout << cnt << endl;
    if (v.size() != 0) {
        sort(v.begin(), v.end());
        for (int i = 0; i < v.size() - 1; i++) {
            cout << v[i] << " ";
        }
        cout << v[v.size() - 1] << endl;
    } else {
        cout << endl;
    }
    return 0;
}
```

# 1049 数列的片段和 (20)
[题目链接](https://pintia.cn/problem-sets/994805260223102976/problems/994805275792359424)
![数列的片段和](a74ee18/1049%20数列的片段和.png)

总结 `printf()` 和 `scanf()` 函数：如果输入的是 `float` 型数据,那么用 `%f` 来接收；如果输入的是 `double` 型数据，那么用 `%lf` 来接收；如果输入的是 `long double` 型数据，那么用 `%llf` 来接收。 
但后来这道题的测试点 2 数据进行了更精细地改动，大概是将 N 改为 10 的 5 次方，此时会用 double 类型数据进行大量的计算，而由于数组中的每个数都小于 1.0，因此很有可能出现 double 小数精度不够用的情况而导致精度丢失，这时候计算出来的结果有可能会有偏差了。

查阅资料后得出以下两种方法：
1. 对每个输入的数据都先乘1000（只能乘以1000，可能乘多了就会导致整数部分精度丢失了，可以把这个结果想象成 63 位的队列，队接近队头部分是整数，接近队尾部分是小数，那么小数位数多了就会从队尾“丢出”丢失了的精度，整数位数多了就会从队头“丢出”丢失了的精度），这样就降低了小数位数，最后再对累加的结果除以之前乘的数字即可。
2. 这个方法就更简单了，每当我们用 `int` 型发现范围不够的时候会怎么办？难道会先去把这个数除以 1000，再把最后的结果乘以之前除掉的数字吗？当然不会，一般都是把 `int` 类型改成 `long long` 类型（为啥不是 `long` 类型，而是 `long long` 类型呢，这就涉及到另一个需要记忆的内容了：因为 `long int、long、int` 这三者都是 4 字节的，一样长，想不到吧？）说了那么多没有用。。。正解就是把 `total` 的类型改成 `long double` 就行了，这时候 `total` 的小数精度范围更大，就能完美解决测试点 2 出现的精度丢失问题。
3. `double` 类型占8字节，用 `%lf` 或 `%f` 控制输出，`long double` 一般占 12 字节用 `%Le %Lf %Lg` 控制输出

```cpp
#include <iostream>
using namespace std;

int main() {
    double a[100005];
    int n;
    cin >> n;
    for (int i = 1; i <= n; i++) {
        cin >> a[i];
    }
    long long c[100005];
    memset(c, 0, sizeof(c));
    long long cnt = 1;
    for (int i = 1; i <= n; i++) {
        c[i] = (long long)i * (n - i + 1);
    }
    long double sum = 0.0f;
    for (int i = 1; i <= n; i++) {
        sum += a[i] * 1000 * c[i];
    }
    printf("%.2Lf", sum / 1000);
    return 0;
}
```

# 1095 解码PAT准考证 (25)

纯模拟题，细心即可
[注]：`ios::sync_with_stdio(0);`可以加速 `cin` 和 `cout` 的输入输出速度，还有 `cin.tie(0);`。

```cpp
#include <algorithm>
#include <cstring>
#include <iostream>
#include <unordered_map>
#include <vector>

using namespace std;

int N, M;

struct PAT {
    string NUM, room, date, stuNum;
    char level;
    int score;
    PAT(string _NUM, int _score) : NUM(_NUM), score(_score) {
        level = _NUM[0];
        room = _NUM.substr(1, 3);
        date = _NUM.substr(4, 6);
        stuNum = _NUM.substr(10, 3);
    }
};
vector<PAT> p;
int cmp(PAT a, PAT b) {
    if (a.score == b.score) return a.NUM < b.NUM;
    else return a.score > b.score;
}

void print_score(int cases, int type, char level) {
    printf("Case %d: %d %c\n", cases, type, level);
    int flag = 0;
    for (int i = 0; i < N; ++i) {
        if (p[i].level == level) {
            printf("%s %d\n", p[i].NUM.c_str(), p[i].score);
            flag = 1;
        }
    }
    if (!flag) printf("NA\n");
}

void print_room(int cases, int type, string room) {
    printf("Case %d: %d %s\n", cases, type, room.c_str());
    int cnt = 0, sum = 0;
    for (int i = 0; i < N; ++i) {
        if (p[i].room == room) {
            cnt++;
            sum += p[i].score;
        }
    }
    if (cnt == 0) printf("NA\n");
    else printf("%d %d\n", cnt, sum);
}

void print_date(int cases, int type, string date) {
    printf("Case %d: %d %s\n", cases, type, date.c_str());
    unordered_map<string, int> m;
    for (int i = 0; i < N; ++i) {
        if (p[i].date == date) {
            m[p[i].room]++;
        }
    }
    vector<pair<string, int>> room_order(m.begin(), m.end());
    sort(room_order.begin(), room_order.end(), [=](pair<string, int> p1, pair<string, int> p2) -> bool {
        if (p1.second == p2.second) return p1.first < p2.first;
        else return p1.second > p2.second;
    });
    if (room_order.size() == 0) printf("NA\n");
    else {
        for (int i = 0; i < room_order.size(); ++i) {
            printf("%s %d\n", room_order[i].first.c_str(), room_order[i].second);
        }
    }
}

int main() {
    ios::sync_with_stdio(0);
    freopen("in", "r", stdin);
    cin >> N >> M;
    string s;
    int score;
    for (int i = 0; i < N; ++i) {
        cin >> s >> score;
        p.push_back(PAT(s, score));
    }
    sort(p.begin(), p.end(), cmp);
    int type;
    char c;
    string room, date;
    for (int i = 0; i < M; ++i) {
        cin >> type;
        switch (type) {
            case 1:
                cin >> c;
                print_score(i + 1, type, c);
                break;
            case 2:
                cin >> room;
                print_room(i + 1, type, room);
                break;
            case 3:
                cin >> date;
                print_date(i + 1, type, date);
                break;
            default:
                break;
        }
    }
    return 0;
}
```

# 1099 性感素数 (20)
[题目链接](https://pintia.cn/problem-sets/994805260223102976/problems/1478633879405998080)

这题的主要问题是当给定的数不是 Sexy Prime 时，要找比他大的最小符合条件的数，可以与比他小 6 的组成一对，并不一定是比他大 6 的数组成一对。

```cpp
#include <iostream>
#include <cmath>

using namespace std;

bool isPrime(int n) {
    if (n <= 1) return false;
    for (int i = 2; i <= sqrt(n); i++) {
        if (n % i == 0) {
            return false;
        }
    }
    return true;
}

void Judge(int n) {
    if (isPrime(n)) {
        if (isPrime(n - 6)) {
            cout << "Yes" << endl;
            cout << n - 6 << endl;
            return;
        }
        if (isPrime(n + 6)) {
            cout << "Yes" << endl;
            cout << n + 6 << endl;
            return;
        }
    }
    cout << "No" << endl;
    for (int i = n + 1; i <= 1000000000; i++) {
        if (isPrime(i) && isPrime(i + 6) || isPrime(i) && isPrime(i - 6)) {
            cout << i << endl;
            return;
        }
    }
}

int main() {
    int n;
    cin >> n;
    Judge(n);
    return 0;
}
```


# 1100 校庆 (20)

这题问题还是超时，用 map 进行查找可以将数量级压缩在 O(n).

# 1104 天长地久 (20)

这题主要就是要注意超时问题，可以推出来末尾一定是 99（我感觉题目“天长地久”已经有暗示了），然后在末尾 99 的基础上每次加 100 就行了，这样循环数量级变成 1e7，就不会超时了。

