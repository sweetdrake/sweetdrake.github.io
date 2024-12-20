---
layout: post
title: 인증서는 어떻게 verification될까?
date: 2024-05-29 10:14:00
description: openssl을 통해서 인증서를 verification 해봅시다
tags: 보안 sidebar
categories: 개발
thumbnail: assets/img/240529_PKI hierarchy.JPG
giscus_comments: true
related_posts: false
toc:
  sidebar: left
---

준비물 openssl이 깔린 환경 (openssl 설치하는 건 생략합니다.. 설치는 cgywin, git bash, wsl, dockerhub 등 자유롭게)

## 인증서는 배웠는데, 실전에서 인증 관계를 잘 모르겠어요!

저 같은 분들이 많을 거라고 생각합니다. PKI를 텍스트로 배우면 인증 flow가 잘 와닿지가 않는 것 같습니다. (개인 경험)
그래서 openssl을 통해 Root인증서, intermediate인증서, 마지막 leaf 인증서를 만들어보고 leaf에서 intermediate가 인증한 인증서를 openssl command를 통해서 검증하면서 PKI를 이해 해봅시다.

예제를 통해 생성할 PKI 구조는 다음과 같습니다.

예제에 사용한 파일들은 <a href="https://github.com/sweetdrake/certificate-verification">github</a>에 올려두었습니다.

<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/240529_PKI hierarchy.JPG" class="img-fluid rounded z-depth-1" %}
</div>

### 키 만들기
각 도메인(rootCA, intermediate, leaf)을 대표하는 3개의 키를 생성합니다.
```bash
openssl genrsa -out rootCA.key 2048
openssl genrsa -out intermediate.key 2048
openssl genrsa -out leaf.key 2048
```

### CSR을 통해 인증서 발급하기
상위 스트림에게 CSR(Certificate Siging Request)를 요청하여 인증서를 발급받을 수 있습니다.
하지만 최상위 스트림의 경우(root CA) 상위 스트림이 없기 때문에 자기 자신에게 signing request를 보내면 되는데 이는 self-sign이라고 합니다. self-sign의 경우, 상위 스트림에 따로 CSR을 보낼 필요없이 자신의 도메인에서 self-sign하면 됩니다.
```bash
# RootCA 도메인에서 self-sign
openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1825 -out rootCA.pem
```
CA(Certificate Authority, 인증기관)인 rootCA.pem이 생성되었습니다.

이제 intermediate 도메인으로 넘어가보죠, <br>
intermediate 키를 갖고 있는 사용자는 CSR을 생성하여 rootCA에 intermediate 키를 갖는 사용자의 신원을 보증을 부탁하는데(Reference), 보증된 문서는 인증서의 형태로 출력됩니다.

  생성하는 CSR에는 `intermediate 도메인 사용자에 대한 정보`와 `intermediate.key의 public key`가 담겨 있습니다. CSR에 사용자에 대한 정보를 실어 보내는 이유는 뭘까요? 만약 Reference목적으로 상위 스트림에게 public key를 전달한다면, 받는 입장에서 이 key가 도대체 누구한테서 전달된 key인지 알 수가 없기 때문입니다. 또한 상위 스트림에서 특정 포맷의 신원 정보를 필터링하여 아무 관계도 없는 제3자의 publicKey를 reference하는 상황도 방지하는 기능도 합니다.

다시 본론으로 돌아와서, CSR을 상위 스트림에 보내는 이유는 앞서 말한대로, 전달하는 publicKey는 상위 스트림(여기서는 rootCA)을 대리인으로 세워 신원을 보증받기 위함입니다. rootCA라는 대리인이 보증하는 인증서는(정확하게 말하면 인증서 포맷안에 패키징된 하위 스트림의 publicKey)는 rootCA 대리인을 믿고 있는 사용자들(바꿔 말하면 rootCA의 publicKey를 갖고 있는 사용자들)이 믿고 쓸 수 있을 겁니다.
 
 물론 믿는다고 해서, 인증서 검증하는 단계가 생략되지는 않습니다. 자세한 검증 단계에 대한 설명은 뒤에서 살펴보고, 우선 하위 스트림에서 생성할 수 있는 CSR과 상위 스트림으로부터 생성되는 인증서부터 만들고 설명해 보죠.

