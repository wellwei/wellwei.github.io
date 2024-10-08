### 前言

有些同学可能对于多组测试样例不熟悉，这里解释一下

题目中说了有t组测试样例的只需要先输入一个t，然后循环t次解题过程即可，例如：

```cpp
void solve() {	// 该函数为主要解题处
    // 你的解题代码
}
int main() {
    int t;
    scanf("%d", &t);	// T 组测试
    while (t--) solve();
    return 0;
}
```

不存在t就代表程序要求你读入数据直到数据文件尾，这里要自行了解scanf函数的返回值以及EOF常量，例：

```cpp
int main() {
    int n;
    while (scanf("%d", &n) != EOF) {	如果==EOF就代表已经读入完了
        // 你的解题代码
    }
    return 0;
}
```

还存在其他的多组测试样例，但是输入格式中都有描述可自行理解



另外观察到部分同学可能在学习C语言过程中会对一些输入输出操作添加提示信息，例如

```cpp
void solve() {
    int a, b;
    printf("请输入两个整数\n");	// 语句1
    scanf("%d%d", &a ,&b);
    printf("两个整数分别是：%d %d", a, b);	//语句2
}
```

上例中语句1以及语句2中的提示语部分在算法题目中都是不背允许的，每个题目都严格要求了输入输出的格式，题目要你输出什么就输出什么，其他的内容都不要输出，否则Wrong Answer





### A

这个题目只需要用一个if语句判断 "a + b == c" 或者 "a - b == c" 这两个条件是否成立即可

```cpp
void solve() {
	int a, b, c;
    scanf("%d%d%d", &a, &b, &c);
	if (a - b == c) cout << "-" << endl;
	else cout << "+" << endl;
}

int main() {
	int t ;
    scanf("%d", &t);
	while(t--) solve();
	return 0;
}
```



### B

这题只需要用if语句判断多个逻辑表达式是否成立即可

```cpp
void solve() {
	int a, b, c;
    scanf("%d%d%d", &a, &b, &c);
	if (a + b == c || a + c == b || b + c == a) 
        printf("YES");
	else printf("NO");
}

int main() {
	int t; 
    scanf("%d", &t);
	while(t--) solve();
	return 0;
}
```





### C

跟上面两题一样只需要if判断条件是否成立

```cpp
void solve() {
	int a, b, c;
    scanf("%d%d%d", &a, &b, &c);
	if (a < b && b < c) printf("STAIR\n");
    else if (a < b && b > c) printf("PEAK\n");
    else printf("NONE\n");
}

int main() {
	int t;
    scanf("%d", &t);
	while(t--) solve();
	return 0;
}
```





### D

只需要判断哪个数字没有出现即可

```cpp
void solve() {
	int a, b;
    scanf("%d%d", &a, &b);
    if (a != 1 && b != 1) printf("1\n");
    else if (a != 2 && b != 2) printf("2\n");
    else printf("3\n");
}

int main() {
	int t = 1;
	while(t--) solve();
	return 0;
}
```



### E

这个题目需要推一下公式，通过观察对称，可以发现 $ a[i][j] $ 与 $a[2 - i][2 - j]$  中心对称  (下标0-2)

```cpp
void solve() {
    char a[3][3];
    scanf("%s%s%s", a[0], a[1], a[2]);
    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 3; j++) {
            if (a[i][j] != a[2 - i][2 - j]) {
                printf("NO\n");
                return;
            }
        }
    }
    printf("YES\n");
}
```





### F

观察到要牌数最小那么就只能用4，那么只需要对 $n/4$  向上取整即可

```cpp
#include <math.h>

void solve() {
    int n;
    scanf("%d", &n);
    printf("%d\n", ceil(n / 4.0));		// ceil函数向上取整，可自行百度
    									// 也可以自己编码实现该功能
}

int main() {
	int t;
    scanf("%d", &t);
	while(t--) solve();
	return 0;
}
```





### G

该题涉及排序操作，根据id和总成绩进行双关键字排序然后循环查找id为1的输出即可（C语言的qsort、C++的sort都可轻松实现）

