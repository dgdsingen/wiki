# Issues

## HSTS로 인한 사이트 접속 불가

> https://campus.barracuda.com/product/ContentShield/doc/73699516/how-to-clear-the-hsts-cache-or-disable-hsts-for-firefox/

Firefox 최신버전(ex: v101.0.1) 설치 후 회사 PC 등에서 Proxy를 통해 https://accouns.google.com와 같은 사이트를 접속하는 경우 HSTS 에러가 나며 더 이상 진행이 안될 수 있다.

이 때는 브라우저 주소창에 `abount:config` 를 치고 들어가서 `security.enterprise_roots.enabled` 의 값을 `true` 로 변경한다.



## 폐쇄망에서 업데이트 팝업 끄기

> https://winaero.com/disable-updates-firefox-63-above/

```sh
# disable_firefox_updates.reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\Software\Policies\Mozilla\Firefox]
"DisableAppUpdate"=dword:00000001

# enable_firefox_updates.reg
Windows Registry Editor Version 5.00

[HKEY_LOCAL_MACHINE\Software\Policies\Mozilla\Firefox]
"DisableAppUpdate"=-
```