```bash
# intermediate 도메인에서 CSR생성
openssl req -new -sha256 -key intermediate.key -out intermediate.csr

# RootCA도메인으로 CSR이 전달되어 rootCA가 reference하는 intermediate.pem 인증서가 발급됨
openssl x509 -req -in intermediate.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out intermediate.pem -days 365 -sha256 -extfile <(echo "basicConstraints=critical,CA:TRUE")
```
rootCA의 인증을 받은 인증서를 intermediate.pem가 생겼습니다. 하지만 인증서를 검증없이 사용할 수 있을까요? 인증서를 사용한다는 의미는 인증서 내부에 있는 publicKey를 사용한다는 뜻 입니다. 사용하고자 하는 publicKey를 무턱대고 쓸 수 없으니 신원을 검사할 필요가 있는데, 이를 상위 스트림이 사전 배포한 publicKey를 통해서 인증서 꼬리에 붙은 서명(Signature)을 검증하여 publicKey에 대한 신원을 확인할 수 있습니다. 물론 검증 과정의 선행으로 상위 스트림의 publicKey가 사용자에게 안전하게 전달된 것을 가정합니다.

이를 그림으로 표현하면 아래와 같습니다. 예제에 등장하지 않는 intermediate B가 나왔는데 intermediate A의 publicKey를 생성하고 싶어하는 도메인으로 이해하면 되겠습니다.
<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/240529_HowCertificateCanBeVerified.JPG" class="img-fluid rounded z-depth-1" zoomable=true %}
</div>

자세한 인증서 검증 과정을 서술하면, `(A)인증서 꼬리에 붙은 서명을 상위스트림의 publicKey로 복호화한 값에 해쉬함수를 적용해 얻은 해쉬 값`과 `(B)서명을 제외한 인증서 자체 값에 해쉬함수를 적용해 얻은 해쉬`를 비교하여 서로 같으면 인증서에 담긴 publicKey의 신원이 확인되어 해당 publicKey를 사용해도 된다는 것이고, 반대의 경우 상위 스트림의 인증을 받지않은 사용자 신원으로 판명되어 사용하면 안된다는 것 입니다.

위 예제로 보면 intermediate 도메인에 있는 사용자가 rootCA에 인증요청(CSR)을 통해 얻은 intermediate certificate는 rootCA의 publicKey를 갖고 있는 다른 사용자들에게 유효한 셈입니다. 

꽤나 복잡한데요, 위에 적어 놓은 검증 과정은 차차 나올테니 걱정마세요.

계속해서 leaf 사용자 인증서도 생성해보겠습니다. 

 다음으로 생성할 leaf 인증서는 intermediate를 상위 스트림으로 갖는 인증서로 생성하겠습니다. 생성한 CSR(leaf.csr)은 외부에 노출된 환경에서 intermediate 도메인으로 전달되며(publicKey는 노출되어도 상관 없습니다!), CSR에 대한 sign은 intermediate의 private key를 통해 이루어지고 이 결과물을 `leaf 사용자 인증서(leaf.pem)`라고 하겠습니다.
```bash
openssl req -new -sha256 -key leaf.key -out leaf.csr # Generate leaf CSR
openssl x509 -req -in leaf.csr -CA intermediate.pem -CAkey intermediate.key -CAcreateserial -out leaf.pem -days 365 -sha256 //Sign leaf CSR by intermediate
```
여기서 -basicConstraints=critical,CA:TRUE이 생략된 채로 leaf 인증서가 만들어 지는데요, 이 의미는 leaf인증서를 통해 더이상 하위 스트림의 인증서를 만들지 않겠다는 뜻이 됩니다.

예제를 통해 만든 인증서 chain 구조를 좀 더 자세히 나타내면 아래와 같습니다.

<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/240529_PKI hierarchy(Detail).JPG" class="img-fluid rounded z-depth-1" %}
</div>

### 인증서 검증하기
인증서를 openssl을 통해 비교해보겠습니다. openssl에서 인증서 검증은 인증서 체인을 검증하기 때문에 rootCA인증서와 중간 단계인 intermediate인증서를 하나로 이어(cat=concatenate) trustChain인증서를 생성하겠습니다.
```bash
cat rootCA.pem intermediate.pem > trustChain.pem
```

