# Chrome 80+ SameSite 이슈 대응

- Chrome 80+ 부터 SameSite 기본값이 None => Lax로 변경됨. 
- 기존에는 a.com 안에 b.com이 iframe이나 img 등으로 들어 있었을 경우 b.com의 쿠키값들이 그대로 전달되었지만, SameSite=Lax가 되면 Cross Site로는 쿠키값이 전달되지 않는다.
- 기존과 같이 SameSite=None으로 유지하고 싶은 경우 Set-Cookie 시 명시적으로 SameSite=None을 줘야 한다. 
- 또한 SameSite=None 적용시 반드시 해당 쿠키에 Secure 속성이 있어야 하니 SSL/TLS 설정을 해준다. 

```nginx
# Apache
ln -s /etc/apache2/mods-available/headers.load /etc/apache2/mods-enabled/headers.load

<VirtualHost *:80>
    <IfModule mod_headers.c>
        Header set Set-Cookie "JSESSIONID=test"
        Header edit Set-Cookie ^(.*)$ "$1; Secure; SameSite=None;"
    </IfModule>
</VirtualHost>
```

```nginx
# Nginx
location / {
  proxy_cookie_path / "/; Secure; SameSite=None";
}
```

```
# Tomcat 8.5+
<Context>
  <CookieProcessor sameSiteCookies="none"/>
</Context>
```

- 어떤 도메인이 SameSite고 어떤 도메인이 CrossSite인가? 1.google.com, 2.google.com 처럼 서브 도메인만 다르면 SameSite 맞다. 즉 이 경우에는 SameSite=Lax or Strict 여도 쿠키값이 잘 전달된다. test01.dgdsingen.com, test02.dgdsingen.com 2개 도메인 만들고 test01에서 test02를 iframe으로 로드봤는데 쿠키 SameSite 값과 상관없이 모두 쿠키값이 잘 전달된다. 그러나 http://dhost 에서 test02.dgdsingen.com을 iframe으로 로드하는 경우 SameSite=None인 쿠키만 전달된다. 

- 그러나 정말 서브 도메인이면 전부 SameSite일까? 예를 들어 1.github.io, 2.github.io 는 서브 도메인이라 SameSite일 것 같지만 사실은 CrossSite다. 

