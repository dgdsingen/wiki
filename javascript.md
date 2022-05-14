# Fetch

## HTTP Basic Auth

```javascript
let apiAuth = `Basic ${btoa('id:pw')}`;

let response = await fetch('https://test.com/api', {
    method:'POST',
    headers: {
        Authorization: apiAuth,
        'Content-Type': 'application/json'
    },
    credentials: 'include',
    body: JSON.stringify({
        key: val
  	})
});

text = await response.text();
```

위와 같이 사용하려면 API 서버에서도 받아줄 준비가 되어야 한다. 

```conf
# nginx에서의 CORS 설정
http {
  server {
    add_header Access-Control-Allow-Methods 'POST, GET, OPTIONS' always;
    add_header Access-Control-Max-Age '3600' always;
    add_header Access-Control-Allow-Origin 'https://test.com' always;
    add_header Access-Control-Allow-Headers 'x-requested-with' always;
    add_header Access-Control-Allow-Headers 'Content-Type' always;
    add_header Access-Control-Allow-Headers 'Authorization' always;
    add_header Access-Control-Allow-Credentials 'true' always;
  }
}
```

```python
# FastAPI에서의 preflight & HTTP Basic Auth & API 구현
import secrets
from fastapi import FastAPI, Depends, HTTPException, status, Body
from fastapi.security import HTTPBasic, HTTPBasicCredentials

app: FastAPI = FastAPI()
security: HTTPBasic = HTTPBasic()
  
async def validateCredential(credentails: HTTPBasicCredentials):
    validUsername = secrets.compare_digest(credentails.username, "id")
    validPassword = secrets.compare_digest(credentails.password, "pw")
    if not (validUsername and validPassword):
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED)

@app.options('/api')
async def preflight():
    return 200

@app.post('/api')
async def mergefile(key: str = Body(...), credentials: HTTPBasicCredentials = Depends(security)):
    await validateCredential(credentials)
    return 200
```

# Async

```javascript
async function p(x) {
    return x;
}

p('done').then(async a => await alert(a));
```

async function을 호출하면, 이 function은 Promise를 반환한다.

async함수가 값을 리턴하면 Promise는 반환된 값을 갖고 resolve될 것이다. 

async함수가 exception이나 에러를 표시하는 특정값을 던졌을때 Promise는 이값과 함께 reject될 것이다.

async함수는 async함수의 실행을 일시정지하고 보내진 Promise의 해결을 위해 대기하고 다시 async함수의 실행을 재개하고 resolve된 값을 반환하는 [await](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/await)표현과 묶일 수 있다. 

await연산자는 Promise를 기다리기 위해 사용되며, 이는 async function 내부에서만 사용될 수 있다.

## Promise

```js
p = x => {
    return new Promise(resolve => { 
        resolve(x)
    });
};

p('done').then(async a => await alert(a));
```

# Ajax

## by jQuery

```javascript
$.ajax({
    type : "post"
  , url : "/myproj/test.do"
  , data : $("#myform").serialize()
  , dataType : "json"
  , success : function(args){
        if (args.resFL == "true") {
            // success
        } else {
            // fail
        }
    }
  , error : function(e) {
        // error
    }
});
```

## Access-Control-Allow-Origin (CORS Header)

> 참조: https://jang8584.tistory.com/250

요청을 받는 서버 사이트에서 아래와 같이 헤더를 설정해준다. 


```java
response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
response.setHeader("Access-Control-Max-Age", "3600");
response.setHeader("Access-Control-Allow-Headers", "x-requested-with");
response.setHeader("Access-Control-Allow-Origin", "*");
```

### response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");

POST, GET, OPTIONS, DELETE 요청에 대해 허용하겠다는 의미


### response.setHeader("Access-Control-Max-Age", "3600");

HTTP Request 요청에 앞서 Preflight Request 라는 요청이 발생되는데, 이는 해당 서버에 요청하는 메서드가 실행 가능한지(권한이 있는지) 확인을 위한 요청이다. Preflight Request는 OPTIONS 메서드를 통해 서버에 전달되는데 (위의 Methods 설정에서 OPTIONS 를 허용함)

여기서 Access-Control-Max-Age 는 Preflight request를 캐시할 시간이다. 단위는 초단위(3,600초 = 1시간) Preflight Request를 웹브라우저에 캐시한다면 최소 1시간동안에는 서버에 재 요청하지 않을 것임. 


### response.setHeader("Access-Control-Allow-Headers", "x-requested-with");

이는 표준화된 규약은 아니지만, 보통 AJAX 호출이라는 것을 의미하기 위해 비공식적으로 사용되는 절차이다. JQuery 또한 AJAX 요청 시, 이 헤더(x-requested-with)를 포함하는 것을 확인할 수 있다. 여기서는 이 요청이 Ajax 요청임을 알려주기 위해 Header 에 x-request-width를 설정한다. Form을 통한 요청과 Ajax 요청을 구분하기 위해 사용된 비표준 규약지만, 많은 라이브러리에서 이를 채택하여 사용하고 있다. (참고로 HTML5 부터는 Form 과 Ajax 요청을 구분할 수 있는 Header가 추가되었다)