```bash
openssl verify -CAfile rootCA.pem intermediate.pem  # Case1) Verify OK
openssl verify -CAfile trustChain.pem leaf.pem      # Case2) Verify OK
openssl verify -CAfile rootCA.pem leaf.pem       # Case3) Verify NOK
openssl verify -CAfile intermediate.pem leaf.pem # Case4) Verify NOK
```
위 1번째 경우, intermediate.pem의 상위 스트림이 root.pem이 맞는지 확인 합니다. OK가 나왔을 겁니다.
2번째 경우, leaf.pem의 상위 스트림이 root.pem, intermediate.pem으로 이루어진 trustChain.pem이 맞는지 확인합니다. openssl에서 상위 스트림에 대한 확인은 연결된 모든 상위 스트림의 인증서를 검증합니다. 꼭 openssl이라서 이런 건 아니고 논리적으로도 모든 상위 스트림을 비교하는게 맞기 때문에 이러는 겁니다.

하지만 이런 의문을 갖을 순 있죠, leaf의 상위 스트림이 intermediate 혹은 rootCA인데 이 경우에도 부분적으로 되어야 하는건 아닌가? 

예제 3번째 경우, leaf의 최상위 스트림이 rootCA이지만, verify는 실패하는데 이는 intermediate에 대한 정보가 없기 때문입니다. 왜냐하면 leaf 인증서의 경우 intermediate.key가 sign을 했기 때문에 rootCA가 갖고있는 publicKey로는 leaf인증서 꼬리에 달린 signature를 검증할 수 없기 때문입니다.

하지만 4번째 경우, leaf의 바로 직전 상위 스트림이 intermediate인데 왜 출력되는 verify결과는 error일까요? 부분적으로 leaf 인증서 꼬리에 달린 signature에 대한 검증은 intermediate의 publicKey로 인증되지만 거시적으로 볼 때 intermediate 인증서 또한 검증하는게 더 안전하기 때문에, 최상위 정보가 없는 인증서에 대한 검증은 openssl에서는 error라고 판단하기 때문입니다.

### Raw level로 인증서 검증하기
4번째 경우, leaf인증서가 intermediate를 통해 발급되었음을 검증하는 일은 불가능한 일 일까요? 위에서 인증서 검증 과정은 인증서 꼬리에 붙은 `(A)인증서 꼬리에 붙은 서명을 상위스트림의 publicKey로 복호화한 값에 해쉬함수를 적용해 얻은 해쉬 값`과 `(B)서명을 제외한 인증서 자체 값에 해쉬함수를 적용해 얻은 해쉬`를 통해 이루어 진다고 했는데요. 한 번 그대로 만들어 보겠습니다.

#### (B)인증서 자체에 대한 해쉬값 얻기
(B)의 경우부터 살펴보죠, 서명을 제외한 인증서 자체 값은 어떻게 뽑아 낼까요?
그 전에 왜 인증서 자체 값이 필요한지 살펴보면, 상위 스트림 privateKey이 인증서를 만들 때, 사인한 서명값 뒤에 붙이는데, 서명을 만드는 방법은 사실 인증서에 대한 암호화를 사인이라고 부르기 때문입니다.

 따라서 서명 검증을 한다는 뜻은 곧 서명에 대한 복호화를 의미하게 되는데, 복호화된 값이 인증서를 사인(암호화)하기 전 값과 정확하게 일치하면 암호화와 복호화에 사용된 비대칭키가 같은 쌍인지 아닌지를 검증할 수 있기 때문입니다.
그림으로 보면 더 쉽죠, 아래 그림에서 보자면 plainText는 인증서 자체 값과 대응합니다.
<div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/240529_AsymmetricKeySignVerify.JPG" class="img-fluid rounded z-depth-1" %}
</div>


