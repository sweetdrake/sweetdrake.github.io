---
layout: post
title: 나 혼자 회사에서 웨폰마스터
date: 2024-05-30 9:14:00
description: 사용하고 있는 tool에 대해 소개합니다
tags: tool sidebar
categories: 개발
giscus_comments: true
related_posts: false
toc:
  sidebar: left
---

나 혼자 레벨업, 나 혼자 탑에서 농사 등 나 혼자~ 물에 영감을 받아 제목을 지었습니다.

사용하는 utility tool에 대해서 소개합니다.


## Powertoys :robot:
MS에서 배포하는 툴로 여러 기능이 담겨 있다. 자세한 기능 소개는 <a href="https://learn.microsoft.com/ko-kr/windows/powertoys">Powertoys</a> 공식 사이트 참고.

가장 많이 쓰는 기능은 마우스 유틸리티 기능, Mac에 있는 spotlight 처럼 빠른 실행을 하는 기능, Shortcut guide가 있다.
### 마우스 유틸리티
 마우스 찾기는 날이 갈수록 넓어지는 모니터에서 마우스 찾기를 도와준다.

 또 Teams 같이 화면공유로 회의할 때, 하이라이트 해주는 목적으로도 활용한다. (근데 보는 쪽에서는 화면 공유 딜레이 땜에 렉걸린 줄 아는게 함정)

 마우스 형광펜 기능 또한 화면 녹화 등에 유용하다.

### PowerToys Run
익숙해지면 편한 기능, MAC의 spotlight과 유사하다. Installer를 통해 설치된 전역 파일의 경우 왠만하면 대부분 찾기가 가능한데, 특정 실행파일을 run 창에 띄우고 싶은 경우가 종종 있다. 그런 경우 등록하고 싶은 프로그램의 바로가기를 만든 뒤에 `해당 경로`에 바로가기 파일(shortcut)을 넣어주면 된다.

여러가지 접미사를 두고 실행하는 기능도 있다. run 실행하면 창에 도움말이 뜨니까 몇 번 사용해보다가 쓰는 것만 쓰게 되는데 내 경우 `?? 검색어`를 제일 많이 쓴다. 기능은 웹 브라우저로 검색

여러가지 플러그인도 지원하니 설치하는 재미가 있다.
github emoji도 깔아서 쓰고있는데 github.io블로그에 alias로 이모지를 넣으니 직접 넣는 것보다 깔끔하니 보기 좋다.
- github alias로 emoji 출력 : :earth_asia:
- Raw emoji로 출력 🌏

잡담으로 Windows에서 `Win key` + `.`으로 Emoji 패널을 띄울 수 있는데, 영어 키보드(US)에서만 이모지에 대한 검색이 가능하다. 한글 키보드의 경우 패널 상에 검색하는 기능이 없다.
~~(Windows 11에서는 되는 듯..)~~
 github emoji 깔면 번거롭게 키보드 스위칭(`Win key` + `space`)할 필요가 없다.

### Shortcut guide
Windows 버튼을 누르고 있으면 Windows OS에서 제공하는 단축키와 설명이 적힌 창이 올라온다. 

가끔 단축키가 긴가민가할 때 사용한다.

## KeyExplorer :closed_lock_with_key:
openssl 커맨드로 딥하게 볼 수 있겠지만, 항상 커맨드가 뭐였지 하고 구글링하게 된다.

이 툴은 간단하게 인증서 파일이나 키 파일 열기, 다른 포맷으로 출력하고 싶을 때 주로 사용하는데, GUI도 지원하기 때문에 간단하게 drag-and-drop으로 끌어다 놓기만 하면 알아서 key file의 내용을 띄워준다. 가끔 포맷이 안 맞는 경우가 있어서 이런 경우 openssl로 까보는게 좋다.


## Everything :mag:
Windows OS의 file searching 기능은 너무 느리고 쓰기도 불편하다.

