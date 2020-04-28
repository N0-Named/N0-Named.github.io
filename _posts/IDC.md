---
layout: post
title: "About IDC"
date: 2020-02-28    
tags: [reversing,ida,idc]
comments: false
---
IDC에 대해서 알아봅시다.

---

# IDC에 대해서 알아봅시다.

---

# IDC란?

---

사용자의 모든 요구를 만족시키는 프로그램은 없다고 생각합니다.

모든 잠재적인 요구는 들어줄 수는 없으니까요.

개발자들은 끊임 없이 새로운 기능에 대한 요구, 문제 해결에 대한 요구를 
받지만 그것을 모두 수용할 수는 없습니다.

IDA에서는 사용자가 직접 프로그램적으로 문제를 해결할 수 있는 
스크립트 기능을 제공합니다.

IDA는 두 가지 스크립트를 지원하는데 IDC, IDAPython 스크립트입니다.

### 오늘은 IDC에 대해서 기초적인 부분을 살펴보겠습니다.

---

# 목차

---

1. 기본적인 스크립트 실행
2. IDC 변수
3. IDC 연산자
4. IDC 문자열
5. IDC 복합문
6. IDC 사용자 정의 함수
7. IDC 이외의 문법
8. IDC 에러 관련 ...
9. IDC 유용한 함수

---

# 기본적인 스크립트 실행

---

스크립트는 3가지 방법을 이용하여 실행할 수 있습니다.

- File 탭에서 Script File 버튼을 클릭
- File 탭에서 Command Script 버튼을 클릭
- OutPut 윈도우에서 직접 스크립트 입력

---

# IDC 변수

---

IDC의 문법은 C의 문법과 유사합니다.

IDC는 느슨한 타입의 언어로, 명시적인 타입이 없습니다.

일단 변수부터 살펴봅시다.

IDC에서는 두가지 유형의 변수가 존재합니다.

1. 전역 변수
2. 지역 변수

지역 변수를 선언할 때는 auto 키워드를 사용합니다.

선언과 동시에 초기화를 진행할 수 있습니다.

    auto a1,a2,a3;
    auto count = 0;
    a1 = 1;
    a2 = 2;
    a3 = 3;
    Message("%d %d %d %d\n",count,a1,a2,a3);

반면 전역 변수를 선언할 때는 extern 키워드를 사용해야 하고

함수 내부, 외부에서 선언해줄 수 있습니다.

그리고 변수를 선언할 때 초기화를 동시에 진행할 수 없습니다.

    extern outsideGlobal1;
    
    static main()
    {
    	extern insideGlobal1;
    	outsideGlobal1 = "Global";
    	insideGlobal1 = 1;
    	Message("outG1 : %s\n",outsideGlobal1);
    	Message("inG1 : %d\n", insideGlobal1);
    }

IDC에서 주석은 C와 비슷합니다.

위에서 봤을 때 눈치채셨겠지만 종료 문자도 ';'로 C와 비슷합니다.

    // han jul jusuk
    /* yara jul jusuk */

---

# IDC 연산자

---

IDC에서는 어떤 연산자를 지원하는지 실험해보겠습니다.

일단 대입연산자, 산술 연산자부터 살펴보도록 하겠습니다.

    auto a1 = 1;
    auto a2 = 2;
    auto res = 0;
    
    // +
    res = res + a1 + a2;
    
    Message("a1 + a2 = %d\n",res); 
    
    // -
    res = 0;
    res = res + (a1 - a2);
    
    Message("a1 + a2 = %d\n",res); 
    
    // *
    res = 0;
    res = res + (a1 * a2);
    
    Message("a1 + a2 = %d\n",res); 
    
    // /
    res = 0;
    res = res + (a1 / a2);
    
    Message("a1 + a2 = %d\n",res); 
    
    // %
    res = 0;
    res = res + (a1 / a2);
    
    Message("a1 + a2 = %d\n",res);

일단 기본적인 대입 연산자, 산술 연산자는 지원이 되는군요...

복합 대입 연산자도 써보겠습니다.

    auto a1 = 3;
    auto a2 = 5;
    auto res = 0;
    
    // +
    res += (a1 + a2);
    
    Message("%d\n",res); 
    
    // -
    res = 0;
    res -= (a1 + a2);
    
    Message("%d\n",res); 
    
    // *
    res = 1;
    res *= (a1 * a2);
    
    Message("%d\n",res); 
    
    // /
    res = 1;
    res /= (a1 / a2);
    
    Message("%d\n",res); 
    
    // %
    res = 132;
    res %= (a1 / a2);
    
    Message("%d\n",res);

음... 세미콜론 관련 에러 메세지가 뜨는데 세미콜론은 잘 사용했습니다.

IDA Pro Book을 살펴봤는데 복합 연산자는 지원하지 않는다고 하네요.

혹시 증감 연산자는 지원하지 않을까요?

    auto a = 5;
    auto b = 5;
    a++;
    b--;
    ++a;
    --b;
    Message("%d, %d\n", a,b);

휴우... 증감 연산자는 지원하네요.