물론 인증서 자체 값에 바로 사인(암호화)를 적용하진 않습니다. 암호화에 앞서 길이를 평준화할 목적과 어떤 값을 사인을 어떤 대상으로 했는지 쉽게 못 알아차릴 목적으로 해쉬 함수를 적용 합니다. 이제 인증서 자체 값(PlainText)을 뽑아내고, 해쉬까지 적용해 보겠습니다.
```bash
# 인증서에서 sign할 때 사용되었던 plainText 뽑아내기
openssl asn1parse -inform PEM -in leaf.pem -strparse 4 -out leef.val.der

# 1) PlaintText에 해쉬함수 적용, Binary file로 보고 싶으면 경우
openssl dgst -sha256 -binary leaf.val.der > leaf.val.der.dgst

# 2) PlaintText에 해쉬함수 적용, 커맨드 창으로 hash 보고 싶은 경우
openssl dgst -sha256 leaf.val.der -out leaf.val.der.dgst
```
저는 (B)의 값으로 da88de0c48c81d92e6e16a026460baad2e5ed0808eb6e5e9d3ab8f8ee2c52fd5의 해쉬값을 얻게 되었습니다. 이 값이 이 후에 나올 과정을 통해서 똑같이 생성되는지 살펴보도록 하죠.


#### (A)서명을 복호화한 값에서 해쉬값 얻기
이제 (A)값을 얻어 볼까요. 먼저 leaf 인증서의 서명을 뽑아보겠습니다. 아래 긴 옵션 없이 -noout까지 출력하셔도 됩니다.
```bash
# leaf 인증서 꼬리부분인 서명만 출력하기
$ openssl x509 -in leaf.pem -text -noout -certopt ca_default -certopt no_validity -certopt no_serial -certopt no_subject -certopt no_extensions -certopt no_signame | sed s/://g
# >> output
    Signature Algorithm sha256WithRSAEncryption
         16dc33615c15c2a7ab4e305c3f9bd42c508a
         f4b6021611ddd3261f279a154b8164e42340
         8d56f9efa954e6f5c1f7d85b1b4dd1cd5576
         adc706b21086232f7884d390007804346e04
         4696b1c30e31537b7fcb28f2f6edb3a2799e
         d3bfe001a7162000fd449d933321bb8dabaa
         9d224f761b65da8fa7c672539c55aaf0be0d
         cd320a5ac48f23e1ee48e7b0c7fca8f2c2e3
         fc92f565e051dbbc3636ed120226edccdae9
         a2a7d75b369b595496b03adb31cf7f26edb3
         718ba92b1183a582c4cc551e6db7e6bc37a4
         40ca40c7ffaaac41a80737c9c41ed50082eb
         e925f27e0bde984c7bf86acb474d57ee2114
         c0bbefdf6a2b9362e1a4272021bfff6690af
         ae56341c
```
이번엔 publicKey가 필요합니다. 필요한 이유는 서명을 검증(복호화)할 때는, 암호화 출력값인 서명에 publicKey를 사용하면 복호화할 수 있기 때문입니다. publicKey를 얻기위해서는 leaf.pem 인증서를 서명할 때 사용한, intermediate.key private키의 반대쪽 쌍인 publicKey가 필요합니다. leaf 도메인 입장에서 바로 직전 상위 스트림의 인증서(intermediate.pem)가 있다고 가정합니다. intermediate.pem에서 publicKey는 아래 커맨드로 뽑아낼 수 있죠.
 ```bash
 # intermediate 인증서에 담긴 publicKey 출력(RSA이므로 modulus만 뽑아내겠습니다)
 openssl x509 -modulus -noout < intermediate.pem | sed s/Modulus=//
 # >> output
 b0aa11aae1e641c54abc1f3bdf52034fd7d497ed11045ad1f8eb78f2c38b91388a0401f422628e794904c2942a6d4c71f997be6e95a1135ea0fe1f1165c6619c86aa342d49f7b528443205b88110208ee0968a367225523a212f326de532a92c924b978ae1677f394403c7e9eda856a533d79ade889936ca2e69d441fe98dd5d5d6b5de1cb168a15526de39839de544906898e546e71ae2bea8ea6092a4edb3ef9186e9b1e906645ae33f6d32e670baba0b787968f6701e66a3d38c1d8970f868dd161f4abbe8e6f45b6a99121feb8c0d8f65cb945e60ffe1c4617f3c74a7b5894064bd136abd408fa98133604a969134aa3907df916b6d810d437b00571fd81
 ```

자 이제 서명값을 검증(복호화) 해보겠습니다. 아래는 파이썬으로 만든 RSA 복호화 코드입니다.