가령 어떤 파일을 찾아야하는데 확장자만 기억한다거나.. 어느 폴더 쯤에 있는지는 아는데 정확히 모른다거나.. 아니면 파일 이름 몇개만 기억한다 든지 하는 상황에서 쓰는 최고의 툴이다. (Unix에 find가 있다면 Windows에는 everything이 있다!!)

정규표현식을 지원하기 때문에 *.pdf 이런식으로 검색해도 되고,

특정 위치에서 파일 찾기는 폴더 우클릭으로 context 메뉴를 열어서 search everything.. 으로 검색하면 해당 폴더를 기준으로 파일 검색을 지원한다. (설치할 때 context 메뉴 추가하기 체크박스가 체크 되어있어야함)

파일 검색 때 제외하고 싶은 경로나 추가하고 싶은 경로 ~~(이제 공유 폴더 다 뒤졌다)~~ 또한 제거/추가 할 수 있다.

다음은 자주 사용하는 tool 단축키
- `Tab`:검색창과 검색한 결과는 이동 가능
- `Enter`: 열고 싶은 파일 열기
- `Ctrl`+`Enter`: 파일의 경로 열기
- `Ctrl`+`Shift`+`C`: Copy full name(path) to clipboard


## Onedrive :cloud:
Onedrive는 사실 SharePoint의 다른 이름이다. 즉, 연결된 SharePoint의(웹) 사이트의 폴더들을 로컬 onedrive에 연동할 수 있다. 
Sharepoint에서 documents 탭으로 이동하여 Add shortcut to Onedrive를 통해 연동할 수 있다. (창이 작으면 ... 아이콘을 클릭하면 보임)

 연동시킨 뒤 everything에서 동기화 시키고 싶은 onedrive의 경로를 탐색하는 범위에 등록하면 SharePoint에 올라간 파일을 쉽게 search할 수 있게 된다. ~~(이제 회사 퇴사자들의 자료를 털어볼까)~~

Sharepoint뿐만 아니라 Teams안에 있는 채널들에 대한 파일들 또한 동기화가 가능하다. Add shortcut to Onedrive를 통해 동기화 시켜두도록 하자.

파일을 실수로 삭제하거나 복원하고 싶을 때는 sharepoint로 접속해 bin에 있는 항목을 살려주거나, 파일 우클릭 후 version history를 통해 복원 가능하다.


## MobaXterm :computer:
여러가지 기능이 있는 터미널 이다. 주로 SSH 목적으로 쓰는데, 가장 편한건 SSH로 접속했을 때 왼쪽에 file explorer 창을 띄워주는 기능이다.
때문에 scp 같은 명령어 없이 drag&drop으로 파일을 손 쉽게 복사/붙여넣기 할 수 있다.

 또 한가지 업무상 ssh를 여러번 접속해야하는 경우가 많은데, 자주 들어가는 ssh에 session에 ID/PW를 저장할 수 있어 편하다. 그리고 ssh gateway목적으로 ssh에 접속한 후 해당 네트워크에서 연결된 다른 ssh로 점프할 수 있는 기능까지 있어 편리하다.

<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/20240530_mobaXterm.JPG" class="img-fluid rounded z-depth-1"%}
</div>

## UMLet :pencil2:

블로그에 올리는 UML은 이 툴을 통해 그리고 있다. UML로 유명한 Enterprise architecture(EA)는 유로이고 또 굉장히 사용자 친화적이지 않기 때문에.. 개인적으로 별로 좋아하는 툴은 아니다. 반면 UMLet는 말이 UML이지 사실 사용자가 맘대로 드로잉이 가능하다. UML을 정확히 알고 쓰는게 아닌 입장에서 이 만한 툴이 없는 것 같다.
 익숙해지면 사용하기 편한 툴이다. (요즘은 VScode extension으로도 지원되던데, 아무래도 오리지널이 편한듯)

 예쁘게 그리려고하면 대신 엄청난 노가다가 필요한게 단점.


