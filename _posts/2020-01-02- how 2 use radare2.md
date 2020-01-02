
<p>Radare2를 이용한 리버스 엔지니어링</p>
<p>&nbsp;</p>
<h1>간단한 소개</h1>
<hr />
<blockquote><p><strong>Radare2</strong>는 리버스 엔지니어링 툴(프레임워크)로 gdb와 같이 <strong>동적분석</strong>을 할 수 있다.  또한 우리에겐 매우 편리한 인터랙티브 쉘로 구성되어 있어서 리눅스에서 평소 사용하던 명령어들을 그대로 이용할 수 있다.  그리고 여러 강력한 기능을 제공하고 전부는 아니지만 최대한 많이 담아 볼 예정이다.</p>
</blockquote>
<p>&nbsp;</p>
<hr />
<h1>설치</h1>
<hr />
<pre><code>1. $  git clone https://github.com/radare/radare2.git
2. $  cd radare2/sys/
3. $  ./install.sh
</code></pre>
<h4>이상 설치가 완료되었으면 간단하게 Radare2에 있는 툴 11가지중 분석에 용이한 7가지를 간단한 예제와 함게 설명 후 ctf 문제를 분석을 진행할 것이다.</h4>
<p>(예제 파일은 18~19년도 코게 문제)</p>
<hr />
<h1>목차</h1>
<h3>1. instruction</h3>
<h3>2. rabin2</h3>
<h3>3. rax2</h3>
<h3>4. radiif2</h3>
<h3>5. rafind2</h3>
<h3>6. rahash2</h3>
<h3>7. rasm2</h3>
<h3>8. r2 + CTF 문제 분석</h3>
<p>&nbsp;</p>
<hr> 
<h1>instructon</h1>
<ol>
<li><h4>분석을 위한 간단한 명령어들</h4>
</li>

</ol>
<p>우선 <strong>r2 filename</strong>으로 radare2를 작동시켜준다. 이번 문서에서는 예시로 IORI의 Crackme문제를 이용하겠다. 여기서 실행을 할 때, 여러가지 옵션이 사용가능한데 -d옵션을 달아줄 경우, 디버깅 모드로 접속이 가능하다. 또한 -w 옵션으로 접속할 경우 바이너리를 수정하는것이 가능해진다. 상황에 맞게 옵션을 이용하자.</p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 10.09.23.png" referrerpolicy="no-referrer" alt="r2 filename"></p>
<p>본격적으로 분석을 하기에 앞서 <strong>rabin2 -I filename</strong> 명령어를 이용하여 해당 바이너리의 세부정보를 확인할 수 있다. 밑의 사진을 보면 canary, nx, relro 등 의 보호기법 여부와 프로그래밍 언어, os 도 확인을 할 수 있다. 문제풀이에 앞서 탐색전을 하기에 유용하다.</p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 7.43.24.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 7.43.24"></p>
<p>그 후 <strong>aa</strong>를 이용하여 본격적인 분석을 시작할 수 있다. aa는 Analyze All 의 약자로, 바이너리 전체를 돌아서 symbol 영역 등을 분석해주는 명령어이다. 이외에 <strong>aaa</strong>를 입력하면 <strong>aa</strong>를 포함한 <strong>aar, aac</strong> 등의 다양한 명령어들을 실행시켜준다. </p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 3.49.08.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 3.49.08"></p>
<p><strong>afl</strong>명령어를 사용하면 해당 바이너리 내의 함수 목록을 표현해준다. 밑의 이미지에서 함수의 이름이 임의로  sym.~~~로 표현되는데 이를 더 직관적으로 확인할 수 있도록 <strong>afn |name| |addr|</strong> 로 함수의 이름을 변경할 수 있다.</p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 7.53.12.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 7.53.12"></p>
<p>Radare2는 <strong>pdf</strong>(Print Disassemble Funtion) 명령어를 제공하므로, 해당 함수의 디스어셈블리를 보고싶다면 <strong>pdf @funcname</strong>을 입력하면 된다. 여기서 @는 명령이 실행되는 탐색 위치를 지정하는데 사용된다. 또한, <strong>pd</strong>라는 명령어를 사용하면 함수가 아닌 개체를 디스어셈블 해준다. 
<img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 3.49.14.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 3.49.14"></p>
<p>다음으로 설명할 명령어는 IDA처럼 그래프 형식으로 바이너리를 보여주는 <strong>VV</strong>명령어이다. 현재 분석중인것은 파란색으로 표시되고, 다음 노드로 넘어가기 위해서는 <strong>Tab</strong>을 누르면된다.  [<strong>g</strong>]나 방향키로 분석을 하다가 현재의 노드로 돌아갈때 에는<strong>.</strong>을 누르면 된다.  </p>
<p><strong>P</strong>키를 누르면 그래프의 형태가 변화하고, <strong>x</strong>를 누르면 현재 분석중인 노드의 위치를 알려준다. 그 후 <strong>q</strong>를 누르면 빠져나올 수 있다.</p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 8.02.23.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 8.02.23"></p>
<p>hex값을 확인할때는 &quot;<strong>x 표시할 라인수</strong>&quot;를 입력해주면 해당 라인만큼의 hex값을 출력해준다. </p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 3.50.20.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 3.50.20"></p>
<p>해당 바이너리를 더 깊게 분석하고 싶다면 seek명령어인 <strong>s</strong>를 입력후 옵션을 추가하여 이용하면 된다. s는 현재의 탐색 위치를 지정하는데 유용하게 사용할 수 있다. 밑의 이미지를 확인하면 원래는 현재의 탐색 주소가 0x08048430 이였는데 s main을 입력후 main함수의 주소인 0x080484e4로 변경되는것을 알 수 있다. 이는 afn명령어로 수정한 이름과 혼합하여 사용하면 분석을 할 때 더욱 편리하게 이용이 가능하다.</p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 8.00.17.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 8.00.17"></p>
<ol>
<li><h4>문자열 관련 명령어들</h4>
</li>

