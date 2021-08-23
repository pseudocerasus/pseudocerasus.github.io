---
layout: post
title:  "repo init/sync에 대하여"
date:   2015-01-07 13:11:38 +0900
---
여러 곳의 git저장소를 모아서 소스트리<span style="color:grey"><sup>source tree</sup></span>를 구성하고 싶을때, 우리는 쉽게 [repo](https://source.android.com/setup/develop?hl=ko#installing-repo)를 사용하고 manifest.xml 파일로 버전을 관리합니다.

repo init, repo sync는 어떻게 동작하는지 이야기 해 보려고 합니다.<br>
<br>
<br>
<br>
> **repo/manifest.xml**
<br>

repo를 사용하면 다음과 같이 `repo init` 부터 하게 됩니다. <br>
`$ repo init -u ssh://source.epicycle.info:29418/platform/manifest.git -b master` <br>
이렇게 repo init을 하면 `.repo` 라는 폴더가 생기는데, 여기에는 어떤 정보가 저장되어 있는지 살펴 보겠습니다.<br>

```bash
$ cd .repo
$ ls -al
total 20
drwxrwxr-x 5 pi.epicycle pi.epicycle 4096  1월 27 21:52 .
drwxrwxr-x 3 pi.epicycle pi.epicycle 4096  1월 27 21:51 ..
lrwxrwxrwx 1 pi.epicycle pi.epicycle   21  1월 27 21:52 manifest.xml -> manifests/default.xml
drwxrwxr-x 3 pi.epicycle pi.epicycle 4096  1월 27 21:52 manifests
drwxrwxr-x 9 pi.epicycle pi.epicycle 4096  1월 27 21:52 manifests.git
drwxrwxr-x 7 pi.epicycle pi.epicycle 4096  1월 27 21:52 repo
```

`manifest.xml -> manifests/default.xml`<br>
symbolic link 인데, 하위 폴더인 `manifests`안의 `default.xml` 파일을 가리키고 있습니다.<br>
`manifests`는 `default.xml`을 포함하여 각종 버전의 xml 파일이 모여 있는 폴더 입니다.<br>
`manifests.git`은 서버의 `manifest.git` 을 Bare<span style="color:grey"><sup>서버쪽 형상</sup></span> 형태로 clone해 온 저장소 입니다.<br>
서버쪽 저장소를 그대로 복사해서 가져다 놓은 것으로 이해하시면 됩니다.<br>

우리가 이 폴더 안에서 주목해야 할 것은 repo init을 할때 어떤 Branch 이름을 이용했는지 찾을 수 있다는 사실입니다.<span style="color:grey"><sup>뒤에서 설명합니다.</sup></span><br>
여기서는 `manifest.xml` 링크와 `manifests.git` 폴더가 있다는 것만 보고 넘어가겠습니다.<br>
   <br>
   <br>
   <br>
> **repo Launcher**
<br>

repo툴<span style="color:grey"><sup>Launcher</sup></span>은 repo init 명령을 통해서 google이 배포한 `git-repo.git`을 clone 합니다.<br>

`git-repo`를 clone해서 보면, 그 안에는 repo를 동작시키는 실제 script들이 있습니다.<br>

repo init 명령을 통해서 생성된 `.repo` 폴더안에는 repo 폴더가 있는데, 이 폴더는 google이 배포한 `git-repo.git`을 clone해 온 것이고, 이 폴더에 실제 repo 도구<span style="color:grey"><sup>script</sup></span>가 모두 모여 있습니다.<br>

설정에 따라서는 `git-repo.git`을 회사내에 미러링해 두고, repo init시에 사내 미러를 바라보고 repo launcher가 동작하게 할 수도 있습니다.<span style="color:grey"><sup>--repo-url 을 찾아보세요</sup></span><br>

보안이나 방화벽 문제, 버전 호환성 문제가 있을때 이런 방법을 사용합니다.<br>

참고로 repo 폴더에 들어가서 `remote` 정보를 보면 일반적으로 아래와 같이 나옵니다.<br>
```bash
$ cd .repo/repo
$ git remote -v
origin  https://gerrit.googlesource.com/git-repo (fetch)
origin  https://gerrit.googlesource.com/git-repo (push)
```
보시듯이 [https://gerrit.googlesource.com](https://gerrit.googlesource.com) 서버에 가서 `git-repo` 라는 것을 가져온 것임을 알 수 있습니다.<br>

   일단 .repo 안에 무엇이 들어 있는지 살펴 보았습니다.<br>
   좀 더 구체적으로 살펴 보겠습니다.<br>
   <br>
<br>
<br>
> **manifest.xml**

`repo sync` 명령이 들어오면 `repo`는 `manifest.xml`을 참고해서 소스트리를 다운로드 합니다.<br>
보통은 `manifest.xml -> manifests/default.xml` 과 같이 `default.xml` 을 가리키고 있습니다.<br>

그런데, 우리는 간혹 Nightly Build로 Tag를 붙이고 이 특정 버전의 xml 파일을 저장하곤 합니다.<br>
그 특정 버전의 xml 파일을 가리키게 symbolic link를 변경하면 어떻게 될까요?<br>
`manifest.xml` 의 link 위치를 바꿔서 Nightly 버전을 참고하도록 강제로 바꿀 수 있을까요?<br>
예. 변경할 수 있습니다.<br>
<br>
예를 들어 보겠습니다.<br>
```bash
$ cd manifests
$ ls -a
NIGHTLY_201308300003.xml  NIGHTLY_201309121202.xml  NIGHTLY_201309271802.xml
NIGHTLY_201308301203.xml  NIGHTLY_201309131203.xml  default.xml  release.xml  .git
```

`manifest.xml -> manifests/default.xml` 는 이 폴더 안에 있는 `default.xml` 파일을 참고하고 있다는 의미입니다.<br>

위 예제에서 보이듯이 `manifests` 폴더안에는 다양한 `xml` 파일들이 있습니다.<br>
    그럼 symbolic link를 변경하면 어떻게 될까요?<br>

```bash
# modify the manifest.xml
$ cd .repo
$ ls -al
total 28
drwxrwxr-x 5 pi.epicycle pi.epicycle  4096  1월 27 21:53 .
drwxrwxr-x 3 pi.epicycle pi.epicycle  4096  1월 27 21:53 ..
lrwxrwxrwx 1 pi.epicycle pi.epicycle    21  1월 27 21:53 manifest.xml -> manifests/default.xml
drwxrwxr-x 3 pi.epicycle pi.epicycle 12288  1월 27 21:53 manifests
drwxrwxr-x 9 pi.epicycle pi.epicycle  4096  1월 27 21:53 manifests.git
drwxrwxr-x 7 pi.epicycle pi.epicycle  4096  1월 27 21:53 repo

$ rm manifest.xml
$ ln -s manifests/gh14_release_branch.xml manifest.xml
$ ls -al
total 28
drwxrwxr-x 5 pi.epicycle pi.epicycle  4096  1월 27 22:24 .
drwxrwxr-x 3 pi.epicycle pi.epicycle  4096  1월 27 21:53 ..
lrwxrwxrwx 1 pi.epicycle pi.epicycle    33  1월 27 22:24 manifest.xml -> manifests/NIGHTLY_201309271802.xml
drwxrwxr-x 3 pi.epicycle pi.epicycle 12288  1월 27 21:53 manifests
drwxrwxr-x 9 pi.epicycle pi.epicycle  4096  1월 27 21:53 manifests.git
drwxrwxr-x 7 pi.epicycle pi.epicycle  4096  1월 27 21:53 repo
```
위에서 보듯이 `manifest.xml` 파일을 삭제하고 `manifests/NIGHTLY_201309271802.xml` 을 바라보도록 symbolic link를 새롭게 생성했습니다.<br>

그러니 이제 이 `.repo`는 `default.xml` 을 바라보지 않고, `NIGHTLY_201309271802.xml` 을 바라보고 있습니다.<br>

그럼 이 상태로 repo sync를 하면 `NIGHTLY_201309271802.xml` 정보대로 Source를 Download 할까요?<br>
예, 맞습니다. `NIGHTLY_201309271802.xml` 정보대로 소스트리를 다운로드 할 겁니다.<br>

하지만 이렇게 직접 symbolic link를 변경하는 방법은 사용하지 않습니다.<br>
대신 repo init 명령어를 사용하죠. 왜냐하면 그것이 툴의 명령어를 사용해서 symbolic link를 정상적으로 변경하는 방법이기 때문입니다.<br>

그럼 이제 `manifest.xml`이 가리키는 파일을 repo init 명령어를 이용해서 `NIGHTLY_201309271802.xml` 을 가리키도록 해 보겠습니다.<br>
```bash

# `-m` 옵션을 주고 `repo init` 을 새로 해 보겠습니다.<br>
# 동일한 workspace에 `.repo`가 있는 상태에서 `repo init` 명령을 새로 실행하는 겁니다.<br>
$ repo init -u ssh://source.epicycle.info:29418/platform/manifest -b master -m NIGHTLY_201309271802.xml

$ cd .repo
$ ls -al
total 28
drwxrwxr-x 5 pi.epicycle pi.epicycle  4096  1월 27 22:33 .
drwxrwxr-x 3 pi.epicycle pi.epicycle  4096  1월 27 21:53 ..
lrwxrwxrwx 1 pi.epicycle pi.epicycle    33  1월 27 22:33 manifest.xml -> manifests/NIGHTLY_201309271802.xml
drwxrwxr-x 3 pi.epicycle pi.epicycle 12288  1월 27 22:31 manifests
drwxrwxr-x 9 pi.epicycle pi.epicycle  4096  1월 27 22:33 manifests.git
drwxrwxr-x 7 pi.epicycle pi.epicycle  4096  1월 27 21:53 repo
```
`-m` 옵션에 `NIGHTLY_201309271802.xml` 을 주고 repo init을 새로 했더니, symbolic link가 가리키는 파일이 `NIGHTLY_201309271802.xml`로 변경 되었습니다.<br>

이 상태에서 repo sync를 하면 당연히 `NIGHTLY_201309271802.xml` 를 기준으로 소스트리를 다운로드 합니다.<br>
여기까지 `.repo`안에 있는 `manifest.xml` 파일이 어떤 곳을 가리키는지 살펴 봤습니다.<br>
<br>
<br>
<br>
> **repo init 정보 확인**
 
마지막으로 repo sync한 소스트리가 어떤 Branch 이름으로 repo init 한 것인지 살펴 보겠습니다.<br>
`.repo` 폴더 안에는 `manifests.git` 폴더가 있습니다.<br>

위에서 간략하게 언급했듯이 이 폴더는 git 서버가 가지고 있는 `manifests.git` 이라는 폴더를 통째로 clone해 놓은 것이라고 이해하시면 됩니다.<br>

`manifests.git` 폴더 안에는 repo init할 때 어떤 Branch 이름으로 `-b` 옵션을 주었는지 기록되어 있습니다.<br>

```bash
$ cat .repo/manifests.git/config
[core]
        repositoryformatversion = 0
        filemode = true
[remote "origin"]
        url = ssh://source.epicycle.info:29418/platform/manifest.git
        fetch = +refs/heads/*:refs/remotes/origin/*
[branch "default"]
        remote = origin
        merge = master
```
보시듯이 `manifests.git/config` 파일안에는<br>

이 `manifest` git을 어디에서 가져온 것인지 `url` 이 있습니다.<br>

그리고 merge 부분에 repo init할 때 사용한 Branch 이름이 보입니다.<br>

위 예문을 통해서 `ssh://source.epicycle.info:29418/platform/manifest.git`가 manifest가 있는 git이고, repo init할 때 Branch는 `master`였음을 알 수 있습니다.<br>

그리고 `.repo/manifest.xml` symbolic link가 가리키는 xml파일에 소스트리를 구성하는 git들의 버전이 적혀 있습니다.