### response.setHeader("Access-Control-Allow-Origin", "*");

이 부분이 가장 중요한 부분이다. * 는 모든 도메인에 대해 허용하겠다는 의미다. 즉 어떤 웹사이트라도 이 서버에 접근하여 AJAX 요청하여 결과를 가져갈 수 있도록 허용하겠다는 의미다.

만약 보안 이슈가 있어서 특정 도메인만 허용해야 한다면 * 대신 특정 도메인만을 지정할 수 있다.


```
response.addHeader("Access-Control-Allow-Origin", "*");
```


대신


```
response.addHeader("Access-Control-Allow-Origin", "http://test1.com");
response.addHeader("Access-Control-Allow-Origin", "http://test2.com");
```


이렇게 쓰면 test1.com, test2.com 2개의 도메인에 대해서만 크로스 도메인을 허용하겠다는 의미


## Ajax 호출로 Target Cross Domain의 setCookie를 받고싶을 경우

위와 같이 CORS Header를 세팅해도 ajax를 호출해보면 Response header의 setCookie가 먹히지 않는다. 

이때 아래와 같이 처리해주면 bbb.com에서 aaa.com으로 ajax 호출한 Response header로 넘어온 setCookie 값이 aaa.com 도메인으로 저장된다. 


### Server side


```
response.setHeader("Access-Control-Allow-Methods", "POST, GET, OPTIONS, DELETE");
response.setHeader("Access-Control-Max-Age", "3600");
response.setHeader("Access-Control-Allow-Headers", "x-requested-with");
response.setHeader("Access-Control-Allow-Origin", "http://aaa.com");
response.setHeader("Access-Control-Allow-Credentials", "true");
```

### Client side


```
$.ajax({
    xhrFields: {
        withCredentials: true
    },
    crossDomain: true,
    url:'http://aaa.com/test.do'
});
```

## Issues

### Status Code가 200인데 error인 경우

contentType이 application/json인데 리턴값이 json형태가 아님

# Puppeteer

## Install


```sh
npm install puppeteer --upgrade
```

## 주간보고 자동화


```js
const puppeteer = require('puppeteer');

puppeteer.launch({
  headless: false, timeout: 3000000, args: [
    '--safebrowsing-disable-auto-update',
    '--safebrowsing-disable-download-protection',
    '--safebrowsing-disable-extension-blacklist',
    '--safebrowsing-manual-download-blacklist',
    '--allow-insecure-localhost',
    '--disable-client-side-phishing-detection'
  ]
})
  .then((browser) => {
    return browser.newPage()
      .then((page) => {
        Date.prototype.getWeek = function () {
          var date = this.getDate();
          this.setDate(1);
          var bias = this.getDay();
          this.setDate(date);
          var month = this.getMonth() + 1 < 10 ? '0' + (this.getMonth() + 1) : '' + (this.getMonth() + 1);
          return '' + this.getFullYear() + month + 'W' + Math.ceil((date + bias) / 7);
        }

        var d = new Date();
        var thisWeekTitle = d.getWeek() + '_주간보고서';
        d.setDate(d.getDate() - 7);
        var lastWeekTitle = d.getWeek() + '_주간보고서';

        var wikiUrl = 'http://test.com/wikis/home?lang=ko#!/wiki/Weekly Meeting/page/';

        return page.goto('http://test.com', { waitUntil: 'domcontentloaded' })
          .then(() => page.frames()[1])
          .then((frame) => {
            frame.type('#txtUserID', 'ID') // ###수정 필요### 계정 아이디
              .then(() => frame.type('#txtPwd', 'PW')) // ###수정 필요### 계정 비밀번호
              .then(() => frame.click('button[type="button"]'))
          })
          .then(() => page.waitForNavigation({ timeout: 4000 }))
          .then(() => null, () => page.goto(wikiUrl + lastWeekTitle))
          .then(() => page.waitForNavigation({ timeout: 4000 }))
          .then(() => null, () => page.$eval('#wikiContentDiv', wikiContentDiv =>
            localStorage.setItem('wikiContent', wikiContentDiv.innerHTML)
          ))
          .then(() => page.waitForNavigation({ timeout: 3000 }))
          .then(() => null, () => page.goto(wikiUrl + thisWeekTitle))
          .then(() => page.waitForNavigation({ timeout: 3000 }))
          .then(() => null, () => page.$eval('body', body =>
            localStorage.setItem('wikiUrl', location.href)
          ))
          .then(() => page.waitForNavigation({ timeout: 3000 }))
          .then(() => null, () => page.click('.lotusBtn a[role="button"]'))
          .then(() => page.waitForNavigation({ timeout: 3000 }))
          .then(() => null, () => page.frames()[3])
          .then((frame) => frame.$eval('body', body =>
            body.innerHTML = localStorage.getItem('wikiContent')
          ))
          .then(() => page.waitForNavigation({ timeout: 3000 }))
          .then(() => null, () => page.click('#edit_saveclose'))
          .then(() => page.waitForNavigation({ timeout: 3000 }))
          .then(() => page.goto('http://test.com/OpenDocument', { waitUntil: 'domcontentloaded' }), // ###수정 필요### 메일 개인 URL
            () => page.goto('http://test.com/OpenDocument', { waitUntil: 'domcontentloaded' })) // ###수정 필요### 메일 개인 URL
          .then(() => page.waitForNavigation({ timeout: 5000 }))
          .then(() => null, () => page.frames()[1])
          .then((frame) => frame.click('span[id*="new-text"]'))
          .then(() => page.waitForNavigation({ timeout: 5000 }))
          .then(() => null, () => page.frames()[1])
          .then((frame) => {
            var sendTo = '"최근" <test@test.com>' // ###수정 필요### 메일 수신자 목록
            frame.type('textarea[id*=sendto]', sendTo)
              .then(() => frame.type('textarea[id*=subject]', thisWeekTitle))

            for (let i = 0; i < page.frames().length; i++)
              page.frames()[i].$eval('body[class*="content"]', content => {
                content.innerHTML = '안녕하세요. <br>금일 주간보고서 작성 요청드립니다. <br><br>'
                  + localStorage.getItem('wikiUrl')
                  + '<br><br>감사합니다.' // ###수정 필요### 메일 내용
              })
          })
          .then(() => page.waitForNavigation({ timeout: 5000 }))
          .then(() => null, () => page.frames()[1])
          .then((frame) => frame.click('span[id*="send-text"]'))
          .then(() => page.waitForNavigation({ timeout: 5000 }))
      })
      .then(() => browser.close());
  });
```

