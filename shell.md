# Concept

>  [리눅스/맥 커맨드라인 셸 입문자를 위한 생존 가이드](https://www.44bits.io/ko/post/linux-and-mac-command-line-survival-guide-for-beginner) 



## Permission

**owner, group 설정** 

```sh
sudo chown -R cr420 wiki # wiki와 그 하위 모든 파일&폴더의 owner user를 cr420으로
sudo chgrp -R cr420 wiki # wiki와 그 하위 모든 파일&폴더의 owner group을 cr420으로
```



**read, write, execute 설정** 

리눅스에서 권한은 다음과 같이 나타난다. 명령어 ls -l를 입력해보면 알수있다.

아래는 각각 디렉토리와 파일의 권한을 나타낸다. 디렉토리는 d, 파일은 -로 나타난다.

```sh
d      rwx    rwx     rwx
dir    user   group   others
 -     rwx    rwx     rwx
file   user   group   others
```

권한을 주는 명령어는 chmod로, rwx를 이용해 권한을 줄 수도 있고, code를 사용해서 줄 수도 있다.

```sh
sudo chmod ugo+rwx ck # ck에 u,g,o 모두 다 rwx권한을 부여한다.
sudo chmod a-rwx ck # ck에 root를 제외한 누구도 접근할수 없게 한다.
sudo chmod -R 700 ck # ck와 그 하위 모든 파일&폴더에 대해 u에만 rwx권한을 부여한다.
```

- u : user
- g : group
- o : others
- a : all
- \+: permission allowed
- \- : permission denied
- r (4) : read
- w (2) : write
- x (1) : execute



**SUID. SGID. STID** 

SUID. SGID는 프로그램 실행시 해당 권한을 임시로 사용하는 것을 의미한다.

STID는 Sticky bit로 rw 권한은 주어지나 변경, 삭제는 불가능한 것을 의미한다. 예를 들어 /tmp의 경우 여러 user가 공용으로 사용하기 때문에 rw는 가능하지만 변경, 삭제는 admin만 가능해야 한다.

- u (4) : SUID
- g (2) : SGID
- t (1) : STID
- 권한 예제
    - 4777 : rwsrwxrwx
    - 2777 : rwxrwgrwx
    - 1777 : rwxrwxrwt
    - 6777 : rwsrwgrwx
    - 7777 : rwsrwgrwt



# Language

## get file extension

```sh
${1##*.}
```



## if

```sh
# directory가 없는지 검사
if [ ! -d .git ]; then
fi

# 문자열 contains 검사
if [[ "$(pwd)" == *"/me" ]]; then
fi

# 문자열 equals 검사
if [ x"${type}" == x"add" ]; then
elif [ x"${type}" == x"rm" ]; then
else
fi

# 파라미터 null 검사
if [ x"${1}" == x ]; then
fi
```



## Loop

```sh
a=1
while [ $a -lt 100 ]; do
    touch $a
    let a=a+1
done
```



## logging

```sh
python t.py > t.log  #항상 새 log를 만든다
python t.py >> t.log #log파일이 없으면 새로 만들고, 있으면 이어붙인다
python t.py > t.log 2>&1 #표준 에러(2) 메시지를 표준 출력(1)으로 바꿔 logging한다. 이 구문이 없으면 표준 에러 출력은 log에 찍히지 않는다.
```



## read file

```sh
# test.txt 파일로부터 각 라인을 읽어서 출력
while read LINE; do
    echo $LINE
done < test.txt
```

> 참조: http://linuxconfig.org/Bash_scripting_Tutorial#8-2-read-file-into-bash-array



## nohup, &

nohup을 붙이면 ssh client를 통해 들어가서 프로세스를 돌리더라도, client가 죽든말든 server에서 돌아가는 프로세스를 만들 수 있다. (원래는 client에서 실행하던 것은 client가 연결이 끊기면 프로세스도 같이 죽는다)

```sh
nohup java test > test.log 2>&1 &
nohup java test2 > test2.log 2>&1 &

# 명령어 끝에 붙은 &는 백그라운드로 돌게 하는 명령어다. 
# 이 명령어로 인해 두 명령어가 동시에 돌게 된다.

# nohup은 hang-up signal이 와도 동작하도록 한다.
# 그래서 터미널 연결이 끊어져도 실행을 멈추지 않는다.

# 결론적으로 nohup만 붙이면 fore-ground에서 꺼지지 않고 돈다.
# 끝에 &만 붙이면 back-ground로 돌다가 터미널 연결이 끊어지면 죽는다.
# nohup에 &까지 붙여주면 백그라운드로 돌고, 터미널 연결과 상관없이 돈다.

# 그런데 여러 명령어에 &를 다 붙여주면 모두 동시에 돈다.
# 그러므로 순서대로 돌아야 하는 명령어라면 &를 빼야한다.
```



## 병렬 처리

```sh
#!/bin/bash

echo start

sleep 3 &
sleep 5 &
sleep 7 &

wait
# 아래와 같이 명시적으로 PID를 줘서 wait도 가능
wait `jobs -l | awk '{print $2}'`

echo end
```



# CLI Tools

## Terminal Shortcuts

- CTRL + E : 줄 마지막으로 이동. 자동완성과 함께 사용할 때 좋다.
- CTRL + A : 줄 처음으로 이동.
- CTRL + F : 한 문자씩 앞으로 이동
- CTRL + B : 한 문자씩 뒤로 이동
- ALT + F : 한 단어씩 앞으로 이동
- ALT + B : 한 단어씩 뒤로 이동
- CTRL + C : 현재 실행 중인 명령을 중단한다.
- CTRL + G : 명령을 실행하지 않고 떠나기. (명령을 실행하지 않고 다음 프롬프트가 뜬다)
- CTRL + H : 뒤로 한 문자씩 지운다. 백스페이스와 같은 효과.
- CTRL + J : 엔터키와 동일하다.
- CTRL + K : 커서뒤에 있는 모든 문자를 삭제한다.
- CTRL + L : 지금까지의 입력을 유지하면서 화면을 삭제(clear) 한다. clear 명령과 비슷하지만 clear는 현재 입력중인 명령어까지 삭제한다.



## tmux

> [tmux 설정, 사용법](https://junho85.pe.kr/320) 

- tmux ls : List Sessions
- tmux at (-t 0) : Attach Session
- Ctrl + b : Command Mode
    - " : Split Screen Horizontally
    - % : Split Screen Vertically
    - Arrow : Move Focus
    - Alt + Arrow : Resize Pane a lot
    - Ctrl + Arrow : Resize Pane less
    - z : Toggle Zoom
    - d : Detach Session
    - c : Create new Session
    - x : Kill Pane
    - Page Up/Down : Scroll Up/Down
        - / : Search Mode
        - q : Quit Scroll Mode



## rsync

rsync는 file size와 file mod time을 기준으로 변화를 감지한다. 이 기준이 부족하다면 옵션을 줘서 checksum 비교로 변화를 감지할 수도 있다.

또한 checksum을 통해 파일의 일부분(chunk)의 변경 내역만 업데이트 할 수도 있다. 그러므로 fstab에 noatime을 옵션으로 주더라도 일반적인 rsync 사용에는 영향을 미치지 않는다.

Server에는 sshd 설치, Client에는 rsync 설치. Server는 sshd를 가동해 둔다. rsync는 설치만 되어있으면 된다. rsyncd를 켤 필요는 없다. Client에서는 다음과 같이 입력해 동기화한다.


```sh
# ssh를 통한 원격지 rsync
rsync -avz --delete -e 'ssh -p 9122' $ID@test.com:/home/dgdsingen/bak /home/dgdsingen
rsync -avz --delete -e 'ssh -p 9122' $ID@test.com:/home/dgdsingen/docs /home/dgdsingen
rsync -avz --delete -e 'ssh -p 9122' $ID@test.com:/home/dgdsingen/music /home/dgdsingen
rsync -avz --delete -e 'ssh -p 9122' $ID@test.com:/home/dgdsingen/photo /home/dgdsingen
# samba mount를 통한 원격지 sync

# 1. Windows에서 Z:에 samba mounting하고 rsync
/bin/rsync -avz --delete /cygdrive/z/bak /cygdrive/d
/bin/rsync -avz --delete /cygdrive/z/docs /cygdrive/d
/bin/rsync -avz --delete /cygdrive/z/music /cygdrive/d
/bin/rsync -avz --delete /cygdrive/z/photo /cygdrive/d

# 2. Ubuntu에서 Nautilus의 samba mounting 후 rsync
rsync -avz --delete /run/user/1000/gvfs/smb-share:port=9139,server=test.com,share=$ID/bak /home/$ID
rsync -avz --delete /run/user/1000/gvfs/smb-share:port=9139,server=test.com,share=$ID/docs /home/$ID
rsync -avz --delete /run/user/1000/gvfs/smb-share:port=9139,server=test.com,share=$ID/music /home/$ID
rsync -avz --delete /run/user/1000/gvfs/smb-share:port=9139,server=test.com,share=$ID/photo /home/$ID

# 로컬 rsync
rsync -avz --delete /home/pi/bak /mnt/rsync
rsync -avz --delete /home/pi/docs /mnt/rsync
rsync -avz --delete /home/pi/photo /mnt/rsync
rsync -avz --delete /home/pi/music /mnt/rsync

# 특정 패턴 제외
rsync -avz --delete -e 'ssh -p 9122' $ID@test.com:/home/$ID/docs/reading /home/$ID --exclude=*mo.pdf

# Windows에서 Cygwin을 통한 Batch 실행
C:\cygwin\usr\local\bin
/bin/rsync -avz --delete /cygdrive/z/docs/news_tablet /cygdrive/c/Users/ele/Documents

r.bat
cd C:\cygwin\bin
bash "/usr/local/bin/r"
```



## qrencode

CLI로 QR Code 만들기

```sh
sudo apt-get install qrencode

qrencode "some texts" -o qr.png
```



## fzf, fd

- [Navigating a filesystem quickly with fzf and fd](https://mike.place/2017/fzf-fd/) 



## peco

> [Peco 사용법: 커맨드라인 텍스트 증분검색 필터링 도구](https://www.44bits.io/ko/post/incremental-search-tool-peco) 



## tail

> [tail + bat](https://www.lesstif.com/lpt/tail-bat-pipe-123338881.html) 



## figlet

> [figlet](http://www.figlet.org/) 



## sendemail

```sh
sudo apt install sendemail

sendemail -l email.log \
-f "sender@domain.com" \
-u "Email Subject 1" \
-t "receiver@domain.com" \
-s "smtp.gmail.com:587" \
-o tls=yes \
-xu "youremail@gmail.com" \
-xp "Email Password" \
-o message-file="/tmp/mailbody.txt"

cat mailbody.txt | sendemail -l email.log \
-f "sender@domain.com" \
-u "Email Subject 2" \
-t "receiver@domain.com" \
-cc "receiver2@domain.com" \
-bcc "receiver3@domain.com" \
-s "smtp.gmail.com:587" \
-o tls=yes \
-xu "youremail@gmail.com" \
-xp "Email Password"
```



## grep

> 참조: http://lkrox.blogspot.com/2013/01/grep.html



**명령어 종류** 

- grep : 파일 전체를 뒤져 정규표현식에 대응하는 모든 행들을 출력한다.
- egrep : grep의 확장판으로, 추가 정규표현식 메타문자들을 지원한다.
- fgrep : fixed grep 이나 fast grep으로 불리며, 모든 문자를 문자 그대로 취급한다. 즉, 정규표현식의 메타문자도 일반 문자로 취급한다.



**Parameters** 

grep에서 사용하는 정규표현식 메타문자

- ^ : 행의 시작 지시자
    - '^love' : love로 시작하는 모든 행과 대응
- $ : 행의 끝 지시자
    - 'love$' : love로 끝나는 모든 행과 대응
- . : 하나의 문자와 대응
    - 'l..e' : l 다음에 두 글자가 나오고 e로 끝나는 문자열을 포함하는 행과 대응
- * : 선행문자와 같은 문자의 0개 혹은 임의개수와 대응
    - ' *love' : 0개 혹은 임의 개수의 공백 문자 후에 love로 끝나는 문자열을 포함한 행과 대응
- [] : [] 사이의 문자 집합중 하나와 대응
    - '[Ll]ove' : love나 Love를 포함하는 행과 대응
- [^ ] : 문자집합에 속하지 않는 한 문자와 대응
    - '[^A-K]love' : A와 K 사이의 범위에 포함되지 않는 한 문자와 ove가 붙어있는 문자열과 대응
- \< : 단어의 시작 지시자
    - '\ : love로 시작하는 단어를 포함하는 행과 대응(vi,grep에서 지원)
- \> : 단어의 끝 지시자
    - 'love\>' : love로 끝나는 단어를 포함하는 행과 대응 (vi,grep에서 지원)
- \(..\) : 다음 사용을 위해 태그를 붙인다.
    - '\(lov\)ing' : 지정된 부분을 태크1에 저장한다. 나중에 태그값을 참고하려면 \1을 쓴다. 맨 왼쪽부터 시작해 태그를 9개가지 쓸 수 있다. 왼쪽 예에서는 lov가 레지스터1에 저장되고 나중에 \1로 참고할 수 있다.
- x\{m\} : 문자 x를 m번 반복한다.
    - 'o\{5\}' : 문자 o가 5회 연속적으로 나오는 모든 행과 대응
- x\{m,\} : 적어도 m번 반복한다.
    - 'o\{5,\}' : 문자 o가 최소한 5회 반복되는 모든 행과 대응
- x\{m,n\} : m회 이상 n회 이하 반복한다.
    - o\{5,10\}' : 문자 o가 5회에서 10회 사이의 횟수로 연속적으로 나타나는 문자열과 대응



**Options** 

- -b : 검색 결과의 각 행 앞에 검색된 위치의 블록 번호를 표시한다. 검색 내용이 디스크의 어디쯤 있는지 위치를 알아내는데 유용하다.
- -c : 검색 결과를 출력하는 대신, 찾아낸 행의 총수를 출력한다. (count)
- -h : 파일 이름을 출력하지 않는다.
- -i : 대소문자를 구분 하지 않는다.(대문자와 소문자를 동일하게 취급). (ignore)
- -l : 패턴이 존재하는 파일의 이름만 출력한다.(개행문자로 구분) (list file)
- -n : 파일 내에서 행 번호를 함께 출력한다. (number)
- -s : 에러 메시지 외에는 출력하지 않는다. 종료상태를 검사할 때 유용하게 쓸 수 있다.
- -v : 패턴이 존재하지 않는 행만 출력한다. (invert)
- -w : 패턴 표현식을 하나의 단어로 취급하여 검색한다. (word)


```sh
# /etc/passwd 파일에서 jack을 찾는다. jack이 행의 맨 앞에 있으면 행 번호를 화면으로 출력한다.
grep -n '^jack:' /etc/passwd
```



**Exit Value** 

grep은 파일 검색의 성공 여부를 종료 상태값으로 되돌려준다.

- 패턴을 찾으면 0
- 패턴을 찾을 수 없으면 1
- 파일이 존재하지 않을 경우 2

sed,a자 등은 검색의 성공 여부에 대한 종료 상태값을 반환하지 않는다. 다만 구문 에러가 있을 경우에만 에러를 보고한다.



**Examples** 


```sh
# grep NW datafile
# grep NW d*
(d로 시작하는 모든 파일에서 NW를 포함하는 모든 행을 찾는다.)
# grep '^n' datafile
(n으로 시작하는 모든 행을 출력한다.)
# grep '4$' datafile
(4로 끝나는 모든 행을 출력한다.)
# grep TB Savage datafile
(TB만 인자이고 Savage와 datafile은 파일 이름이다.)
# grep 'TB Savage' datafile
(TB Savage를 포함하는 모든 행을 출력한다.)
# grep '5\.' datafile
(숫자 5, 마침표, 임의의 한 문자가 순서대로 나타나는 문자열이 포함된 행을 출력한다.)
# grep '\.5' datafile
(.5가 나오는 모든 행을 출력한다.)
# grep '^[we]' datafile
(w나 e로 시작하는 모든 행을 출력한다.)
# grep '[^0-9]' datafile
(숫자가 아닌 문자를 하나라도 포함하는 모든 행을 출력한다.)
# grep '[A-Z][A-Z] [A-Z]' datafile
(대문자 2개와 공백 1개, 그리고 대문자 하나가 연이어 나오는 문자열이 포함된 행을 출력한다.)
# grep 'ss* ' datafile
(s가 한 번 나오고, 다시 s가 0번 또는 여러번 나온 후에 공백이 연이어 등장하는 문자열을 포함한 모든 행을 출력한다.)
# grep '[a-z]\{9\}' datafile
(소문자가 9번 이상 반복되는 문자열을 포함하는 모든 행을 출력한다.)
# grep '\(3\)\.[0-9].*\1 *\1' datafile
(숫자 3,마침표,임의의 한 숫자,임의 개수의 문자,숫자 3(태그),임의 개수의 탭 문자,숫자 3의 순서를 갖는 문자열이 포한된 모든 행을 출력한다.)
# grep '\
(north로 시작하는 단어가 포함된 모든 행을 출력한다.)
# grep '\' datafile
(north라는 단어가 포함된 모든 행을 출력한다.)
# grep '\<[a-z].*n\>' datafile
(소문자 하나로 시작하고, 이어서 임의 개수의 여러 문자가 나오며, n으로 끝나는 단어가 포함된 모든 행을 출력한다. 여기서 .*는 공백을 포함한 임의의 문자들을 의미한다.)
# grep -n '^south' datafile
(행번호를 함께 출력한다.)
# grep -i 'pat' datafile
(대소문자를 구별하지 않게 한다.)
# grep -v 'Suan Chin' datafile
(문자열 Suan Chin이 포함되지 않은 모든 행을 출력하게 한다. 이 옵션은 입력 파일에서 특정 내용의 입력을 삭제하는데 쓰인다.
# grep -v 'Suan Chin' datafile > black
# mv black datafile
)
# grep -l 'SE' *
(패턴이 찾아진 파일의 행 번호 대신 단지 파일이름만 출력한다.)
# grep -w 'north' datafile
(패턴이 다른 단어의 일부가 아닌 하나의 단어가 되는 경우만 찾는다. northwest나 northeast 등의 단어가 아니라, north라는 단어가 포함된 행만 출력한다.)
# grep -i "$LOGNAME" datafile
(환경변수인 LOGNAME의 값을 가진 모든 행을 출력한다. 변수가 큰따옴표로 둘러싸여 있는 경우, 쉘은 변수의 값으로 치환한다. 작은따옴표로 둘러싸여 있으면 변수 치환이 일어나지 않고 그냥 $LOGNAME 이라는 문자로 출력된다.)
```



**grep -r 옵션 안될경우** 

보통 리눅스에서는 grep -r 하위 디렉토리까지 파일을 검색 할수 있게 recursive 옵션을 지원하지만 전통? grep에는 -r 옵션이 없는지 AIX ,HP,Solaris 에서는 -r 옵션을 사용 할수 없다. 그렇다면 여기서 find 와 xargs 를 이용하여 -r 옵션과 같은 실행을 할수 있는 방법은 아래와 같다.


```sh
ex) dir : /home/search/cgi-src
1. -r 옵션 이용 : grep -r "include"  /home/search/cgi-src
2.  find 와 xargs  이용 : find /home/search/cgi-src | xargs grep "include"
여기서 xargs는  간단하게 말해 파이프 '|' 를 통해 입력 받아서 xargs 뒤에 있는 명령어(grep)한테 파라미터를 주는것.
```



**xargs 활용** 

예를들어 test.cpp, test1.cpp, test2.cpp 이런식으로 다수의 파일이 있을떄 일일히 cp 명령어로 .bak 파일을 만드려면 번거로울 것이다. 한번에 처리할 수 있는 방법이 있다.


```sh
$ ls test* | xargs -t -i cp {} {}.bak
```



**egrep** 

egrep(extended grep) : grep에서 제공하지 않는 확장된 정규표현식 메타문자를 지원한다. grep와 동일한 명령행 옵션을 지원한다.

egrep에서 지원하는 확장 메타문자

- + : 선행문자와 같은 문자의 1개 혹은 임의 개수와 대응
    
    - '[a-z]+ove' : 1개 이상의 소문자 뒤에 ove가 붙어있는 문자열과 대응. move,approve,love,behoove 등이 해당된다.
- ? : 선행문자와 같은 문자의0개 혹은 1개와 대응
  
    - 'lo?ve' : l 다음에 0개의 문자 혹은 하나의 문자가 o가 나오는 문자열과 대응. love,lve 등이 해당된다.
- a|b : a 혹은 b와 대응
  
    - 'love|hate' : love 혹은 hate와 대응.'
- () : 정규표현식을 묶어준다
  
    - 'love(able|ly)' : lovable 혹은 lovely와 대응.
- '(ov)+' : ov가 한 번 이상 등장하는 문자열과 일치.



**Examples** 


```sh
# egrep 'NW|EA' datafile
(NW나 EA가 포함된 행을 출력한다.)
# egrep '3+' datafile
(숫자 3이 한 번 이상 등장하는 행을 출력한다.)
# egrep '2\.?[0-9]' datafile
(숫자 2 다음에 마침표가 없거나 한 번 나오고, 다시 숫자가 오는 행을 출력한다.)
# egrep ' (no)+' datafile
(패턴 no가 한 번 이상 연속해서 나오는 행을 출력한다.)
# egrep 'S(h|u)' datafile
(문자 S 다음에 h나 u가 나오는 행을 출력한다.)
# egrep 'Sh|u' datafile
(패턴 Sh나 u를 포함한 행을 출력한다.)
```



**fgrep** 

fgrep : grep 명령어와 동일하게 동작한다. 다만 정규표현식 메타문자들을 특별하게 취급하지 않는다.


```sh
# fgrep '[A-Z]****[0-9]..$5.00' file
([A-Z]****[0-9]..$5.00 이 포함된 행을 출력한다. 모든 문자들을 문자 자체로만 취급한다.)
```



## find

**-exec** 


```sh
find . -name base.dat.Z -exec zcat {} \; | grep 072
find . -name "*.pdf" -exec cp {} /home/dgdsingen/downloads \;
```



**Parameters** 

각각의 인수들의 의미는 다음과 같다.

- path : 찾기 시작할 위치를 나타낸다. 예를들어, `.'은 현재 디렉토리를 나타내고, `/'은 루트 디렉토리부터 찾을 겻을 나타낸다.
- expression : 특정 파일을 찾기 위한 여려가지 조건들을 표현하는 부분으로 option, test, action, operator 등의 구문으로 구성된다. 그럼, expression의 각각의 구성 요소에 대하여 알아보자. option은 test와 상관 없이 항상 적용된다. option의 방법에는 다음과 같은 것이 있다.



**Options** 

- -name : 확장자가 txt 인 화일을 찾는다.
    - find / -name '*.txt'
- -perm : 퍼미션이 666(-rw-rw-rw-)인 화일을 찾는다.
    - find . -perm 666
- -type : 파일의 타입을 지정하여, 찾고자하는 파일을 찾는다. 타입의 종류는 다음과 같다.
    - b : 블록 특수 파일(block device)
    - c : 캐릭터 특수 파일 (character deice)
    - d : 디렉토리(directory)
    - f : 일반파일(file)
    - l : 심볼릭 링크(link)
    - p : 파이프 (pipe)
    - s : 소켓 (socket)
    - find . -type d : 현재 디렉토리 아래에 있는 서브디렉토리를 모두 찾는다.
- -atime +n/-n/n : 최근 n일 이전에 액세스된 파일을 찾아준다.(accessed time)
    - +n은 n일 또는 그보다 더 오래 전의 파일
    - -n은 오늘 부터 n일 전까지의 파일
    - n은 정확히 n일 전에 액세스되었음을 의미한다.
    - find / -atime +30 -type d : 시스템 전체에서 한 달 또는 그 이상의 기간동안 한번도 액세스하지 않은 디렉토리
- -ctime +n/-n/n : ctime은 파일의 퍼미션을 마지막으로 변경시킨 날짜를 의미한다. (changed time)
    - +n은 n일 또는 그보다 더 오래 전의 파일
    - -n은 오늘 부터 n일 전까지의 파일
    - n은 정확히 n일 전에 수정되었음을 의미한다.
    - find . -ctime -7 : 현재 디렉토리 아래에서 최근 일주일 동안 고친 파일
- -mtime +n/-n/n : mtime은 파일내의 data를 마지막으로 변경한 날짜를 의미한다.(modified time)
    - +n은 n일 또는 그보다 더 오래 전의 파일
    - -n은 오늘 부터 n일 전까지의 파일
    - n은 정확히 n일 전에 수정되었음을 의미한다.
- -cnewer 파일명 : '파일명' 부분에 적어준 파일보다 더 최근에 수정된 파일들을 찾아준다.
    - find . -cnewer test.txt -print : test.txt 화일이 생성된 이후의 화일을 찾는다.
- -user 유저네임 : '유저네임' 부분에 지정한 유저 소유의 파일을 찾아준다.
    - find / -user nalabi : nalabi 라는 계정의 화일을 찾아준다.
- -maxdepth n : 0이 아닌 정수값으로 경로 깊이를 지정하여 검색을 할 경우에 사용한다. 예를들어, '-maxdepth 1'은 시작위치로 지정한 디렉토리만 검색하고 하위 디렉토리는 찾지 않는다.
- -mindepth n : 반대로 동작한다. 즉, 지정한 숫자만큼의 깊이부터 그 하위 디렉토리를 검색한다. (GNU find 버전)
- -follow : 심볼릭 링크된 디렉토리도 검색을 할 경우에 사용한다.
- -mount : 현재의 파일 시스템과 동일한 타입의 파일 시스템에서만 검색을 할 경우에 사용한다. test에는 다음과 같은 방법들이 있으며, test에 사용하는 인수에는 보다 큰 수를 의미하는 `'나, 보다 작은 수를 의미하는 `'를 함께 사용할 수 있다. 인수에 아무 연산자가 없을 경우에는 정확히 그 인수 값을 의미한다.
- -group : 특정 그룹 소유의 파일들을 찾을 경우에 사용한다.
- -nouser : 소유자가 없는 파일을 찾을 경우에 사용한다. 즉, /etc/passwd 파일에 없는 소유자의 파일을 찾을 경우에 사용한다.
- -nogroup : 올바른 그룹의 소유가 아닌 파일을 찾을 경우에 사용한다. 즉, /etc/groups 파일에 없는 그룹의 소유인 파일을 찾을 경우에 사용한다.
- -newer file1 file2 : `file1' 보다는 이후에 `file2' 보다는 이전에 생성되거나 변형된 파일들을 찾을 경우에 사용한다.
- -size n[bckw] : 크기가 n 유닛(unit)인 파일을 찾을 경우에 사용한다. 유닛은 기본 설정('b'와 함께 사용한 경우와 동일)인 512 바이트의 블럭, `c'를 사용할 경우에는 1 바이트, `k'를 사용할 경우에는 킬로바이트, `w'를 사용할 경우에는 2 바이트의 워드 크기를 나타낸다.
- -empty : 비어있는 파일이나 디렉토리를 찾을 경우에 사용한다. (GNU find 버전)
- -regex : 정규표현식(regular expression)을 이용하여 파일들을 찾을 경우에 사용한다. `-iregex'는 대소문자를 구별하지 않을 경우에 사용한다. (GNU find 버전) action은 test에서의 조건과 일치하는 파일들에 대해 수행할 작업을 명시하는 것으로 다음과 같은 방법들이 있다.
- -print : 찾은 파일들을 표준출력(stdout)으로 출력한다. 기본으로 설정되어 있다.
- -fprint file : 찾은 파일들을 `file'로 출력한다. `file'이 존재 하지 않을 경우에는 새로 생성되고, 존재할 경우에는 기존의 파일은 없어진다. (GNU find 버전)
- -exec : 파일을 찾았을 경우, 찾은 파일들에 대해 특정 명령을 수행 할 때 사용한다. 일반적으로 `-exec command {} \;' 혹은 `-exec command {} ;'의 형식을 취한다.
- -ok : -exec와 동일한 작업을 한다. 다른 점은, 명령을 실행할 때마다 실행 의사를 물어본다.
- -ls : `ls -dils' 형식으로 찾은 파일들의 정보를 출력할때 사용한다.
- -fls file : `ls'와 동일하게 동작하며 결과를 `file'로 출력한다. operator는 test에서 사용한 옵션들을 조합하여 조건식을 만들고자 할때 사용는 것으로 다음과 같은 방법들이 있다. (설명 순서는 우선순위(precedence)에 따른다.)



**Examples** 


```sh
자신의 홈 디렉토리에서 확장자가 '.txt'인 파일을 찾을 경우 
$ find   -name "*.txt'' -print
현재 디렉토리 밑에서 첫글자가 영어 대문자인 모든 파일을 찾을 경우 
$ find . -name "[A-Z]*'' -print
'/usr/local'에서 첫 두글자는 영어 소문자이고 세번째 한자리는 숫자로 시작하는 이름을 가진 파일을 찾을 경우 
$ find /usr/local -name "[a-z][a-z][0-9]*'' -print
확장자가 .txt 인 파일을 찾으면서 현재 디렉토리와 한 단계 밑의 디렉토리에서만 파일을 찾을 경우 
$ find   -maxdepth 2 -name "*.txt'' -print
현재 디렉토리 밑에서 `zzang'이라는 이름을 가진 사용자 소유의 파일을 찾을 경우 
$ find . -user zzang -print
시스템에서 소유자나 그룹이 없는 파일을 찾을 경우 (크래커가 만들어 놓은 파일일 경우도 있음) 
$ find / -nouser -o -nogroup -print
자신의 홈 디렉토리에서 최근 3일 동안 변경된 파일들을 찾을 경우 
$ find . -mtime -3 -print
'/tmp'에서 최근 5일 동안 변경되지 않은 파일들을 찾아서 삭제할 경우 (파일을 삭제할 때마다 삭제할 것인가를 물어보도록) 
$ find . -mtime +5 -print -ok rm {} ;
현재 디렉토리 밑에 있는 모든 포스트 스크립트 파일(.ps)을 찾아서 gzip으로 압축을 하고 그 목록을 result.txt라는 파일에 저정할 경우 
$ find . -name "*.ps" -fprint result.txt -exec gzip {} ;
크랙커의 침입이 의심스러워 자신의 시스템에서 suid와 guid가 설정된 일반 파일들을 찾아서 권한을 확인할 경우 
$ find / -type f -perm +6000 -print -ls
시스템 관리의 실수로 일반 사용자가 쓰기 권한을 갖도록 설정되어 있는 파일을 찾아서 실행 권한을 없애는 경우 (단, 링크 파일은 제외함)
$ find / -perm +2 ! -type l -print -exec chmod o-w {} ;
현재 디렉토리에서 가장 큰 파일을 찾기
$ find  .  -type  f  | xargs  du  -s  | sort  -n  |  tail  -1
위 한 줄의 명령어는 현재 디렉토리에 서브 디렉토리 포함하여 가장 큰 파일을 하나 찾아서 이를 출력하라는 의미이다.
간혹, 파일 시스템의 FULL이 되어서 가장 큰 파일을 찾으려고 할 경우 아주 유용하다.
현재 디렉토리에서 확장자가 cpp 이고 string 이란 문자열이 포함된 파일 검색
$ find / -name "*.cpp" -print -exec grep string {} \
현재 디렉토리 위치에서 하위디렉토리를 포함하여 string 이란 문자열을 포함한 파일 검색
$ find . -type f | xargs grep "string"

위 방법은 파일 이름에 공백이 들어갈 경우 문제가 생길 수 있다. 공백이 포함된 파일은 다음과 같이 찾는다.
$ find . -type f -print0 | xargs -0 grep "string"
현재 디렉토리내에서 확장자가 cpp 이고 string 문자열을 포함하는 파일 검색
파일이름과 내용을 보여주려면..
$ grep string `find . -name \*\.cpp`

파일이름만 보여주려면..
$ grep -l string `find . -name \*\.cpp`
특정 내용을 가지고 있는 파일 목록 찾기

1. find와 grep -l 사용
find . -name '*.*' -exec grep -l "maps.google.com" {} \;


2. cat과 grep 사용
jsp=$(find . -name '*.jsp')

for f in $jsp
do
    if [ -f $f ]
    then
        inner=$(cat $f | grep "${1}")
        if [ -n "${inner}" ]
        then
                echo $f
                echo $inner
        fi
    fi
done


3. cat과 grep 사용
# t.sh
v1=$(cat "${1}" | grep test1)
v2=$(cat "${1}" | grep test2)

if [ -n "${v1}" -o -n "${v2}" ]
then
    echo "${1}"
    echo "${v1}"
    echo "${v2}"
fi

# using t.sh
find . -name '*.txt' -exec t.sh {} \; > t.txt
# 특정 디렉토리 제외하고 검색
find . -name '*' ! -path './q*'

# AIX ksh에서 특정 디렉토리 제외하고 tar로 묶기
find . ! -path '*c*' | xargs tar -c -v -D -f a.tar;
# 특정기간의 파일 찾기

1. 우선 touch명령어를 가지고 특정기간(검색하고 싶은 기간)의 파일 생성
touch -t 201107010000 start.txt
touch -t 201107022359 end.txt
--> 이 방법을 이용하면 지정한 날짜와 시간에 생성된 파일이 만들어짐

2. 위에서 만든 파일을 가지고 파일 검색
find . -newer start.txt -a ! -newer end.txt -exec rm -rf {} \;
--> 7월1일 ~2일 사이에 생성된 파일을 검색하여 삭제
```

## rename

```sh
rename s/source/"target text"/g *.txt
```

## sed

파일 내용 변경

```sh
sed -i 's/old/new/g' test.txt

# macOS에서는 -i 옵션이 달라서 위 명령어로 되지 않는다. 모든 환경에서 동작하게 하려면 아래와 같이 해준다.
cat test.txt | sed 's/old/new/g' > tmp.txt
mv tmp.txt test.txt
```



`/` 가 포함된 문자열 처리

```sh
sed "s#old#new#g"
```



파일 날짜를 유지하며 내용 변경

```sh
cd /home/repo/me/test/myapp
files=$(find . -name '*.jsp' | egrep '\.js$|\.jsp$|\.html$|\.properties$|\.property$' | egrep -v '\.class$')

for f in $files
do
    if [ -f $f ]
    then
        touch buf_src.txt
        touch buf_date.txt
 
        touch -r $f buf_date.txt
 
        sed 's/charset=KSC5601/charset=UTF-8/g' $f > buf_src.txt
        cat buf_src.txt > $f
 
        touch -r buf_date.txt $f
 
        rm buf_src.txt
        rm buf_date.txt
    fi
done
```



## diff

```sh
# 일반 diff (다른 내용까지 출력)
diff file1 file2

# 다른지 여부만 report
diff -q file1 file2

# 같은 디렉토리 구조를 가진 2개의 디렉토리 이하 모든 파일을 비교하여 다른것만 report
diff -qr dir1 dir2
```



## curl

```sh
# ignore ssl error
curl -k $URL

# download a file
curl $URL -o test.sh

# follow redirects
curl -L $URL
```



# Sample

## Get relative path

```sh
py -c 'import os, sys; print(os.path.relpath(*sys.argv[1:]))' ~/.zshrc ~/.config
# 결과는 ../.zshrc
```



## Simple File Backup


```sh
now_datetime=$(date "+%Y%m%d%H%M%S")
backup_dir="backup_${now_datetime}"
mkdir "${backup_dir}"

while read line;
do
    echo "${line}" | cpio -pd ./"${backup_dir}"
done < backup.txt
```



## Wiki Backup

**Bash only** 

[bak.sh](http://docs.google.com/bak.sh.codeblock.2.html)


```sh
cd /var/www
sources=`ls`
log='/home/pi/bak/bak.log'
parse='/home/pi/bak/bak_parse.log'

echo $sources > $log
echo parse > $parse

for s in $sources
do
7z a $s.7z $s >> $log 2>&1
ls -al $s.7z >> $log 2>&1
done
 
cat $log | grep -v Compressing >> $parse
 
mv *.7z /home/pi/bak
mpack -s 'Bak Report' -d $parse /dev/null dgdsingen@gmail.com
```



**Bash & Python (SMTP)** 

bak.sh


```bash
src='/mnt/data/backup/wiki.7z'
tgt='/var/www/wiki'
log='/home/dgdsingen/programming/wikibackup.log'
 
rm $src > $log 2>&1
7z a $src $tgt >> $log 2>&1
 
python /home/dgdsingen/programming/python/bak.py
```

bak.py


```python
#*-* encoding:utf8 *-*
f = open('/home/dgdsingen/programming/wikibackup.log', 'r')
lines = f.readlines()
results = []
 
for line in lines :
    if line.find('Compress') >= 0 :
        pass
    else :
        results.append(line)
 
result = ''.join(results)
 
f.close()
 
 
import smtplib
 
mailTo = 'dgdsingen@gmail.com'
mailFrom = mailTo
mailFromPwd = '$PW'
mailSubject = 'wikibackup report'
 
smtpserver = smtplib.SMTP("smtp.gmail.com",587)
smtpserver.ehlo()
smtpserver.starttls()
smtpserver.ehlo
smtpserver.login(mailFrom, mailFromPwd)
 
header = """To : {mailTo}
From : {mailFrom}
Subject : {mailSubject}
""".format(mailTo = mailTo, mailFrom = mailFrom, mailSubject = mailSubject)
print (header)
 
msg = header + result
 
smtpserver.sendmail(mailFrom, mailTo, msg)
smtpserver.close()
```



**Bash & Python (dropbox & mpack)** 

bak.sh


```bash
#!/bin/bash

bak=/mnt/data/bak
src=$bak/www.7z
tgt=/var/www
log=$bak/bak.log

rm $src
7z a $src $tgt

ls -al $src > $log

python $bak/bak.py $src >> $log

mpack -s 'Bak Report' -d $log /dev/null dgdsingen@gmail.com
```


bak.py


```python
import sys
 
args = []
 
if len(sys.argv) > 1 : 
    for arg in sys.argv : 
        if arg != sys.argv[0] :
            args.append(arg)
 
from dropbox import client, rest, session
 
APP_KEY = '$APP_KEY'
APP_SECRET = '$APP_SECRET'
ACCESS_TYPE = 'dropbox'
TOKEN_KEY = '$TOKEN_KEY'
TOKEN_SECRET = '$TOKEN_SECRET'
 
sess = session.DropboxSession(APP_KEY, APP_SECRET, ACCESS_TYPE)
sess.set_token(TOKEN_KEY, TOKEN_SECRET)
 
client = client.DropboxClient(sess)
 
for arg in args : 
    f = open(arg)
    response = client.put_file('/' + arg, f, True)
    print "uploaded : ", response
    f.close()
```



## svn-deploy.sh

```bash
pre='/cygdrive/c/egov/repo/me'
 
# Before running this script, you should commit all your own resources first.
while read line;
do
    path=$(echo $line | awk '{print $1}')
    rev=$(echo $line | awk '{print $2}')
    svn_rev=$(svn ls -v -r HEAD $pre$path | awk '{print $1}')
 
    if [ $rev -gt $svn_rev ]
    then
        echo NOT updated: $path
        echo Entered revision: $rev
        echo SVN Head revision: $svn_rev
    else
        echo Update now: $path 
        svn update -r $rev $pre$path
    fi
done < deploy.txt

# init for Packaging
today=$(date +%Y%m%d)
pre='/cygdrive/c/Users/IBM_ADMIN/_work/deploy'
 
rm -rf mail
rm -rf *_deploy
rm mail_$today.zip
rm static_$today.zip
rm src_$today.zip
 
echo > deploy_non_rev.txt
while read line;
do
    echo $line | awk '{print $1}' >> deploy_non_rev.txt
done < deploy.txt
 
# change file extension : java -> class
paths=$(sed s/\.java$/*\.class/g deploy_non_rev.txt)
 
for path in $paths
do
    # admin이나 webform이 존재하지 않을 경우 해당 케이스는 find -name 조건이 비게 되어 pass된다
    admin=$(echo $path | egrep -v 'webform' | awk -F/ '{print $NF}')
    find ./admin -name $admin | grep -v '.svn' | cpio -pd ./admin_deploy
 
    webform=$(echo $path | egrep 'webform' | awk -F/ '{print $NF}')
    find ./webform -name $webform | grep -v '.svn' | cpio -pd ./webform_deploy
done
 
# handling mail
mails=$(find ./admin_deploy | grep '\/jsp\/mail\/')
mkdir ./mail
echo $mails | cpio -pd ./mail
cd ./mail
mv ./admin_deploy/admin ./
rm -rf ./admin_deploy
zip -rq ./mail_$today.zip ./admin
mv ./mail_$today.zip ../
cd ../
rm -rf $mails
 
# Packaging files
mkdir ./webform_deploy/css
mv ./webform_deploy/webform ./webform_deploy/css/webform
 
# handling common
mv ./admin_deploy/admin/common ./webform_deploy/css
 
cd ./webform_deploy
zip -rq ../static_$today.zip ./css
 
cd ../admin_deploy
zip -rq ../src_$today.zip ./admin
 
# remove temp files
cd ../
rm deploy_non_rev.txt
 
# check results
echo 
echo '-------------------- CHECK RESULTS --------------------'
unzip -l mail_$today.zip
echo
unzip -l src_$today.zip
echo
unzip -l static_$today.zip
 
# 위 스크립트에 들어갈 input file 구조
/project/test/test.jsp 4470
```



## Crontab

```sh
# crontab -e 실행 후 다음과 같은 작업을 추가한다.
00 12 * * * sh /home/dgdsingen/programming/backup.sh

/etc/crontab에는 system-wide cron job을 추가한다. 

# cron으로 sh을 돌릴때는 환경변수가 setting되지 않는다. 그러므로 반드시 다음 구문을 넣어준다
. /home/as/bd01/ORA_ENV/ora_env.sh

# 예시 : 매일 12시(정오)에 sh 실행
00 12 * * * sh /home/dgdsingen/programming/bak.sh
```



# Issues

## 파라미터 띄어쓰기 문제

```sh
# 이 경우 파라미터에 띄어쓰기가 들어가면 제대로 인식하지 못한다
process $1

# 아래와 같이 선언하면 "full path"의 형태가 되므로 정상 인식한다. 
process "${1}"
```

## 중복 제거

```sh
cat t.txt | sort -u
```

## Script 명령어 안먹힐 때

'와 \`를 체크해보자. 시스템에 따라 'test'는 그냥 문자열로 잡히고 \`test\`가 명령어로 먹히게 된다.

