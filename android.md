# Termux

- nginx에서 auth_basic을 사용할때 termux에 htpasswd 패키지가 없어서 .htpasswd 파일을 만들지 못한다. 다른 Linux 머신에서 .htpasswd 파일을 만든 뒤 termux로 가져오자. 
- `File has unexpected size Mirror sync in progress?` 발생하는 경우 `apt clean` , `pkg clean` 실행 및 termux restart 후 `pkg update` 실행



## install jupyterlab

```sh
pkg install clang python-dev libzmq libzmq-dev
pip install jupyterlab
```



# MIUI Config

- 앱 비활성화 : 아래 여러 방법 중 하나를 선택. 아래로 내려갈수록 설정 범위가 넓음
    - 설정 > 앱 > 앱 관리 > 앱 선택
    - FX > Apps > 앱 선택 > Control Panel
    - 노바 런처 > 위젯 > 액티비티 > 설정 > ManageApplications
    - 건강, 검색, 기본 프린터 서비스, 나침반, 동영상, 마켓 피드백 에이전트, 메모, 메일, 브라우저, 사용자 사전, 소리, 슬라이드형 배경화면, 음악, 캘린더, 클라우드 프린트, 태그, 파일 탐색기, Android Auto, Bookmark Provider, Gboard, Google TTS 엔진, Mi Doc Viewer, OK Google enrollment, Print Service Recommendation Service, X Google enrollment
- 앱 자동시작
    - 설정 > 앱 > 앱 관리 > 권한 > 자동 시작
    - 보안 > 앱 관리 > 권한 > 자동 시작
    - 내 기기 찾기, 어시스턴트, 카카오뱅크, 카카오톡, 캘린더, 피트니스, Chrome, FolderSync Pro, Gmail, Google 한국어 입력기, KB국민카드, Vanced microG
- 앱 잠금
    - 멀티 태스킹 > Long Click > 앱 잠금
    - 보안 > 설정 > 속도 향상 > 앱 잠금
    - 어시스턴트, Vanced microG, Gmail, 피트니스, Chrome, 카카오뱅크, 캘린더, KB국민카드, 카카오톡
- MIUI 배터리 절약
    - 설정 > 앱 > 앱 관리 > 앱 선택
    - 설정 > 배터리 & 성능 > 설정 > 앱 배터리 절약
    - 백그라운드 앱 제한 : 뷰티포인트, AhnLab V3 Mobile Plus, ISP/페이북
    - 제한 없음 : 어시스턴트, 카카오뱅크, 카카오톡, 캘린더, 피트니스, Chrome, KB국민카드, Gmail, Vanced microG
    - 여기서 "제한 없음"으로 설정하면 아래 배터리 최적화에서도 "최적화되지 않음"으로 설정된다.
- 배터리 최적화
    - 노바 런처 > 위젯 > 액티비티 > 설정 > ManageApplications
    - 설정 > 개인정보 보호 > 관리 > 특별 권한 > 배터리 최적화
- 개발자 옵션
    - 설정 > 내 기기 > 전체 사양 > MIUI 버전 5연타
    - 설정 > 추가 설정 > 개발자 옵션
    - 애니메이션 0.5x, 로거 버퍼 크기 256KB > OFF, 로그 레벨 Warn > OFF, 적응형 알림 우선순위 ON > OFF, 위험한 기능 알림 ON > OFF, 시스템 추적 > 디버깅 가능한 앱 추적 ON > OFF
- 게임 터보, 동영상 도구 상자, 위치기록 끄기. 지역 미국으로 설정
- 설정 > 개인정보 보호 > 특별 권한 > 적응형 알림 > 끄기
- Xiaomi Overclock
- 시스템 앱 삭제

