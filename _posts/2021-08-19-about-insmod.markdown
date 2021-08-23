---
layout: post
title:  "insmod가 가능해야 한다는 문의에 대하여"
date:   2021-08-19 15:06:10 +0900
---
Linux kernel은 모듈 프로그래밍을 해서 kernel에 인스톨 해볼 수 있습니다.

c 언어로 kernel에서 동작시킬 프로그램을 작성하고, 컴파일해서 생성한 kernel 모듈을 인스톨할 때 사용하는 명령이 insmod 입니다.

보통 c 언어로 프로그램을 만들어서 라이브러리로 만들면, .so 가 만들어지고 실행파일로 만들면 executable 가능한 실행 파일이 만들어지죠.

linux kernel의 module은 확장자 .ko 를 갖습니다.

예를 들어서 코드를 작성한 후에 hello.ko 를 컴파일해서 만들어내고,

```bash
$ sudo insmod hello.ko 하면 인스톨 되는 겁니다.
```

아래와 같은 시험을 해 보세요.<span style="color:grey"><sup>Docker image는 ubuntu 18.04 정도면 충분합니다</sup></span>

* Docker Container를 하나 실행해서 insmod가 동작하는지 확인해 보세요.
* Container를 실행할때 docker run 옵션으로 `--privileged` 를 주고 실행하면 어떻게 되는지 확인해 보세요.

커널 개발을 하려면, kernel header 파일들을 가지고 있어야 합니다.

그리고 insmod 도 툴인데, kernel 개발을 하는 서버가 아니면 설치가 안되어 있을 수도 있으니, kmod 라는 패키지를 설치해야 합니다.

```bash
$ sudo apt-get install linux-headers-$(uname -r)
$ sudo apt-get install kmod
```

아래는 hello.c 와 Makefile 예제입니다.

Makefile은 명령어 시작할때, 반드시 제일 앞에 탭이 들어가야 함을 기억하세요.

```c
//FILE NAME : hello.c

#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

static int __init init_hello(void){
        printk(KERN_ALERT "Hello, my kernel!\n");
        return 0;
}


static void __exit exit_hello(void){
        printk(KERN_ALERT "Good-bye, my kernel!\n");
}

module_init(init_hello);
module_exit(exit_hello);
MODULE_LICENSE("GPL");
```

```makefile
#FILE NAME : Makefile

obj-m = hello.o
KDIR := /lib/modules/$(shell uname -r)/build
PWD := $(shell pwd)

default:
	$(MAKE) -C $(KDIR) SUBDIRS=$(PWD) modules

clean:
	rm -rf *.kr *.mod.* .*.cmd *.o
```


두개의 파일을 넣어 두고,
```bash
$ make
```
하면 빌드가 되서 ko 파일이 생겨야 하고,

```bash
$ sudo insmod hello.ko
$ lsmod | grep hello
$ sudo rmmod hello
```
를 해 보고,

```bash
$ dmesg
```
로 kernel 로그를 확인해 보면 됩니다.
