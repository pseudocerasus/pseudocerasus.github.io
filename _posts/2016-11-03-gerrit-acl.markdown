---
layout: post
title:  "Gerrit 권한 설정"
date:   2016-11-03 11:22:06 +0900
---
Gerrit<span style="color:grey"><sup> 2.1x 버전</sup></span>의 권한 설정을 설명 합니다.<br>
<br>

> **Rights Inherit From**
<br>

Gerrit의 권한 설정은 개별 Project<span style="color:grey"><sup>\*</sup></span>마다 Access 메뉴에서 합니다.<br>
<span style="color:grey"><sup>* Gerrit 에서는 각각의 Git Repository를 Project 라고 부릅니다. 
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
3개 정도면 할 수 있을지 모르겠지만 Android Open Source Project 같이 큰 규모의 source code는 Git이 600여개가 넘기 때문에, 일일이 Gerrit에서 Access 설정을 할 수 없습니다.<br>

이럴 경우 Gerrit에서 `Alpha-Projects` 라는 Project를 만들어서 권한을 설정해 놓았다고 가정해 보겠습니다.<br>
`Alpha-Projecst`도 Git이기는 한데 아무도 사용하지 않을 것이고 source code도 가지고 있지 않을 겁니다.<br>
단지 Gerrit Access 권한 설정만 상세하게 해 놓았다고 가정하겠습니다.<br>

