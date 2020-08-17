1.飞行棋，输入K(距离)，N（掷骰子的次数），问最后飞行棋到终点的距离是多少？，需要返回多少次？（因为飞行棋如果超出终点的话会返回的）如果为0的话输出“paradox”。

输入：

```
10 2
6 3
```

输出：

```
1 0
```

```cpp
#include <iostream>
using namepspace std;
int main(){
    int K, N, temp;
    cin >> K >> N;
    int count = 0;
    for(int i = 0; i < N; ++i){
        cin >> temp;
        if(temp == K){
            cout << "paradox";
            return 0;
        }else if(temp > K){
            K = temp - K;
            ++count;
        }else{
            K = K - temp;
        }
    }
    cout << K << " " << count;
    return 0;
}
```

