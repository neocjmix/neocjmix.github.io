---
layout: post
title: Old Browser Compatibility Issues
categories: []
tags: []
published: True

---

## Form

### element attribute에 object 를 할당하는 경우

```js
var input = document.createElement("input");
input.setAttribute("value", [1,2,3]);
```

위의 코드는 IE9 이하에서 오동작한다.
input.value 를 다시 접근하면 결과는 다음과 같다.

| IE 7,8,9 | ModernBrowser |
|---|---|
| `[object]` | `"1,2,3"` |


따라서 오래된 브라우저에서는 다음과 같이 파싱해서 할당해야 한다.

```js
input.setAttribute("myArray",[1,2,3].join(","));
```

