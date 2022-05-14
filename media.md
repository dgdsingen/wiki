# Video

## Convert to mp4 by avconv

```sh
sudo apt install libavcodec-extra
avconv -i in.avi -c:v libx264 out.mp4
```

## YouTube downloader

```sh
################# install #################
# To install it right away for all UNIX users (Linux, OS X, etc.), type:
sudo curl https://yt-dl.org/downloads/2015.02.18.1/yt-dlp -o /usr/local/bin/yt-dlp
sudo chmod a+x /usr/local/bin/yt-dlp

# If you do not have curl, you can alternatively use a recent wget:
sudo wget https://yt-dl.org/downloads/2015.02.18.1/yt-dlp -O /usr/local/bin/yt-dlp
sudo chmod a+x /usr/local/bin/yt-dlp

# You can also use pip:
sudo apt-get install python-pip
sudo pip install --upgrade youtube_dl

# This command will update yt-dlp if you have already installed it. See the pypi page for more information.
# You can use Homebrew if you have it:
brew install yt-dlp

# To check the signature, type:
sudo wget https://yt-dl.org/downloads/2015.02.18.1/yt-dlp.sig -O yt-dlp.sig
gpg --verify yt-dlp.sig /usr/local/bin/yt-dlp
rm yt-dlp.sig

################# youtube download #################
# check downloadable formats
yt-dlp -F http://www.youtube.com/watch?v=HRIF4_WzU1w

# download
yt-dlp -f 141 http://www.youtube.com/watch?v=HRIF4_WzU1w
```

## Convert Subtitle Formats

```sh
sudo apt-get install libsubtitles-perl

# smi -> srt
subs -c srt "${1}" -o "${1}.srt"
sed -i 's/ //g' "${1}.srt"

# srt -> smi
subs -c smi "${1}" -o "${1}.smi"
```

## MPlayer

### Config

```sh
subfont-encoding=unicode
unicode=yes
utf8=yes
 
fontconfig=1 
font=/usr/X11R6/lib/X11/fonts/truetype/gulim.ttf 
subcp=cp949
```

### Streaming via sftp

Linux에선 Nautilus에서 sftp를 연결해 들어가서 그냥 영상 실행하면 스트리밍된다. 자막도 알아서 불러온다.

Windows는 그런거 없다.

### Streaming via ftp

