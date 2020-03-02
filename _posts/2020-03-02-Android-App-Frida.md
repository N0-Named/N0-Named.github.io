---
layout: post
title: "Android-App-Frida"
date: 2020-03-02 
tags: [reversing,pwnable,hacking,exploit,system,hook]
comments: false
---





Frida를 이용한 안드로이드 후킹을 Araboza


 
 
 
 
 
# Frida를 이용한 안드로이드 후킹


# 간단한 소개

 
> **frida**는 DBI(Dynamic Binary Instrumentation) 프레임워크중 하나로 강력한 스크립팅을 제공하고 무료로 사용할 수 있다.  frida는 아래 구조로 동작하며 js 기반의 여러 언어의 스크립팅을 제공해주며 특히 파이썬이 가장 많이 이용한다. 또한,  구조상 양방향 통신을 하면서 명령을 주고받으며 동작하고 꼭 이런구조가아닌 내부적으로 동작 가능하다.
 
 
![image-20200301222231888](/assets/img/img3/image-20200301222231888.png)



# 개요 

frida 자체의 사용법 보다  OWASP Uncrackable Crackme 문제들을 분석해보고 후킹을 통해 풀이를 할 것 입니다.



# 설치 



```bash
$ sudo apt-get install build-essential gcc-multilib lib32stdc++-5-dev \
    python-dev python3-dev git
$ curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
$ sudo apt-get install -y nodejs
$pip install frida-tools # CLI tools
$pip install frida       # Python bindings
$npm install frida       # Node.js bindings
```

![image-20200302022657509](/assets/img/img3/image-20200302022657509.png)

위 사진 처럼 wsl 사용하면 기묘하게 exe 실행 시킬 수 있습니다.

![image-20200302024749002](/assets/img/img3/image-20200302024749002.png)

![image-20200302042726634](/assets/img/img3/image-20200302042726634.png)

하지만 path 에러 때문에 파워셀을 씁니다. 

https://github.com/frida/frida/releases/download/12.8.13/frida-server-12.8.13-android-x86.xz

위 주소에서 다운 받은 다음 push 명령어로 올려주시고 권한 설정과 실행 시켜주었습니다.



예제 문제를 안드로이드에 올립니다.

https://github.com/OWASP/owasp-mstg/tree/master/Crackmes

![image-20200302044406522](/assets/img/img3/image-20200302044406522.png)



# UnCrackable-Level1

 위 문제 1번 문제입니다. 먼저 실행 시켜 보면 아래 사진 처럼 루팅 탐지가 되어 종료가 되었다고 뜨며 ok 버튼을 눌르면 종료 됩니다.

![image-20200302044536169](/assets/img/img3/image-20200302044536169.png)



본격적으로 GDA 툴을 사용하여 apk 분석 해보겠습니다.

![image-20200302044811093](/assets/img/img3/image-20200302044811093.png)

![image-20200302044821517](/assets/img/img3/image-20200302044821517.png)

메인 onCreate()에서 아까 발견 한 문자열이 보입니다. 또한, 디버깅 탐지하는 루틴이 같이 확인 할 수 있고 

두 루틴 모두 MainActivity.a()에 메세지를 전달하는 형식으로 되어 있습니다. 

![image-20200302045729213](/assets/img/img3/image-20200302045729213.png)

MainActivity.a()함수 내부입니다 dialog 함수로 문자열 출력이후 MainActivity$1()로 this 인자를 넘기네요.

![image-20200302045948926](/assets/img/img3/image-20200302045948926.png)

![image-20200302045958572](/assets/img/img3/image-20200302045958572.png)

MainActivity$1() 내부 루틴에 system.exit를 발견 할 수 있었습니다.

정리 해보자면 

MainActivity -> onCreate ->  MainActivity.a -> MainActivity$1 -> MainActivity$1.onClick

그래서 system.exit 함수를 frida를 통해 후킹하여 종료되지 않는 루틴으로 만들어 보겠습니다.

```python
import sys,frida

Hook_package = "owasp.mstg.uncrackable1"

def on_message(message,data):
    print("{} -> {}".format(message,data))

jscode = """
    Java.perform(function()
    {
        console.log("[+] Hooking[System.exit]");
        var exitClass=Java.use("java.lang.System");
        exitClass.exit.implementation=function()                                               
        {
            console.log("[+] System.exit Called");
        }
    });
"""

try:
    device = frida.get_usb_device(timeout=10)
    pid = device.spawn([Hook_package])
    print("[+] App is starting pid:{}".format(pid))                                 
    process = device.attach(pid)
    device.resume(pid)
    script = process.create_script(jscode)
    script.on('message',on_message)
    print('[+] Running Frida')
    script.load()
    sys.stdin.read()

except Exception as e:
    print(e)
```

 아래와 같이 정상적으로 후킹되어 dialog  메세지는 뜨지만 ok눌러도 종료되지 않습니다.

![image-20200302075939437](/assets/img/img3/image-20200302075939437.png)

![image-20200302080115690](/assets/img/img3/image-20200302080115690.png)

다음 루틴  위 메세지 박스의 스트링이 보입니다. p0을 입력했을때 a.a()함수에 들어갑니다.

![image-20200302080338158](/assets/img/img3/image-20200302080338158.png)

특정한 암호문과 특정 함수에 들어가 복호화 된 값이랑 input 값을 비교합니다.

![image-20200302083718512](/assets/img/img3/image-20200302083718512.png)

위 루틴을 통해  아래 sg.vantagepoint.a.a.a 함수를 후킹하여 input과 비교할 password 값을 후킹을 통해 출력 하였습니다. 

![image-20200302083951038](/assets/img/img3/image-20200302083951038.png)



```python
import sys,frida
 
Hook_package = "owasp.mstg.uncrackable1"
 
def on_message(message,data):
    print("{} -> {}".format(message,data))
 
jscode = """
    Java.perform(function()
    {
        console.log("[+] Hooking[System.exit]");
        var exitClass=Java.use("java.lang.System");
        exitClass.exit.implementation=function()                                               
        {
            console.log("[+] System.exit Called");
        }
        console.log("[+] System.exit modified");

		var aa = Java.use("sg.vantagepoint.a.a");
        aa.a.implementation = function(arg1,arg2)
        {
			console.log("[+] Hooking[a.a.a]");
			var retval = this.a(arg1,arg2);
			var Password="";
			for(var i=0; i<retval.length; i++)
			{
				Password += String.fromCharCode(retval[i]);
			}                                                                       
			console.log("[+] Password: "+Password);                                       
			return retval;
        }

        console.log("[+] sg.vantagepoint.a.a.a modified");

    });
"""
 
try:
    device = frida.get_usb_device(timeout=10)
    pid = device.spawn([Hook_package])
    print("[+] App is starting pid:{}".format(pid))                                 
    process = device.attach(pid)
    device.resume(pid)
    script = process.create_script(jscode)
    script.on('message',on_message)
    print('[+] Running Frida')
    script.load()
    sys.stdin.read()
 
except Exception as e:
    print(e)
```



아래 처럼 결과를 확인 할 수 있습니다.

![image-20200302083558748](/assets/img/img3/image-20200302083558748.png)

메세지 확인

![image-20200302084220174](/assets/img/img3/image-20200302084220174.png)





# 참고 

https://www.codemetrix.io/hacking-android-apps-with-frida-1/

https://github.com/OWASP/owasp-mstg

https://www.frida.re

