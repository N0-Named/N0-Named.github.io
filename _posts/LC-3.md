---
layout: post
title: "LC-3(Little Computer 3)"
date: 2020-02-28 
tags: [ISA, LC-3]
comments: false
---
LC-3(Little Computer 3)�� ���ؼ� �˾ƺ��ô�.

------

# LC-3(Little Computer 3)�� ���ؼ� �˾ƺ��ô�.

------

# LC-3��?

------

> LC-3�� ������ ������ ���α׷��� ���� low-level ���α׷��� ����Դϴ�.

> x86���� �� ���������� ����� ���α׷��� �ۼ��ϴ� �� ����� �� �ֽ��ϴ�.

> Yale N. Patt�� Sanjay J. Patel�� ���� ���ߵǾ����ϴ�.

------

# LC-3 �ùķ����� & ������

------

http://highered.mheducation.com/sites/0072467509/student_view0/lc-3_simulator.html

�� ��ũ�� ���� LC-3 Simulator & Eidtor�� �ٿ���� �� �ֽ��ϴ�.

------

# ����

------

1. Memory and Registers

2. Memory Map

3. Opcodes
 
4. ���� ����


------

# 1. Memory and Registers

------

### Memory

- address space : 2 ** 16 (65536)
- Addressability : 16 bits

### Register

- 8 general purpose register : R0 ~ R7
- 1 program counter : PC
- 1 condition flag register : COND

------

# 2. Memory Map

------

LC-3�� �޸� ������ �Ʒ��� �����ϴ�.

![MemoryMap](/_posts/img/MemoryMap.jpg)

------

# 3. Opcodes

------

LC-3�� 16���� ��ɾ�� �����Ǿ� �ֽ��ϴ�.

- BR (Branch)
- ADD (Add)
- LD (Load)
- ST (Store)
- JSR (Jump Register)
- AND (Bitwise And)
- LDR (Load Register)
- STR (Store Register)
- RTI 
- NOT (Bitwise Not)
- LDI (Load Indirect)
- STI (Store Indirect)
- JMP (Jump)
- RES (Reserved)
- LEA (Load Effective Address)
- TRAP (Execute Trap)

LC-3 Instruction Set Format�Դϴ�.

![InstructionSet.jpg](/_posts/img/InstructionSet.jpg)

�̹� ����Ʈ������ 7���� ��ɾ ���ؼ�  �ٷﺼ �����Դϴ�.

------

## ADD / AND 

------

![ADDAND.jpg](/_posts/img/ADDAND.jpg)

ADD, AND ��ɾ�� 5��° bit�� 0�� ��� 3���� �������͸� ����Ͽ� ������ �����մϴ�.

- DR = SR1 + SR2
- DR = SR1 & SR2

ADD, AND ��ɾ�� 5��° bit�� 1�� ��� 2���� �������Ϳ� 1���� imm�� ����Ͽ� ������ �����մϴ�.

- DR = SR1 + imm5
- DR = SR1 & imm5

------

## NOT

------

![NOT.jpg](/_posts/img/NOT.jpg)

NOT ��ɾ�� 2���� �������͸� ����Ͽ� ������ �����մϴ�.

- DR = ~SR

------

## LDR / STR

------

![LDRSTR.jpg](/_posts/img/LDRSTR.jpg)

LDR, STR ��ɾ�� 2���� �������Ϳ� 1���� offset�� ����Ͽ� �޸𸮿� �����մϴ�.

- DR = memory[BaseR + offset6]
- memory[BaseR + offset6] = SR

------

## JMP

------

![JMP.jpg](/_posts/img/JMP.jpg)

JMP ��ɾ�� 1���� �������͸� ����Ͽ� PC ���������� ���� �����մϴ�.

- PC = BaseR

------

## BR

------

![BR.jpg](/_posts/img/BR.jpg)

condition flag register ���¿� ���� PC ���������� ���� �����մϴ�.

- if(conditionRegister[n] == n &&  conditionRegister[z] == z && conditionRegister[p] == p)
-   PC += PCoffset9

------

# ���� ����

------

> https://en.wikipedia.org/wiki/Little_Computer_3

> https://slidesplayer.org/slide/15146969/

> https://justinmeiners.github.io/lc3-vm/#1:3

> https://www.cs.colostate.edu/~cs270/.Fall18/resources/PattPatelAppA.pdf