## ZoomIt :gem:
MS 소개 페이지에 있을 정도로 유명한 utility tool이다. 기능은 화면 확대, 스크린에 펜슬로 드로잉 가능, 화면 녹화 기능등 다양하다.
이 툴을 통해서 스크린에다가 드로잉 하거나, 글자 등을 쓸 수 있어서 화상으로 일하면서 화면 공유 하는 일이 많은 직종에 강추하는 툴이다.

기본 키 매핑은 Ctrl + 1, Ctrl +2 등으로 되어 있으며, 사용자 메뉴얼은 <a href="https://learn.microsoft.com/ko-kr/sysinternals/downloads/zoomit">MS내 툴 소개 페이지</a>로 대체한다.

자주 쓰는 기능은
- 스크린에 드로잉(`Ctrl`+`2`)
- 드로잉 기능 켜졌을 때 글자 입력(`Shirt`+`t`)
- 칠판 모드 `W` or `K`
- 색 변경 `R, G, B`
- 클립보드에 복사 `Ctrl`+`C` or `Ctrl`+`Shirt`+`C`.

## Windows :heavy_check_mark:

툴의 영역은 아닌데 Windows OS에서 사용하는 꿀 커맨드를 소개한다.
- `Win`+`V`: 복사했던 내용을 히스토리 패널로 불러온다.  이 기능을 알고 나면 절대 이전으로 못 돌아갈 정도로 편한 기능
- `Ctrl`+`Alt`+`V`: 복사한 문자 서식/포맷 없이 붙여 넣기.
- `Shift`+`Enter`: Confluence나 wiki 혹은 MS-word 등에서 엔터는 줄 나눔을 뜻한다. 하지만 여기에 shift를 붙이면 간격 없이 줄 나눔을 할 수 있는데, 이러면 글 쓸 때 굉장히 깔끔해진다.

<div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/20240530_shiftEnter.JPG" class="img-fluid rounded z-depth-1"%}
</div>

- `Ctrl`+`Shift` 누른채로 Drag: 텍스트 중간에서 드래그 하고 싶을 때 사용

<div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/20240530_CtrlShiftDrag.JPG" class="img-fluid rounded z-depth-1"%}
</div>

`Win`+`숫자`: 작업표시줄에 있는 프로그램을 실행시킨다.

## Beyond compare :traffic_light:
파일이나 폴더 자체를 1:1로 비교할 때 사용하는 툴. 아쉽게도 유로이나 왠만하면 회사에서 사준다.

없을 때는 notepad++에서 plugin에 compare로 볼 수 있는데 기능이 다소 아쉬운 편이다.

## VScode :vs:

말이 없는 G.O.A.T :goat:

사용하고 있는 extension으로는
- project manager: workspace 관리에 좋다
- git graph: git terminal 커맨드로 고통받는 스타일이 아니기 때문에 이툴로 git을 관리한다
- remote-ssh: 매 번 SSH 접속할 때 ID/PW 입력에 고통받는 다면 꼭 ID/PW 설정을 해주도록 하자
- copilot: 잘 쓰면 신세계. compiler 에러, polyspace coding rule violation 등 여러상황에서 `해줘`하면 거의 찾아내주는 듯
- doxygen documentation generator: 함수 헤더에 doxygen 스타일로 코멘트를 달 수 있다.

커스터마이징 한 vscode는 새 개발환경 구축할 때 마다 번거러운데 설정 동기화를 해놓으면 편하다.

예전에는 extension으로 설정 동기화를 해주었는데, 요즘은 VScode 내에서 동기화해주는 기능이 있어 이를 사용하는 중

자주 사용하는 단축키
- `Ctrl` + `p` : workspace내에 파일 검색
- `Ctrl` + `Shift` + `f`: workspace 내에 단어 검색. 확장자 별로 files to include/exclude 설정 가능
- `F1`: extension에서 파생되는 기능시 사용 ex. F1 후 git graph 검색하여 열기 기능
- `F12`: Go to definition
- `Ctrl`+` : 터미널 open/close