```sh
adb kill-server
adb shell

pm list packages

pm uninstall -k --user 0 com.android.bookmarkprovider
pm uninstall -k --user 0 com.android.calendar
pm uninstall -k --user 0 com.android.dreams.basic
pm uninstall -k --user 0 com.android.dreams.phototable
pm uninstall -k --user 0 com.android.hotwordenrollment.okgoogle
pm uninstall -k --user 0 com.android.hotwordenrollment.xgoogle
pm uninstall -k --user 0 com.android.htmlviewer
pm uninstall -k --user 0 com.android.midrive
pm uninstall -k --user 0 com.android.providers.userdictionary
pm uninstall -k --user 0 com.android.quicksearchbox
pm uninstall -k --user 0 com.android.wallpaper.livepicker
pm uninstall -k --user 0 com.dsi.ant.server
pm uninstall -k --user 0 com.duokan.phone.remotecontroller.peel.plugin
pm uninstall -k --user 0 com.facebook.appmanager
pm uninstall -k --user 0 com.facebook.services
pm uninstall -k --user 0 com.facebook.system
pm uninstall -k --user 0 com.google.android.apps.tachyon
pm uninstall -k --user 0 com.google.android.apps.turbo
pm uninstall -k --user 0 com.google.android.feedback
pm uninstall -k --user 0 com.google.android.googlequicksearchbox
pm uninstall -k --user 0 com.google.android.inputmethod.latin
pm uninstall -k --user 0 com.google.android.markup
pm uninstall -k --user 0 com.google.android.marvin.talkback
pm uninstall -k --user 0 com.google.android.music
pm uninstall -k --user 0 com.google.android.soundpicker
pm uninstall -k --user 0 com.google.android.tts
pm uninstall -k --user 0 com.google.android.videos
pm uninstall -k --user 0 com.google.android.webview
pm uninstall -k --user 0 com.google.ar.lens
pm uninstall -k --user 0 com.mfashiongallery.emag
pm uninstall -k --user 0 com.mi.android.globalpersonalassistant
pm uninstall -k --user 0 com.milink.service
pm uninstall -k --user 0 com.mipay.wallet.id
pm uninstall -k --user 0 com.mipay.wallet.in
pm uninstall -k --user 0 com.miui.analytics
pm uninstall -k --user 0 com.miui.android.fashiongallery
pm uninstall -k --user 0 com.miui.bugreport
pm uninstall -k --user 0 com.miui.cloudbackup
pm uninstall -k --user 0 com.miui.cloudservice
pm uninstall -k --user 0 com.miui.cloudservice.sysbase
pm uninstall -k --user 0 com.miui.fm
pm uninstall -k --user 0 com.miui.fmservice
pm uninstall -k --user 0 com.miui.hybrid
pm uninstall -k --user 0 com.miui.hybrid.accessory
pm uninstall -k --user 0 com.miui.micloudsync
pm uninstall -k --user 0 com.miui.notes
pm uninstall -k --user 0 com.miui.personalassistant
pm uninstall -k --user 0 com.miui.player
pm uninstall -k --user 0 com.miui.touchassistant
pm uninstall -k --user 0 com.miui.translation.kingsoft
pm uninstall -k --user 0 com.miui.translation.xmcloud
pm uninstall -k --user 0 com.miui.translation.youdao
pm uninstall -k --user 0 com.miui.translationservice
pm uninstall -k --user 0 com.miui.videoplayer
pm uninstall -k --user 0 com.xiaomi.glgm
pm uninstall -k --user 0 com.xiaomi.joyose
pm uninstall -k --user 0 com.xiaomi.midrop
pm uninstall -k --user 0 com.xiaomi.mipicks
pm uninstall -k --user 0 com.xiaomi.mirecycle
pm uninstall -k --user 0 com.xiaomi.payment
pm uninstall -k --user 0 com.xiaomi.scanner
pm uninstall -k --user 0 com.google.android.apps.wellbeing
pm uninstall -k --user 0 com.mi.health
pm uninstall -k --user 0 com.miui.miservice
pm uninstall -k --user 0 com.google.android.apps.cloudprint
pm uninstall -k --user 0 com.miui.greenguard
pm uninstall -k --user 0 cn.wps.xiaomi.abroad.lite
pm uninstall -k --user 0 com.mi.android.globalminusscreen
pm uninstall -k --user 0 com.google.android.projection.gearhead
pm uninstall -k --user 0 com.miui.mishare.connectivity
pm uninstall -k --user 0 com.miui.yellowpage
pm uninstall -k --user 0 com.miui.msa.global
pm uninstall -k --user 0 com.mi.globalbrowser

# Wifi 접속 시 ID/PW를 입력해야 하는 환경에서 문제가 될 수도 있음 (테스트 필요)
pm uninstall -k --user 0 com.android.browser

# 삭제 시 Exchange Server 연동 불가
pm uninstall -k --user 0 com.android.email

# 삭제 시 녹음기 앱 강제종료됨
pm uninstall -k --user 0 com.xiaomi.micloud.sdk

# 배터리 정보 보기
pm uninstall -k --user 0 com.miui.powerkeeper
pm uninstall -k --user 0 com.xiaomi.powerchecker

# Mi Device Unlock 유지
pm uninstall -k --user 0 com.xiaomi.account
pm uninstall -k --user 0 com.xiaomi.finddevice
```



