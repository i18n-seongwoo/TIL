# ulimit

```
ulimit [-acdfHlmnpsStuv] [제한값]
```
* ulimit는 쉘 명령어로 시스템 자원의 제한 값을 조정한다. 값은 쉘에서 실행되는 프로세스에 적용된다.
* 리눅스에만 있다.

## 옵션
* -c core file의 최대크기. 
* -d 프로세스 data segment의 최대 크기
* -p pipe(:12) 버퍼의 크기 
* -s 최대 stack 크기 
* -n open 가능한 파일의 개수 
* -f 쉘에서 생성하는 파일의 최대 크기 
* -t 할당받을 수 있는 cpu time 
* -v 프로세스가 할당 받을 수 있는 virtual memory의 크기 
* -a 모든 제한 값을 출력한다.

## 예제
### 오픈 가능한 파일의 개수를 2048로 조정
```
$ ulimit -n       // 현재 오픈 가능한 개수를 확인
1024
$ ulimit -n 2048
$ uliimt -n
2048                // 2048로 바뀌었다.
```

### 전체 목록 조회 
* ulimit -a
```
# ulimit -a
core file size        (blocks, -c) 0
data seg size         (kbytes, -d) unlimited
file size             (blocks, -f) unlimited
max locked memory     (kbytes, -l) unlimited
max memory size       (kbytes, -m) unlimited
open files                    (-n) 1024
pipe size          (512 bytes, -p) 8
stack size            (kbytes, -s) 8192
cpu time             (seconds, -t) unlimited
max user processes            (-u) 4087
virtual memory        (kbytes, -v) unlimited
```