(RSA 복호화가 겨우 1줄.. ㅋㅋ)

위에서 얻은 서명과 공개키를 아래와 같이 넣어주었습니다.
```python
modulus="b0aa11aae1e641c54abc1f3bdf52034fd7d497ed11045ad1f8eb78f2c38b91388a0401f422628e794904c2942a6d4c71f997be6e95a1135ea0fe1f1165c6619c86aa342d49f7b528443205b88110208ee0968a367225523a212f326de532a92c924b978ae1677f394403c7e9eda856a533d79ade889936ca2e69d441fe98dd5d5d6b5de1cb168a15526de39839de544906898e546e71ae2bea8ea6092a4edb3ef9186e9b1e906645ae33f6d32e670baba0b787968f6701e66a3d38c1d8970f868dd161f4abbe8e6f45b6a99121feb8c0d8f65cb945e60ffe1c4617f3c74a7b5894064bd136abd408fa98133604a969134aa3907df916b6d810d437b00571fd81"
exponent=65537
signature ="16dc33615c15c2a7ab4e305c3f9bd42c508af4b6021611ddd3261f279a154b8164e423408d56f9efa954e6f5c1f7d85b1b4dd1cd5576adc706b21086232f7884d390007804346e044696b1c30e31537b7fcb28f2f6edb3a2799ed3bfe001a7162000fd449d933321bb8dabaa9d224f761b65da8fa7c672539c55aaf0be0dcd320a5ac48f23e1ee48e7b0c7fca8f2c2e3fc92f565e051dbbc3636ed120226edccdae9a2a7d75b369b595496b03adb31cf7f26edb3718ba92b1183a582c4cc551e6db7e6bc37a440ca40c7ffaaac41a80737c9c41ed50082ebe925f27e0bde984c7bf86acb474d57ee2114c0bbefdf6a2b9362e1a4272021bfff6690afae56341c"
# Decrypt RSA: (sign ** public_exp) % modulus
padded_hash = hex(pow(int(signature, 16), exponent, int(modulus, 16)))
print(padded_hash)

#Output
#0x1ffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff003031300d060960864801650304020105000420da88de0c48c81d92e6e16a026460baad2e5ed0808eb6e5e9d3ab8f8ee2c52fd5
```
서명을 복호화 했더니 0x1fffff...인 값인 어떤 포맷된 결과가 출력되었습니다. 이는 RSA_PKCS_v1_5 포맷형식의 sign이 적용된 서명을 복호화 했기 때문인데, openssl을 통한 인증서 생성시 기본 설정값은 RSA-PKCS_v1_5를 통해서 sign을 하게끔 되어있기 때문입니다. RSA_PKCS_v1_5는 0x1ffffff...ffff..00 Hash-identifier..hashValue의 형태로 encoding하게끔 되어있습니다.
여기서 3031300d060960864801650304020105000420는 <a href="https://datatracker.ietf.org/doc/html/rfc8017#section-9.2">RSA PKCS_v1_5</a> 에서 정의하는 SHA256에 대한 hash identifier입니다. 저희에게는 hash identifier 뒤에 위치하는 H(hash)da88de0c48c81d92e6e16a026460baad2e5ed0808eb6e5e9d3ab8f8ee2c52fd5값이 의미가 있죠

자 그러면 방금 과정을 통해  `(A)인증서 꼬리에 붙은 서명을 상위스트림의 publicKey로 복호화한 값에 해쉬함수를 적용해 얻은 해쉬 값`인 da88de0c48c81d92e6e16a026460baad2e5ed0808eb6e5e9d3ab8f8ee2c52fd5를 얻었습니다.
'Raw level로 인증서 검증하기' 맨 처음 과정에서 얻은 `(B)서명을 제외한 인증서 자체 값에 해쉬함수를 적용해 얻은 해쉬`는 어떤 값이였나요? da88de0c48c81d92e6e16a026460baad2e5ed0808eb6e5e9d3ab8f8ee2c52fd5가 출력되었습니다.

따라서 본 목적이였던 leaf 인증서의 상위 스트림이 intermediate 인증서가 맞는지 아닌지에 대한 검증은 위 (A)와 (B)가 서로 같기 때문에 인증서 검증이 완료되었다고 볼 수 있겠습니다.

