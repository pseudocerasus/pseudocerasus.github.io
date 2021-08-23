---
layout: post
title:  "Gerrit 권한 설정"
date:   2016-11-03 11:22:06 +0900
---
Gerrit<span style="color:green"><sup> 2.1x 버전</sup></span>의 권한 설정을 설명 합니다.

<br>
<br>
<br>
> **Rights Inherit From**
<br>

Gerrit의 권한 설정은 각 Git Repository(Project※)마다 Access 메뉴에서 합니다.<br>
<span style="color:green"><sup>* Gerrit 에서는 각각의 Git Repository를 Project 라고 부릅니다. 
</sup></span>
<br>
<br>
Access 메뉴에 들어가 보면 가장 먼저 `Rights Inherit From: All-Projects` 라는 것이 보입니다.<br>

이는 다른 프로젝트인 `All-Projects`로부터 Access 권한을 상속 받겠다는 뜻 입니다.<br>

즉, `All-Projects`라는 Git이 Gerrit 서버에는 존재하고 이 Git은 아무런 source code도 가지고 있지 않고 Access 권한 설정만 되어 있는데, 이 `All-Projects`라는 Gerrit Project에 설정되어 있는 권한을 상속 받겠다는 뜻이 됩니다.<br>

보통은 이렇게 사용합니다.<br>

예를 들어서 Alpha라는 프로젝트를 진행하게 되었는데, Alpha는 src, lib, include 등의 Git Project를 가지고 있어야 한다면, 총 3개의 Git Repository가 만들어질 겁니다.<br>
`Alpha/src`<br>
`Alpha/lib`<br>
`Alpha/include`<br>
정도가 될 겁니다.<br>

그러면 Gerrit에서 이 3군데 Project에 모두 Access 설정을 해 주어야 합니다.<br>
3개 정도면 할 수 있을지 모르겠지만 Android Open Source Project 같이 큰 규모의 source code는 Git이 400여개가 넘기 때문에,일일이 Gerrit에서 Access 설정을 할 수 없습니다.<br>

이럴 경우 Gerrit에서 `Alpha-Projects` 라는 Project를 만들어서 권한을 설정해 놓았다고 가정해 보겠습니다.<br>
즉, `Alpha-Projecst`도 Git이기는 한데 아무도 사용하지 않을 것이고 source code도 가지고 있지 않을 겁니다.<br>
단지 Gerrit Access 권한 설정만 상세하게 해 놓았다고 가정하겠습니다.<br>

그럼 이제 `Alpha/src, Alpha/lib, Alpha/include` 와 같은 Project들은 `All-Projects`가 아닌 `Alpha-Projects`를 상속 받는다고 설정하면 됩니다.<br>
즉 이렇게 다른 Git의 권한을 상속 받을 수 있으며, 보통 큰 규모의 프로젝트를 진행할 경우 권한 설정용 Git Project를 하나 만들고, 이 Project를 상속 받도록 하면서 필요한 Git들을 만듭니다.<br>
<br>
<br>
<br>
> **refs/**
<br>

`refs/` 는 Git의 branch와 tag와 같은 정보에 접근하기 위한 path로 생각하면 됩니다.<br>

예를 들어서 `refs/heads/master` 는 master Branch를 의미하고, `refs/tags/v1.0` 은 v1.0 이라는 tag를 의미합니다.<br>
<span style="color:green"><sup>* 보시듯이 차이점은 heads와 tags 입니다. 보시듯이 Branch와 Tag는 heads와 tags 처럼 path (경로)가 다릅니다.</sup></span>

물론 Git 명령어 사용시에 `refs/` 를 직접 사용하는 경우는 드물지만 Git 명령어에서 `refs/` 를 사용하지 않더라도 Git은 암묵적으로 `refs/` 가 생략된 것이라고 생각하고 실제 명령 실행 레벨에서는 `refs/` 를 붙여서 명령을 실행 합니다.<br>

예를 들면, 아래 명령어는 `refs/` 가 보이지 않지만 사실 Git은 `master` 대신에 `refs/heads/master` 라고 해석 합니다.<br>

```bash
$ git push origin HEAD:master
```
<span style="color:green"><sup>* HEAD:master를 Git은 실제로는 HEAD:refs/heads/master 라고 해석 합니다.</sup></span>
