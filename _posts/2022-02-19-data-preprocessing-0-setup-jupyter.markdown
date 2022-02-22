---
layout: post
title:  "Data Preprocessing - Jupyter notebook 실습환경 만들기"
date:   2022-02-19 13:01:05 +0900

---
Jupyter notebook을 이용한 실습 환경 만들기

>1. docker image 'continuumio/miniconda3' 로  jupyter notebook 실습 환경을 만든다.
>2. 실습에 필요한 jupyter notebook 단축키를 익힌다.
>3. Data preprocessing 실습에 필요한 python module을 추가 설치한다.

<br>
### Jupyter notebook 도커 컨테이너로 실행하기

[miniconda image](https://hub.docker.com/r/continuumio/miniconda3)를 사용한다.
```bash
$ docker pull continuumio/miniconda3
```
도커 이미지를 pull 했으면 적절한 옵션을 주고 실행하면 된다.
아래와 같이 실행한다.
```bash
docker run -i -t -v $(pwd)/notebooks:/opt/notebooks \
--name jupyter_notebook \
-p 8888:8888 continuumio/miniconda3 /bin/bash \
-c "/opt/conda/bin/conda install jupyter -y --quiet && mkdir -p \
/opt/notebooks && /opt/conda/bin/jupyter notebook \
--notebook-dir=/opt/notebooks --ip='*' --port=8888 \
--no-browser --allow-root"
```

위 명령으로 docker container가 실행되면 아래와 같은 화면을 볼 수 있다.
```bash
Preparing transaction: ...working... done
Verifying transaction: ...working... done
Executing transaction: ...working... done
[I 10:49:00.169 NotebookApp] Writing notebook server cookie secret to /root/.local/share/jupyter/runtime/notebook_cookie_secret
[W 10:49:00.498 NotebookApp] WARNING: The notebook server is listening on all IP addresses and not using encryption. This is not recommended.
[I 10:49:00.509 NotebookApp] Serving notebooks from local directory: /opt/notebooks
[I 10:49:00.509 NotebookApp] Jupyter Notebook 6.4.8 is running at:
[I 10:49:00.509 NotebookApp] http://9024eec70122:8888/?token=898dksie8dkfow9ekskdfheys8dif9eosju08df97a6f072888
[I 10:49:00.510 NotebookApp]  or http://127.0.0.1:8888/?token=898dksie8dkfow9ekskdfheys8dif9eosju08df97a6f072888
[I 10:49:00.510 NotebookApp] Use Control-C to stop this server and shut down all kernels (twice to skip confirmation).
[C 10:49:00.515 NotebookApp]

    To access the notebook, open this file in a browser:
        file:///root/.local/share/jupyter/runtime/nbserver-1-open.html
    Or copy and paste one of these URLs:
        http://98765soei928342:8888/?token=898dksie8dkfow9ekskdfheys8dif9eosju08df97a6f072888
     or http://127.0.0.1:8888/?token=898dksie8dkfow9ekskdfheys8dif9eosju08df97a6f072888
```

위 상태가 되었다면 docker container로 jupyter notebook이 실행된 것이다.

`Ctrl + p + q` 키를 누르면 docker container를 계속 실행하게 두고 화면에서 빠져나와<sup>docker container detached</sup> 원래 shell로 돌아간다.

원래 shell로 돌아왔다면, 아래와 같이 docker container에 interactive 모드로 shell 다이빙을 한다.
```bash
$ docker exec -it jupyter_notebook /bin/bash
```
shell 다이빙이 되면 아래와 같은 모습을 볼 수 있다.
```bash
(base) root@90cc0896776d:/#
```
앞으로 쓰게 될테니, 우선 vim을 설치해 두도록 한다.

```bash
(base) root@90cc0896776d:/# apt-get update && apt-get install vim
```

<br>
### Jupyter notebook password 설정

우선 jupyter notebook이 설치된 docker container 에 root로 접속되어<sup>shell 다이빙 되어</sup> 있는 현재 상태에서 jupyter notebook config 파일을 생성한다.
```bash
(base) root@90cc0896776d:/# jupyter notebook --generate-config
```

아래 위치에 config 파일이 생성된다.
```bash
(base) root@90cc0896776d:/# ls -al ~/.jupyter/jupyter_notebook_config.py
-rw-r--r-- 1 root root 54275 Feb 19 03:24 /root/.jupyter/jupyter_notebook_config.py
```

config 파일을 생성해 두고, 다음으로 jupyter notebook에 password를 설정한다.
```bash
(base) root@90cc0896776d:/# jupyter notebook password
Enter password:
Verify password:
[NotebookPasswordApp] Wrote hashed password to /root/.jupyter/jupyter_notebook_config.json
```

Password가 설정되면 위와 같이 jupyter_notebook_config.json 파일에 hashed password가 기록되었다는 메시지를 볼 수 있다.

jupyter_notebook_config.json 파일을 읽어보면 아래와 같이 hashed password가 적혀 있음을 볼 수 있다.
```bash
# cat /root/.jupyter/jupyter_notebook_config.json
{
  "NotebookApp": {
    "password": "argon2:$argon2id$v=19$m=10240,t=10,p=8$9dksiwldoe8rt7djskejfidhqyei348d9ckf3HexTIXygI4tQITxFjYt2qad0MXqpWARIUI"
  }
}(base) root@90cc0896776d:/#
```

위 예제에서 argon2: 로 시작해서 ARIUI로 끝나는 문자열 전체가 hashed password 다.

이 hashed password는 암호를 분실했거나  또는 암호가 유출되었다고 판단될때 비밀번호를 변경하고 로그인 되어 있는 기존 모든 세션을 무효화 시키는데 사용할 수 있다.

암호를 분실했을때 사용할 수 있는 위 hashed password 말고, jupyter notebook config에 설정할 hashed password를 추가로 만들 수 있다. 아래와 같이 만든다.
```bash
(base) root@90cc0896776d:/# ipython
In [1]: from notebook.auth import passwd

In [2]: passwd()
Enter password:
Verify password:
Out[2]: 'argon2:$argon2id$v=19$m=10240,t=10,p=8$n10SX9cuiKI1qCPXFJAVTw$lydrwJg0APOWVMN3muP9bO/dqA9EJInsyHT9YNe5u98'

In [3]:exit
(base) root@90cc0896776d:/#
```
위와 같이  passwd() 로 새로운 password를 준비할 수 있다. 역시 argon2로 시작되는 문자열 전체가 hashed password 값이다.

위에서 두번에 걸쳐서 hashed password를 만들어냈는데, 우선 처음 `$ jupyter notebook password` 로 만들어서 `jupyter_notebook_config.json` 파일에 저장된 hashed password는 암호를 분실했을때 등을 대비한 만약의 경우에 대한 파일이라고 생각하면 된다.

그리고 두번째 `notebook.auth`의 `passwd()`를 이용해서 생성해 낸 hashed password는 이제 jupyter notebook에 로그인할 때 사용할 password 이며, 이 password는 아래 config 파일에 설정한다.

```bash
(base) root@90cc0896776d:/# vi ~/.jupyter/jupyter_notebook_config.py
```
`jupyter_notebook_config.py` 파일을 보면 `#c.NotebookApp.password = ''` 와 같이 주석처리된 부분이 있다. 여기에 passwd()로 생성해 낸 hashed password 값을 적어 준다. 아래 예제와 같다.

```bash
c.NotebookApp.password = 'argon2:$argon2id$v=19$m=10240,t=10,p=8$n10SX9cuiKI1qCPXFJAVTw$lydrwJg0APOWVMN3muP9bO/dqA9EJInsyHT9YNe5u98'
```
이제 2개의 파일에 hashed password가 적혀 있는 상태가 되었다.

첫번째는 암호 분실시에 사용할 수 있는 `jupyter_notebook_config.json` 파일에 기록되어 있는 hashed password 이고, 두번째는 `jupyter_notebook_config.py` 파일에 기록한 hashed password 값이 있다.

이제 jupyter notebook에 로그인할때 두번째 hashed password 문자열을 생성해 낼때 입력했던 password 를 사용하면 된다.

그런데, 이렇게 하고 jupyter notebook에 로그인 하려고 하면 로그인이 안될 수 있다.

jupyter notebook doc을 보면 Json(`jupyter_notebook_config.json`) 파일에 password가 기록되어 있으면 `jupyter_notebook_config.py` 에도 기록한 암호가 동작하지 않을 수 있다고 설명되어 있다.

https://jupyter-notebook.readthedocs.io/en/stable/public_server.html
>Automatic password setup will store the hash in `jupyter_notebook_config.json` while this method stores the hash in `jupyter_notebook_config.py`. The `.json` configuration options take precedence over the `.py` one, thus the manual password may not take effect if the Json file has a password set.

위와 같은 이유로, jupyter_notebook_config.json 파일을 이름을 변경해서 저장해 놓기로 한다.
```bash
(base) root@90cc0896776d:/# cd ~/.jupyter/
(base) root@90cc0896776d:~/.jupyter# mv jupyter_notebook_config.json jupyter_notebook_config.json.bak
```

이제 `.py` 파일에 설정된<sup>두번째로 생성했던</sup> password로 jupyter notebook에 로그인이 될 것이다.

설정이 끝났으니, jupyter notebook을 재시작한다. `Ctrl + p + q` 로 docker container에서 빠져나간 후 아래와 같이 container를 재시작 한다.
```bash
$ docker restart jupyter_notebook
```
 
Jupyter notebook 실행을 8888번 포트로 시켜 놓았다. localhost 혹은 Jupyter notebook이 실행중인 서버에 8888 포트로 접속하면 Jupyter notebook을 볼 수 있다.
 <br><br>
![](/assets/images/jupyter_notebook_login_screen.png)

<br>
<b>Jupyter notebook 기본적인 단축키</b>
아래와 같은 기본적인 단축키를 외워두자.
  
|단축키||내용|
|---|---|---|
|a|:|현재 선택된 셀 위에 새로운 실행 셀을 만든다|
|b|:|현재 선택된 셀 아래에 새로운 실행 셀을 만든다|
|x|:|현재 선택된 셀을 삭제한다|
|m|:|현재 선택된 셀을 markdown 모드로 변경한다|
|y|:|현재 선택된 셀을 python 입력 셀로 변경한다|
|Ctrl+Enter|:|현재 선택된 셀의 내용 실행|

이 정도만 외우고 있으면 Jupyter notebook에서 실습을 하는데 불편함이 없을 것 같다.

<br>
<b>추가 설치할 package</b>

Jupyter notebook의 실행 셀에서 `!`를 붙이고 명령어를 입력하면 Jupyter notebook이 실행되고 있는 서버에 shell 명령으로 전달된다.

예를 들면 `!ls -al ~/.jupyter`  와 같이 입력하면 아래와 같은 화면을 볼 수 있다.

![](/assets/images/jupyter_notebook_command_run_screen.png)

<br>

이제 Jupyter notebook에서 `!명령어`를 입력해서 아래와 같이 몇개의 package를 더 설치하자.

```bash
!apt-get update
```
```bash
!pip install bs4
```
```bash
!pip install seaborn
```
```bash
!pip install folium
```
```bash
!pip install openpyxl
```
```bash
!pip install matplotlib
```
```bash
!pip install --upgrade pandas

!pip install pandas_datareader
```
```bash
!apt-get install -y fonts-nanum
```

이제 Data Preprocessing 실습을 위한 환경이 모두 갖춰졌다.