```cpp
struct sc {
    int id;
    int sum;
} arr[1010];

void selectionSort(int n) {     // 执行选择排序
    for (int i = 0; i < n - 1; i++) {
        int maxIdx = i;
        for (int j = i + 1; j < n; j++) {
            // 按总成绩降序排列，如果总成绩相同则按id升序排列
            if (arr[j].sum > arr[maxIdx].sum || 
                (arr[j].sum == arr[maxIdx].sum && arr[j].id < arr[maxIdx].id)) {
                maxIdx = j;
            }
        }
        // 交换当前元素和找到的最大元素
        if (maxIdx != i) {
            sc temp = arr[i];
            arr[i] = arr[maxIdx];
            arr[maxIdx] = temp;
        }
    }
}

void solve() {
    int n;
    scanf("%d", &n);
    int a, b, c, d;
    for (int i = 0; i < n; i++) {
        scanf("%d%d%d%d", &a, &b, &c, &d);
        arr[i].id = i;
        arr[i].sum = a + b + c + d;
    }
    selectionSort(n);
    for (int i = 0; i < n; i++) {
        if (arr[i].id == 0) {
            printf("%d\n", i + 1);
            break;
        }
    }
}
```





### H

关键的观察是 $i^i$ 的奇偶性与 $i$ 相同。因此，问题归结为找出以 $n$ 结尾的 $k$ 个连续整数的和是奇数还是偶数。这可以通过找出 $n-k+1, n-k+2, \ldots, n-1, n$ 的和来完成，即$S_{1 \sim n} - S_{n - k \sim n}$，使用等差公式$n \times (n + 1) / 2$，然后检查它的奇偶性。或者，可以计算那些 $k$ 个连续整数中奇数的数量。

答案运算过程中，可能超过int的数据范围，所以要用long long

```cpp
void solve() {
    long long n, k;
    scanf("%lld%lld", &n, &k);
    long long res = n * (n + 1) / 2 - (n - k) * (n - k + 1) / 2; 
    // 1~n的和减去1~(n-k)的和就是 (n-k+1)~n的和
    if (res % 2) printf("NO\n");
    else printf("YES\n");
}
```



### I

解决问题的时间取决于x和y中的最小值

```cpp
void solve() {
    int n, x, y;
    scanf("%d%d%d", &n, &x, &y);
    printf("%d\n", (int)ceil(1.0 * n / min(x, y)));		// 要向上取整
}
```



### J

通过观察可以得知如果两个区间不相交的话那么只需要关一扇门，否则要将相交区的每个门都关上（注意判断边界没门的情况）

```cpp
void solve() {
    int l, r, L, R;
    scanf("%d%d%d%d", &l, &r, &L, &R);
    int ll = max(l, L), rr = min(r, R);     // 相交区间的左右边界
    int ql = min(l, L), qr = max(r, R);     // 整个区间的左右边界
    if (ll > rr) printf("1\n");     // 如果两个区间不相交那么只需关一扇门
    else {                          // 否则要将相交区的每个门关上
        int ans = rr - ll;      // 相交房间之间的门
        if (ll > ql) ++ans;     
        if (rr < qr) ++ ans;    // 相交区最外围的两个门（如果存在）
        printf("%d\n", ans);
    }
}
```





### K

先分别求出正x方向和正y方向各要走多少步cx、cy。因为先走x方向，如果cy >= cx 代表y方向要多走，又因为是交替方向前进，所以x方向需要停留等待y，即最后一步要走y方向，答案为 2 * cy 。如果cx > cy 那么最后一步走x方向，所以答案为 2 * cx - 1

```cpp
void solve5()
{
    int x, y, k;
    scanf("%d%d%d", &x, &y, &k);
    int cx = ceil(1.0 * x / k), cy = ceil(1.0 * y / k);
    if (cy >= cx) printf("%d\n", 2 * cy);
    else printf("%d\n", 2 * cx - 1);
}
```





### L、M、N

后面的题以后再来探索吧.........