우선 ftp를 열어야 하는데, Passive Mode로 접속시 Passive Port 범위 접속이 안될때가 있다. 해결책은 다음을 참고 : [issues](http://docs.google.com/dev/web/server/issues.html)

ftp 연결이 정상적으로 되면 mplayer를 설치하고 다음과 같이 pipe를 사용해 streaming하면 된다.

wget ftp://id:pass@localhost:9000/video/Interstellar.mp4 -O - | ./mplayer.exe -cache 8192 -

아래는 동영상 파일 이름을 받아 그에 상응하는 자막 파일도 같이 불러와 스트리밍하는 스크립트다.

```sh
video="${1}"
smi=$(echo "${1}" | awk -F. '{print $1}').smi
ftp_url="ftp://$ID:$PW@localhost:9000/video"
 
wget "${ftp_url}"/"${smi}" -O -.smi
wget "${ftp_url}"/"${video}" -O - | ./mplayer.exe -cache 8192 -
 
wget "${ftp_url}"/"${smi}" --remote-encoding=utf-8 --local-encoding=utf-8 -O -.smi
wget "${ftp_url}"/"${video}" -O - | ./mplayer.exe -utf8 -subcp cp949 -font /cygdrive/c/mplayer-svn-37391-x86_64/mplayer/l_10646.ttf -cache 8192 -
```

### Streaming via http

ftp로 안되면 걍 http로 하자. 우선 video를 /var/www/html/video로 옮긴다. (심볼릭 링크가 아닌 원본이 옮겨져야 하며, 영상 파일 권한은 775로 주자. video 디렉토리는 770으로 줘도 된다)

이 스크립트는 'm Interstellar.mp4'와 같이 실행한다.

```sh
video="${1}"
smi=$(echo "${1}" | awk -F. '{print $1}').smi
http_url="http://test.com/video"
mplayer="/cygdrive/c/mplayer-svn-37391-x86_64/mplayer.exe"
 
wget "${http_url}"/"${smi}" -O -.smi
wget "${http_url}"/"${video}" -O - | "${mplayer}" -cache 8192 -
 
#wget "${http_url}"/"${smi}" --remote-encoding=utf-8 --local-encoding=utf-8 -O -.smi
#wget "${http_url}"/"${video}" -O - | "${mplayer}" -utf8 -subcp cp949 -font /cygdrive/c/mplayer-svn-37391-x86_64/mplayer/l_10646.ttf -cache 8192 
```

이 스크립트는 'm [http://test.com/video/Interstellar.mp4](http://test.com/video/Interstellar.mp4)'와 같이 실행한다.

```sh
video=$(echo "${1}" | awk -F/ '{print $5}')
smi=$(echo "${video}" | awk -F. '{print $1}').smi
http_url="http://test.com/video"
mplayer="/cygdrive/c/mplayer-svn-37391-x86_64/mplayer.exe"

wget "${http_url}"/"${smi}" -O -.smi
wget "${http_url}"/"${video}" -O - | "${mplayer}" -cache 8192 -
```

스크립트 실행 다 귀찮으면 걍 SMPlayer로 http 열자.

[http://test.com/video/](http://test.com/video/) 열어서 자막파일은 로컬에 다운받고, 영상은 링크를 복사해 SMPlayer에서 연다.

이후 로컬의 자막파일을 선택해주면 OK.

UI를 터치하기 편하도록 맞추려면 인터페이스 - GUI - 기본 GUI를 선택한 뒤, 메인 툴바를 편집하여 아이콘 크기를 80 이상으로 늘려주자.



# Audio

## ffmpeg

### Time cut

```sh
# ffmpeg -i {input_file} --ss {start_time} -to {end_time} {output_file}
ffmpeg -i input.m4a -ss 00:30 -to 03:30 output.m4a

# without re-encoding (faster!)
ffmpeg -i input.m4a -ss 00:30 -to 03:30 -vcodec copy -acodec copy output.m4a
```



## Convert m4a to mp3

/bin/mp3
```sh
sudo apt-get install libav-tools
 
# originalFile startTime duration
#avconv -ss startTime -i originalFile -t duration -codec mp3 destinationFile
 
avconv -i "${1}" "${1/%m4a/mp3}"
```

## id3 tag edit in terminal

### 기본 명령어

```sh
# id3v1
sudo apt-get install id3tool

id3tool -t title -a album -r artist file

# id3v2
sudo apt-get install id3v2

sudo vi /bin/id3
# remove all tags
id3v2 -D "${1}"

# title album artist
id3v2 -t "${2}" -A "${3}" -a "${4}" "${1}"
```

### 태그 입력 자동화

디렉토리 구조를 "아티스트/앨범/타이틀.mp3"로 잡고, 앨범 디렉토리에서 id3all를 실행하면 id3v2 태그가 자동으로 삽입된다.

/bin/id3

```sh
id3v2 -r TIT2 "${1}" # Remove Title
id3v2 -r TALB "${1}" # Remove Album
id3v2 -r TPE1 "${1}" # Remove Artist
id3v2 -r TPE2 "${1}" # Remove Album Artist
id3v2 -r TYER "${1}" # Remove Year
 
title=$(echo "${1}" | awk -F./ '{print $2}' | awk -F.mp3 '{print $1}')
id3v2 --TIT2 "${title}" --TALB "${2}" --TPE1 "${3}" "${1}"
 
id3v2 -s "${1}" # Remove id3v1 all
```

/bin/id3all

```sh
album=$(pwd | awk -F/ '{print $NF}')
artist=$(pwd | awk -F/ '{artistidx = NF-1}{print $artistidx}')
find . -type f -maxdepth 1 -name '*.mp3' -exec id3 {} "${album}" "${artist}" \;
```

### 가사 입력하기

```sh
# 우선 기존에 존재하는 태그를 깨끗이 지워준다
id3v2 -r USLT test.mp3

# 가사 태그를 입력한다. 줄바꿈이 있을 경우에는 다음과 같이 입력한다. 유의할점은 : 문자가 들어가면 태그가 중간에 끊기고 안들어간다. 
id3v2 --USLT '1. test1
2. test2' test.mp3
```

## HTML5 Audio


```Javascript
// imageCaptchaUrl로부터는 image stream을 받아온다. 
var imageCaptchaUrl = '/bin/beautypoint/simpleCaptcha';

// audioCaptchaUrl로부터는 audio file url을 받아온다. 
var audioCaptchaUrl = '/bin/beautypoint/audioCaptcha';
var audioUrlList = null;

function isIE() {
    return /(Trident|MSIE)/g.test(window.navigator.userAgent);
}

function isHTML5AudioApplicable() {
    return document.createElement('audio').canPlayType;
}

function imageCaptcha() {
    $('#imageCaptcha').html('<img src="' + imageCaptchaUrl + '?rand=' + Math.random() + '" style="width:100%;height:100%;" />');
}

function playBgsound(url) {
    $('#audioCaptcha').attr('src', url);
}

function playHTML5Audio(url) {
    var audioObj = $('#audioCaptchaHTML5')[0];
    audioObj.src = url;
    audioObj.play();
}

// SetTimeout 사용시 index값 유지를 위해 Closure 사용
var sequencer = function() {
    var s = 0;
    return function() {
        return s++;
    }
};

var seq = null;

function audioCaptcha() {
    // seq 함수를 초기화
    seq = sequencer();

    $.ajax({
         type:"get"
        ,url: audioCaptchaUrl
        ,success:function(data) {
            audioUrlList = data ? data.split('|') : {};

            for (var i=0; i<audioUrlList.length; i++)
                setTimeout(callbackAudioCaptcha, i * 1.4 * 1000);
         }
    });
}

// setTimeout으로 실행될때 동적으로 파라미터를 넣어줄 방법이 없다. 
// 만약 audioCaptcha() 안에서 setTimeout 예약시 i를 파라미터로 넣었다면 for문이 종료되면서 i는 스택에서 사라져 값이 0으로 나올것이다. 
// 그러므로 Closure를 사용하여 Context가 유지되는 값을 순차적으로 불러와 사용한다. 
// 초기화가 필요하면 seq = sequencer();를 재실행하면 된다. 
function callbackAudioCaptcha() {
    var url = audioUrlList[seq()];

    if (isIE()) {
        playBgsound(url + '?agent=msie&rand=' + Math.random());
    } else if (isHTML5AudioApplicable()) {
        try {
            playHTML5Audio(url);
        } catch(e) {
            playBgsound(url);
        }
    } else {
        window.open(url, '', 'width=1,height=1');
    }
}


// Android, iOS에서는 HTML5 Audio를 재생하려면 반드시 User Interaction(click, touch...)이 1회 이상 있어야 한다.
// User Interaction 1회시 click 이벤트에서 audio.play()를 최초 1회 실행해주고, 그 다음부터는 무시한다.
var isFirstPlay = true;

$(document).ready(function(){
    imageCaptcha();

    if (isHTML5AudioApplicable()) {
        $('#audioCaptchaHTML5Btn').click(function() {
            if (isFirstPlay) {
                isFirstPlay = false;
                var audioObj = $('#audioCaptchaHTML5')[0];
                audioObj.play();
            }

            audioCaptcha();
        });
    } else {
        $('#audioCaptchaHTML5Btn').click(function() {
            audioCaptcha();
        });
    }
});
```

```HTML
<div id='imageCaptcha'></div>
<bgsound id="audioCaptcha" src="" autostart="true" loop="1" />
<audio id="audioCaptchaHTML5" src=''></audio>
<input type="button" id="audioCaptchaHTML5Btn" />
<input type="text" name="captchaValue" />
<button id="refreshCaptcha" onclick='imageCaptcha();' >Refresh</button>
```



# Image
