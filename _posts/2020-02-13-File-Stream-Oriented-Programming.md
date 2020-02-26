File Stream Oriented Programming 의 기본은 File Structure 을 분석하는것으로 부터 시작된다.

먼저, 기본적인 File Structure 를 보자.

```c
struct _IO_FILE
{
  int _flags;                /* High-order word is _IO_MAGIC; rest is flags. */
  /* The following pointers correspond to the C++ streambuf protocol. */
  char *_IO_read_ptr;        /* Current read pointer */
  char *_IO_read_end;        /* End of get area. */
  char *_IO_read_base;        /* Start of putback+get area. */
  char *_IO_write_base;        /* Start of put area. */
  char *_IO_write_ptr;        /* Current put pointer. */
  char *_IO_write_end;        /* End of put area. */
  char *_IO_buf_base;        /* Start of reserve area. */
  char *_IO_buf_end;        /* End of reserve area. */
  /* The following fields are used to support backing up and undo. */
  char *_IO_save_base; /* Pointer to start of non-current get area. */
  char *_IO_backup_base;  /* Pointer to first valid character of backup area */
  char *_IO_save_end; /* Pointer to end of non-current get area. */
  struct _IO_marker *_markers;
  struct _IO_FILE *_chain;
  int _fileno;
  int _flags2;
  __off_t _old_offset; /* This used to be _offset but it's too small.  */
  /* 1+column number of pbase(); 0 is unknown. */
  unsigned short _cur_column;
  signed char _vtable_offset;
  char _shortbuf[1];
  _IO_lock_t *_lock;
#ifdef _IO_USE_OLD_IO_FILE
};
```

먼저 **_flags** 는 읽기 전용, 쓰기 전용 등의 권한을 설정하는 flag 이다. 

기본적으론 **0xfbad0000** 가 기본 매직값이고, 하위 2비트는 여러 플래그들이 들어갈 수 있다.

```c
/* _flags bit */
#define _IO_MAGIC         0xFBAD0000 /* Magic number */
#define _IO_MAGIC_MASK    0xFFFF0000
#define _IO_USER_BUF          0x0001 /* Don't deallocate buffer on close. */
#define _IO_UNBUFFERED        0x0002
#define _IO_NO_READS          0x0004 /* Reading not allowed.  */
#define _IO_NO_WRITES         0x0008 /* Writing not allowed.  */
#define _IO_EOF_SEEN          0x0010
#define _IO_ERR_SEEN          0x0020
#define _IO_DELETE_DONT_CLOSE 0x0040 /* Don't call close(_fileno) on close.  */
#define _IO_LINKED            0x0080 /* In the list of all open files.  */
#define _IO_IN_BACKUP         0x0100
#define _IO_LINE_BUF          0x0200
#define _IO_TIED_PUT_GET      0x0400 /* Put and get pointer move in unison.  */
#define _IO_CURRENTLY_PUTTING 0x0800
#define _IO_IS_APPENDING      0x1000
#define _IO_IS_FILEBUF        0x2000
                           /* 0x4000  No longer used, reserved for compat.  */
#define _IO_USER_LOCK         0x8000
```

예를 들어 

```c
#include <stdio.h>
int main() {
	FILE *fp = fopen("secret", "r");
}
```

다음과 같은 코드가 있다고 하자. 그럼 이 경우에 플래그 값은 0xfbad2488로, **0xfbad0000 + 0x2000 + 0x400 + 0x80 + 0x8** 이다,

각각 **_IO_MAGIC**, **_IO_IS_FILEBUF**, **_IO_TIED_PUT_GET**, **_IO_LINKED**, **_IO_NO_WRITES** 를 나타낸다. 이 뜻은, **열려있는 파일에 write 할 수 없음** 을 뜻 한다. 대충 이런 식으로 어떻게 이루어지는지만 볼 수 있으면 된다.

그리고 이러한 플래그 값들은 fopen 으로 파일을 열 때 _IO_new_file_fopen 함수가 호출 되는데, 이 플래그 값을 인자(mode) 로 전달하여

플래그 값을 이용해 파일을 열게 된다.