관계 연산자는 당연히 지원하겠죠?

    auto res1 = 5 > 3;
    auto res2 = 3 < 5;
    auto res3 = 3 == 3;
    auto res4 = 5 != 3;
    auto res5 = 4 >=3;
    auto res6 = 3 <= 4;
    
    Message("%d %d %d %d %d %d\n",res1,res2,res3,res4,res5,res6);

역시 지원하는군요.

논리 연산자, 조건 연산자도 시험해보겠습니다.

    auto res1 = 1 && 0;
    auto res2 = 1 || 0;
    auto res3 = !0;
    
    auto res4 = 5 > 3 ? 5 : 3;
    auto res5  = 5 > 3 ? 3 : 5;
    
    Message("%d %d %d %d %d\n", res1, res2, res3, res4, res5);
    

넵 역시나 지원합니다.

비트 논리 연산자, 비트 이동 연산자도 한 번 사용해보겠습니다.

    auto res1 = 5 & 3;
    auto res2 = 5 | 3;
    auto res3 = 5 ^ 3;
    auto res4 = ~5;
    auto res5 = 1 << 1;
    auto res6 = 2 >> 1;
    
    Message("%d %d %d %d %d %d\n", res1, res2, res3, res4, res5, res6);

복합 대입 연산자 빼곤 기본적인 연산자는 다 지원하는 듯 싶습니다.

---

# IDC 문자열

---

C와 IDC의 차이는 여기서 확연하게 들어나는데 ,
문자열은 IDC의 기본 타입 중 하나인 관계로 
strcpy, strdup같은 문자열 복사, 복제 함수가 필요없습니다.

그리고 Python 처럼 슬라이스 연산을 지원합니다.

    auto str = "You should wash your hands";
    auto s1, s2, s3;
    
    s1 = str[0:3];
    s2 = str[4:10];
    s3 = str[21:];
    Message("%s %s %s\n",s1,s2,s3);

슬라이스 연산... 확실히 편안합니다.

---

# IDC 복합문

---

일단 if문부터 살펴보겠습니다.

    if(1)
    {
    	auto insideVal;
    	insideVal = 10;
    }
    else
    {
    	auto notExecute;
    	notExecute = 3;
    }
    
    Message("%d\n",insideVal);
    Message("%d\n", notExecute);

실행시켜보면 if문이 잘 동작합니다.

근데 특이한 점이 두 가지가 존재합니다.

- 블럭 내부에 선언된 변수를 블럭 외부에서 접근할 수 있다.
- 실행하지도 않는 블럭의 변수에도 접근이 가능하다.

switch문도 지원되는지 살펴볼까요?

    auto i = 5;
    
    switch(i)
    {
        case 5 :
            Message("Good\n");
            break;
        case 7 : 
            Message("Luck\n");
            break;
    }

앗... 실행이 안되는 것을 보니 switch는 지원하지 않는군요...

IDA Pro Book에서도 지원하지 않는다고 나와있습니다.

그래도 for문은 지원할 것 같습니다.

    auto i=0;
    for(i;i<10;i++)
    {
    	Message("%d\n",i);
    }

실행시켜보면 역시 잘 실행됩니다.

while문도 살펴보겠습니다.

    auto i = 0;
    while(i<10)
    {
    	Message("%d\n",i);
    	i++;
    }

요것도 잘됩니다.

do~while문도 될까요?

    auto i =0;
    do
    {
    	Message("%d\n",i);
    	i++;
    }while(i<10);

요것도 잘됩니다.

---

# IDC 사용자 정의 함수

---

IDC에서 사용자 정의 함수를 사용할 수 있을까요?

아래 목록과 같은 종류의 함수를 선언하고 호출해보겠습니다.

- 반환값, 받는 인자가 없는 함수
- 반환값은 없고 받는 인자는 있는 함수
- 반환값은 있고 받는 인자는 없는 함수
- 반환값도 있고 받는 인자도 있는 함수

    static func1()
    {
        Message("aaa\n");
    }
    
    static func2(func, arg)
    {
        func(arg);
    }
    
    static func3()
    {
        return "Hello";
    }
    
    static func4(arg)
    {
        return arg;
    }
    
    static main()
    {
        func1();
        func2(Message,"He\n");
        auto a1 = func3();
        Message("%s\n",a1);
        auto a2 = func4(5);
        Message("%d\n",a2);
    }

C와 마찬가지로 비슷하게 사용할 수 있습니다.

참조에 의한 호출도 지원하는지 궁금해지네요.

    static func1(arg)
    {
        arg = 5;
    }
    
    static main()
    {
       auto i = 3;
       func1(&i);
       Message("%d\n",i);
    }

오호... 5가 아니라 3이 출력되는 모습을 확인할 수 있습니다.

이렇게 되면 함수 선언만 봤을 때 함수 수행 후 결과 값을 명시적으로 
리턴하는지, 혹은 리턴하는 값의 타입이 무엇일지 알 수가 없겠네요.

---

# 이외의 문법은 ...

---