</ol>
<p><strong>axt</strong>명령어는 해당 함수의 참조위치를 확인할 수 있는 명령어이다. <strong>afl</strong>명령어를 이용하여 함수의 목록을 확인 후, 이를 이용하여 참조를 확인할 수 있다.</p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 8.08.10.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 8.08.10"></p>
<p><strong>izzq~[text]</strong>는 해당 텍스트의 위치를 바이너리 내에서 검색하는 명령어이다. 이 문제에서는 Password인증을 우회해야 하기 때문에 Password를 찾아봤다. </p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 8.09.45.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 8.09.45"></p>
<p>이번에는 flagspaces를 확인할 수 있는 <strong>fs</strong>명령어이다. </p>
<p><img src="N0-Named.github.io/_posts/스크린샷 2019-12-04 오후 8.07.21.png" referrerpolicy="no-referrer" alt="스크린샷 2019-12-04 오후 8.07.21"></p>
<ol>
<li><h4>바이너리 수정을 위한 명령어들</h4>
</li>

</ol>
<p>이번에는 r2 -w옵션을 이용하여 바이너리 수정 모드에서 이용되는 명령어들을 알아보자. 보통 이렇게 바이너리를 직접 수정하는 경우에는 원본파일을 복사해서 복사본으로 진행하는것이 좋다. 그럼 시작해보자 </p>
<p>우선 처음으로 해야할 것은, 위에서 설명한 s명령어를 이용하여 현재의 탐색주소를 선정해야한다.  그 후, <strong>wx</strong> 명령어를 이용하면 현재 탐색 위치의 값을 변경하고자 하는 hex 값으로 수정할 수 있다.  또한 <strong>wa</strong> 명령어는 현재 탐색 위치의 값을 입력한 opcode로 수정할 수 있다. </p>
<ol>
<li><h4>디버깅을 위한 명령어들</h4>
</li>