```c
FILE * _IO_new_file_fopen (FILE *fp, const char *filename, const char *mode, int is32not64)
{
  int oflags = 0, omode;
  int read_write;
  int oprot = 0666;
  int i;
  FILE *result;
  const char *cs;
  const char *last_recognized;
  if (_IO_file_is_open (fp))
    return 0;
  switch (*mode)
    {
    case 'r':
      omode = O_RDONLY;
      read_write = _IO_NO_WRITES;
      break;
    case 'w':
      omode = O_WRONLY;
      oflags = O_CREAT|O_TRUNC;
      read_write = _IO_NO_READS;
      break;
    case 'a':
      omode = O_WRONLY;
      oflags = O_CREAT|O_APPEND;
      read_write = _IO_NO_READS|_IO_IS_APPENDING;
      break;
     ...
}
```

그리고 이제 3 분류의 Stream buffer pointer 를 알아보도록 하겠다. 각각 read buffers, write buffers, reserve buffers 로 이루어져 있다.

* **Read buffers**
  1. _IO_read_ptr
  2. _IO_read_end
  3. _IO_read_base

- **Write buffers**
  1. _IO_write_ptr
  2. _IO_write_end
  3. _IO_write_base

- **Reserve buffers**
  1. _IO_buf_base
  2. _IO_buf_end

여기서 **_IO_read_ptr, _IO_write_ptr** 등의 ptr 은 현재 버퍼의 위치를 가르킨다. 

그리고 **_fileno** 는 open 할때의 file descriptor 이다. 보통 0,1,2 가 있고, 각각 input, output, error 를 뜻한다.

지금 까지 **_IO_FILE** 구조체를 분석 했다. 하지만 실제로 file stream을 열때는 **_IO_FILE_plus** 구조체가 리턴된다.

## _IO_FILE_plus

**_IO_FILE_plus** 란, 파일 스트림에서 함수 호출을 잘 써먹기 위해 _IO_FILE에 함수 포인터 테이블을 가리키는 포인터를 추가한

구조체 이다. stdin, stdout, stderr 도 이 구조체를 사용하고, 모든 파일 관련된 작업들은 이 **_IO_FILE_plus** 를 사용한다.

이는 아래와 같이 정의된다.

```c
struct _I_FILE_plus
{
  _IO_FILE file;
  const struct _IO_jump_t *vtable;
};
```

**_IO_jump_t** 파일에 관련된 여러 함수 포인터들이 저장 되어있다.

이들은 **fread**, **fopen**, **fwrite** 와 같은 여러 표준 함수에서 호출된다. **__IO_jump_t** 는 다음과 같이 정의된다.

```c
struct _IO_jump_t
{
    JUMP_FIELD(size_t, __dummy);
    JUMP_FIELD(size_t, __dummy2);
    JUMP_FIELD(_IO_finish_t, __finish);
    JUMP_FIELD(_IO_overflow_t, __overflow);
    JUMP_FIELD(_IO_underflow_t, __underflow);
    JUMP_FIELD(_IO_underflow_t, __uflow);
    JUMP_FIELD(_IO_pbackfail_t, __pbackfail);
    /* showmany */
    JUMP_FIELD(_IO_xsputn_t, __xsputn);
    JUMP_FIELD(_IO_xsgetn_t, __xsgetn);
    JUMP_FIELD(_IO_seekoff_t, __seekoff);
    JUMP_FIELD(_IO_seekpos_t, __seekpos);
    JUMP_FIELD(_IO_setbuf_t, __setbuf);
    JUMP_FIELD(_IO_sync_t, __sync);
    JUMP_FIELD(_IO_doallocate_t, __doallocate);
    JUMP_FIELD(_IO_read_t, __read);
    JUMP_FIELD(_IO_write_t, __write);
    JUMP_FIELD(_IO_seek_t, __seek);
    JUMP_FIELD(_IO_close_t, __close);
    JUMP_FIELD(_IO_stat_t, __stat);
    JUMP_FIELD(_IO_showmanyc_t, __showmanyc);
    JUMP_FIELD(_IO_imbue_t, __imbue);
#if 0
    get_column;
    set_column;
#endif
};
```

이것들은 read, write, close 등 여러 함수로 JUMP 하는데, 즉 여기 있는 포인터들중 원하는 함수를 덮어준다면,

그 포인터는 원하는 함수로 JUMP 할 것이다! (exploit 하는데 생각 해 볼수 있는 방법!)

참고로 **fread** 함수를 호출할 때 아래와 같은 코드는 vtable의 내부 함수 포인터를 이용해

저수준 동작이 수행됨을 보여준다.

