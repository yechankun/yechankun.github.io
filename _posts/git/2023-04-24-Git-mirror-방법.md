---
layout: post
title: Git mirror, 원격저장소에 미러링 하는 법
date: 2023-04-24 23:15 +0900
categories: 개발 Git
tags: 깃 Git 도구 깃허브 미러링 깃랩 복사
summary:
image: /assets/img/develop/github.png
---

## 다른 깃저장소에 기존 저장소를 옮기고 싶을 때

<img class="col-md-5" src="/assets/img/develop/github.png" >
<em>항상 깃허브를 쓰고 싶지만 마음대로 될리가..</em>

간혹가다 깃랩에서 깃허브로 리포지토리를 옮기고 싶거나 깃허브에서 다른 리포지토리로 기존 리포지토리를 옮기고 싶을 때가 있다.

그럴 때 적용 가능한 팁

> 💡 **이동할 저장소에는 아무것도 생성을 안한 상태로 진행해 주세요.**

### 원본 저장소 이력 복사

```bash
git clone --mirror [원본 저장소 주소]
```

### 디렉토리 이동

```bash
cd [원본 저장소 이름].git
```

### 이동할 원격 저장소 경로 지정

```bash
git remote set-url --push origin [이동할 저장소 주소]
```

### Push

```bash
git push --mirror
```

## 완료!
