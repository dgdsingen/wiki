# Chrome

## Network logging

- 개발자 콘솔은 현재 탭의 요청은 잘 볼 수 있지만, 새 창이나 탭이 열리는 경우 트래픽을 추적하기는 어렵다. 
- chrome://net-export/ 에서 브라우저 레벨에서 네트워크 트래픽을 캡쳐할 수 있다. (Firefox는 about:networking#logging)



## Flags

```
chrome://flags/#explore-sites
OFF

chrome://flags/#smooth-scrolling
OFF

chrome://flags/#enable-gpu-rasterization
ON

chrome://flags/#enable-zero-copy
ON

chrome://flags/#read-later
OFF

chrome://flags/#side-panel
OFF

chrome://flags/#sharing-qr-code-generator
ON

chrome://flags/#interest-feed-v2
OFF
```



## Headless Chrome

```sh
"C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" --headless --no-sandbox --disable-setuid-sandbox --disable-gpu --hide-scrollbars --disable-web-security --print-to-pdf=C:\Users\dgdsi\Downloads\t.pdf --window-size=1280,15000 https://www.test.com
```



## Extensions

- https://chrome.google.com/webstore/detail/google-translate/aapbdbdomjkkjkaonfhkkikfgjllcleb?hl=ko
- https://chrome.google.com/webstore/detail/ublock-origin/cjpalhdlnbpafiamejdnhcphjbkeiagm?hl=ko
- https://chrome.google.com/webstore/detail/imacros-for-chrome/cplklnmnlbnpmjogncfgfijoopmnlemp?hl=ko
- https://chrome.google.com/webstore/detail/editthiscookie/fngmhnnpilhplaeedifhccceomclgfbg?hl=ko
- https://chrome.google.com/webstore/detail/save-to-google-drive/gmbmikajjgmnabiglmofipeabaddhgne?hl=ko
- https://chrome.google.com/webstore/detail/office-editing-for-docs-s/gbkeegbaiigmenfmjfclcdgdpimamgkj?hl=ko
- https://chrome.google.com/webstore/detail/linkclump/lfpjkncokllnfokkgpkobnkbkmelfefj?hl=ko
- https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=ko
- https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=ko
- https://chrome.google.com/webstore/detail/accessibility-insights-fo/pbjjkligggfmakdaogkfomddhfmpjeni?hl=ko



# Chrome OS

## OpenVPN

Obtain a .ovpn file from your network administrator or VPN provider and save it to your Downloads folder.

Enter the following in the CLI (Ctrl+alt+t):

- `shell`
- `sudo su`
- `Password: enter "chrome"`
- `sudo stop shill`
- `sudo start shill BLACKLISTED_DEVICES=tun0`
- `cd /home/chronos/user/Downloads`
- `openvpn --config [filename].ovpn`
- `Enter Auth Username: enter username assigned to the ovpn file`
- `Enter Auth Password: enter password assigned to the ovpn file`
- `The connection is complete when "Initialization Sequence Completed" is displayed.`
- `Leave the CLI window open and connect to the remote computer using Chrome RDP app from the Chrome Web Store.`
- `To disconnect, go back to the CLI tab and enter Ctrl+c`

While it works, it is too slow for my use. Maybe someone else will have better results.

좀 더 편하게 사용하고 싶다면 ovpn 파일을 `/usr/local/bin/dgdsingen.ovpn`로 옮기고 실행파일을 아래 경로에 작성한다.


```sh
/usr/local/bin/vpn
sudo stop shill
sudo start shill BLACKLISTED_DEVICES=tun0
sudo openvpn --config /usr/local/bin/dgdsingen.ovpn
```



## Crostini


### 한글 입력

```bash
# Locale
sudo vi /etc/locale.gen
#ko_KR.UTF-8 UTF-8 주석 해제

sudo locale-gen

# fcitx
sudo apt install fcitx-hangul
fcitx-autostart
fcitx-configtool
# + 버튼 누르고 Hangul 추가

# autostart
sudo vi /etc/systemd/user/cros-garcon.service.d/cros-garcon-override.conf
Environment="GTK_IM_MODULE=fcitx"
Environment="QT_IM_MODULE=fcitx"
Environment="XMODIFIERS=@im=fcitx"

vi ~/.sommelierrc
/usr/bin/fcitx-autostart

# im-config
sudo vi /etc/default/im-config
IM_CONFIG_DEFAULT_MODE=fcitx

im-config
# default 선택
```



## Crouton

/usr/bin/fcitx-autostart


### install


```sh
sudo sh ~/Downloads/crouton -t xfce
sudo sh ~/Downloads/crouton -r trusty -t xfce,xiwi,touch
```

### uninstall


```sh
sudo edit-chroot -a
sudo edit-chroot -d trusty
```

## JDK

Download the latest Oracle JDK from here : [http://www.oracle.com/technetwork/java/javase/downloads/index.html](http://www.oracle.com/technetwork/java/javase/downloads/index.html)


```sh
sudo tar zxvf jdk-8u91-linux-arm32-vfp-hflt.tar.gz -C /usr/lib/jvm/

sudo update-alternatives --install /usr/bin/java java /usr/lib/jvm/jdk1.8.0_91/bin/java 1081
sudo update-alternatives --install /usr/bin/javac javac /usr/lib/jvm/jdk1.8.0_91/bin/javac 1081
sudo update-alternatives --install /usr/bin/javaws javaws /usr/lib/jvm/jdk1.8.0_91/bin/javaws 1081
sudo update-alternatives --install /usr/lib/mozilla/plugins/libnpjp2.so libnpjp2.so /usr/lib/jvm/jdk1.8.0_91/jre/lib/amd64/libnpjp2.so 1081

sudo update-alternatives --display java
```

## Input Method


```sh
sudo apt install fcitx
```



# References

- [Chrome Status](https://www.chromestatus.com/features) 
- [chromeOS.dev](https://chromeos.dev/en) 
- [크롬 항상 이러한 유형의 링크를 연결된 앱에서 열기 해제 방법](http://www.inven.co.kr/mobile/board/powerbbs.php?come_idx=2152&my=chu&l=26780) 
- [chrome://system](chrome://system/) 
- [Crostini libreoffice 6 installation frustration](https://www.reddit.com/r/Crostini/comments/aie4zk/libreoffice_6_installation_frustration/een5j87/) 
- [리눅스 동영상 플레이어 mplayer 사용법](https://hiseon.me/linux/mplayer-tutorial/) 
- [Chromebook Comparison](https://www.starryhope.com/chromebooks/chromebook-comparison-chart/) 
- [sebanc/brunch: Boot ChromeOS on x86_64 PC (supports most Intel CPU/GPU or AMD Stoney Ridge)](https://github.com/sebanc/brunch) 
