---
layout: post
title: File Uploading
categories: []
tags: []
published: True

---

## HTML Form

일반적인 form에서 input[type="file"]을 추가하고, post 로 전송하면 파일이 바이너리로 전송된다. 이때 일반적인 폼 데이터가 아닌 바이너리 파일을 전송하기 위해서는 `enctype` attibute를 설정해 주어야 한다.

```html
<form action="전송할 주소" method="post" enctype="multipart/form-data">
    <input name="testField" value="testValue" />
    <input name="file" type="file" multiple="true" />
    <input type="submit" />
</form>
```

`enctype` attribute 는 전송될 data의 MIME 타입을 설정한다. 이 값은 http header의 Content-Type 필드를 통해서 서버측에 전달된다.

HTML form은 해당 값에 따라서 다음의 encoding을 제공한다.

 - application/x-www-form-urlencoded (default)  
   모든 공백과 특수문자를 인코딩 한다.
 - multipart/form-data  
   아무 글자도 인코딩 되지 않는다. 여러개의 content type을 boundary string으로 구분해서 보낼 때 사용되며, 일반적으로 binary 파일이 포함된 폼을 전송할 때 사용한다.
 - text/plain  
   공백문자만 인코딩한다.

### HTTP 전송내용

전송시 header와 body에는 다음 내용이 포함된다.

```sh
POST 전송될 주소
HTTP/1.1
Host: 호스트 이름
Content-Type: multipart/form-data; boundary=------this-is-boundary ### 바운더리 경계를 나타낼 문자열. 보통 브라우저에서 임의로 정한다.### 
### 여기까지 헤더. 공백 줄을 넣고 body가 시작된다.### 

------this-is-boundary ### Content-Type에서 정한 바운더리 문자로 part의 경계를 나눈다.### 
Content-Disposition: form-data; name="testField"

testValue
------this-is-boundary
Content-Disposition: form-data; name="file"; filename="sample.png"
Content-Type: image/png

바이너리 데이터...
------this-is-boundary-- ### 마지막에 "--"가 붙어서 전송내용의 끝을 알린다.### 
```

## VanillaJS


### 파일 접근

일단 js 는 로컬 파일 시스템에 직접 접근할 수 없으므로 파일을 읽어서 뭘 좀 해보려면 html input element 를 사용하는 것이 편하다.

```js
var fileInput = document.createElement("input");
fileInput.type = "file";
document.body.appendChild(fileInput);
```

여러 파일을 선택 가능하게 하려면 multiple 속성을 true 로 해준다.

```js
fileInput.multiple = true;
```

브라우저에서 사용자가 파일을 선택하면 해당 정보가 element 객체에 들어온다. 객체를 살펴보면 value에는 마지막 추가된 파일의 보안상 가짜 경로가 들어있고, "files" property 에 배열 형식으로 파일이 들어있다. files[0]에 파일 이름, 크기, MIME 타입 등이 들어있다.

```js
input = {
	value : "fake/path/to/file"
	files : [File, File, File, File]
};

input.files[0] = {
     lastModified: 1446104232000,
    lastModifiedDate: Thu Oct 29 2015 16:37:12 GMT+0900 (KST),
    name: "filename.png",
    size: 9755,
    type: "image/png"
}
```

이 File 은 Blob 객체를 확장한 타입이다.

```js
fileInput.files[0].__proto__.__proto__; //Blob
```

Blob 타입은 FileReader를 통해 읽어들이거나 XMLHttpRequest 를 통해 전송할 수 있다.

#### 파일 읽기

```js
var fr = new FileReader();

//비동기로 읽어들이므로, 읽기 완료시 callback 지정
fr.addEventListener("load", function(){
    console.log(this.result);
});

fr.readAsBinaryString(fileInput.files[0]); //fr.result 에 string으로 변환된 binary가 들어온다.
```


### 파일 전송

전송은 굳이 FileReader 를 사용하지 않고 Blob 타입을 그대로 편리하게 전송할 수 있다.

``` js
        var xhr = new XMLHttpRequest();
        xhr.open("POST",url,true);
        xhr.send(fileInput.files[0]);
```

 하지만 보통 파일 전송을 위해서는 바이너리 파일 하나만 딱 보내지 않고, HTML Form에서도 사용한 "multipart/form-data" 형식을 사용한다. 이를 위해서 FormData 객체를 생성하고 파일객체를 포함한 필드들을 추가한다.

```js
var formData = new FormData();
formData.append("field1","value1");
formData.append("field2","value2");
...
formData.append("file",file);
```

그리고 이 폼 데이터를 XHR 객체로 전송한다. 전송시 encrypt type을 설정해서 보내야 한다.

``` js
        var xhr = new XMLHttpRequest();
        xhr.open("POST",url,true);
        xhr.enctype = "multipart/form-data"
        xhr.send(formData);
```

### 전송 과정 표시

