---
layout: post
title: "Bootstrap Beginner Tutorial"
description: ""
category: "tools"
tags: [bootstrap, css, html, javascript, framework, responsible, mobile]
---
{% include JB/setup %}

<style>
.demo {
  margin-bottom: 5em;
  margin: 0 0 3em;
  padding: 10px 32px;
  border: solid 1px #ddd;
  box-shadow: 1px 3px 12px -6px inset;
}

.demo h1,
.demo .h1,
.demo h2,
.demo .h2,
.demo h3,
.demo .h3,
.demo h4,
.demo .h4,
.demo h5,
.demo .h5,
.demo h6,
.demo .h6{
  margin : 1em 0 .3em !important;
}

.demo.outline .row [class*="col"]{
  outline:solid 1px;
  padding-top: 1em;
  padding-bottom: 1em;
  overflow: hidden;
  white-space: nowrap;
}
</style>


> 부트스트랩은 반응형이며 모바일 우선인 웹프로젝트 개발을 위한 가장 인기있는 HTML, CSS, JS 프레임워크입니다. - [http://bootstrapk.com/](http://bootstrapk.com/)

부트스트랩은 css 파일과 js 파일, 그리고 아이콘을 나타내기 위한 글꼴 파일들로 이루어져 있습니다. 소스코드를 이용해서 필요한 부분만 설치할 수도 있고, `npm`, `bower` 등을 이용해서 설치할 수도 있지만 여기서는 사전지식이 필요없는 직접 다운로드 방식을 설명합니다.

##다운로드
[http://getbootstrap.com/getting-started/#download](http://getbootstrap.com/getting-started/#download)

위 링크에서 Download Bootstrap 을 클릭하면 zip 파일을 다운받을 수 있습니다.
압축을 해제하면 `css`, `fonts`, `js` 세개의 디렉토리가 나오는데 모두 작업할 디렉토리에 복사해줍니다.

##기본 템플릿

작업 디렉토리에 index.html 을 만들고 기본적인 html 문서의 뼈대를 만듭니다. 

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="utf-8">
<title>부트스트랩</title>
</head>
<body>

</body>
</html>
```

Mobile device를 위한 `viewport` 설정을 합니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
...
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no">
...
</head>
...
```

`head` 태그 안쪽에 `bootstrap.css` 파일을 로딩 하는 태그를 작합니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
...
<link href="css/bootstrap.min.css" rel="stylesheet">
...
</head>
...
```

`body` 가 끝나기 바로 전에 `jquery.js`와 `bootstrap.js` 파일을 추가합니다.


**[주의]** Bootstrap이 jQuery를 필요로 하므로 반드시 Bootstrap 보다 jQuery를 먼저 로드해야 합니다.  

**[주의]** 이미 문서 어디에서든 jQuery를 로딩했다면 jQuery를 불러오는 코드가 중복되지 않도록 주의해야 합니다.

```html
...
<script src="https://code.jquery.com/jquery-1.11.3.min.js"></script>
<script src="js/bootstrap.min.js"></script>
</body>
</html>
```



필요한 리소스를 모두 로딩하고 나면 아래와 같은 기본 템플릿이 완성됩니다.

```html
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="utf-8">
<title>부트스트랩</title>
<meta name="viewport" content="width=device-width, initial-scale=1, user-scalable=no">
<link href="css/bootstrap.min.css" rel="stylesheet">
</head>
<body>


<script src="https://code.jquery.com/jquery-1.11.3.min.js"></script>
<script src="js/bootstrap.min.js"></script>
</body>
</html>
```

##테마 적용

부트스트랩의 기본 컴포넌트 이외에 향상된 스타일을 적용하기 위한 추가적인 css를 추가할 수 있습니다. 기본 디자인을 수정하고자 하면 또 다른 css를 추가하거나 이 테마 css를 수정, 또는 작성해서 커스텀 스타일을 적용할 수 있습니다.

아래는 부트스트랩 다운로드시 제공되는 기본적인 테마를 적용하는 예입니다.

```html
<link href="css/bootstrap-theme.min.css" rel="stylesheet">
```

##기본 요소

###컨테이너

내용 전체를 감싸는 컨테이너. 그리드 시스템을 만들때도 필요합니다.

####반응형 고정폭 컨테이너

화면 안에서 좌우 여백을 남기고 일정 폭을 유지하는 반응형 div.

```html
<div class="container"></div>
```

####반응형 유동성 컨테이너

언제나 화면에 꽉 차는 div.

```html
<div class="container-fluid"></div>
```

###그리드 시스템

####row
`row`는 한 행을 나타냅니다.  
반드시 `container` 또는 `container-fluid` 안에 위치해야 합니다.

아래 예제는 두 행이 있는 내용을 보여줍니다. 아직 행 안에 아무 내용이 없으므로 브라우저 화면에는 아무것도 보이지 않을 것입니다.

```html
<div class="container">
<div class="row"></div>
<div class="row"></div>
</div>
```

####col
`col`은 열을 나타냅니다.  
반드시 `row` 안에 위치해야 하며, `row` 바로 밑에는 `col`만이 위치할 수 있습니다.

```html
<div class="container">
<div class="row">
<div class="col-xs-6"></div>
<div class="col-xs-6"></div>
</div>
</div>
```

각 `col-` 클래스 뒤에 붙는 `xs-숫자` 는 해당 행에서 각 열이 row 를 12칸으로 나누었을 때, 그중에 몇칸 분량의 너비를 차지할 것인지를 나타냅니다. (자세한 사항은 뒤의 항목에서 설명합니다.)

####기본예제

다음과 같이 작성합니다.

***코드***

```html
<div class="container">

<h2>한 줄에 하나</h2>
<div class="row">
<div class="col-xs-12">col 12/12</div>
</div>

<h2>한 줄에 둘</h2>
<div class="row">
<div class="col-xs-6">col 6/12</div>
<div class="col-xs-6">col 6/12</div>
</div>

<h2>한 줄에 셋</h2>
<div class="row">
<div class="col-xs-4">col 4/12</div>
<div class="col-xs-4">col 4/12</div>
<div class="col-xs-4">col 4/12</div>
</div>

<h2>한 줄에 넷</h2>
<div class="row">
<div class="col-xs-3">col 3/12</div>
<div class="col-xs-3">col 3/12</div>
<div class="col-xs-3">col 3/12</div>
<div class="col-xs-3">col 3/12</div>
</div>

<h2>한 줄에 여섯</h2>
<div class="row">
<div class="col-xs-2">col 6/12</div>
<div class="col-xs-2">col 6/12</div>
<div class="col-xs-2">col 6/12</div>
<div class="col-xs-2">col 6/12</div>
<div class="col-xs-2">col 6/12</div>
<div class="col-xs-2">col 6/12</div>
</div>

<h2>섞어서 배치</h2>
<div class="row">
<div class="col-xs-5">col 5/12</div>
<div class="col-xs-3">col 3/12</div>
<div class="col-xs-1">col 1/12</div>
<div class="col-xs-3">col 3/12</div>
</div>

</div>
```

`div`가 어떻게 배치되는지 명확히 확인하기 위해서 다음과 같은 코드를 html에 추가한 후에 브라우저에서 확인해봅시다.

```html
<style>
div{
  outline:solid 1px;
  overflow: hidden;
  white-space: nowrap;
}
</style>
```

***결과***

<article class="demo outline">
  <h2>한 줄에 하나</h2>
  <div class="row">
    <div class="col-xs-12">col 12/12</div>
  </div>

  <h2>한 줄에 둘</h2>
  <div class="row">
    <div class="col-xs-6">col 6/12</div>
    <div class="col-xs-6">col 6/12</div>
  </div>

  <h2>한 줄에 셋</h2>
  <div class="row">
    <div class="col-xs-4">col 4/12</div>
    <div class="col-xs-4">col 4/12</div>
    <div class="col-xs-4">col 4/12</div>
  </div>

  <h2>한 줄에 넷</h2>
  <div class="row">
    <div class="col-xs-3">col 3/12</div>
    <div class="col-xs-3">col 3/12</div>
    <div class="col-xs-3">col 3/12</div>
    <div class="col-xs-3">col 3/12</div>
  </div>

  <h2>한 줄에 여섯</h2>
  <div class="row">
    <div class="col-xs-2">col 6/12</div>
    <div class="col-xs-2">col 6/12</div>
    <div class="col-xs-2">col 6/12</div>
    <div class="col-xs-2">col 6/12</div>
    <div class="col-xs-2">col 6/12</div>
    <div class="col-xs-2">col 6/12</div>
  </div>

  <h2>섞어서 배치</h2>
  <div class="row">
    <div class="col-xs-5">col 5/12</div>
    <div class="col-xs-3">col 3/12</div>
    <div class="col-xs-1">col 1/12</div>
    <div class="col-xs-3">col 3/12</div>
  </div>
</article>

####반응형 예제
위 예제를 다시 보면 `col-` 클래스 뒤에 `xs` 라는 문자열이 보입니다. 이 문자열은 'extra small' 사이즈 일때도 해당되는 폭 비율을 유지하라는 뜻입니다. 여기에 넣을 수 있는 다른 옵션들은 다음과 같습니다.

기기 종류|모바일폰|태블릿|데스크탑|큰 데스크탑
---|--- | --- | --- | ---
class|.col-xs-숫자 | .col-sm-숫자 | .col-md-숫자 |.col-lg-숫자
뷰포트 폭|< 768px|≥ 768px|≥ 992px|≥ 1200px
고정폭 container의 크기|100%|750px |970px| 1170px|

각 클래스 뒤에 숫자를 넣으면 "숫자/12"에 해당되는 너비가 클래스마다 할당된 부포트 범위 **이상**에서 적용됩니다.
예를 들어서, 아래와 같이 입력하면 데스크탑 이상에서는 4/12 사이즈로 나오고, 태블릿에서는 6/12 사이즈로 나오며, 그 이하에는 해당되는 클래스가 없으므로 가로로 정렬되지 않고 한 행을 전부 차지하게 됩니다.

```html
<div class="col-sm-6 col-md-4"></div>
```

이를 container 및 row에 적용하면 아래와 같은 결과를 볼 수 있습니다. 브라우저 폭을 변화시키면서 확인하시 바랍니다.

***코드***

```html
<div class="container">
<div class="row">
<div class="col-sm-6 col-md-4">col 3/12</div>
<div class="col-sm-6 col-md-4">col 3/12</div>
<div class="col-sm-6 col-md-4">col 3/12</div>
<div class="col-sm-6 col-md-4">col 3/12</div>
<div class="col-sm-6 col-md-4">col 3/12</div>
<div class="col-sm-6 col-md-4">col 3/12</div>
</div>
</div>
```

***결과***

<article class="demo outline">
  <div class="row">
    <div class="col-sm-6 col-md-4">col 3/12</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>
  </div>
</article>


####Clearfix
각 컬럼의 높이가 다를 때 clearfix로 처리해주지 않으면 아래처럼 레이아웃이 깨질 수 있습니다.

<article class="demo outline">
  <div class="row">
    <div class="col-sm-6 col-md-4">col 3/12<br>line1<br>line2<br>line3</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>
    <div class="col-sm-6 col-md-4">col 3/12<br>line1</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>
    <div class="col-sm-6 col-md-4">col 3/12<br>line1<br>line2</div>
    <div class="col-sm-6 col-md-4">col 3/12<br>line1</div>
  </div>
</article>

이럴 때에는 뷰포트별로 줄바꿈이 되는 포인트에서 다음과 같이 clearfix를 해줍니다.

```html
<article class="demo outline">
  <div class="row">
    <div class="col-sm-6 col-md-4">col 3/12<br>line1<br>line2<br>line3</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>

    <div class="clearfix visible-sm"><!--줄당 2개 나올 때만--></div> 

    <div class="col-sm-6 col-md-4">col 3/12<br>line1</div>

    <div class="clearfix visible-md"><!--줄당 3개 나올 때만--></div> 

    <div class="col-sm-6 col-md-4">col 3/12</div>

    <div class="clearfix visible-sm"><!--줄당 2개 나올 때만--></div> 

    <div class="col-sm-6 col-md-4">col 3/12<br>line1<br>line2</div>
    <div class="col-sm-6 col-md-4">col 3/12<br>line1</div>
  </div>
</article>
```

`visible-sm`, `visible-md` 클래스를 주의해서 봅시다. 해당되는 뷰포트 사이즈일때만 해당 엘리먼트를 보여주고 나머지 상황에서는 감춥니다. 따라서 뷰포트 사이즈에 따라서 다른 위치에서 clearfix 해줄 수 있습니다. 윈도우 사이즈를 르게 하면서 아래 결과를 확인해 봅시다.

<article class="demo outline">
  <div class="row">
    <div class="col-sm-6 col-md-4">col 3/12<br>line1<br>line2<br>line3</div>
    <div class="col-sm-6 col-md-4">col 3/12</div>

    <div class="clearfix visible-sm"></div> 

    <div class="col-sm-6 col-md-4">col 3/12<br>line1</div>

    <div class="clearfix visible-md"></div> 

    <div class="col-sm-6 col-md-4">col 3/12</div>

    <div class="clearfix visible-sm"></div> 

    <div class="col-sm-6 col-md-4">col 3/12<br>line1<br>line2</div>
    <div class="col-sm-6 col-md-4">col 3/12<br>line1</div>
  </div>
</article>

####기타

그외에도 coloumn 안에 row를 넣어서 중첩한다든지, column의 위치를 offset 한다든지 할 수 있습니다. 이러한 기능은 아래 링크에 자세히 나와 있습니다.

http://bootstrapk.com/css/#grid-offsetting

##기타 CSS styles & Components

[예시 문서](/assets/bootstrap-tutorial/)

위 링크의 문서는 부트스트랩에서 기본으로 제공하는 css 스타일과 제공하는 기본적인 Component들을 보여줍니다. 그리드 레이아웃과 조합해서 사용하면 별도의 스타일을 입히지 않아도 html만으로 괜찮은 레이아웃 뿐만 아니라 Javascript 코드가 미리 완성되어 있는 웹 컴포넌트를 손쉽게 만들 수 있습니다.

위 문서는 90% 이상 부트스트랩 공식 홈페이지의 css 항목과 components 항목의 코드를 단순히 copy &amp; paste 해서 만들어진 것입니다. 위 링크 문서의 소스코드를 살펴보면서 간단한 사용 예를 보고 공식 홈페이지의 코드를 참조하면 어렵지 않게 컴포넌트 사용법을 익힐 수 있습니다.

참조자료 : [http://getbootstrap.com/](http://getbootstrap.com/), [http://bootstrapk.com/](http://bootstrapk.com/)