- [Public suffix](https://publicsuffix.org/list/public_suffix_list.dat)에 TLD(Top Level Domain) 목록이 있다. 여기를 보면 github.io가 있다. 즉 google.com의 TLD는 com이지만 github.io의 TLD는 github.io다. 

- Registrable Domain은 TLD+1의 도메인을 의미한다. 예를 들면 google.com에서 TLD는 com, Registrable Domain은 google.com이다. 

- Set-Cookie Domain의 Registrable Domain과 브라우저 주소창의 Registrable Domain이 같을 때 SameSite가 된다. 

    > [web.dev](https://web.dev/samesite-cookies-explained/#explicitly-state-cookie-usage-with-the-samesite-attribute) 
    >
    > Key Term: If the user is on [www.web.dev](http://www.web.dev/) and requests an image from static.web.dev then that is a same-site request. The public suffix list defines this, so it’s not just top-level domains like .com but also includes services like github.io. That enables your-project.github.io and my-project.github.io to count as separate sites.

- 참고로 [Draft 7](https://datatracker.ietf.org/doc/html/draft-ietf-httpbis-rfc6265bis-07) 부터는 Scheme(HTTP, HTTPS)까지 같아야 SameSite다. 



- SameSite=None
    - 반드시 해당 쿠키에 Secure 속성이 있어야 하니 SSL/TLS 설정을 해준다.
- SameSite=Lax
    - Top Level Navigations(which use a safe HTTP method) 일 경우에만 cross-site 상황에서 쿠키 생성 및 전달을 허용한다.
    - safe HTTP method = "GET", "HEAD", "OPTIONS", "TRACE" (RFC7231)
- SameSite=Strict
    - 모든 cross-site subresource requests, cross-site nested navigations 상황에서 쿠키 생성 및 전달을 허용하지 않는다.
    - 여기서 subresource requests 는 html 파일 안에서 js, css, img 등의 리소스 가져올 때를 말한다.
- Set-Cookie 에 SameSite=Strict; SameSite=Lax; SameSite=None 여러개 붙은 경우 마지막 SameSite=None가 사용된다.
- Lax-Allowing-Unsafe 가 새롭게 생겼는데 HTTP 요청 method와 상관없이 top-level request 일 때 cross-site 상황에서 쿠키 생성 및 전달이 가능해진다. 예를 들어 로그인 로직 플로우에서 cross-site top-level POST 요청을 하게 되면 기존 Lax 에서는 적절하지 않게 된다. 그렇다고 None 으로 하기엔 모든 cross-site 에 가능해지니 나름의 절충안이다. 그래서 Lax 알고리즘이 아래와 같이 바뀌었다.
    - 쿠키 same-site-flag 가 None 이 아닐때, 그리고 HTTP 요청이 cross-site 이면 정책에 따라 쿠키 전달을 차단하는데 아래 3개 구문을 모두 만족할 때는 예외가 된다.
    - 첫째, same-site-flag 가 Lax 이거나 Default 다.
    - 둘째, HTTP 요청의 메서드가 safe 메서드이거나 브라우저의 Lax-Allowing-Unsafe 요구사항을 만족한다.
    - 셋째, top-level navigation 일 때다. 
- 하지만 Lax-Allowing-Unsafe 모드는 Lax 모드의 관대한 변형(permissive variant) 이다. 그래서 CSRF 공격에 대한 방어력이 낮을 수 밖에 없다. 따라서 궁극적으로는 모두 Lax 로 가야하고 Lax-Allowing-Unsafe 는 임시 조치로 간주되어야 한다.



# Service Worker

## fetch 시 http 로딩 이슈

예를 들어 아래와 같은 Service Worker를 사용한다고 하자. 


```js
/sw.html
if ('serviceWorker' in navigator) {
    window.addEventListener('load', function() {
        navigator.serviceWorker.register('/sw.js', {scope: '/'}).then(function(registration) {
            console.log('ServiceWorker registration successful with scope: ', registration.scope);
        }).catch(function(err) {
            console.log('ServiceWorker registration failed: ', err);
        });
    });
}

/sw.js
self.addEventListener('install', function (event) {
    event.waitUntil(
        caches.open("v1")
            .then(function (cache) {
                return cache.addAll([
                    '/a.png',
                    '/b.font'
                ]);
            })
    );
});

self.addEventListener('fetch', function (event) {
    event.respondWith(
        caches.match(event.request)
            .then(function (response) {
                return response || fetch(event.request);
            })
    );
});
```


만약 이때 /a.png를 [http://test.com/a.png와](http://test.com/a.png와) 같이 http 프로토콜로 하드코딩되어 있다고 하면, Service Worker가 실행되는 웹사이트는 무조건 https 프로토콜을 사용하기 때문에 sw.js에서 http 리소스를 fetch할 수 없다. (오류가 발생하며 fetch 실패)

아래와 같이 img src를 변경하는 등의 작업으로 re-fetch를 실행하면 http 리소스들을 정상적으로 fetch할 수 있다. 그러나 http 리소스 fetch시 발생한 오류 메시지는 그대로 남는데… Service Worker의 fetch 이벤트 발생 전에 아래 스크립트를 실행할 수는 없을까?


```js
var imgs = document.getElementsByTagName('img')
for (i=0; i<imgs.length; i++) {
    var src = imgs[i].src ? imgs[i].src : '';
    if (src.indexOf('http:') > -1)
        imgs[i].src = src.replace(/http:/g, 'https:');
}
```


그럴때는 fetch할때 아래와 같이 처리해주면 된다. 


```js
self.addEventListener('fetch', function (event) {
    event.respondWith(
        caches.match(event.request)
            .then(function (response) {
                if (response) {
                    return response;
                } else {
                    var request = event.request.url.indexOf('http:') > -1 ? 
                        new Request(event.request.url.replace(/http:/g, 'https:'), {mode:'no-cors'}) : 
						event.request;

                    return fetch(request);
                }
            })
    );
});
```

# References
- [MDN](https://developer.mozilla.org/ko/) 
- [web.dev](https://web.dev/) 
- [Google Dev](https://developers.google.com/) 
- [Google Dev Library](https://devlibrary.withgoogle.com/) 
- [브라우저는 어떻게 동작하는가?](https://d2.naver.com/helloworld/59361) 
- [AMP](https://www.ampproject.org/) 
- [W3C](https://www.w3.org/) 
- [W3Schools](http://www.w3schools.com/) 
- [The Web Robots Pages](http://www.robotstxt.org/) 
- [Can I use](http://caniuse.com/) 
- [OverAPI](http://overapi.com/) 
- [User Agent String](http://www.useragentstring.com/pages/useragentstring.php?name=All) 
- [Kakao Developers - 카카오스토리 공유하기 ](https://developers.kakao.com/docs/social-plugins/story-share) 
- [Kakao Developers - 미리보기 Cache 삭제하기 ](https://developers.kakao.com/docs/cache) 
- [배민 API GATEWAY - spring cloud zuul 적용기](http://woowabros.github.io/r&d/2017/06/13/apigateway.html) 
- [Webhooks - 그래프 API](https://developers.facebook.com/docs/graph-api/webhooks/?locale=ko_KR) 
- [REST API의 이해와 설계-#1 개념 소개](http://bcho.tistory.com/953) 
- [Referrer-Policy - HTTP | MDN](https://developer.mozilla.org/ko/docs/Web/HTTP/Headers/Referrer-Policy) 
- [schema.org](https://schema.org/) 
- [인스타그램 API – ACCESS TOKEN 발급](https://studio-jt.co.kr/%EC%9D%B8%EC%8A%A4%ED%83%80%EA%B7%B8%EB%9E%A8-api-access-token-%EB%B0%9C%EA%B8%89%EB%B0%9B%EA%B8%B0/) 
- [A Tinder Progressive Web App Performance Case Study](https://medium.com/@addyosmani/a-tinder-progressive-web-app-performance-case-study-78919d98ece0) 
- [HTTPS 전환 과정에서 read timeout 오류 해결 과정](http://d2.naver.com/helloworld/1469717) 
- [Facebook Graph API Explorer](https://developers.facebook.com/tools/explorer/) 
- [공유 디버거 - Facebook for Developers](https://developers.facebook.com/tools/debug/sharing) 
- [API Guidelines](https://adidas-group.gitbooks.io/api-guidelines/content/) 
- [깃허브(GitHub), L4 로드 밸런서 GLB 디렉터 오픈소스로 공개](https://www.44bits.io/ko/post/news--github-release-glb-github-load-balancer-as-open-source) 
- [Preload, Prefetch And Priorities in Chrome](https://medium.com/@koh.yesl/preload-prefetch-and-priorities-in-chrome-15d77326f646) 
- [웹 폰트 사용과 최적화의 최근 동향](https://d2.naver.com/helloworld/4969726) 
- [PageSpeed Insights](https://developers.google.com/speed/pagespeed/insights/?hl=ko) 
- [WebPageTest - Website Performance and Optimization Test](https://www.webpagetest.org/) 
- [Documenting your REST API](https://gist.github.com/iros/3426278) 
- [인스타그램 API – 인스타그램 OPEN API의 업데이트와 Instagram Graph API | 스튜디오 제이티](https://studio-jt.co.kr/%EC%9D%B8%EC%8A%A4%ED%83%80%EA%B7%B8%EB%9E%A8-open-api%EC%9D%98-%EC%97%85%EB%8D%B0%EC%9D%B4%ED%8A%B8%EC%99%80-instagram-graph-api/) 
- [SameSite cookie recipes](https://web.dev/samesite-cookie-recipes/) 
- [GitHub - GoogleChromeLabs/samesite-examples](https://github.com/GoogleChromeLabs/samesite-examples) 
- [RESTful API 설계 가이드](https://sanghaklee.tistory.com/57) 
- [Cookie SameSite 기본편](https://yangbongsoo.tistory.com/5) 
- [Cookie SameSite Lax 모드 업데이트 정리](https://yangbongsoo.tistory.com/22) 
- [CA certificates extracted from Mozilla](https://curl.se/docs/caextract.html) 
- [Adding trusted root certificates to the server](https://manuals.gfi.com/en/kerio/connect/content/server-configuration/ssl-certificates/adding-trusted-root-certificates-to-the-server-1605.html) 
- [REST Architecture 논문](https://www.ics.uci.edu/~fielding/pubs/dissertation/top.htm) 

## PWA
- [Service Worker Spec](https://www.w3.org/TR/service-workers-1/) 
- [Push API Spec](https://www.w3.org/TR/push-api/) 
- [Service Worker Resources](https://jakearchibald.github.io/isserviceworkerready/resources.html) 
- [ServiceWorker Cookbook](https://serviceworke.rs/) 
- [Service Worker Sample: Pre-fetching Resources During Registration](https://googlechrome.github.io/samples/service-worker/prefetch/index.html) 
- [samples/service-worker at gh-pages · GoogleChrome/samples · GitHub](https://github.com/GoogleChrome/samples/tree/gh-pages/service-worker) 
- [서비스 워커: 소개  |  Web  |  Google Developers](https://developers.google.com/web/fundamentals/primers/service-workers/?hl=ko) 
- [서비스워커 cache API 업데이트  |  Web  |  Google Developers](https://developers.google.com/web/updates/2015/09/updates-to-cache-api?hl=ko) 
- [웹 프로젝트는 PWA이어야 한다](http://webactually.com/2017/09/%EC%9B%B9-%ED%94%84%EB%A1%9C%EC%A0%9D%ED%8A%B8%EB%8A%94-pwa%EC%9D%B4%EC%96%B4%EC%95%BC-%ED%95%9C%EB%8B%A41/) 
- [Chrome, the Background Sync API and exponential backoff](https://notes.eellson.com/2018/02/11/chrome-the-background-sync-api-and-exponential-backoff/) 
- [Prioritizing Your Resources with link rel=&#39;preload&#39;  |  Web  |  Google Developers](https://developers.google.com/web/updates/2016/03/link-rel-preload) 
- [Introduction to Push Notifications  |  Web  |  Google Developers](https://developers.google.com/web/ilt/pwa/introduction-to-push-notifications) 
- [웹 앱에 푸시 알림 추가  |  Web  |  Google Developers](https://developers.google.com/web/fundamentals/codelabs/push-notifications/?hl=ko) 
- [node.js, FCM, 웹앱(서비스워커) 으로 푸시 구현하기. – Sejong De Kang – Medium](https://medium.com/@sejongdekang/node-js-fcm-%EC%9B%B9%EC%95%B1-%EC%84%9C%EB%B9%84%EC%8A%A4%EC%9B%8C%EC%BB%A4-%EC%9C%BC%EB%A1%9C-%ED%91%B8%EC%8B%9C-%EA%B5%AC%ED%98%84%ED%95%98%EA%B8%B0-43c49b761dba) 
- [프로그레시브 웹앱(PWA) 푸시 알람 A to Z • Captain Pangyo](https://joshua1988.github.io/web-development/pwa/pwa-push-noti-guide/) 
- [The Web Push Protocol  |  Web Fundamentals  |  Google Developers](https://developers.google.com/web/fundamentals/push-notifications/web-push-protocol) 
- [Workbox](https://developers.google.com/web/tools/workbox/) 
- [웹 앱에 푸시 알림 추가  |  Web  |  Google Developers](https://developers.google.com/web/fundamentals/codelabs/push-notifications?hl=ko) 
- [Push Companion](https://web-push-codelab.glitch.me/) 
- [Build a PWA Using Only Vanilla JavaScript - Level Up Coding](https://levelup.gitconnected.com/build-a-pwa-using-only-vanilla-javascript-bdf1eee6f37a) 
- [PWABuilder](https://www.pwabuilder.com/) 
- [PWA Universal Builder](https://pwa.cafe/) 

## Google
- [SEO 초보자 가이드](https://support.google.com/webmasters/answer/7451184) 
- [TechnicalSEO.com](https://technicalseo.com/) 
- [How To Remove Pages From Google in 12 Hours](http://www.orchidbox.com/how-to-remove-pages-from-google-in-12-hours) 
- [Google에서 정보 삭제](https://support.google.com/webmasters/answer/6332384?hl=ko) 
- [다지역 및 다국어 사이트 관리](https://support.google.com/webmasters/answer/182192?hl=ko) 