XHR 객체에는 전송 과정에서 계속 발생하는 이벤트 핸들러가 총 세개 있다. 
 - `XMLHttpRequest.upload.onProgress` : 데이터를 브라우저에서 서버로 업로드할 시 발생
 - `XMLHttpRequest.download.onProgress` : 데이터를 서버에서 브라우저로 다운로드할 시 발생
 - `XMLHttpRequest.onProgress` : `download.onProgress`와 동일 (bubbling up되는것인지는 잘 모르겠다.)

이 이벤트핸들러는 `addEventListener` 로도 등록 가능하며, 콜백에 전달되는 이벤트 객체 안에는 총 전송할 데이터가 얼마이고 그 중에 얼마나 완료되었는지가 들어있다. 다음과 같이 활용 가능하다.

```js
xhr.upload.onprogress = function(e){
    console.log((e.loaded / e.total)*100);   //전송 과정을 퍼센트로 보여준다.
};
```

## jQuery.ajax 활용

좀 짜증나는 일이지만, XHR객체를 표준적으로 지원 안하는 브라우저의 폴리필을 위해서는 jQuery 등의 라이브러리를 활용하는 것이 편하다.
일반적으로 POST 요청을 ajax 로 처리하기 위해서는 $.post 를 사용하면 편하지만, multipart 형식을 이용하기 위해서는 $.ajax 를 이용해서 다소 귀찮은 세부 설정을 해주어야 한다.

```js
$.ajax({
    url : url,
    method : 'POST',
    contentType : false,    //이 부분을 "multipart/form-data" 라고 String 으로 하드코딩 하면 boundary 정보가 전송되지 않는다.
    processData : false,   //false로 overriding 해주지 않으면 바이너리 데이터를 form 으로 urlEncode 하려고 하면서 $.param에서 에러가 발생한다.
    data : formData,  //위에서 만든 FormData 객체 
}).done(function(data){
    console.log(data);  //파일 전송 완료 후 서버측에서 보내온 응답을 처리한다.
});
```

### onprogress 이벤트

jQuery.ajax를 활용할 때는 XHR 객체를 직접 생성해서 사용하지 않으므로 onprogress 이벤트를 처리하기가 까다롭다. ajax 설정 오브젝트의 xhr 필드에 팩토리 메서드를 지정하면 XHR 객체를 직접 생성해서 return 할 수 있다. 그러면 여기에서 생성된  XHR 객체를 이용해서 ajax 통신을 처리하므로 이 객체에 기존 방식대로 이벤트 핸들러를 등록할 수 있다.

```js
$.ajax({
    xhr : function(){
        var xhr = new XMLHttpRequest();
        xhr.upload.onProgress = function(e){
            console.log(e.loaded / e.total);
        };
        return xhr;
    }
})
```

그런데 이런 방식을 사용하면 표준 XHR 객체를 직접 생성하게 되므로 브라우저 호환성을 편하게 처리하기 위해서 jQuery를 활용하는 의의가 무용지물이 된다. 따라서 jQuery에서 브라우저에 따라 생성하는 XHR 객체를 가져와서 활용해야 한다. 이를 위해서 $.ajaxSettings 객체를 이용한다. 이 객체에는 ajax 통신을 하기위해서 설정한 url 주소 등의 각정 설정값과, xhr 이라는 메소드가 들어있는데 이 메소드를 이용해서 jQuery가 생성한 XHR 객체를 얻어올 수 있다.

```js
var xhr = $.ajaxSettings.xhr();
    
if(xhr.upload){  //upload.onprogress 를 지원하지 않는 브라우저에서 upload 객체가 undefined라서 나는 에러를 방지
    xhr.upload.onprogress = function(e){
        console.log(e.loaded / e.total);
    };
} 

return xhr;
```

## 종합

위의 내용을 모두 적용한 완성된 코드는 다음과 같다.

```js
//파일 input 을 생성.  파일 지정은 유저가 ui를 통해서 직접 해야한다.
var fileInput = document.createElement("input");
fileInput.type = "file";
fileInput.multiple = true;
document.body.appendChild(fileInput);


function uploadFile(url){
    //폼 데이터 생성
    var formData = new FormData();
    formData.append("field1","value1");
    formData.append("field2","value2");
    ...
    formData.append("file",fileInput.files[0]);

    //ajax전송
    $.ajax({
        url : url,
        method : 'POST',
        contentType : false,
        processData : false,
        data : formData,
        xhr : function(){
            var xhr = $.ajaxSettings.xhr();
            if(xhr.upload){
                xhr.upload.onprogress = function(e){
                    console.log(e.loaded / e.total);
                };
            } 
            
            return xhr;
        }
    }).done(function(data){
        console.log(data);
    });
}

```

file input 에 UI를 통해서 파일을 지정하고 uploadFile(전송할 주소)를 실행하면 파일이 전송된다.

