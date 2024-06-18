---
layout: post
title: ssh python 삽질기록
date: 2024-06-17 9:14:00
description: ssh로 host간 jump를 사용해보자
tags: 삽질 sidebar
categories: 개발
giscus_comments: true
related_posts: false
toc:
  sidebar: left
---
powershell script를 통해 remote access를 수행하고<br>
MobaXterm의 기능인 ssh gateway (jump host)를 python으로 구현해봅니다.

#### Overview

 회사에서 서로 다른 사용자들이 빌드서버에 접속 / 공용 리모트 컴퓨터(Windows)에 접속해서 테스트할 일이 꽤 많은데,
빌드서버에서 빌드 이미지를 개인 PC로 가져오는 것도 귀찮고, 리모트 컴퓨터에 접속하면 상대방이 튕기는 여러 불편함이 많아
이를 해결하기 위한 툴을 만들었다.

 해당 포스트에서 소개하는 내용은,
 1. JSON으로 개인 보안 정보가 담긴 외부 파일을 만들어 각자 관리하고, (MyConfig.json)
 2. 파이썬을 통해 SSH server가 설치된 ubuntu 빌드 서버에 접속, 파일 가져오기 그리고 가져온 파일을 공유 폴더에 올리고, (sshClient.py)
 3. Powershell을 통해 remote desktop에 명령을 내린다. 명령은 remote desktop에 있는 test.py를 실행하기 위한 목적.  (command.ps1)
 4. remote desktop의 test.py은 연결된 target device로 공유 폴더의 바이너리 파일을 복사 후 실행한다. (test.py)

간단히 도식화하면 아래와 같다.
<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/2024-06-18-ssh_experiment_overview.JPG" class="img-fluid rounded z-depth-1" %}
</div>


#### [1] MyConfig.json
`MyConfig.json`이라는 파일을 아래와 같이 생성
```JSON
{
  "config":{
    "User": "YOUR_ACCOUNT_NAME",
    "SSHServer": "YOUR_SSH_IP",
    "PassWord": "YOUR_PASSWORD",
    "BuildImagePath": "YOUR_BUILDIMAGE_PATH",
    "SharedPath" : "YOUR_SHARED_PATH"
  }
}
```

파이썬으로 config정보가 담긴 파일을 로드<br>
(c_XXX등 으로 관리하면 헷갈려서 클래스로 관리하는데 여기서는 귀찮으니까 그냥 c_XXX로 네이밍함)
```python
import json
def readJSON(config_file):
    with open(config_file, "r") as f:
        return json.load(f)

config = readJSON("MyConfig.json")

c_user = config['config']['User']
c_sshServer  = config['config']['SSHServer']
c_passWord   = config['config']['PassWord']
c_buildImagePath = config['config']['BuildImagePath']
c_sharedPath = config['config']['SharedPath']
```

#### [2] sshClient.py
config 정보를 바탕으로 SSH client를 통해 build server(SSH server)에 접속 후 이미지 파일을 공유 폴더로 복사하는 코드

```python
import paramiko

def printTotals(transferred, toBeTransferred):#복사한 image 파일 크기 출력
    if(transferred == toBeTransferred):
        print(f"Transferred {c_buildImagePath.split('/')[-1]} file {transferred/(1024*1024):.2f}MB to {c_sharedPath}")

buildserver_ssh = paramiko.SSHClient()
buildserver_ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
try:
    buildserver_ssh.connect(c_sshServer, username=c_user, password=c_passWord)
    print(f"Connection established ...{c_sshServer}")
    sftp = buildserver_ssh.open_sftp()
    sftp.get(c_buildImagePath,
    c_sharedPath+c_buildImagePath.split('/')[-1], #bulidImage를 shared path에 복사
        callback=printTotals)
except paramiko.AuthenticationException:
    print("Failed to authenticate.")
except Exception as e:
    print(f"An error occurred: {e}")
finally:
    if 'sftp' in locals():
    sftp.close()
    buildserver_ssh.close()
```