# 키 입력받기


```javascript
// VanillaJS
function testOnEnter(e) {
    if (e.keyCode == 13)
        alert(0);
}

<input type="text" onKeyDown='testOnEnter(event);'>

// jQuery
$('input[name=test]').keydown(function(e) {
    if (e.keyCode == 13)
        select();
});
```

# 모바일 앱 연동

```javascript
// 앱이 깔려있지 않으면 스토어 & 마켓으로, 앱이 깔려있다면 앱으로 이동한다. 
$('#appdownbtn').click(function() {
    var openAt = new Date,
        uagentLow = navigator.userAgent.toLocaleLowerCase(),
        chrome25,
        kitkatWebview;
 
    //var androidAppUrl = "com.test.handset";
    //var androidIntentUrl = "intent:testmembers://#Intent;package=com.test.handset;end";
    //var androidMarketUrl = "https://play.google.com/store/apps/details?id=com.test.handset&hl=ko";
    var androidAppUrl = "market://details?id=com.test.handset&url=test://home";
    var androidIntentUrl = androidAppUrl;
    var androidMarketUrl = androidAppUrl;
    var iosAppUrl = "testmembers://";
    var iosStoreUrl = "https://itunes.apple.com/kr/app/id12345?mt=8";
 
    $("body").append("<iframe id='____appdownlink____'></iframe>");
    $("#____appdownlink____").hide();
 
    setTimeout( function() {
        if (new Date - openAt < 4000) {
            if (uagentLow.match(/android/)) {
                //alert(0);
                //$("#____appdownlink____").attr("src", androidMarketUrl);
            } else if (uagentLow.match(/iphone|ipad|ipod/)) {
                //alert(1);
                location.replace(iosStoreUrl);
            }
        }
    }, 1000);
 
    if (uagentLow.match(/android/)) {
        chrome25 = uagentLow.match(/chrome/) && navigator.appVersion.match(/Chrome\/\d+.\d+/)[0].split("/")[1] > 25;
        kitkatWebview = uagentLow.match(/naver/) || uagentLow.match(/daum/);
 
        if (chrome25 && !kitkatWebview) {
            //alert(2);
            document.location.href = androidIntentUrl;
        } else{
            //alert(3);
            $("#____appdownlink____").attr("src", androidAppUrl);
        }
    } else if (uagentLow.match(/iphone|ipad|ipod/)) {
        //alert(4);
        location.href = iosAppUrl;
    }
});
```



# 팝업 종료시 이벤트 실행

```javascript
var openDialog = function(uri, name, options, closeCallback) {
    var win = window.open(uri, name, options);
    var interval = window.setInterval(function() {
        try {
            if (win == null || win.closed) {
                window.clearInterval(interval);
                closeCallback(win);
            }
        }
        catch (e) {
        }
    }, 1000);
    return win;
};
```

# Closure

```js
var sequencer = function() {
    var s = 0;
    return function() {
        return s++;
    }
};

seq = sequencer();

seq(); // 0
seq(); // 1
seq(); // 2
```

