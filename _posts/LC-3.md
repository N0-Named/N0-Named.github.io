---
layout: post
title: "About LC-3"
date: 2020-02-28   
tags: [reversing,memory,hacking,system,register]
comments: false
---

LC-3(Little Computer 3)에 대해서 알아봅시다.



\------



\# LC-3(Little Computer 3)에 대해서 알아봅시다.



\------



\# LC-3란?



\------



\> LC-3는 일종의 교육용 프로그래밍 언어로 low-level 프로그래밍 언어입니다.



\> x86보단 덜 복잡하지만 어셈블리 프로그램을 작성하는 데 사용할 수 있습니다.



\> Yale N. Patt과 Sanjay J. Patel에 의해 개발되었습니다.



\------



\# LC-3 시뮬레이터 & 에디터



\------



http://highered.mheducation.com/sites/0072467509/student_view0/lc-3_simulator.html



위 링크를 통해 LC-3 Simulator & Eidtor를 다운받을 수 있습니다.



\------



\# 목차



\------



\1. Memory and Registers



\2. Memory Map



\3. Opcodes

 

\4. 참고 문헌





\------



\# 1. Memory and Registers



\------



\### Memory



\- address space : 2 ** 16 (65536)

\- Addressability : 16 bits



\### Register



\- 8 general purpose register : R0 ~ R7

\- 1 program counter : PC

\- 1 condition flag register : COND



\------



\# 2. Memory Map



\------



LC-3의 메모리 구조는 아래와 같습니다.



![MemoryMap](/_posts/img/MemoryMap.jpg)



\------



\# 3. Opcodes



\------



LC-3의 16가지 명령어로 구성되어 있습니다.



\- BR (Branch)

\- ADD (Add)

\- LD (Load)

\- ST (Store)

\- JSR (Jump Register)

\- AND (Bitwise And)

\- LDR (Load Register)

\- STR (Store Register)

\- RTI 

\- NOT (Bitwise Not)

\- LDI (Load Indirect)

\- STI (Store Indirect)

\- JMP (Jump)

\- RES (Reserved)

\- LEA (Load Effective Address)

\- TRAP (Execute Trap)



LC-3 Instruction Set Format입니다.



![InstructionSet.jpg](/_posts/img/InstructionSet.jpg)



이번 포스트에서는 7가지 명령어에 대해서  다뤄볼 예정입니다.



\------



\## ADD / AND 



\------



![ADDAND.jpg](/_posts/img/ADDAND.jpg)



ADD, AND 명령어는 5번째 bit가 0일 경우 3개의 레지스터를 사용하여 연산을 진행합니다.



\- DR = SR1 + SR2

\- DR = SR1 & SR2



ADD, AND 명령어는 5번째 bit가 1일 경우 2개의 레지스터와 1개의 imm를 사용하여 연산을 진행합니다.



\- DR = SR1 + imm5

\- DR = SR1 & imm5



\------



\## NOT



\------



![NOT.jpg](/_posts/img/NOT.jpg)



NOT 명령어는 2개의 레지스터를 사용하여 연산을 진행합니다.



\- DR = ~SR



\------



\## LDR / STR



\------



![LDRSTR.jpg](/_posts/img/LDRSTR.jpg)



LDR, STR 명령어는 2개의 레지스터와 1개의 offset을 사용하여 메모리에 접근합니다.



\- DR = memory[BaseR + offset6]

\- memory[BaseR + offset6] = SR



\------



\## JMP



\------



![JMP.jpg](/_posts/img/JMP.jpg)



JMP 명령어는 1개의 레지스터를 사용하여 PC 레지스터의 값을 수정합니다.



\- PC = BaseR



\------



\## BR



\------



![BR.jpg](/_posts/img/BR.jpg)



condition flag register 상태에 따라 PC 레지스터의 값을 수정합니다.



\- if(conditionRegister[n] == n &&  conditionRegister[z] == z && conditionRegister[p] == p)

\-   PC += PCoffset9



\------



\# 참고 문헌



\------



\> https://en.wikipedia.org/wiki/Little_Computer_3



\> https://slidesplayer.org/slide/15146969/



\> https://justinmeiners.github.io/lc3-vm/#1:3



\> https://www.cs.colostate.edu/~cs270/.Fall18/resources/PattPatelAppA.pdf