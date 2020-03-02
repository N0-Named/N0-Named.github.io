---
layout: post
title: "How to Calculate Sum of count for divisors?"
date: 2020-01-30    
tags: [algorithm,programming,number theory,math,code]
comments: false
---


이 글에서는 NYPC 2019 예선 문제 **약수**를 기반으로 어떻게 약수 개수의 합을 효율적으로 구할 수 있는가를 알아본다.

우리의 주요 관심사는

### *$$a$$, $$b$$가 1 이상이고 $$10^{12}$$ 이하 일때, $$a$$ 이상 $$b$$ 이하의 약수 개수의 합은?* 

가 우리가 해결해야 할 문제이다. 먼저 첫번째 방법으로는 1부터 자기 자신의 숫자를 모두 나눠보며 확인하는 방법이 기본이겠다.

```c++
#include <iostream>
using namespace std;
typedef long long ll;

int main() {
    ll a, b;
    ll counting = 0;
    ll result = 0;
    cin >> a >> b;
    for(int i = a; i <= b; i++) {
        counting = 0;
        for(int j = 1; j <= i; j++) {
            if(i % j == 0) {
                counting++;
            }
        }
        result += counting;
    }
    cout << result << "\n";
}
```

하지만 이러면, 매우 비효율적이고 속도가 느리다.. 이제 어떻게 하면 더 효율적으로 구할 수 있을까?

여기서 기초 정수론 지식을 하나 꺼내보자.

### $$a$$ 이하의 $$n$$의 배수의 개수는 $$[{a\over n}]$$ 개 이다.

왜 갑자기 뜬금없이 배수가 나오는가? 의문이 나올 수 있다. (위 식이 직관적으로 다가오지 않는다면, 직접 나열하면서 살펴보라,)

만약 10의 약수의 개수라고 하면, 반대로 말하자면 어떠한 수의 배수가 10이 되는가? 를 묻는 것과 같다.

10은 약수가 1,2,5,10 이고, 10은 1의 배수, 2의 배수, 5의 배수, 10의 배수로 4번 등장하므로, 

약수의 개수는 배수로 등장하는 횟수를 세는 것과 같다. 즉, 어떠한 배수의 개수를 구하는 것이 더 쉽기 때문에 

약수의 개수를 구하는 것과 달리 더 빨리 구할 수 있다. 이러한 아이디어를 **더블 카운팅(Double Counting)** 이라 한다.

**즉, 각 자연수의 약수 개수를 세는 것은 그 수가 어떤 수의 배수인지를 세는 것과 같다.** 그리고 이 문제를

**1 이상 b 이하의 약수 개수는?** 로 고쳐보자. 1이상 n 이하의 약수의 개수를 $$f(n)$$ 이라 하면,

a이상 b이하의 약수의 개수는 $$f(b) - f(a-1)$$ 로 나타낼 수 있다. 그리고 위 문제로 바뀌면서 

### *$$f(b)=\sum_{k=1}^{b} [{b\over k}]$$*

로 나타낼 수 있다.

즉, 본 우리가 구하고 싶은 문제에 위 이론을 적용하면, 시간 복잡도가 $O(b)$  으로 확 줄어든다! 

전 코드와 비교했을때 매우 효율적으로 변했다는것을 알 수 있다. 하지만 이 코드 조차도, $10^{12}$ 의 범위를 체크하기엔, 무리가 있다

(실제 대회 문제에선, 시간 제한이 1초 였다!) 그래서 우리는, 어떻게 더 효율적으로 구할 수 있는가를 알아보아야 한다.

1부터 10의 약수의 개수의 합을 구하는 것을 예로 들어보자.

$$[{10\over1}]+[{10\over2}]+[{10\over3}]+[{10\over4}]+[{10\over5}]+[{10\over6}]+[{10\over7}]+[{10\over8}]+[{10\over9}]+[{10\over10}]=10+5+3+2+2+1+1+1+1+1$$ 

로 나열 할 수 있다. 여기서 뒤에 1이 신경 쓰이지 않는가? 자세히 살펴보면, $$[{10\over5}]$$ 다음은 계속 1로 더해진다.

즉, $$k$$가 $$[{b \over 2}]$$보다 크다면, 1을 더해주기만 하면 된다! 즉, $$k = [{b\over2}]+1$$ 부터 $$b$$ 까진 탐색 할 필요가 없다. 1을 한번에 더해주면 되니까,

이제 탐색할 구간이 반으로 줄어들었다. 하지만 $$10^{12}$$ 를 커버하기엔 부족하다.. 위와 같은 논리로 접근하면,

$$k = [{b\over 3}] + 1$$ 부터 $$[{b\over2}]$$까진 2의 개수만큼 2를 더해주면 된다. 보면 점점 더하는 범위가 줄어들고 있다. 이제 중요한건

**어디까지 합을 구해야 하는가?** 이다. 생각해보면. 이 방법은 같은 값을 가지는 항이 2개 이상 있을때 유용하다.

하나만 있으면 그냥 더하면 되니까. 그래서, $$[{b\over k}]=x$$ 인 k의 범위를 한번 생각해 보자,

### $$[{b\over k}] = x \Rightarrow x \le {b\over k} < x+1 \Rightarrow {b\over {x+1}} < k \le {b\over x}$$

### $$[{b\over{x+1}}] + 1 \le k \le [{b\over x}]$$

### $$k의\ 개수 : [{b\over x}] - [{b\over{x+1}}] \Rightarrow 합은 \ k([{b\over x}] - [{b\over{x+1}}])$$

이제 k의 범위를 정하는 것만 남았다. 대략적으로 관찰해보면, $$\sqrt{b}$$ 근방 까지만 더해보면 된다는 것을 직관적으로 알 수 있을것이다. 대충 $$\sqrt{b}$$ 를 넘어가면, k와 x의 대소관계가 바뀌기 때문에 의미가 없어지기 때문이다. (수학적인 증명은 잘 모르겠다.) 즉 위 식을 정리하면,

### **$$f(b) = \sum_{k=1}^{\sqrt(b)}{[{b\over k}]} + k({[{b\over k}]}-{[{b\over {k+1}}}])$$**

로 정리 가능하고, 이를 그대로 코드로 옮기면 된다. 조심해야할 점은, k = [b/k]인 경우에는 제외 시켜야한다. 중복 되기 때문이다.

```c++
#include <iostream>

using namespace std;
typedef long long ll;
ll f(ll b) {
    ll i, cnt = 0;
    for(i = 1; i*i <= b; i++) {
        cnt += b/i + i*(b/i - b/(i+1));
        if(i == b/i) cnt -= i;
    }
    return cnt;
}
int main() {
		ll a, b;
    cin >> a >> b;
    cout << f(b) - f(a-1) << '\n';
}
```

## Reference 

-------

http://blog.naver.com/PostView.nhn?blogId=sg7360&logNo=221621812298&parentCategoryNo=&categoryNo=77&viewDate=&isShowPopularPosts=false&from=postList

https://www.acmicpc.net/board/view/41491

https://cheet0se.tistory.com/6