</ol>
<p>이번에는 r2 -d옵션을 이용하여 디버그 모드에서 이용되는 명령어들을 알아보자. </p>
<p><strong>dc</strong>명령어를 입력하면 실행을 하게되며, 첫 실행이나 break point에 걸렸을때 계속 실행하는 명령어이다.</p>
<p><strong>db 주소</strong>명령어를 이용하면 break point를 설정할 수 있다. 또한 <strong>db- 주소</strong>하면 해당 break point를 제거할 수 있고 <strong>db-</strong> 의경우 모든  break point를 제거한다.</p>
<p><strong>dr</strong> 명령어는 디버그 레지스터의 상태를 보여준다. </p>
<p><strong>afvd</strong> 명령어는 디버깅 진행중 해당 변수들의 상태를 보여준다.</p>
<p>이후에 <strong>dc</strong> 를 입력하면 디버깅이 종료되게 된다. </p>
<hr />
<h1>2. rabin2</h1>
<hr />
<p>기본 usage 예제는 <strong>rabin2 -l 파일이름</strong>으로 아래 와 같은 기능을 수행한다.</p>
<blockquote><p><strong>$ rabin2 -I ./RedVelvet</strong>
arch     x86
baddr    0x400000
binsz    11797
bintype  elf
bits     64
canary   true
class    ELF64
compiler GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.5) 5.4.0 20160609
crypto   false
endian   little
havecode true
intrp    /lib64/ld-linux-x86-64.so.2
laddr    0x0
lang     c
linenum  true
lsyms    true
machine  AMD x86-64 architecture
maxopsz  16
minopsz  1
nx       true
os       linux
pcalign  0
pic      false
relocs   true
relro    partial
rpath    NONE
sanitiz  false
static   false
stripped false
subsys   linux
va       true</p>
</blockquote>
<p>각종 메모리 보호 기법과 아키텍쳐까지 굉장히 많은 정보를 한 눈에 확인 할 수 있습니다.</p>
<hr />
<h1>3. rax2</h1>
<hr />
<p>usage는 <strong>rax2 value</strong>이며 아래 예제 보다 휠 많은 옵션들이 있지만 분석 도중 간단히 사용할 수 있는 진법 변환기</p>
<blockquote><p><strong>$ rax2 0xf</strong>
15
<strong>$ rax2 15</strong>
0xf
<strong>$ rax2 b15</strong>
1111b
<strong>$ rax2 1111b</strong>
0xf</p>
</blockquote>
<p>무려 float 형인 실수까지 변환 가능하다.(물론 강제 캐스팅?!)</p>
<hr />
<h1>4. radiff2</h1>
<hr />
<p>usage는 <strong>radiff2 file1 file2</strong></p>
<blockquote><p><strong>$ cat ./1.txt</strong>
123
<strong>$ cat ./2.txt</strong>
321
<strong>$ radiff2 ./1.txt ./2.txt</strong>
0x00000000 31 =&gt; 33 0x00000000
0x00000002 33 =&gt; 31 0x00000002</p>
</blockquote>
<p>&nbsp;</p>
<p>위같은 너무~ 간단한 비교부터 여러 옵션으로 아래 주소 같은 정밀하게 비교도 가능한 r2의 비교 툴 radiff2!!
<a href='https://radareorg.github.io/blog/posts/binary-diffing/' target='_blank' class='url'>https://radareorg.github.io/blog/posts/binary-diffing/</a></p>
<hr />
<h1>5. rafind2</h1>
<hr />
<p>usage는 <strong>rafind2 file</strong></p>
<blockquote><p><strong>$ rafind2 Kingmaker</strong>
0x00405cc0  | |/ (<em>)</em> <em>_   __ _|  \/  | __ _| | <strong>_</strong> _ __
0x00405cf8  | &#39; /| | &#39;</em> \ / <em><code>| |\\/| |/ _</code> | |/ / _ \ &#39;__|
0x00405d30  | . \| | | | | (</em>| | |  | | (<em>| |   &lt;  __/ |
0x00405d68  |</em>|\<em>\</em>|<em>| |</em>|\<strong>, |<em>|  |</em>|\</strong>,<em>|</em>|\<em>\___|</em>|
0x00405da0                |___/                            \n
0x00405dd8 Once upon a time, there was a kingdom with 7 princes.
0x00405e10 One day the king thinks to decide the who will be the next king.
0x00405e58 So he made 5 tests for the princes.
0x00405e80 If you pass all the tests, you can be a king!\n
0x00405eb0 <strong>**</strong><strong>**</strong><strong>**</strong><strong></strong>KING MAKER START<strong>**</strong><strong>**</strong><strong>**</strong><strong></strong>\n
0x00405eee .......
0x00405ef6 Who am I...??
0x00405f04 1&gt; Ask to someone
0x00405f16 2&gt; Look around
0x00406158 \e\f\a\b
0x00406188 \e\f\a\b
0x004061af ;*3$&quot;
0x00406211 V\f\a\b
0x004062b1 Y\f\a\b
0x004062f1 Y\f\a\b
0x00406312 I\f\a\b
0x00406331 Y\f\a\b
0x00406479 Y\f\a\b
0x004064b9 Y\f\a\b
0x004066f9 Y\f\a\b
0x0040673a I\f\a\b
0x00406759 y\f\a\b
0x004067b9 Y\f\a\b
0x00406879 Y\f\a\b
0x00406899 l\f\a\b
0x004068d9 Y\f\a\b
0x00406939 j\f\a\b
0x00406959 V\f\a\b
0x006070d0 Dæúæ
0x006070e4 DDN&gt;ÍƍÃðú
0x00000000 GCC: (Ubuntu 5.4.0-6ubuntu1~16.04.10) 5.4.0 20160609
0x00000001 .shstrtab
0x0000000b .interp
0x00000013 .note.ABI-tag
0x00000021 .note.gnu.build-id
0x00000034 .gnu.hash
0x0000003e .dynsym
0x00000046 .dynstr
0x0000004e .gnu.version
0x0000005b .gnu.version_r
0x0000006a .rela.dyn
0x00000074 .rela.plt
0x0000007e .init
0x00000084 .plt.got
0x0000008d .text
0x00000093 .fini
0x00000099 .rodata
0x000000a1 .eh_frame_hdr
0x000000af .eh_frame
0x000000b9 .init_array
0x000000c5 .fini_array
0x000000d1 .jcr
0x000000d6 .dynamic
0x000000df .got.plt
0x000000e8 .data
0x000000ee .bss
0x000000f3 .comment</p>
</blockquote>
<p> 위 처럼 문자열 뿐만 아니라 데이터 영역 .bss 영역등과 아래 와 같이 함수 이름을 확인 할 수 있었습니다.</p>
<blockquote><p><strong>$ rafind2 ./RedVelvet | grep &quot;func&quot;</strong>
0x00000131 func8
0x00000137 func10
0x00000185 func14
0x000001ab func3
0x000001ef func7
0x00000258 func13
0x0000028c func2
0x000002a2 func12
0x000002a9 func6
0x00000324 func5
0x0000032a func9
0x00000330 func1
0x00000336 func11
0x0000033d func15
0x00000366 func4</p>
</blockquote>
<p>r2용 find 툴~</p>
<hr />
<h1>6. rahash2</h1>
<hr />
<p>usage <strong>rahash2 file</strong>이며 아래 rahash2툴로 할 수 있는 hash 목록</p>
<blockquote><p><strong>Available Hashes</strong>: 
md5
sha1
sha256
sha384
sha512
crc16
crc32
md4
xor
xorpair
parity
entropy
hamdist
pcprint
mod255
xxhash
adler32
luhn
crc8smbus
crc15can
crc16hdlc
crc16usb
crc16citt
crc24
crc32c
crc32ecma267</p>
</blockquote>
<p>또한 인코딩, 디코딩 까지 ~</p>
<blockquote><p><strong>Available Encoders/Decoders</strong>: 
base64
base91
punycode</p>
</blockquote>
<p>그리고 암호화 알고리즘 까지 지원!</p>
<blockquote><p><strong>Available Crypto Algos</strong>: 
rc2
rc4
rc6
aes-ecb
aes-cbc
ror
rol
rot
blowfish
cps2
des-ecb
xor
serpent-ecb</p>
</blockquote>
<p>기본으로는 아래와 같이 sha256으로 설정</p>
<blockquote><p><strong>$ rahash2 ./KingMaker</strong>
./KingMaker: 0x00000000-0x000079af 
sha256: 44d00c9ae24574b8d8e1f94e58c84913f5bd5356ba36a5d1ed65b174d9556150</p>
</blockquote>
<hr />
<h1>7. rasm2</h1>
<hr />
<p>rasm2은 어셈블러 + 디어셈블러 툴 </p>
<blockquote><p><strong>$ rasm2 -d 90</strong>
nop
<strong>$ rasm2 -a x86 -b 32 &#39;mov eax, 33&#39;</strong>
b821000000
<strong>$ rasm2 &quot;nop&quot;</strong>
90</p>
</blockquote>
<p>opcode나 어셈블리 명령어를 넣어도 바로 변환해주는 툴</p>
<hr />
<h1>8.  r2</h1>
<hr />
<p>usage </p>
<blockquote><p><strong>$ r2 file</strong>
정적 모드</p>
</blockquote>
<blockquote><p><strong>$ r2 -d file</strong>
Debugger 모드</p>
</blockquote>
<blockquote><p><strong>$ r2 -w file</strong>
write(수정) 모드</p>
</blockquote>
<p>r2 처음 바이너리를 분석할때 사용자는 <strong>aa</strong>([a]nalyze [a]ll)이라는 명령어로 r2가 프로그램을 분석하라고 알려주면 잡히지 않았던 변수나 함수들이 정상적으로 잡히게된다 특히 aa에서 a를 추가하여 <strong>aaa</strong>사용하여 aa 보다 더 많은 알고리즘을통해 프로그램을 분석 시킨다.</p>
<blockquote><p><strong>$ r2 -d RedVelvet</strong>
Process with PID 18907 started...
= attach 18907 18907
bin.baddr 0x00400000
Using 0x400000
asm.bits 64
-- There is no F5 key in radare2 yet
<strong>[0x7f4d3d400c30]&gt; aa</strong>
[x] Analyze all flags starting with sym. and entry0 (aa)
<strong>[0x7f4d3d400c30]&gt; aaa</strong>
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze function calls (aac)
[x] Analyze len bytes of instructions for references (aar)
[x] Check for objc references
[x] Check for vtables
[TOFIX: aaft can&#39;t run in debugger mode.ions (aaft)
[x] Type matching analysis for all functions (aaft)
[x] Propagate noreturn information
[x] Use -AA or aaaa to perform additional experimental analysis.
<strong>[0x7f4d3d400c30]&gt;</strong></p>
</blockquote>
<p>aaaa도 존재하지만 시간이 쫌 걸리고 aaa에서 잡히지 않는 이상 거의 무족건 안잡히는거라고 생각하면 된다.</p>
<h4><strong>afl</strong> 명령어를 통해서 현재 프로그램에서 심볼이 잡힌 함수들의 목록을 프린트해준다.</h4>
<blockquote><p><strong>&gt;[0x7f0da9e00c30]&gt; afl</strong>
0x00400890    1 41           entry0
0x004007e0    1 6            sym.imp.<strong>libc_start_main
0x004008c0    4 50   -&gt; 41   sym.deregister_tm_clones
0x00400900    4 58   -&gt; 55   sym.register_tm_clones
0x00400940    3 28           entry.fini0
0x00400960    4 38   -&gt; 35   entry.init0
0x004016b0    1 2            sym.</strong>libc_csu_fini
0x004007d0    1 6            sym.imp.exit
0x004007c0    1 6            sym.imp.puts
0x004016b4    1 9            sym.<em>fini
0x00401640    4 101          sym.__libc_csu_init
0x00400780    3 26           sym.</em>init
0x004007b0    1 6            sym.imp.printf
0x004007f0    1 6            sym.imp.fgets
0x00400800    1 6            sym.imp.strlen
0x00400810    1 6            sym.imp.sprintf
0x00400820    1 6            sym.imp.ptrace
0x00400830    1 6            sym.imp.SHA256_Update
0x00400840    1 6            sym.imp.__stack_chk_fail
0x00400850    1 6            sym.imp.strcmp
0x00400860    1 6            sym.imp.SHA256_Final
0x00400870    1 6            sym.imp.SHA256_Init
0x00400986    9 126          sym.func1
0x00400a04    6 79           sym.func2
0x00400a53    6 88           sym.func3
0x00400aab    7 98           sym.func4
0x00400b0d    6 91           sym.func5
0x00400b68   12 169          sym.func6
0x00400c11   12 166          sym.func7
0x00400cb7   11 166          sym.func8
0x00400d5d    9 133          sym.func9
0x00400de2   10 136          sym.func10
0x00400e6a   11 169          sym.func11
0x00400f13   11 166          sym.func12
0x00400fb9   12 178          sym.func13
0x0040106b   10 138          sym.func14</p>
<p><strong>0x004010f5   11 180          sym.func15</strong>
0x004011a9   14 1440 -&gt; 1176 main</p>
</blockquote>
<p>또한 리눅스의 ailas 같은 기능으로 <strong>afn</strong> 명령어로 자신이 원하는 함수의 이름을 재설정 할 수 있습니다 ! 
간단한 예제로 func 15를 fun1 로 바꿔 보겠습니다.</p>
<blockquote><p><strong>[0x7f0da9e00c30]&gt; afn fun1 0x004010f5</strong>
<strong>[0x7f0da9e00c30]&gt; afl</strong>
0x00400890    1 41           entry0
0x004007e0    1 6            sym.imp.<strong>libc_start_main
0x004008c0    4 50   -&gt; 41   sym.deregister_tm_clones
0x00400900    4 58   -&gt; 55   sym.register_tm_clones
0x00400940    3 28           entry.fini0
0x00400960    4 38   -&gt; 35   entry.init0
0x004016b0    1 2            sym.</strong>libc_csu_fini
0x004007d0    1 6            sym.imp.exit
0x004007c0    1 6            sym.imp.puts
0x004016b4    1 9            sym.<em>fini
0x00401640    4 101          sym.__libc_csu_init
0x00400780    3 26           sym.</em>init
0x004007b0    1 6            sym.imp.printf
0x004007f0    1 6            sym.imp.fgets
0x00400800    1 6            sym.imp.strlen
0x00400810    1 6            sym.imp.sprintf
0x00400820    1 6            sym.imp.ptrace
0x00400830    1 6            sym.imp.SHA256_Update
0x00400840    1 6            sym.imp.__stack_chk_fail
0x00400850    1 6            sym.imp.strcmp
0x00400860    1 6            sym.imp.SHA256_Final
0x00400870    1 6            sym.imp.SHA256_Init
0x00400986    9 126          sym.func1
0x00400a04    6 79           sym.func2
0x00400a53    6 88           sym.func3
0x00400aab    7 98           sym.func4
0x00400b0d    6 91           sym.func5
0x00400b68   12 169          sym.func6
0x00400c11   12 166          sym.func7
0x00400cb7   11 166          sym.func8
0x00400d5d    9 133          sym.func9
0x00400de2   10 136          sym.func10
0x00400e6a   11 169          sym.func11
0x00400f13   11 166          sym.func12
0x00400fb9   12 178          sym.func13
0x0040106b   10 138          sym.func14</p>
<p><strong>0x004010f5   11 180          fun1</strong>
0x004011a9   14 1440 -&gt; 1176 main</p>
</blockquote>
<p>위와 같이  0x004010f5 함수 sys.func15가 fun1으로 바뀌는 모습을 볼 수 있습니다.
또한 <strong>~</strong>를 사용하여 <strong>afl~something</strong>으로 마치 grep을 사용하는 형태로 이용할 수 있습니다.</p>
<blockquote><p><strong>[0x7f0da9e00c30]&gt; afl~sym</strong>
0x004007e0    1 6            sym.imp.<strong>libc_start_main
0x004008c0    4 50   -&gt; 41   sym.deregister_tm_clones
0x00400900    4 58   -&gt; 55   sym.register_tm_clones
0x004016b0    1 2            sym.</strong>libc_csu_fini
0x004007d0    1 6            sym.imp.exit
0x004007c0    1 6            sym.imp.puts
0x004016b4    1 9            sym.<em>fini
0x00401640    4 101          sym.__libc_csu_init
0x00400780    3 26           sym.</em>init
0x004007b0    1 6            sym.imp.printf
0x004007f0    1 6            sym.imp.fgets
0x00400800    1 6            sym.imp.strlen
0x00400810    1 6            sym.imp.sprintf
0x00400820    1 6            sym.imp.ptrace
0x00400830    1 6            sym.imp.SHA256_Update
0x00400840    1 6            sym.imp.__stack_chk_fail
0x00400850    1 6            sym.imp.strcmp
0x00400860    1 6            sym.imp.SHA256_Final
0x00400870    1 6            sym.imp.SHA256_Init
0x00400986    9 126          sym.func1
0x00400a04    6 79           sym.func2
0x00400a53    6 88           sym.func3
0x00400aab    7 98           sym.func4
0x00400b0d    6 91           sym.func5
0x00400b68   12 169          sym.func6
0x00400c11   12 166          sym.func7
0x00400cb7   11 166          sym.func8
0x00400d5d    9 133          sym.func9
0x00400de2   10 136          sym.func10
0x00400e6a   11 169          sym.func11
0x00400f13   11 166          sym.func12
0x00400fb9   12 178          sym.func13
0x0040106b   10 138          sym.func14</p>
</blockquote>
<p>위와 같이 <strong>sym</strong>문자열이 들어간 결과들만 출력되는걸 확인 할 수 있습니다.</p>
<p>r2에서 어셈블리를 출력하는 방법은 총 3가지가 있습니다. </p>
<ol>
<li><strong>s</strong>([s]eek) 명령어를 사용한 주소 이동후 <strong>pdf</strong>[[p]rint [d]isassembly [f]unction] 명령어 출력</li>
<li><strong>pdf</strong> 명령어를 통한 주소 명시 출력</li>
<li><strong>pd</strong>[[p]rint [d]isassembly]명령어로 함수가 아닌 구역 어셈블리 출력</li>

</ol>
<p>첫번째 케이스, afl로 얻은 정보로 아래와 같이 s로 주소 이동</p>
<blockquote><p>0x004011a9   14 1440 -&gt; 1176 main
<strong>[0x7f482a400c30]&gt; s main</strong>
<strong>[0x004011a9]&gt;</strong></p>
</blockquote>
<p>이후 pdf 명령어를 통해 현재 주소 기준으로 함수 디셈블리한 어셈들을 출력한다.ㅇ</p>
<p>이미지 613</p>
<p>두번째 케이스, 그냥 <strong>pdf main</strong>(주소 변동없음 지칭 함수만 출력).</p>
<p>세번쨰 케이스, <strong>pd</strong> 명령어를 통한 <strong>함수가 아닌 구역</strong> 디셈블리한 내용 출력한다.</p>
<p>이 케이스가 가장 특이한데 afl통해 심볼릭이 잡히지 않는 구역의 코드를 인식해서 강제로 디셈블리해서 출력해주는 명령어다. 첫번째와 두번째는 주소 변동 말고는 차이 없는 같은 출력을 해준다.</p>
<p>또한 <strong>px</strong> 명령어를 통해서 gdb의 x/?? 참조해서 출력하는 역할과 똑같이 출력해 줄 수 있다,</p>
<blockquote><p><strong>[0x7f19dda00c30]&gt; px 0x15 @0x4016e2</strong></p>
<ul>
<li>offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x004016e2  666c 6167 203a 207b 2220 2573 2022 7d0a  flag : {&quot; %s &quot;}.
0x004016f2  0000 011b 03                             .....</li>

</ul>
</blockquote>
<p><strong>@</strong>를 통해서 *(참조)똑같은 의미이고 0x4016e2부터 0x15크기 만큼 출력하라는 의미이다.</p>
<hr />
<h2>String 출력 관련 명령어</h2>
<hr />
<p><strong>fs</strong> 명령어를 통해서 프로그램이 인식한 <strong>flagspaces</strong>를 출력해주고 
<strong>fs strings; f</strong> 를 사용하여 문자열 플래그 값을 전부 출력한다.</p>
<blockquote><p><strong>[0x7f19dda00c30]&gt; fs</strong>
7 * classes
0 * functions
17 * imports
19 * regs
15 * relocs
31 * sections
10 * segments
4 * strings
50 * symbols
27 * symbols.sections
<strong>[0x7f19dda00c30]&gt; fs strings; ffs strings; f</strong>
0x004016c4 12 str.HAPPINESS:
0x004016d0 13 str.Your_flag<em>:
0x004016dd 5 str.02x
0x004016e2 17 str.flag</em>:<strong>_</strong>s</p>
</blockquote>
<p>또한 <strong>axt @@ str.*</strong> 통해서 모든 문자열의 참조되어 있는 명령어를 출력해준다.</p>
<blockquote><p><strong>[0x7f19dda00c30]&gt; axt @@ str.*</strong>
sym.func1 0x4009e1 [DATA] mov edi, str.HAPPINESS:
sym.func2 0x400a30 [DATA] mov edi, str.HAPPINESS:
sym.func3 0x400a93 [DATA] mov edi, str.HAPPINESS:
sym.func4 0x400af5 [DATA] mov edi, str.HAPPINESS:
sym.func5 0x400b50 [DATA] mov edi, str.HAPPINESS:
sym.func6 0x400bf9 [DATA] mov edi, str.HAPPINESS:
sym.func7 0x400c9f [DATA] mov edi, str.HAPPINESS:
sym.func8 0x400d45 [DATA] mov edi, str.HAPPINESS:
sym.func9 0x400dca [DATA] mov edi, str.HAPPINESS:
sym.func10 0x400e52 [DATA] mov edi, str.HAPPINESS:
sym.func11 0x400efb [DATA] mov edi, str.HAPPINESS:
sym.func12 0x400fa1 [DATA] mov edi, str.HAPPINESS:
sym.func13 0x401053 [DATA] mov edi, str.HAPPINESS:
sym.func14 0x4010dd [DATA] mov edi, str.HAPPINESS:
sym.func15 0x401191 [DATA] mov edi, str.HAPPINESS:
main 0x40125a [DATA] mov edi, str.Your_flag<em>:
main 0x4015b6 [DATA] mov esi, str.02x
main 0x4015fc [DATA] mov edi, str.flag</em>:<strong>_</strong>s</p>
</blockquote>
<p>어디서 함수에서 호출되는지  어떤 어셈블리어인지 전부 출력된다.
그리고 <strong>izzq~string</strong> 이라는 명령어를 통해 특정 스트링을 검색또한 가능하다.</p>
<blockquote><p><strong>[0x7f19dda00c30]&gt; izzq~flag</strong>
0x4016d0 13 12 Your flag :
0x4016e2 17 16 flag : {&quot; %s &quot;}\n</p>
</blockquote>
<hr />
<h2>바이너리 패치</h2>
<hr />
<blockquote><p>반드시 바이너리 패치는 -w 옵션으로 실행해야하고 패치된 바이너리는 바로 저장되기때문에 백업 필수</p>
</blockquote>
<p><strong>wx</strong> 명령어로 통해서 <strong>hex 데이터</strong>로 데이터를 변경할 수 있고 변경할 위치에 꼭 <strong>s</strong>로 이동해야한다.</p>
<blockquote><p><strong>[0x7f19dda00c30]&gt; px 0x10</strong></p>
<ul>
<li>offset -       0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x7f19dda00c30  4889 e7e8 780d 0000 4989 c48b 0537 5022  H...x...I....7P&quot;
<strong>[0x7f19dda00c30]&gt; wx 90</strong>
<strong>[0x7f19dda00c30]&gt; px 0x10</strong></li>
<li>offset -       0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x7f19dda00c30  9089 e7e8 780d 0000 4989 c48b 0537 5022  ....x...I....7P&quot;</li>

</ul>
</blockquote>
<p>또한 <strong>wa</strong> 를 사용한  <strong>opcode  string 데이터</strong>로 패치를 할 수 있다.</p>
<blockquote><p><strong>0x004015f0      752f           jne 0x401621</strong>
<strong>[0x004015f0]&gt; px</strong></p>
<ul>
<li>offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x004015f0  752f 488b 8588 feff ff48 89c6 bfe2 1640  u/H......H.....@
<strong>[0x004015f0]&gt; wa je 0x401621</strong>
Written 2 byte(s) (je 0x401621) = wx 742f
<strong>[0x004015f0]&gt; px</strong></li>
<li>offset -   0 1  2 3  4 5  6 7  8 9  A B  C D  E F  0123456789ABCDEF
0x004015f0  742f 488b 8588 feff ff48 89c6 bfe2 1640  t/H......H.....@</li>

</ul>
</blockquote>
<hr />
<h2>디버그 모드</h2>
<hr />
<p>반드시 -d 옵션으로 시작해야한다.</p>
<p><strong>dc</strong>를 통해서  시작 시키고 <strong>db 주소 지칭(심볼릭)</strong> breakpoint를 설정하며 -키워드를 사용하여 <strong>db -주소(심볼릭)</strong> 으로 해체 할 수 있다. </p>
<blockquote><p><strong>[0x7f4a23600c30]&gt; db main</strong>
<strong>[0x7f4a23600c30]&gt; dc</strong>
hit breakpoint at: 4011a9
<strong>[0x004011a9]&gt; afl~main</strong>
0x004007e0    1 6            sym.imp.__libc_start_main
0x004011a9   14 1440 -&gt; 1176 main</p>
</blockquote>
<p>또한 <strong>dr</strong> 통해서 현재 레지스터 상태를 출력한다.</p>
<blockquote><p><strong>[0x004011a9]&gt; dr</strong>
rax = 0x004011a9
rbx = 0x00000000
rcx = 0x00000000
rdx = 0x7fffcb6855b8
r8 = 0x004016b0
r9 = 0x7f4a23610ac0
r10 = 0x00000846
r11 = 0x7f4a22e00740
r12 = 0x00400890
r13 = 0x7fffcb6855a0
r14 = 0x00000000
r15 = 0x00000000
rsi = 0x7fffcb6855a8
rdi = 0x00000001
rsp = 0x7fffcb6854c8
rbp = 0x00401640
rip = 0x004011a9
rflags = 0x00000246
orax = 0xffffffffffffffff</p>
</blockquote>
<p>그리고 <strong>afvd</strong> 명령어를 통해서 현재 생성된 변수와 값을 여러 형태로 출력해준다.</p>
<blockquote><p><strong>[0x004011a9]&gt; afvd</strong>
var var_8h = 0x00401638 = (qword)0x00401f0f00000000
var var_70h = 0x004015d0 = (qword)0xb47e1ffffffe48bd
var var_68h = 0x004015d8 = (qword)0x40858d4890558d48
var var_60h = 0x004015e0 = (qword)0x8948d68948ffffff
var var_58h = 0x004015e8 = (qword)0xc085fffff262e8c7
var var_50h = 0x004015f0 = (qword)0xfffe88858b482f75
var var_48h = 0x004015f8 = (qword)0x4016e2bfc68948ff
var var_40h = 0x00401600 = (qword)0xa5e800000000b800
var var_38h = 0x00401608 = (qword)0x00000000b8fffff1
var var_30h = 0x00401610 = (qword)0x34334864f8758b48
var var_28h = 0x00401618 = (qword)0xeb11740000002825
var var_20h = 0x00401620 = (qword)0xa5e800000001bf0a
var var_18h = 0x00401628 = (qword)0xfffff210e8fffff1
var var_10h = 0x00401630 = (qword)0x00841f0f2e66c3c9
var var_100h = 0x00401540 = (qword)0xb5e8c78948fffffe
var var_178h = 0x004014c8 = (qword)0x7c8589fffffaece8
var var_ffh = 0x00401541 = (qword)0xf2b5e8c78948ffff
var var_1b4h = 0x0040148c = (qword)0xffffff1185b60fc8
var var_feh = 0x00401542 = (qword)0xfff2b5e8c78948ff
var var_1b0h = 0x00401490 = (qword)0x89c0be0fffffff11
var var_fdh = 0x00401543 = (qword)0xfffff2b5e8c78948
var var_1ach = 0x00401494 = (qword)0xe8c789ce89c0be0f
var var_fch = 0x00401544 = (qword)0x48fffff2b5e8c789
var var_1a8h = 0x00401498 = (qword)0xfffffa73e8c789ce
var var_fbh = 0x00401545 = (qword)0x8948fffff2b5e8c7
var var_1a4h = 0x0040149c = (qword)0xfe788589fffffa73
var var_f9h = 0x00401547 = (qword)0x48c28948fffff2b5
var var_fah = 0x00401546 = (qword)0xc28948fffff2b5e8
var var_1a0h = 0x004014a0 = (qword)0xb60ffffffe788589
var var_f7h = 0x00401549 = (qword)0x8d8b48c28948ffff
var var_f8h = 0x00401548 = (qword)0x8b48c28948fffff2
var var_19ch = 0x004014a4 = (qword)0xffff1585b60fffff
var var_f5h = 0x0040154b = (qword)0xfe888d8b48c28948
var var_f6h = 0x0040154a = (qword)0x888d8b48c28948ff
var var_198h = 0x004014a8 = (qword)0xd0be0fffffff1585
var var_f3h = 0x0040154d = (qword)0xfffffe888d8b48c2
var var_f4h = 0x0040154c = (qword)0xfffe888d8b48c289
var var_194h = 0x004014ac = (qword)0x1485b60fd0be0fff
var var_f1h = 0x0040154f = (qword)0x8d48fffffe888d8b
var var_f2h = 0x0040154e = (qword)0x48fffffe888d8b48
var var_190h = 0x004014b0 = (qword)0x0fffffff1485b60f
var var_efh = 0x00401551 = (qword)0x90858d48fffffe88
var var_f0h = 0x00401550 = (qword)0x858d48fffffe888d
var var_18ch = 0x004014b4 = (qword)0xb60fc8be0fffffff
var var_edh = 0x00401553 = (qword)0xfffe90858d48ffff
var var_eeh = 0x00401552 = (qword)0xfe90858d48fffffe
var var_188h = 0x004014b8 = (qword)0xffff1385b60fc8be
var var_ebh = 0x00401555 = (qword)0x48fffffe90858d48
var var_ech = 0x00401554 = (qword)0xfffffe90858d48ff
var var_184h = 0x004014bc = (qword)0xc0be0fffffff1385
var var_e9h = 0x00401557 = (qword)0xce8948fffffe9085
var var_eah = 0x00401556 = (qword)0x8948fffffe90858d
var var_180h = 0x004014c0 = (qword)0xc789ce89c0be0fff
var var_e7h = 0x00401559 = (qword)0x8948ce8948fffffe
var var_e8h = 0x00401558 = (qword)0x48ce8948fffffe90
var var_17ch = 0x004014c4 = (qword)0xfffaece8c789ce89
var var_170h = 0x004014d0 = (qword)0xff1785b60ffffffe
var var_e0h = 0x00401560 = (qword)0x48fffff2c9e8c789
var var_1b8h = 0x00401488 = (qword)0x85b60fc8be0fffff
var var_c0h = 0x00401580 = (qword)0x0000fffffe4885c7</p>
</blockquote>
<p>그리고 VV 명령어를 사용하면 아래 와 같은 그래픽 뷰를 지원해주기 때문에 어셈분석할때 큰 도움을 받을 수 있다. </p>
<p>이미지 614</p>
<p>종료는 <strong>q</strong></p>
<p>그리고 마지막으로 아래와 같이 떴을때 <strong>oob</strong> 명령어를 통해서 프로그램을 다시 재설정해야지 디버깅할 수 있기 때문에 <strong>oob arg1 arg2_</strong> 이런식의 입력이다.(올리디버거 ctrl + f2 랑 비슷한 맥락)</p>
<blockquote><p><strong>[0x7f2e08400c30]&gt; dc</strong>
Your flag :
<strong>[0x7f2e07cac748]&gt; dc</strong>
child exited with status 1</p>
</blockquote>
<blockquote><p>==&gt; Process finished</p>
</blockquote>
<blockquote><p><strong>[0x7f2e07cac748]&gt; oob</strong>
Process with PID 18954 started...
= attach 18954 18954
File dbg://./RedVelvet reopened in read-write mode
Unable to find filedescriptor 5
Unable to find filedescriptor 5
18954
<strong>[0x7fd1e3800c30]&gt; dc</strong>
Your flag :</p>
</blockquote>
