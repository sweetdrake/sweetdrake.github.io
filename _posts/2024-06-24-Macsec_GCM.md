---
layout: post
title: gpt와 춤을, MACSec 예제 뿌수기
date: 2024-06-24 9:14:00
description: gpt를 사용해 python GCM-AES-128 코드를 작성해 Macsec 예제를 이해합니다
tags: 삽질 python
categories: 개발
giscus_comments: true
related_posts: false
---
IEEE 802.1AE 2018 MAC(Media access control)은 autumotive MACSec가 참고하는 표준 문서 입니다.
802.1AE 에서 예제가 나와있는데, 잘 이해가 안 가서 python을 통해 값을 유도하는 코드를 작성했습니다. gpt로 예제 해석해달라고 하니까 코드까지 만들어주는 세상이네요..

gpt3.5로 생성한 코드이고 중간마다 padding이 깨지고, 출력값이 missing나는 경우도 있었지만, 고쳐달라고 다시 타이핑 해주니 뚝딱뚝딱 고쳐버리는 AI입니다.
~~(Nvidia/MS 만세)~~
짧은 시간 대비 아웃풋이 정말 잘 나온 것 같아서.. 놀랍네요

코드는 제 github <a href="https://github.com/sweetdrake/MACSec_GCM_AES">MACSec_GCM_AES</a>에 올려두었습니다.

블로그에서 jupyter가 지원되서 jupyter로 올려봅니다

#### Jupyter 코드
{::nomarkdown}
{% assign jupyter_path = 'assets/jupyter/blog.ipynb' | relative_url %}
{% capture notebook_exists %}{% file_exists assets/jupyter/blog.ipynb %}{% endcapture %}
{% if notebook_exists == 'true' %}
  {% jupyter_notebook jupyter_path %}
{% else %}
  <p>Sorry, the notebook you are looking for does not exist.</p>
{% endif %}
{:/nomarkdown}

#### Table C-9, refer to IEEE std 802.1AE
<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/2024-06-224-Masec_GCM_tableC9.JPG" class="img-fluid rounded z-depth-1" %}
</div>