# Clipboard

```js
function copyToClipboard() {
    var url = 'url';

    var msie = $.browser.msie ? true : false;

    if (msie) {
        window.clipboardData.setData("Text", url);
        alert('복사되었습니다.');
    } else {
        temp = prompt("Ctrl+C를 눌러 클립보드로 복사하세요", url);
    }
}
```

# Cookie

```js
function getCookie( cookieName ) {
    var search = cookieName + "=";
    var cookie = document.cookie;

    if( cookie.length > 0 ) {
        startIndex = cookie.indexOf( cookieName );

        if( startIndex != -1 ) {
            startIndex += cookieName.length;
            endIndex = cookie.indexOf( ";", startIndex );

            if( endIndex == -1) 
            endIndex = cookie.length;

            return unescape( cookie.substring( startIndex + 1, endIndex ) );
        } else {
            return false;
        }
    } else {
        return false;
    }
}

function setCookie(cookieName, cookieValue, expireDate) {
    var today = new Date();
    today.setDate( today.getDate() + parseInt( expireDate ) );
    document.cookie = cookieName + "=" + escape( cookieValue ) + "; path=/; expires=" + today.toGMTString() + ";";
}
 
function deleteCookie( cookieName ) {
    var expireDate = new Date();
    expireDate.setDate( expireDate.getDate() - 1 );
    document.cookie = cookieName + "= " + "; expires=" + expireDate.toGMTString() + "; path=/";
}

function setMyCookie() {
    setCookie( form.setName.value, form.setValue.value, form.expire.value );
    viewCookie();
}

function getMyCookie() {
    alert( "쿠키 값 : " + getCookie( form.getName.value ) );
}

function deleteMyCookie() {
    deleteCookie( form.deleteName.value );
    viewCookie();
}

function viewCookie() {
    if( document.cookie.length > 0 )
        cookieOut.innerText = document.cookie;
    else
        cookieOut.innerText = "저장된 쿠키가 없습니다.";
}
```

# Date

## Countdown timer


```js
// Time global variables
var now;
var future;
var interval;

// Set time to after 3 minutes
function initTime() {
    clearInterval(interval);
    now = new Date();
    future = new Date(Date.parse(now) + 1000 * 179); // after 3 minutes
    interval = setInterval(function () {
        calcTime();
    }, 500);
}

function calcTime() {
    now = new Date();
    days = Math.floor((future - now) / 1000 / 60 / 60 / 24);
    hours = Math.floor((future - now) / 1000 / 60 / 60 - (24 * days));
    minutes = Math.floor((future - now) / 1000 / 60 - (24 * 60 * days) - (60 * hours));
    seconds = Math.round((future - now) / 1000 - (24 * 60 * 60 * days) - (60 * 60 * hours) - (60 * minutes));

    if (seconds != 60)
        $('#time').text(minutes + ":" + (seconds < 10 ? '0' + seconds : seconds));

    if (minutes <= 0 && seconds <= 0)
        clearInterval(interval);
}
```

## setMonth() 2월 -> 3월 변경되는 이슈

오늘이 1월 30일이라고 가정해보자. 

```js
var d = new Date(2015, 0, 30);
```

2월 28일자의 Date 객체를 만들고 싶은데 아래와 같이 설정하면 3월 28일이 된다. 

```js
d.setFullYear(2015);
d.setMonth(1);
d.setDate(28);
```

1월 30일인 상태에서 setMonth(1)을 실행하면 2월 30일이 되는데 이는 자동 계산되어 3월 2일로 변환된다. 
3월 2일이 된 후 setDate(28)을 실행하므로 최종 날짜는 3월 28일이 된다. 
그러므로 임의의 시간을 set하려면 아래와 같이 하자. 

```js
var d = new Date(2015, 1, 28);
```

## add 함수


```js
Date.MILLI = 'ms';
Date.SECOND = 'ss';
Date.MINUTE = 'mi';
Date.HOUR = 'hr';
Date.DAY = 'dy';
Date.MONTH = 'mo';
Date.YEAR = 'yr';

Date.prototype.clone = function () {
        return new Date(this.getTime());
    }

Date.prototype.getFirstDateOfMonth = function() {
    return new Date(this.getFullYear(), this.getMonth(), 1);
}

Date.prototype.getLastDateOfMonth = function() {
    return new Date(this.getFullYear(), this.getMonth(), this.getDaysInMonth());
}

Date.prototype.isLeapYear = function() {
    var year = this.getFullYear();
    return !!((year & 3) == 0 && (year % 100 || (year % 400 == 0 && year)));
}

Date.prototype.getDaysInMonth = function() {
    var daysInMonth = [31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31];

    return function() { // return a closure for efficiency
        var m = this.getMonth();

        return m == 1 && this.isLeapYear() ? 29 : daysInMonth[m];
    };
}()

Date.prototype.add = function (interval, value) {
    var d = this.clone();
    if (!interval || value === 0) return d;

    switch(interval.toLowerCase()) {
        case Date.MILLI:
            d.setMilliseconds(this.getMilliseconds() + value);
            break;
        case Date.SECOND:
            d.setSeconds(this.getSeconds() + value);
            break;
        case Date.MINUTE:
            d.setMinutes(this.getMinutes() + value);
            break;
        case Date.HOUR:
            d.setHours(this.getHours() + value);
            break;
        case Date.DAY:
            d.setDate(this.getDate() + value);
            break;
        case Date.MONTH:
            var day = this.getDate();
            if (day > 28) {
                day = Math.min(day, this.getFirstDateOfMonth().add('mo', value).getLastDateOfMonth().getDate());
            }
            d.setDate(day);
            d.setMonth(this.getMonth() + value);
            break;
        case Date.YEAR:
            d.setFullYear(this.getFullYear() + value);
            break;
    }
    return d;
}
```

