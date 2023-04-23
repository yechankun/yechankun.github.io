---
layout: post
title: Jekyll, Html 캐시 비활성화 방법 cache-busting
date: 2023-04-23 15:32 +0900
description: 사이트 캐시 자동 삭제, 사이트 캐시 비활성화 방법, Jekyll 캐시 사용 중지
image: /assets/img/blog/cache_1_thumnail.png
categories: 개발 블로그
tags: Jekyll 캐시 캐시중지 캐시비활성화 자동삭제 캐시삭제 예찬군
sitemap: true
---

## 지킬 블로그 대공감.. 레이아웃을 수정했는데 갱신이 안돼!

<img class="col-md-5" src="/assets/img/blog/cache_1.png">
<em>분명 레이아웃을 수정했는데 왜 안되지… 웨.. 웨? (냐옹)</em>

Github를 이용해 Jekyll 블로그를 하다보면 레이아웃과 같은 화면을 분명 바꿨음에도 갱신이 되지 않는 경우가 있다.

그것은 바로 웹캐시(Web cache)로 인해 발생하는 문제인데 브라우저가 사이트의 탐색을 빠르게 하기 위해 브라우저 내부에 html, js, css 등의 자산들을 기록하고 재방문시 이를 가져오는 방식을 택하기 때문이다.

본론부터 말하면 어느정도 해결할 방법들이 있다. 여러 해결법 중에 택해서 사용하자.

## 메타 태그 활용으로 해결 (캐시 사용 자체를 비활성화)

자주 갱신되는 목적의 블로그에선 캐시를 사용하지 않기 위해 다음과 같은 방법을 적용하는 것도 괜찮은 방법이다.

```html
<!-- 다음 코드를 html 헤더에 삽입-->

<!--
no-cache : 캐시 사용 전 재검증을 위한 요청을 강제
no-store : 클라이언트의 요청, 서버의 응답 등을 저장 안함
must-revalidate : 캐시 사용 전 반드시 만료된 것인지 검사
(http 1.1에서 적용됨)
-->
<meta
  http-equiv="Cache-Control"
  content="no-cache, no-store, must-revalidate"
/>
<!--
특정시간 이후엔 리소스가 만료됐음을 의미. 0, -1 등은 즉시 만료
Ex) Expires: Wed, 21 Oct 2015 07:28:00 GMT
-->
<meta http-equiv="Expires" content="0" />

<!-- 위의 명시된 날짜 이후가 되면 페이지가 캐싱되지 않는다.(1990년 이후 쭉) -->
<meta http-equiv="Expires" content="Mon, 06 Jan 1990 00:00:01 GMT" />

<!-- 페이지 로드시마다 페이지를 캐싱하지 않는다.(HTTP 1.0) -->
<meta http-equiv="Pragma" content="no-cache" />
```

## 빌드 버전(Build version) 사용

브라우저는 파일명을 기반으로 캐시를 저장한다. 따라서 js, css 등 html에 삽입되어 사용되는 에셋에 캐싱을 이용하지 않으려면 파일명을 달라지게 해주어야 한다.

### Github Pages 를 이용해서 Jekyll 사이트를 빌드할 경우

Ruby와 Liquid 템플릿 언어가 적용된 Jekyll에서 활용 가능하다. (이게 보통 기본 사양임)

`_include/` 위치에 `build_versions.html`이란 파일을 만들어보자.

```
<!--  build_versions.html -->
{%- if site.github.build_revision and jekyll.environment == "production" -%}
  {%- assign build_version = site.github.build_revision -%}
{%- else -%}
  {%- assign build_version = site.time | date: '%s' -%}
{%- endif -%}
```

그리고 이를 다른 `header` 문구가 있는 html 파일에 다음과 같이 적용한다.

{% raw %}

```liquid
{% include build_versions.html %}

<!-- 각종 css -->
<link
  rel="stylesheet"
  href="{{ 'assets/css/style.css' | relative_url }}?v={{ build_version }}"
/>

<!-- 각종 js -->
<script src="{{ 'assets/js/bundle.js' | relative_url }}?v={{ build_version }}"></script>
```

{% endraw %}

### Github Actions로 Jekyll 사이트를 빌드할 경우

만약 깃허브 Actions로 빌드할 경우 기본적으로 Ubuntu 환경에서 빌드되며 jekyll의 환경에선 기본적으로 [site.time](https://jekyllrb.com/docs/variables/#site-variables)에 build된 시간이 저장된다.

{% raw %}

```liquid
<link
  href="{{ 'assets/js/core.min.js' | relative_url }}?v={{ site.time | date: '%s' }}"
  rel="stylesheet"
/>
```

이처럼 에셋을 가져오는 문구의 href 마지막에 `?v={{ site.time | date: '%s' }}` 를 붙이면 된다.

`%s`는 기본적으로 date의 **strftime 필터를 가져오며** `1970-01-01 00:00:00 UTC.` 이후로 지난 초 형식으로 변환한다.

{% endraw %}

<br/>

> 이제부터 잘 갱신되는 블로그와 행복한 나날을 보내자!

---

### 참조

[https://stalker5217.github.io/javascript/cache/](https://stalker5217.github.io/javascript/cache/)

[https://milanaryal.com.np/jekyll-static-assets-cache-busting/](https://milanaryal.com.np/jekyll-static-assets-cache-busting/)
