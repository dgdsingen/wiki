# ScriptRunner

> [ScriptRunner Docs](https://docs.adaptavist.com) 



# API

## Create Page

### Mac

```sh
#!/bin/bash

# 주간 원격 근무 현황판 자동 생성 스크립트. 매주 월요일에 진행하는 것으로 상정
delta=$((8-$(date +%u)))
title="$(date -v +${delta}d +'%-m')월 $((($(date -v +${delta}d +%-d)-1)/7+1))주 ($(date -v +${delta}d +'%y.%m.%d') ~ $(date -v +$((delta+4))d +'%m.%d'))"

# http://confluence.com/pages/viewpage.action?pageId=1 를 템플릿으로 사용
value=`http -a $ID:$PW "http://confluence.com/rest/api/content/1?expand=body.view"`
value=`echo $value \
    | jq ".body.view.value" \
    | sed "s/5월 10일 (월)/$(date -v +$((delta))d +'%-m월 %-d일 (%a)')/g" \
    | sed "s/5월 11일 (화)/$(date -v +$((delta+1))d +'%-m월 %-d일 (%a)')/g" \
    | sed "s/5월 12일 (수)/$(date -v +$((delta+2))d +'%-m월 %-d일 (%a)')/g" \
    | sed "s/5월 13일 (목)/$(date -v +$((delta+3))d +'%-m월 %-d일 (%a)')/g" \
    | sed "s/5월 14일 (금)/$(date -v +$((delta+4))d +'%-m월 %-d일 (%a)')/g" \
    | sed 's/\\"/"/g' | sed 's/^"//g' | sed 's/"$//g'
`

# http://confluence.com/pages/viewpage.action?pageId=2 하위 페이지 신규 생성
http -a $ID:$PW POST "http://confluence.com/rest/api/content/" \
    type="page" title="${title}" ancestors:='[{"id":2}]' space:='{"key":"ist"}' body:="{\"storage\":{\"value\":\"${value}\",\"representation\": \"storage\"}}"
```

```sh
# 주간 보고 페이지 자동 생성 스크립트. 매주 수요일에 진행하는 것으로 상정
delta=0
if [ $(date +%u) -le 3 ]; then
    delta=$((3-$(date +%u)))
else
    delta=$((10-$(date +%u)))
fi
title="$(date -v +${delta}d +'%Y.%m.%d') 주간 보고"

# http://confluence.com/pages/viewpage.action?pageId=3 를 템플릿으로 사용
value=`http -a $ID:$PW "http://confluence.com/rest/api/content/3?expand=body.view"`
value=`echo $value \
    | jq ".body.view.value" \
    | sed 's/\\"/"/g' | sed 's/^"//g' | sed 's/"$//g'
`

# http://confluence.com/pages/viewpage.action?pageId=4 하위 페이지 신규 생성
http -a $ID:$PW POST "http://confluence.com/rest/api/content/" \
    type="page" title="${title}" ancestors:='[{"id":4}]' space:='{"key":"ist"}' body:="{\"storage\":{\"value\":\"${value}\",\"representation\": \"storage\"}}"
```

### Linux

```sh
#!/bin/bash

# 주간 원격 근무 현황판 자동 생성 스크립트
# 매주 금요일마다 실행하는 것으로 상정
title="$(date -d '3 days' +'%-m')월 $((($(date -d '3 days' +%-d)-1)/7+1))주 ($(date -d '3 days' +'%y.%m.%d') ~ $(date -d '7 days' +'%m.%d'))"

# http://confluence.com:8080/pages/viewpage.action?pageId=1 를 템플릿으로 사용
value=`curl -u id:password -X GET "http://confluence.com:8080/rest/api/content/1?expand=body.view" \
    | jq ".body.view.value" \
    | sed "s/5월 10일 (월)/$(date -d '3 days' +'%-m월 %-d일 (%a)')/g" \
    | sed "s/5월 11일 (화)/$(date -d '4 days' +'%-m월 %-d일 (%a)')/g" \
    | sed "s/5월 12일 (수)/$(date -d '5 days' +'%-m월 %-d일 (%a)')/g" \
    | sed "s/5월 13일 (목)/$(date -d '6 days' +'%-m월 %-d일 (%a)')/g" \
    | sed "s/5월 14일 (금)/$(date -d '7 days' +'%-m월 %-d일 (%a)')/g" \
    | sed 's/\\"/"/g' | sed 's/^"//g' | sed 's/"$//g'
`

# http://confluence.com:8080/pages/viewpage.action?pageId=2 하위 페이지 신규 생성
curl -u id:password -X POST -H 'Content-Type: application/json' \
     -d "{\"type\":\"page\",\"title\":\"${title}\",\"ancestors\":[{\"id\":2}],\"space\":{\"key\":\"ist\"},\"body\":{\"storage\":{\"value\":\"${value}\",\"representation\": \"storage\"}}}" \
     http://confluence.com:8080/rest/api/content/
     

# 주간 보고 페이지 자동 생성 스크립트
# 매주 금요일마다 실행하는 것으로 상정
title="IST-WEEKLY. $(date -d '5 days' +'%y.%m.%d') 주간 보고"

# http://confluence.com:8080/pages/viewpage.action?pageId=3 를 템플릿으로 사용
value=`curl -u id:password -X GET "http://confluence.com:8080/rest/api/content/3?expand=body.view" \
    | jq ".body.view.value" \
    | sed 's/\\"/"/g' | sed 's/^"//g' | sed 's/"$//g'
`

# http://confluence.com:8080/pages/viewpage.action?pageId=4 하위 페이지 신규 생성
curl -u id:password -X POST -H 'Content-Type: application/json' \
     -d "{\"type\":\"page\",\"title\":\"${title}\",\"ancestors\":[{\"id\":4}],\"space\":{\"key\":\"ist\"},\"body\":{\"storage\":{\"value\":\"${value}\",\"representation\": \"storage\"}}}" \
     http://confluence.com:8080/rest/api/content/
```



# Server

## Migration

```sh
#!/bin/bash

# variables
yyyy=$(date +%Y)
mm=$(date +%m)
dd=$(date +%d)

bak_data="conf-test-data-backup-${yyyy}-${mm}-${dd}.zip"
bak_plugins="conf-test-plugins-${yyyy}-${mm}-${dd}.tar.gz"
bak_attachments="conf-test-attachments-${yyyy}-${mm}-${dd}.tar.gz"
bak_install="conf-test-install-${yyyy}-${mm}-${dd}.tar.gz"

gs_path="gs://jira-test/"
sign_key="202206241732-compute@developer.gserviceaccount.com.json"
url_txt="conf-url-${yyyy}-${mm}-${dd}.txt"

# create backup files
sudo -u jira cp "/data/conf-data/backups/$(sudo -u jira ls /data/conf-data/backups | grep .zip | tail -1)" "${bak_data}"
sudo chown $(whoami) "${bak_data}"
sudo -u jira tar cvzf "${bak_plugins}" /data/conf/confluence/WEB-INF/atlassian-bundled-plugins
sudo -u jira tar cvzf "${bak_attachments}" /data/conf-data/attachments
sudo -u jira tar cvzf "${bak_install}" --exclude=/data/conf/logs --exclude=/data/conf/temp /data/conf

# upload backup files to cloud storage
gsutil mv "${bak_data}" "${gs_path}"
gsutil mv "${bak_plugins}" "${gs_path}"
gsutil mv "${bak_attachments}" "${gs_path}"
gsutil mv "${bak_install}" "${gs_path}"

# create signed url
echo > "${url_txt}"
gsutil signurl -d 12h "${sign_key}" "${gs_path}${bak_data}" >> "${url_txt}"
gsutil signurl -d 12h "${sign_key}" "${gs_path}${bak_plugins}" >> "${url_txt}"
gsutil signurl -d 12h "${sign_key}" "${gs_path}${bak_attachments}" >> "${url_txt}"
gsutil signurl -d 12h "${sign_key}" "${gs_path}${bak_install}" >> "${url_txt}"
gsutil mv "${url_txt}" "${gs_path}"
```

```sh
# nohup 실행
nohup ./bak.sh 2>&1 > log &
tail -f log
```



# References

- [Confluence Cloud REST API](https://developer.atlassian.com/cloud/confluence/rest/intro/)
- [Confluence REST API Examples](https://developer.atlassian.com/server/confluence/confluence-rest-api-examples/)