# Form

## Form 동적 생성하여 submit


```js
var parent_element = '';
parent_element += "<input type='hidden' name='key' value='val' />";

var new_form = document.createElement("form");

$(new_form).attr({
    'action': 'http://test.com/test.do',
    'target': '_blank',
    'method': 'get'
});

$(new_form).append(parent_element);

document.body.appendChild(new_form);
new_form.submit();
document.body.removeChild(new_form);
```

# Popup

```js
// 중앙에 띄우기
var width   = 280;
var height  = 310;
//var leftVal = screenX || screenLeft;
//var topVal  = screenY || screenTop;
var topVal  = (screen.availHeight - height) / 2;
var leftVal = (screen.availWidth - width) / 2;

// 랜덤으로 띄우기
var width   = 10;
var height  = 10;
var leftVal = Math.random() * (screen.availWidth - (width + 10));
    leftVal = leftVal <= 50 ? leftVal + 50 : leftVal;
var topVal  = Math.random() * (screen.availHeight - (height + 10));
    topVal  = topVal <= 120 ? topVal + 120 : topVal;

var popOption = "top=" + topVal + ", left=" + leftVal + ", width=" + width + ", height=" + height + ", resizable=no, scrollbars=no, status=no;";

window.open("/pop_20140422.html","",popOption);
```

# Validation

## How to toggle a boolean?

```js
bool ^= true;
```

## 일부 문자열 replace 하기


```js
var teststr = '/etc/designs/hera/ko/1wea/sadf'
var phoneRegex = /\/etc\/designs\/hera\/ko(\/[^.]+)/;
alert(teststr.replace(phoneRegex, '/kr/ko/static$1'));
```

## 숫자만 입력 가능하게 제한


```js
function digit_check(str) {
    return str.replace(/[^0-9$]/gi, ""); 
}

<input type="text" onkeyup='this.value=digit_check(this.value);'/>
```

## 한글만 입력 가능하게 제한


```js
function hangul_check(str) {
    return (/^[ㄱ-ㅎ\s|가-힣\s|ㅏ-ㅣ\s|\*]+$/).test(str); 
}

<input type="text" onkeyup='this.value=digit_check(this.value);'/>
```

## Phone, Email, Date


```js
var phoneRegex = /^[0-9]*-[0-9]*-[0-9]*$/; 
if (phoneRegex.test(frm.mobPhoneNo.value) == false) { 
    alert('Please check your Contact No.');
    return false;
}
 
var emailRegex=/^([\w-]+(?:\.[\w-]+)*)@((?:[\w-]+\.)*\w[\w-]{0,66})\.([a-z]{2,6}(?:\.[a-z]{2})?)$/; 
if (emailRegex.test(frm.newEmail.value) == false) { 
    alert('Please check your Email.');
    return false;
}
 
var dateRegex = /[0-9]{2}-[0-9]{2}-[0-9]{4}/; // dd-mm-yyyy
if (dateRegex.test(frm.birthDate.value) == false) { 
    alert('Please check your Birthday (ex : 31-03-2014).');
    return false;
}
```

## input text length


```js
function checklen(fform, maxlength) {
    var msgtext;
    msgtext = fform.value;
    var i=0;
    var l=0;
    var temp;
    var lastl;
 
    while(i < msgtext.length) {
        temp = msgtext.charAt(i);
        l += getBytes(temp);
 
        // OverFlow
        if(l>maxlength) {
            alert("한글은 300자, 영문은 600자까지 입력 가능합니다.");
            temp = fform.value.substr(0,i);
            fform.value = temp;
            l = lastl;
            return false;
            //break;
        }
        lastl = l;
        i++;
    }
 
    return true;
}
 
function getBytes(s,b,i,c) {
    for (b = i = 0; c = s.charCodeAt(i++); b += c >> 11 ? 3 : c >> 7 ? 2 : 1);
    return b;
}
 
$('#id').bind('keyup', function() {
    if (!checklen(id[0], 600))
        $('#id').focus();

    $('#idCnt').text(getBytes($('#id').val()));
});
```

