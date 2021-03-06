# fd - 1 pt

## 문제

```text
Mommy! what is a file descriptor in Linux?

* try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link:
https://youtu.be/971eZhMHQQw

ssh fd@pwnable.kr -p2222 (pw:guest)
```

## 풀이

일단 `fd`의 소스코드를 보자.

```c
fd@ubuntu:~$ cat fd.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
        if(argc<2){
                printf("pass argv[1] a number\n");
                return 0;
        }
        int fd = atoi( argv[1] ) - 0x1234;
        int len = 0;
        len = read(fd, buf, 32);
        if(!strcmp("LETMEWIN\n", buf)){
                printf("good job :)\n");
                system("/bin/cat flag");
                exit(0);
        }
        printf("learn about Linux file IO\n");
        return 0;

}
```

`argv`로 받은 숫자를 `read`함수에 넣는 파일 디스크립터로 사용한다. 어떻게 저렇게 해서 `read`로 읽은 데이터가 "LETMEWIN\n"이면 `flag`파일 내용을 출력하도록 되어있다.

유닉스 시스템에선 모든 것을 파일로 관리를 하는데, 프로세스에서 이 파일에 접근할 때 파일 디스크립터를 사용한다. 파일 디스크립터는 0과 양의 정수값을 갖는데, 프로세스가 파일을 사용하기 위해 `open`하면 해당 프로세스의 파일 디스크립터 숫자 중에 사용하지 않는 가장 작은 값을 할당한다.

> int fd = open( pathname, flags, mode )
>
>  pathname 이 가리키는 파일을 열고 열린 파일을 이후 호출에서 참조 할 때 쓸 파일 디스크립터를 리턴.
>
>  flags는 파일을 읽기, 쓰기, 둘다를 위해 열지를 지정 한다.
>
> 출처: http://dev-ahn.tistory.com/96 [Developer Ahn]

| 파일디스크립터  | 목적          | POSIX 이름   | stdio 스트림 |
|-------------- |:--------------:| -----:| ----- |
|  0  | 표준 입력 | STDIN_FILENO  | stdin |
|  1  | 표준 출력 | STDOUT_FILENO | stdout |
|  2  | 표준 에러 | STDERR_FILENO | stderr |

쉘에서 프로세스를 시작하면 해당 프로세스는 쉘의 파일 디스크립터 테이블의 복사본을 상속받게 되는데, 쉘은 기본적으로 위의 3개 디스크립터를 열어놓는다. 한마디로 `read`함수의 디스크립터 매개변수에 `0`을 넘겨주면 표준 입력 디스크립터를 읽게 된다는 것이다.

```c
int fd = atoi( argv[1] ) - 0x1234;
int len = 0;
len = read(fd, buf, 32);
if(!strcmp("LETMEWIN\n", buf)){
        printf("good job :)\n");
        system("/bin/cat flag");
        exit(0);
}
```

이래저래 해서 아무튼 `flag`를 얻기 위해선 `if(!strcmp("LETMEWIN\n", buf))`을 통과해야 한다. 여기서 우린 fd를 `stdin`으로 만들고 직접 콘솔에서 입력한 값을 `buf`에 넣도록 할 것이다.

```c
int fd = atoi( argv[1] ) - 0x1234;
```

`fd`가 `stdin`을 뜻하는 `0`이 되기 위해선 `atoi(argv[1])-0x1234`가 `0`이 되어야 하므로 로그램의 인자로 4660을 넘겨준다. 그렇게 되면 `read`함수에선 표준입력에서 데이터를 가져와 `buf`에 넣는 작업을 하게 된다. 콘솔에서 "LETMEWIN"을 입력하고 엔터를 누르면 된다.

## 레퍼런스와 출처

- ["지맨네집 :: 파일디스크립터.."](http://z-man.tistory.com/151)