C에서 자주 사용하는 전처리문도 사용할 수 있고

try ~ catch 문, class 같은 C++에서 지원하는 기능도 사용할 수 있다고

IDA Pro Book에서 나와 있습니다.

---

# IDC 에러 관련 ...

---

스크립트를 수십 번 실행하면서 알아낸 것이 있다면

IDC의 에러 보고 기능은 그렇게 좋지 않습니다.

IDC에서의 에러는 파싱 에러, 런타임 에러 두 종류로 나뉩니다.

파싱 에러는 실행 전에 문제될 만한 것이 있는지 알려주고

런타임 에러는 실행하면서 발생한 에러를 알려줍니다.

제 경험을 일부 들려드리자면 ...

제가 IDC를 처음 접했을 때 30개 정도의 파싱 에러를 낸 적이 있었는데

파싱 단계에서 발생된 맨 처음 구문에 대해서만 에러를 알려줘서

30번 실행시키고 나서야 모든 에러를 잡을 수 있었습니다.

(에러 보고 기능이 별로 좋지 않다는 걸 체감한 순간이였습니다.)

특히 런타임 에러가 발생했을 때 출력 구문을 이용한 방법을 제외하면

IDC 스크립트를 디버깅할 방법이 없어서 불편함을 느낀 적이 있습니다.

---

# IDC 유용한 함수

---

IDC에서 유용하게 사용해볼만한 함수들을 정리해봤습니다.

## 입출력 함수

- void Message(string format, ...)
    - C의 printf 함수와 비슷하고 
    Output Window에 문자열을 출력해줍니다.
- void print(...)
    - 출력 창에 인자로 들어온 값을 출력해줍니다.
- string AskStr(string default, string prompt)
    - 사용자가 문자열을 입력하게 해줍니다.
    - 사용자가 입력한 문자열을 반환해줍니다.

## 문자열 처리 함수

- long atol(string val)
    - 문자열 형태의 10진수 값을 10진수로 변환해줍니다.
- long xtol(string val)
    - 문자열 형태의 16진수 값을 정수 값으로 변환해줍니다.
- long ord(string ch)
    - 단일 문자 ch와 매핑되는 ASCII 값을 리턴합니다.
- long strlen(string str)
    - str 문자열의 길이를 리턴해줍니다.

## 파일 입출력 함수

- long fopen(string filename, string mode)
    - 파일을 열어주는 함수입니다.
    - 인자값은 C의 fopen 함수와 비슷합니다.
- void fclose(long handle)
    - 파일을 닫습니다.
- long filelength(long handle)
    - 파일의 길이를 리턴합니다.
- long fgetc(long handle)
    - 파일에서 한 바이트를 읽습니다.
- long fputc(long val, long handle)
    - 파일에 한 바이트를 씁니다.
- long writestr(long handle, string str)
    - 파일에 특정 문자열을 씁니다.
- string or long readstr(long handle)
    - 파일에서 문자열을 읽습니다.
    - 모든 문자(non-ASCII 포함), 라인피드 문자까지 읽습니다.

## 데이터베이스 검색 함수

- long find_code(long addr, long flags)
    - 입력 주소에서 명령어 찾기
- long find_data(long addr, long flags)
    - 입력 주소에서 데이터 아이템 찾기
- long findBinary(long addr, long flags, string binary)
    - 지정된 주소에서부터 연속된 바이트를 대상으로 검색합니다.
    - 바이너리 문자열을 검색하려면 16진수 바이트로 기술합니다.

## 디스어셈블리 명령 관련 함수

- string GetDisasm(long addr)
    - 입력한 주소의 디스어셈블리 텍스트를 리턴합니다.
- String GetOpnd(long addr, long opnum)
    - 주어진 주소의 주어진 오퍼랜드를 표시하는 텍스트를 추출합니다.

---

# IDC에서 더 많은 정보를 얻고싶다면

---

간단하게 IDC 문법에 대해서만 알아봤습니다.

혹시 더 많은 정보를 얻고 싶으시다면 IDC Language, IDC 키워드로 
검색을 진행하면 생각보다 더 많은 정보를 찾을 수 있습니다.

다음에 더 재미있는 포스트를 들고 찾아오겠습니다.

---

# 참조

---

1. [https://www.hex-rays.com/products/ida/support/idadoc/158.shtml](https://www.hex-rays.com/products/ida/support/idadoc/158.shtml)
2. [https://www.hex-rays.com/products/ida/support/idadoc/159.shtml](https://www.hex-rays.com/products/ida/support/idadoc/159.shtml)
3. [https://www.hex-rays.com/products/ida/support/idadoc/161.shtml](https://www.hex-rays.com/products/ida/support/idadoc/161.shtml)
4. [https://www.hex-rays.com/products/ida/support/idadoc/160.shtml](https://www.hex-rays.com/products/ida/support/idadoc/160.shtml)
5. [https://www.hex-rays.com/products/ida/support/idadoc/162.shtml](https://www.hex-rays.com/products/ida/support/idadoc/162.shtml)
6. IDA Pro Book