## Check Mobile Browser & Device


### PC, Mobile 단순 구분


```js
// navigator.platform이 존재하고,
// 그 값이 PC Platform 값이 아니라면 Mobile Platform
// 이외의 case는 전부 PC Platform으로 인식
function isMobilePlatform() {
    return ( navigator.platform && "win16|win32|win64|mac|macintel|linux x86|linux x86_64|linux i686|freebsd i386|freebsd amd64|sunos|hp-ux|x11".indexOf(navigator.platform.toLowerCase()) < 0 )
}

// 그러나 이 방법으로는 폰과 태블릿 구분이 안된다!
```

### Platform 상세구분


```js
var MobileUA = ( function () {
    var ua = navigator.userAgent.toLowerCase();

    var mua = {
        IOS: /ipod|iphone|ipad/.test(ua), //iOS
        IPHONE: /iphone/.test(ua), //iPhone
        IPAD: /ipad/.test(ua), //iPad
        ANDROID: /android/.test(ua), //Android Device
        WINDOWS: /windows/.test(ua), //Windows Device
        TOUCH_DEVICE: ('ontouchstart' in window) || /touch/.test(ua), //Touch Device
        MOBILE: /mobile/.test(ua), //Mobile Device (iPad 포함)
        ANDROID_TABLET: false, //Android Tablet
        WINDOWS_TABLET: false, //Windows Tablet
        TABLET: false, //Tablet (iPad, Android, Windows)
        SMART_PHONE: false //Smart Phone (iPhone, Android)
    };
    
    mua.ANDROID_TABLET = mua.ANDROID && !mua.MOBILE;
    mua.WINDOWS_TABLET = mua.WINDOWS && /tablet/.test(ua);
    mua.TABLET = mua.IPAD || mua.ANDROID_TABLET || mua.WINDOWS_TABLET;
    mua.SMART_PHONE = mua.MOBILE && !mua.TABLET;

    return mua;
}());
```

### 확인용 페이지 소스


```js
<script>
// This function can't judge phone or tablet.
function isMobilePlatform() {
    return ( navigator.platform && "win16|win32|win64|mac|macintel|linux x86|linux x86_64|linux i686|freebsd i386|freebsd amd64|sunos|hp-ux|x11".indexOf(navigator.platform.toLowerCase()) < 0 )
}

// This can.
var MobileUA = (function () {
    var ua = navigator.userAgent.toLowerCase();

    var mua = {
        IOS: /ipod|iphone|ipad/.test(ua),//iOS
        IPHONE: /iphone/.test(ua),//iPhone
        IPAD: /ipad/.test(ua),    //iPad
        ANDROID: /android/.test(ua),    //Android Device
        WINDOWS: /windows/.test(ua),    //Windows Device
        TOUCH_DEVICE: ('ontouchstart' in window) || /touch/.test(ua),    //Touch Device
        MOBILE: /mobile/.test(ua),//Mobile Device (iPad ..)
        ANDROID_TABLET: false,    //Android Tablet
        WINDOWS_TABLET: false,    //Windows Tablet
        TABLET: false,    //Tablet (iPad, Android, Windows)
        SMART_PHONE: false//Smart Phone (iPhone, Android)
    };

    mua.ANDROID_TABLET = mua.ANDROID && !mua.MOBILE;
    mua.WINDOWS_TABLET = mua.WINDOWS && /tablet/.test(ua);
    mua.TABLET = mua.IPAD || mua.ANDROID_TABLET || mua.WINDOWS_TABLET;
    mua.SMART_PHONE = mua.MOBILE && !mua.TABLET;

    return mua;
}());

var result = '';
//result += 'navigator.platform(' + navigator.platform + ') ' + (isMobilePlatform() ? 'is NOT Mobile Platform' : 'is Mobile Platform') + '\n';
result += "'" + navigator.platform + "'" + (isMobilePlatform() ? ' is Mobile Platform' : 'is NOT Mobile Platform') + '\n';
result += '\n';

for ( var prop in MobileUA )
    if ( MobileUA[prop] == true )
        result += prop + '\n';

alert(result);
</script>
```

# History

## history.pushState

웹 브라우저에서 뒤로가기 클릭시 돌아가는 URL을 조작하고 싶다면 아래와 같이 history에 조작된 URL을 넣어준다. 

```js
if ('pushState' in history) {
	history.pushState(null, document.title, "?p=" + p);
}
```

- 참조 : https://blog.outsider.ne.kr/1276

# Issues

## JSP EL과 연동시 html escape 안됨

다음과 같은 경우 &와 같은 문자들이 그대로 html string으로 노출되는 문제가 있다. 

```js
$('#id').html('<c:out value="${url}"/>');
```

그럴 때는 이렇게 하면 된다.

