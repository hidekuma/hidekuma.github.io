---
layout: post
title: "JavaScript: Iphone 뒤로가기시 브라우저 새로고침 안되는 경우" 
categories:
  - JavaScript
  - JQuery
  - history
tags: 
  - history
  - javascript
  - jquery
---

## Iphone 뒤로가기시 브라우저 새로고침이  안되는 현상 Debugging
보통 크롬기반 브라우저의 history.back()은 이전 페이지 정보를 refresh 한다. 하지만 Safari의 경우 그렇지 않다.
페이지에 대한 context들을 Safari에서 저장하고 있기 때문이다.

따라서 Safari에서 뒤로가기를 클릭하면, 다시 페이지를 refresh하지 않는 이상 JavaScript의 window.onload, JQuery의 document.ready()는 정상동작하지 않는다는 것이다.

해결을 위해선 별도로 이벤트를 구현해야하는데 다음과 같다.
window 객체의 onpageshow라는 이벤트를 구현하면 아이폰에서 브라우저 백버튼을 눌렀을 때도, 원하는 결과를 얻을 수 있다.
```javascript
//case by JavaScript
window.onpageshow =  function(event) {
    //back 이벤트 일 경우
    if (event.persisted) {
    		//todo
    }
}
//case by JQuery
$(window).bind("pageshow", function(event) {
    //back 이벤트 일 경우
    if (event.originalEvent && event.originalEvent.persisted) {
         //todo
    }
});
```
이와 같이 function을 짜준다면 history.back()시에도 원하는 결과를 얻을 수 있다.

JQuery에서, event.originalEvent 를 체크하는 이유는 브라우저에서 발생시킨 event 내에 event.originalEvent 가 없을 수 있기 때문에 존재하는지 체크가 먼저 선행되어야한다. 그리고 persisted 를 통해, 이 event가 백버튼에 대한 이벤트인지 확인 할 수 있다.
{: .notice--info}