그럼 이제 `Alpha/src, Alpha/lib, Alpha/include` 와 같은 Project들은 `All-Projects`가 아닌 `Alpha-Projects`를 상속 받는다고 설정하면 됩니다.<br>
이렇게 다른 Git의 권한을 상속 받을 수 있으며, 보통 큰 규모의 프로젝트를 진행할 경우 권한 설정용 Git Project를 하나 만들고, 이 Project를 상속 받도록 하면서 필요한 Git들을 만듭니다.<br>
<br>
<br>
<br>
> **refs/**
<br>

`refs/` 는 Git의 branch와 tag와 같은 정보에 접근하기 위한 path로 생각하면 됩니다.<br>

예를 들어서 `refs/heads/master` 는 master Branch를 의미하고, `refs/tags/v1.0` 은 v1.0 이라는 tag를 의미합니다.<br>

물론 Git 명령어 사용시에 `refs/` 를 직접 사용하는 경우는 드물지만 Git 명령어에서 `refs/` 를 사용하지 않더라도 Git은 암묵적으로 `refs/` 가 생략된 것이라고 생각하고 실제 명령 실행 레벨에서는 `refs/` 를 붙여서 명령을 실행 합니다.<br>

예를 들면, 아래 명령어는 `refs/` 가 보이지 않지만 사실 Git은 `master` 대신에 `refs/heads/master` 라고 해석 합니다.<br>

```bash
$ git push origin HEAD:master
```
<span style="color:grey"><sup>* HEAD:master를 Git은 실제로는 HEAD:refs/heads/master 라고 해석 합니다.</sup></span>

<br>
<br>
<br>
> **Gerrit이 refs/ 를 hooking 합니다.**
<br>

`refs/heads` 와 `refs/tags` 는 Git의 본래의 path 입니다.<br>

Gerrit이라는 시스템은 refs/heads/master 와 같은 Branch에 code가 merge되기 전에 review를 하는 시스템 입니다.<br>
따라서 Gerrit은 refs/heads 에 권한을 주지 않고 다른 어떤 path에 push 권한을 부여한 후 push 들어온 code의 Review가 끝난 후에 refs/heads 에 merge 해야 하는 의무를 가지게 됩니다.<br>

따라서 Gerrit에게는 임시로 code를 올려 놓고 Review 할 수 있는 공간이 필요합니다.<br>
Gerrit이 이렇게 임시로 push를 받아두는 공간(path)은 refs/for 입니다.<br>

Gerrit 권한 설정을 할때 Review용 code push를 허용하기 위해서는 `refs/for/refs/heads/master` 와 같이 적어 두어야 하는데,
이는 아래와 같이 해석해 볼 수 있습니다.<br>

`refs/for/refs/heads/master` 는 Gerrit의 Review를 위한 path인 refs/for 로 code가 들어오는데, 결국 이 code가 가야할 Branch는 refs/heads/master 와 같이 Git의 master Branch로 가야 한다는 뜻으로 해석하면 됩니다.<br>

Gerrit이 refs/for 라는 path로 push가 들어오면 이를 가로채서 Review할 수 있게 사용하고 나중에 refs/heads/ 로 merge 한다. 그리고 이 설정은 refs/for/refs/heads/ 와 같이 하면 된다고 이해하면 됩니다.<br>

<br>
<br>
<br>

> **사용자가 취하려는 Action 마다 권한을 설정할 수 있습니다. (설정 해야 합니다!)**

Gerrit은 read, write 와 같이 단순한 권한 설정이 아닌 이보다 훨씬 복잡한 권한 설정이 가능하게 설계되어 있습니다.
Gerrit을 거쳐서 사용자가 source code를 download(read) 할 수 있는지, push 할 수 있는지, Review해서 점수를
부여할 수 있는 사용자인지, Verify 점수를 부여할 수 있는 사용자인지, Tag를 push할 수 있는 사용자인지 등을
아주 세세하게 설정할 수 있습니다. 실제 Sample을 설정하면서 각각 설명해 보겠습니다.<br>
<br>

```bash
[access "refs/for/refs/heads/master"]
    push = group UI_DEVELOPER
    pushMerge = group UI_DEVELOPER
[access "refs/heads/master"]
    read = group UI_DEVELOPER
    read = group UI_MANAGER
    label-Code-Review = -2..+2 UI_MANAGER
    submit = UI_MANAGER
    submit = UI_DEVELOPER
    push = +force UI_MANAGER
```

위 첫번째 예제는 `refs/for/refs/heads/master`로 즉, master Branch에 code를 merge 하고 싶은데, 그 전에 Review를 받아 볼 수 있는 path에 대한 권한 설정을 의미합니다.<br>

위 두번째 예제는 `refs/heads/master`로 사용자 이름으로 Git의 master Branch에 접근하는 권한을 의미합니다.<br>

이해를 위해서 간단하게 생각해 보면,<br>
`refs/for/refs/heads/master` 는 Review data가 쌓이는 곳이라고 생각하고 권한 설정을 하면 됩니다.<br>
`refs/heads/master` 는 실제 Git으로 전달되는 data가 쌓이는 곳이라고 생각하고 권한 설정을 하면 됩니다.<br>

위 예제를 설명해 보면 다음과 같습니다.<br>
우선 첫번째 예제를 보면 `refs/for/refs/heads/master` 에 push 할 수 있는 그룹으로 UI_DEVELOPER 라는 그룹이 설정되어 있습니다.<br>

그리고 Merge push 라는 걸 할 수 있는 그룹으로도 UI_DEVELOPER 그룹이 설정 되어 있습니다.<br>
지금 시점에서는 무언지 정확히는 모르겠지만 우선은 `refs/for` 로 시작하는 path, 즉 Review를 위해서 code가 쌓이는 공간에,
UI_DEVELOPER 그룹에 속한 사람들은 code를 push 할 수 있다는 것을 의미합니다.<br>
code를 올려야(push 해야) Review를 할 수 있을테니까 필요해 보이는 권한 입니다.<br>

두번째 예제를 보면 `refs/heads` 입니다.<br>
여기에는 read 권한이 설정되어 있습니다. 즉 Git에 merge된 후의 Branch path가 refs/heads 라고 했으니, 일단 Git의 refs/heads/master 라는 Branch에 어떤 코드가 올라가 있더라도 read (clone, pull) 할 수 있는 권한이라고 생각하면 됩니다.<br>

두번째 예제에는 label-Code-Review 가 설정되어 있습니다. -2점부터 +2점까지 Review 점수를 부여할 수 있는 권한인데, Review를 하는 code는 아직 merge되기 전인 Review 위치에 있는 code일테니 `refs/for/refs/heads/master`에 label-Code-Review를 설정하는게 맞을 것 같아 보입니다.<br>

그런데 Gerrit은 Review 권한 설정을 `refs/heads/` 에 하게 되어 있습니다.<br>
실제로 Review한 후에 code가 올라갈 곳이 어딘지를 기준으로 Review 권한을 설정하도록 되어 있다고 생각할 수도 있는데,
어쨌든 조금 혼란스럽긴 합니다.<br>

계속해서 두번째 예제를 보면 submit 이라는 권한이 있는데, 이는 code Review가 끝난 후에 code를 merge 하는 행위를 의미합니다.<br>
그러니 submit은 `refs/heads/` 쪽에 설정하는게 맞겠다는 생각이 듭니다.<br>

두번째 예제의 마지막을 보면 push 권한이 +force 하게 UI_MANAGER 에게 부여되어 있음을 볼 수 있습니다.<br>
이는 UI_MANAGER 그룹에 속한 사용자는 Review 없이 Git의 `refs/heads/master`에 직접<span style="color:grey"><sup>\*</sup></span> push해서 code를 올릴 수 있음을 의미합니다.<br>
<span style="color:grey"><sup>* 또한 git push시에 -f 옵션을 사용할 수도 있습니다</sup></span><br>

이제 Gerrit에서 설정 가능한 권한들을 살펴 보겠습니다.<br>

참고로 구체적인 권한 내용은 Gerrit의 도움말<span style="color:grey"><sup>\*</sup></span>에서 찾아볼 수 있습니다.<br>
<span style="color:grey"><sup>* Gerrit에 로그인 한 후 Documentation > Access Controls 에서 볼 수 있습니다.</sup></span>

<br>
<br>
**Label Verified**<br>

사용자가 Verify 점수를 부여할 수 있는 권한 입니다. `refs/heads` 에 설정하면 됩니다.<br><br>

**Level Code-Review**<br>

사용자가 Review 점수를 부여할 수 있는 권한 입니다. `refs/heads` 에 설정하면 됩니다.<br><br>

**Abandon**<br>

push한 사용자 말고 다른 사용자가 push된 code를 Abandon 처리할 수 있는 권한 입니다. `refs/heads` 에 설정하면 됩니다.<br><br>

**Create Reference**<br>

사용자가 새로운 Branch를 만들 수 있는 권한 입니다. `refs/heads` 에 설정하면 됩니다.<br><br>

**Delete Drafts**<br>

다른 사용자가 올린 Draft를 삭제할 수 있습니다. `refs/heads` 에 설정하면 됩니다.<br>
> <span style="color:gery"><sup>git push origin HEAD:refs/drafts/master 와 같이 push 하면 Review push가 아닌 Draft 버전의 push가 됩니다.<br>
Draft는 특정인들만 code를 보다가 원하는 시점에 Review용 code로 publish 할 수 있습니다. Draft 자체가 이런 용도로 사용 됩니다.</sup></span>

<br><br>

**Edit Topic Name**<br>
git commit 시에 적은 commit message를 Gerrit에서 수정할 수 있습니다.<br>
본인 것은 언제든지 수정 가능하고 이 권한은 다른 사람이 push 한 것에 대해서 commit message를 수정할 수 있습니다.<br><br>

**Forge Author Identity**<br>
Author가 다른 commit을 push 할 수 있습니다. `refs/heads` 에 설정하면 됩니다.<br>
Git은 Author와 Committer가 다를 수 있습니다.<br>
Forge 관련 권한은 3개를 동시에 설정하면 되고, `Create Reference`나 force push 권한 설정시 꼭 함께 설정해야 합니다.<br><br>

**Forge Committer Identity**<br>
Committer가 다른 commit을 push 할 수 있습니다. `refs/heads` 에 설정하면 됩니다.<br><br>

**Forge Server Identity**<br>
인증되지 않은 서버 정보를 갖는 commit을 push 할 수 있습니다. `refs/heads`에 설정하면 됩니다.<br><br>

**Owner**<br>
접근 권한 설정을 변경할 수 있는 권한 입니다.<br><br>

**Publish Drafts**<br>
Drafts로 push 된 것을 Review로 전환(publish) 할 수 있는 권한 입니다. `refs/heads`에 설정하면 됩니다.<br><br>

**Push**<br>
code push 권한 입니다.<br>
`refs/for/refs/heads`에 설정하는 것은 Review용 push를 할 수 있는 것이고,<br>
`refs/heads` 에 설정하는 것은 Review 없이 direct push 할 수 있도록 설정하는 것 입니다.<br><br>

**Push Merge Commit**<br>
push하려는 commit 줄기에 merged commit이 있어도 push 할 수 있게 하는 권한 입니다. `refs/heads`에 설정하면 됩니다.<br><br>

**Push Annotated Tag**<br>
설명 주석을 가지고 있는 tag를 push 할 수 있는 권한 입니다. `refs/heads`에 설정하면 됩니다.<br><br>

**Push Signed Tag**<br>
GPG sign이 되어 있는 tag를 push 할 수 있는 권한 입니다. `refs/heads`에 설정하면 됩니다.<br><br>

**Read**<br>
clone, pull 등 code를 download 할 수 있고 Gerrit에서 해당 Git(Project)을 볼 수 있는 권한 입니다. `refs/*` 으로 설정하는 게 좋습니다.<br><br>

**Rebase**<br>
다른 사람의 code를 rebase 할 수 있습니다. 많이 사용하지 않습니다. `refs/heads`에 설정하면 됩니다.<br><br>

**Remove Reviewer**<br>
Gerrit에 push한 commit에 설정되어 있는 Reviewer를 삭제할 수 있습니다. `refs/heads`에 설정하면 됩니다.<br><br>

**Submit**<br>
Review가 완료된 code를 Git에 merge 하기 위한 권한 입니다. `refs/heads`에 설정하면 됩니다.<br><br>

**View Draft**<br>
누군가 Draft push를 했을때 draft code를 볼 수 있는 권한 입니다. `refs/heads`에 설정하면 됩니다.<br>