#### [3] MyConfig.json
Remote desktop에 명령을 내리는 Powershell script, `command.ps1` <br>
아래 `command.ps1`을 실행하기 위해서는 PsExec.exe 파일이 있어야한다. MS에서 제공하는 utility로 Windows remote access를 지원한다.
다운로드와 자세한 내용은 <a href="https://learn.microsoft.com/en-us/sysinternals/downloads/psexec">MS 내 PsExec 소개 페이지</a>를 참고


```bash
.\PsExec.exe -i \\YOUR_REMOTE_IP -u YOUR_REMOTE_ACCOUNT -p YOUR_PASSWORD cmd /c python3 test.py 2>$null
```

근데 PsExec를 powershell에서 단독으로 사용하면 cmd 결과가 출력되는데, python으로 powershell을 실행하여 위 `command.ps1`을 실행하면 결과가 안나온다.
커맨드 실행 결과가 출력되지 않고 실행 여부가 출력됨 :cry:

 그래서 remote desktop에 있는 test.py를 통해 공유폴더에 log를 남기고, `command.ps1`를 실행한 컴퓨터 쪽에서 log를 확인하는 식으로 활용함.

아래는 python 내에서 powershell을 통해 `command.ps1`을 실행하는 코드<br>
(위 서술한 log가 실시간으로 출력되지 않아, 아래 python은 실행안하고 그냥 powershell로 확인하는 경우가 더 많았음)
```python
import subprocess as sp
try:
    res = sp.Popen(['powershell.exe', "YOURPATH\\command.ps1", ], shell=True, stdout=sp.PIPE, stderr=sp.PIPE)
    for line in res.stderr:
        print(line)
    for line in res.stdout:
        print(line)
except Exception as e:
        print(f"Exception {e}")
```

#### [4] test.py
마지막 remote desktop내에서 타겟 디바이스의 ssh--> ssh --> 바이너리 실행하는 파이썬 코드
```python
import paramiko
import os

def upload_directory(sftp, local_dir, device_dir):
    for filename in os.listdir(local_dir):
        print(os.listdir(local_dir))
        local_path = local_dir + "\\" + filename
        try:
            sftp.chdir(device_dir)
        except IOError:
            sftp.mkdir(device_dir)
            sftp.chdir(device_dir)
        sftp.put(local_path, filename)
        print(f"copied file {local_path}")

ssh = paramiko.SSHClient()
ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())

try:
    ssh.connect('YOUR_SSH1_IP', username='YOUR_SSH1_ID', password='YOUR_SSH1_PW')
    jump_transport = ssh.get_transport()
    channel = jump_transport.open_channel("direct-tcpip", ('YOUR_SSH2_IP', 22), ('YOUR_SSH1_IP',0))
    target_ssh = paramiko.SSHClient()
    target_ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    target_ssh.connect('YOUR_SSH2_IP', username='YOUR_SSH2_ID', password='YOUR_SSH2_PW', sock=channel)
    print("Connection established ...")

    sftp = target_ssh.open_sftp()

    # move TEST_BINARY to TAEGET_DEVICE_PATH
    local_dir = os.getcwd() + "\\TEST_BINARY" # Remote desktop
    device_dir = "/TARGET_DEVICE_PATH"        # PATH in Target device
    upload_directory(sftp, local_dir, device_dir)

    # commands to run on TARGET_DEVICE
    commands = [
        'date',
        'chmod +x /TARGET_DEVICE_PATH/TEST_BINARY',
      #  './TARGET_DEVICE_PATH/TEST_BINARY'
    ]
    for command in commands:
        stdin, stdout, stderr = target_ssh.exec_command(command)
        output = stdout.read().decode().strip()
        print(f'Output of {command}: {output}')
except paramiko.AuthenticationException:
    print("Failed to authenticate.")
except Exception as e:
    print(f"An error occurred: {e}")
finally:
    if 'sftp' in locals():
        sftp.close()
    target_ssh.close()
    ssh.close()
```