```c
#define fread(p, m, n, s) _IO_fread (p, m, n, s)
size_t
_IO_fread (void *buf, size_t size, size_t count, FILE *fp) {
    ...
    bytes_read = _IO_sgetn (fp, (char *) buf, bytes_requested);
    ...
}
size_t
_IO_sgetn (FILE *fp, void *data, size_t n)
{
  /* FIXME handle putback buffer here! */
  return _IO_XSGETN (fp, data, n);
}
#define _IO_XSGETN(FP, DATA, N) JUMP2 (__xsgetn, FP, DATA, N)
#define JUMP2(FUNC, THIS, X1, X2) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1, X2)
# define _IO_JUMPS_FUNC(THIS) (IO_validate_vtable (_IO_JUMPS_FILE_plus (THIS)))
```



## _IO_flush_all_lockp Exploit

```c
#define fflush(s) _IO_flush_all_lockp (0)
 
fp = (_IO_FILE *) _IO_list_all;
while (fp != NULL)
{
      run_fp = fp;
      if (do_lock)
    _IO_flockfile (fp);
 
      if (((fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)
#if defined _LIBC || defined _GLIBCPP_USE_WCHAR_T
       || (_IO_vtable_offset (fp) == 0
           && fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr
                    > fp->_wide_data->_IO_write_base))
#endif
       )
      && _IO_OVERFLOW (fp, EOF) == EOF)
    result = EOF;
 
      if (do_lock)
    _IO_funlockfile (fp);
      run_fp = NULL;
 
      if (last_stamp != _IO_list_all_stamp)
      {
      /* Something was added to the list.  Start all over again.  */
      fp = (_IO_FILE *) _IO_list_all;
      last_stamp = _IO_list_all_stamp;
      }
      else
    fp = fp->_chain;
}
```

모든 _IO_FILE 의 구조는  _chain 변수로 인해 링크드 리스트로 목록이 관리된다. 이 링크드 리스트의 헤더는 _IO_list_all 인데, 이 공격 방법의 핵심은, _IO_list_all 을 후킹해 _IO_OVER_FLOW 를 적절히 덮는 것 이다. 실제 fflush 는 _IO_flush_all_lockp 로 매크로 선언 되어있다.

_IO_flush_all_lockp 는 다음과 같은 3가지 경우에서 호출 된다.

1. 라이브러리가 중단된 프로세스를 실행 할 때
2. exit 함수를 호출 할 때
3. 실행 흐름이 main 함수에서 return 될 때

_IO_OVERFLOW 를 실행 하기 위해선, 다음 조건 중 하나를 만족해야한다.

```c
(fp->_mode <= 0 && fp->_IO_write_ptr > fp->_IO_write_base)
```

or

```c
(_IO_vtable_offset (fp) == 0 && fp->_mode > 0 && (fp->_wide_data->_IO_write_ptr > fp>_wide_data->_IO_write_base)
```

이 조건을 맞춰주고, 적절히 _IO_OVERFLOW 를 원하는 함수로 덮어준다면 그 함수가 실행 될 것이다.

FSOP 기법을 더 잘 이해하기 위해, 독자들은 스스로 fread, fopen, fclose, fwrite 등의 대한 함수들을 분석하는 것 과 추가로

내부적으로 file stream 을 사용하는 함수들을 어떻게 하면 exploit 할 수 있을지 생각해보자.

+추가로 버전 마다 다른 익스플로잇 방법이 있으니 그것도 알아보는것이 좋을 것 이다.

## Recommend Challenge

### http://pwn.0x0.site/ - babyfsop

_IO_flush_all_lockp 를 exploit 해보는 문제

### https://pwnable.tw/ - seethefile

fake file pointer struct 를 만들어서 _IO_FILE_plus 를 잘 변조해주는 문제

### http://pwnable.xyz/ - fclose 

fclose 의 내부 동작 과정을 알고 exploit(어떻게 익스플로잇 할까?) 하는 문제

### HITCON 2016 - house of orange 

약간의 힙 지식을 가지고 File Stream Oritend Programming 기법을 사용하는 문제

## Refrence

https://dangokyo.me/2018/01/01/advanced-heap-exploitation-file-stream-oriented-programming/

https://ctf-wiki.github.io/ctf-wiki/pwn/linux/io_file/fsop/

https://www.slideshare.net/AngelBoy1/play-with-file-structure-yet-another-binary-exploit-technique





