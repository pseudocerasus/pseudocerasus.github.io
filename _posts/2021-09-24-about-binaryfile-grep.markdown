---
layout: post
title:  "특정 log 파일이 grep 사용시 binary file로 인식되는 현상에 대하여"
date:   2021-09-24 15:21:11 +0900
---

text 파일인 log 파일을 grep으로 확인하려고 했는데, binary 파일로 인식 된다는 글을 읽었다.

>이유를 알 수 없지만 "xxx.log" 파일을 grep 하면 binary 라고 읽을 수 없다는 메시지가 뜬다.<br>
하지만 다른 파일들은 잘 읽힘.<br>
이 문제를 해결하기 위해서 grep에 -a 옵션을 추가함.<br>

그래서 이 현상을 설명해 보려고 한다.<br>
<br>
<br>

---

CRLF로 문장이 구분되지 않거나, ascii 또는 문자열 인코딩이 아닌 데이터가 있는 파일을 binary 파일로 인식합니다.<br>
간혹 Log를 출력하는 프로그램쪽에서 한 라인에 0x00만 채워 넣던가 하면 해당 파일이 대부분 문자열로 인코딩 된 파일이지만,특정 라인 때문에 전체적으로 data file로 인식될 수 있고, 이 경우 grep은 이를 binary 파일로 일단 간주합니다.
<br>
<br>
아래 예를 참고해 보세요.

```bash
$ vi helloworld.log

This is a text file
Hello World
```
위와 같이 두줄을 적어 보세요. 그리고 저장하세요.<br>
당연히 ascii 파일입니다. 확인해 보죠.<br>
```bash
$ file helloworld.log
helloworld.log: ASCII text
```

이제 이 파일에 ascii 코드값 또는 문자열 인코딩 값이 아닌 다른 값을 넣어 보죠.<br>
vi에서 hex code로 파일을 보고 편집할 수 있습니다.<br>
0x00을 채워 보겠습니다.<br>

```bash
$ vi helloworld.log

vi가 떠 있는 상태에서,

:%!xxd

를 입력하면 hex 모드로 보입니다. 아래처럼 말이죠. 아참, xxd가 설치되어 있어야 합니다. sudo apt-get install xxd

00000000: 5468 6973 2069 7320 6120 7465 7874 2066  This is a text f
00000010: 696c 650a 4865 6c6c 6f20 576f 726c 640a  ile.Hello World.


hex로 보이는 값을 잘 보면, 두번째줄 끝에 0a 가 보일 겁니다.
0a 는 ascii 코드로 LF 즉, line feed, new line이죠.
네. hex 코드로 잘 보이네요. 이제 이 파일을 vi로 hex 모드로 열어 놓은 상태에서 아래와 같이 한줄 더 추가해 보세요.

00000000: 5468 6973 2069 7320 6120 7465 7874 2066  This is a text f
00000010: 696c 650a 4865 6c6c 6f20 576f 726c 640a  ile.Hello World.
00000020: 0000 0000 0000 0000 0000 0000 0000 0000  ................

그리고 vi 에서
:%!xxd -r 을 입력하면 다시 hex 모드가 아닌 문자열 모드로 보일 겁니다.
아래처럼 말이죠.

This is a text file
Hello World
^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@^@

이상한 문자가 보이네요. 보통 바이너리 파일을 vi로 열었을때 보이는 그 문자들 ^@ 이 보이는군요.
어쨌든 저장하고 나가보죠.
```

예상하듯이 이 파일은 이제 data file로 간주 됩니다.
```bash
$ file helloworld.log
helloworld.log: data
```

data file (binary file)을 만들었는데, cat으로 읽어보면 어떻게 나올까요?
```bash
$ cat helloworld.log
This is a text file
Hello World

```

0x00 데이터 부분은 그냥 보여주지 않고 출력됐습니다. cat으로 보면, 그냥 New Line만 한 줄 들어간 것 처럼 보일 뿐이죠.<br>
<br>
이제 grep으로 읽어 보죠. 당연히 binary 파일이라고 할 겁니다.
```bash
$ grep "Hello" helloworld.log
Binary file helloworld.log matches
```

그래서 grep 에서 -a 옵션을 제공하고 있습니다.<br>
grep 입장에서 이건 binary file인데 싶어도 binary 파일안에 문자열 값이 포함되어 있다면 그건 찾아줄께 라는 옵션이 -a 인거죠.
```
$ grep -a "Hello" helloworld.log
Hello World
```

문제가 된 로그 파일을 읽어 보면 어딘가 (보통은) 0x00 ^@이 들어가 있을 겁니다.<br>
로그 파일을 출력하는 프로그램이 0x00 등을 남기거나, 혹은 문자열 끝에 0x0a (LF) 0x0d (CR) 등을 남기지 않으면,이 파일은 종종 binary file로 인식되곤 합니다.