```js
<input type='hidden' id='temp' value='<c:out value="${url}"/>'/>

$('#id').html($('#temp').val());
```

# References
- [You Might Not Need jQuery](http://youmightnotneedjquery.com/) 
- [You-Dont-Need-jQuery/ko-KR](https://github.com/nefe/You-Dont-Need-jQuery/blob/master/README.ko-KR.md) 
- [ECMAScript 6 compatibility table](http://kangax.github.io/compat-table/es6/) 
- [프론트엔드 개발자가 되려면 무엇을 학습해야 하나요? 2018](https://medium.com/@Jbee_/%EC%8B%A0%EC%9E%85-%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C%EC%9E%90%EA%B0%80-%EB%90%98%EB%A0%A4%EB%A9%B4-%EB%AC%B4%EC%97%87%EC%9D%84-%ED%95%99%EC%8A%B5%ED%95%B4%EC%95%BC-%ED%95%98%EB%82%98%EC%9A%94-1dd59a14e084) 
- [Next.js 튜토리얼 1편: 시작하기](https://brunch.co.kr/@hee072794/81) 
- [JavaScript Event Loop](http://asfirstalways.tistory.com/362) 
- [An Overview of JavaScript Testing in 2018](https://medium.com/welldone-software/an-overview-of-javascript-testing-in-2018-f68950900bc3) 
- [문자열의 바이트(Byte) 길이를 구하는 방법](http://programmingsummaries.tistory.com/239) 
- [Ajax를 사용할 때 웹브라우저 &quot;뒤로 가기&quot;의 구현](https://blog.outsider.ne.kr/1276) 
- [Github - puppeteer](https://github.com/GoogleChrome/puppeteer) 
- [Puppeteer as a service](http://pptraas.com/) 
- [GoogleChromeLabs/puppeteer-examples: Use case-driven examples for using Puppeteer and headless chrome](https://github.com/GoogleChromeLabs/puppeteer-examples) 
- [checkly/puppeteer-recorder: Puppeteer recorder is a Chrome extension that records your browser interactions and generates a Puppeteer script.](https://github.com/checkly/puppeteer-recorder) 
- [TypeScript](http://www.typescriptlang.org/index.html) 
- [Typescript의 모듈 해석 방식](https://vomvoru.github.io/blog/typescript-module-resolution/) 
- [jsdev.kr](https://jsdev.kr/) 
- [TypeScript-Handbook ko](https://typescript-kr.github.io/) 
- [트리 쉐이킹으로 자바스크립트 페이로드 줄이기](https://github.com/nhnent/fe.javascript/wiki/%23167-%ED%8A%B8%EB%A6%AC-%EC%89%90%EC%9D%B4%ED%82%B9%EC%9C%BC%EB%A1%9C-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%ED%8E%98%EC%9D%B4%EB%A1%9C%EB%93%9C-%EC%A4%84%EC%9D%B4%EA%B8%B0) 
- [javascript var, let, const 차이점](https://gist.github.com/LeoHeo/7c2a2a6dbcf80becaaa1e61e90091e5d) 
- [let과 const는 호이스팅 될까?](https://medium.com/korbit-engineering/let%EA%B3%BC-const%EB%8A%94-%ED%98%B8%EC%9D%B4%EC%8A%A4%ED%8C%85-%EB%90%A0%EA%B9%8C-72fcf2fac365) 
- [Cross Domain Web Worker? - Stack Overflow](https://stackoverflow.com/questions/20410119/cross-domain-web-worker) 
- [The definitive Node.js handbook](https://medium.freecodecamp.org/the-definitive-node-js-handbook-6912378afc6e) 
- [The 12 Things You Need to Consider When Evaluating Any New JavaScript Library](https://medium.freecodecamp.org/the-12-things-you-need-to-consider-when-evaluating-any-new-javascript-library-3908c4ed3f49) 
- [The Cost Of JavaScript In 2018](https://medium.com/@addyosmani/the-cost-of-javascript-in-2018-7d8950fbb5d4) 
- [자바스크립트는 어떻게 작동하는가: 웹어셈블리와의 비교 + 언제 웹어셈블리를 사용하는 게 좋은가](https://engineering.huiseoul.com/%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8%EB%8A%94-%EC%96%B4%EB%96%BB%EA%B2%8C-%EC%9E%91%EB%8F%99%ED%95%98%EB%8A%94%EA%B0%80-%EC%9B%B9%EC%96%B4%EC%85%88%EB%B8%94%EB%A6%AC%EC%99%80%EC%9D%98-%EB%B9%84%EA%B5%90-%EC%96%B8%EC%A0%9C-%EC%9B%B9%EC%96%B4%EC%85%88%EB%B8%94%EB%A6%AC%EB%A5%BC-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94-%EA%B2%8C-%EC%A2%8B%EC%9D%80%EA%B0%80-cf48a576ca3) 
- [#164: 웹에서 자바스크립트 모듈 사용하기 · nhnent/fe.javascript Wiki](https://github.com/nhnent/fe.javascript/wiki/%23164:-%EC%9B%B9%EC%97%90%EC%84%9C-%EC%9E%90%EB%B0%94%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%EB%AA%A8%EB%93%88-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0) 
- [How do I update each dependency in package.json to the latest version?](https://stackoverflow.com/questions/16073603/how-do-i-update-each-dependency-in-package-json-to-the-latest-version) 
- [2018 JavaScript Rising Stars](https://risingstars.js.org/2018/en/) 
- [React와 Apollo, Parcel 기반 서비스 개발](https://d2.naver.com/helloworld/2838729) 
- [The React Handbook](https://medium.freecodecamp.org/the-react-handbook-b71c27b0a795) 

## Webpack
- [webpack.js.org](https://webpack.js.org/) 
- [Webpack 4 설정하기](https://www.zerocho.com/category/Webpack/post/58aa916d745ca90018e5301d) 
- [Webpack 4 CSS와 기타 파일 번들링하기](https://www.zerocho.com/category/Webpack/post/58ac2d6f2e437800181c1657) 
- [Webpack 4 청크 관리 및 코드 스플리팅 하기](https://www.zerocho.com/category/Webpack/post/58ad4c9d1136440018ba44e7) 
- [요즘 잘나가는 프론트엔드 개발 환경 만들기(2018): Webpack 4](https://meetup.toast.com/posts/153) 
- [Webpack 완전정복하기!! - Dev. DY](https://kdydesign.github.io/2017/11/04/webpack-tutorial/) 
- [Babel + Webpack](https://poiemaweb.com/es6-babel) 
- [Webpack + SpringBoot 기반 Front-end 개발 환경 구축](https://medium.com/@alvin.h/%EC%9B%B9%ED%8C%A9-%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-%EA%B8%B0%EB%B0%98%EC%9D%98-%ED%94%84%EB%A1%A0%ED%8A%B8%EC%97%94%EB%93%9C-%EA%B0%9C%EB%B0%9C-%ED%99%98%EA%B2%BD-%EA%B5%AC%EC%B6%95%ED%95%98%EA%B8%B0-87cd758e1eae) 

## React
- [React](https://reactjs.org/) 
- [React 적용 가이드 - React와 Redux](https://d2.naver.com/helloworld/1848131) 
- [React 적용 가이드 - React 작동 방법](https://d2.naver.com/helloworld/9297403) 
- [ReactJS의 Virtual DOM과 Repaint, Reflow](http://blog.drakejin.me/React-VirtualDOM-And-Repaint-Reflow/) 
- [리액트에 대해서 그 누구도 제대로 설명하기 어려운 것 – 왜 Virtual DOM 인가?](https://velopert.com/3236) 
- [Virtual DOM과 DOM의 차이점](https://github.com/FEDevelopers/tech.description/wiki/%EA%B0%80%EC%83%81-%EB%8F%94%EA%B3%BC-%EB%8F%94%EC%9D%98-%EC%B0%A8%EC%9D%B4%EC%A0%90) 
- [What forces layout/reflow. The comprehensive list.](https://gist.github.com/paulirish/5d52fb081b3570c81e3a) 
- [10 Ways to Minimize Reflows and Improve Performance](https://www.sitepoint.com/10-ways-minimize-reflows-improve-performance/) 
- [How Virtual-DOM and diffing works in React](https://medium.com/@gethylgeorge/how-virtual-dom-and-diffing-works-in-react-6fc805f9f84e) 
- [Change And Its Detection In JavaScript Frameworks](http://teropa.info/blog/2015/03/02/change-and-its-detection-in-javascript-frameworks.html) 
- [Introduction to Layout in Mozilla](https://developer.mozilla.org/en-US/docs/Mozilla/Introduction_to_Layout_in_Mozilla) 
- [How browsers work](http://taligarsiel.com/Projects/howbrowserswork1.htm#Parsing_general) 
- [React 교육](https://www.fastcampus.co.kr/dev_camp_react/) 
- [Slack | react_9기 | FC_React](https://fcreact.slack.com/) 
- [Slides](https://slides.com/woongjae) 
- [GitHub - 2woongjae/js-for-react](https://github.com/2woongjae/js-for-react) 
- [GitHub - axios/axios: Promise based HTTP client for the browser and node.js](https://github.com/axios/axios) 
- [ZEIT](https://zeit.co/) 
- [Next.js](https://nextjs.org/) 
- [2woongjae/tic-tac-toe](https://github.com/2woongjae/tic-tac-toe) 
- [GitHub - gothinkster/realworld: The mother of all demo apps - Exemplary fullstack Medium.com clone powered by React, Angular, Node, Django, and many more 🏅](https://github.com/gothinkster/realworld) 
- [Progressive React](https://houssein.me/progressive-react) 
                                                              