# Pixel Experience (Android 10) Config

- flash TWRP recovery in fastboot
    - [https://drive.google.com/file/d/1W2tF5wgYVbwl7ZL5AIwKwZa0gu4zyppF/view?usp=drive_open](https://drive.google.com/file/d/1W2tF5wgYVbwl7ZL5AIwKwZa0gu4zyppF/view?usp=drive_open)
    - recovery-twrp一键刷入工具.bat 실행 후 2번 → 엔터키 연속으로 누르면 자동으로 처리됨
- flash pixel experience rom in recovery
    - [https://download.pixelexperience.org/lavender](https://download.pixelexperience.org/lavender)
    - [pixelexperience_lavender-10.0-20191109-1552-official.zip](https://download.pixelexperience.org/changelog/lavender/PixelExperience_lavender-10.0-20191109-1552-OFFICIAL.zip)
    - Rom 설치 후 reboot 하여 초기 설정을 진행한다. 이 때 /data 영역은 encrypt 되며 이후 TWRP recovery에서는 decrypt 불가능하다. 
- flash OrangeFox recovery in fastboot
    - OrangeFox recovery를 통해 /data 영역을 decrypt 한다. 그래야만 이후 작업이 가능하다. 
    - [https://files.orangefox.tech/OrangeFox-Stable/lavender/](https://files.orangefox.tech/OrangeFox-Stable/lavender/)
    - [https://drive.google.com/file/d/1M5881rdvAlqJA9rF5wwAUR8KL5cuTPa8/view](https://drive.google.com/file/d/1M5881rdvAlqJA9rF5wwAUR8KL5cuTPa8/view)
    - 압축 풀고 fastboot flash recovery recovery.img
    - 이후 recovery 모드로 부팅하여 flash OrangeFox-R10.0_2-Stable-lavender.zip
- flash unroot in recovery
    - 금융앱을 사용하기 위해서는 언루팅 상태로 체크되어야 한다. 
    - [https://drive.google.com/drive/folders/0ByHvsPuqi27DMHp6ZjQ4Vk1YaDQ](https://drive.google.com/drive/folders/0ByHvsPuqi27DMHp6ZjQ4Vk1YaDQ)
    - 위 unroot zip들을 차례대로 flashing
- Install Magisk in recovery
    - reboot 후 Magisk Manager 실행하고 각 금융앱, V3 등에서 Magisk Hide 처리하면 금융앱 사용이 가능해진다.
    - Youtube Vanced - Magisk Repo
    - Call Recorder - SKVALEX
- 이후 시스템 및 애플리케이션 설정



# Issues

## Redmi Note 9S MIUI 12 EU ROM Issues

- 긴급재난문자 안옴
- 차단해도 문자옴
- Widevine L3



# References

- [MIUI Fastboot Roms](https://c.mi.com/oc/miuidownload/detail?guide=2) 
- [MIUI EU Roms](https://xiaomi.eu/) 
- [Google Camera Port: BSG apks](https://www.celsoazevedo.com/files/android/google-camera/dev-bsg/) 
- [ADB, Fastboot for Windows](https://forum.xda-developers.com/showthread.php?t=2588979) 
- [선탑재 앱, ADB로 루팅 없이 제거해보자.](https://brunch.co.kr/@ericbaek/32) 
- [홍미노트7 한글 커스텀롬(eu롬) 설치](https://ruinses.tistory.com/1839) 
- [홍미노트7 중국롬 EU ROM으로 설치 및 셋팅](http://blog.naver.com/PostView.nhn?blogId=choboangel&logNo=221509406669&categoryNo=1&parentCategoryNo=0&viewDate=&currentPage=1&postListTopCurrentPage=1&from=postView) 
- [The Open GApps Project](https://opengapps.org/) 
- [Xiaomi Firmware Updater](https://xiaomifirmwareupdater.com/) 
- [Call Recorder - ACR APKs - APKMirror](https://www.apkmirror.com/apk/nll/call-recorder-acr/) 
- [GCam Hub](https://www.xda-developers.com/google-camera-port-hub/) 
- [Termux - MariaDB](https://wiki.termux.com/wiki/MariaDB)
