# ScriptRunner

> [ScriptRunner Docs](https://docs.adaptavist.com) 



- Jira > 관리 > 앱 관리 > SCRIPTRUNNER > Console

아래와 같은 테스트 스크립트를 넣고 실행하면 python 스크립트 수행 결과가 Result, Logs에 찍히는 것을 확인할 수 있다.

```groovy
import groovy.json.JsonBuilder

def sout = new StringBuilder()
def serr = new StringBuilder()
def proc = ["/usr/bin/python3", "/home/jira/test.py"].execute()
proc.consumeProcessOutput(sout, serr)
proc.waitFor()

log.error "sout: $sout"
log.error "serr: $serr"

return new JsonBuilder([
  result: "${sout}"
]).toString()
```



`/home/jira/test.py` 

```python
import sys
import datetime

# print, sys.stdout.write 둘 다 결과는 같다.
print(f"python script result: print {datetime.datetime.now()}")
sys.stdout.write(f"python script result: stdout {datetime.datetime.now()}")
```



## Assignee 지정되면 Webhook 전송

- Jira > 관리 > 앱 관리 > SCRIPTRUNNER > Listeners > Create Listener > Custom listener
    - Note: `담당자 지정 알림` 
    - Applied to: `Select project(s)` , `적용하고 싶은 프로젝트 선택` 
    - Events: `Issue Created` , `Issue Assigned`  
    - Script:

```groovy
import com.atlassian.jira.component.ComponentAccessor
import groovyx.net.http.ContentType
import groovyx.net.http.HTTPBuilder
import groovyx.net.http.Method

// variables
def baseurl = ComponentAccessor?.getApplicationProperties()?.getString("jira.baseurl")
def user = ComponentAccessor?.getJiraAuthenticationContext()?.getLoggedInUser()
def thisIssue = event?.getIssue()
String assigneeDisplayName = thisIssue?.getAssignee()?.getDisplayName()

// create webhook payload
String text = "";
if (assigneeDisplayName == null || assigneeDisplayName?.trim()?.equals("")) {
    text = "[${thisIssue.getSummary()}](${baseurl}/browse/${thisIssue.getKey()}) 건이 들어왔고 아직 담당자가 지정되지 않았습니다."
} else {
    text = "[${thisIssue.getSummary()}](${baseurl}/browse/${thisIssue.getKey()}) 건의 담당자가 ${assigneeDisplayName} 님으로 지정되었습니다."
}

// send webhook
def http = new HTTPBuilder("http://slack.com")
http.ignoreSSLIssues()
http.request(Method.POST, ContentType.JSON) {
    uri.path = "/hooks/1234"
    body = ["text":text]

    response.success = { resp ->
        log.error("Response Status: ${resp.status}")
    }

    response.failure = { resp ->
        log.error("Response Status: ${resp.status}")
    }
}

return true
```



## Issue Status 변경시 Script 실행 후 Comment 달기

아래는 이슈 등록 시 특정 Status(승인 등)에서 동작하여 필드값을 읽어서 python 스크립트를 수행하고 그 결과 text, image file을 comment로 추가하는 예제임.

참고로 모든 이벤트에 대해 실행되도록 하는 경우 여러 번 실행하더라도 문제가 되지 않도록 중복 처리를 해주자. 

예를 들면 이슈에 고객이 comment 달 때마다 ScriptRunner가 트리거되고, 그에 따라 comment 추가하는 로직이 계속 반복 실행되면 같은 comment가 계속 달리게 된다.

- Jira > 관리 > 앱 관리 > SCRIPTRUNNER > Listeners > Create Listener > Custom listener
    - Note: `스크립트명` 
    - Applied to: `Select project(s)` , `적용하고 싶은 프로젝트 선택` 
    - Events: `적용하고 싶은 이벤트 선택` 
    - Script: 

```groovy
import com.atlassian.jira.component.ComponentAccessor
import com.atlassian.jira.issue.attachment.CreateAttachmentParamsBean
import com.atlassian.servicedesk.api.requesttype.RequestTypeService
import com.onresolve.scriptrunner.runner.customisers.WithPlugin

def user = ComponentAccessor?.getJiraAuthenticationContext()?.getLoggedInUser()

def thisIssue = event?.getIssue()
String thisStatus = thisIssue?.getStatus()?.getName()
def lastStatus = ComponentAccessor?.getChangeHistoryManager()?.getChangeItemsForField(thisIssue, 'status')?.last()?.fromString

@WithPlugin("com.atlassian.servicedesk")
def requestTypeService = ComponentAccessor?.getOSGiComponentInstanceOfType(RequestTypeService)
def reqQ = requestTypeService?.newQueryBuilder()?.issue(thisIssue?.getId())?.build()
def reqT = requestTypeService?.getRequestTypes(user, reqQ)
def requestTypeName = reqT?.getResults()[0]?.getName()

// validate issue type
if (thisIssue.getIssueType().getName().equals("VPN요청_Issue")) {
  if (requestTypeName.equals("VPN 신규 계정신청")) {
    if (!lastStatus.equals(thisStatus) && thisStatus.equals("작업중")) {
      // get field values
      def cfManager = ComponentAccessor.getCustomFieldManager()

      def cfInfraVpnAccount = cfManager.getCustomFieldObjectsByName("VPN-계정").getAt(0)
      def cfInfraVpnEngName = cfManager.getCustomFieldObjectsByName("VPN-영문성명").getAt(0)
      def cfInfraVpnEmail = cfManager.getCustomFieldObjectsByName("VPN-Email").getAt(0)
      def cfInfraVpnMacAddress = cfManager.getCustomFieldObjectsByName("VPN-MacAddress").getAt(0)

      String infraVpnAccount = thisIssue.getCustomFieldValue(cfInfraVpnAccount).toString()
      String infraVpnEngName = thisIssue.getCustomFieldValue(cfInfraVpnEngName).toString()
      String infraVpnEmail = thisIssue.getCustomFieldValue(cfInfraVpnEmail).toString()
      String infraVpnMacAddress = thisIssue.getCustomFieldValue(cfInfraVpnMacAddress).toString()

      // run python script
      def sout = new StringBuilder()
      def serr = new StringBuilder()
      def proc = ["/usr/bin/python3", "/home/jira/create.py", "${infraVpnAccount}", "${infraVpnEngName}", "${infraVpnEmail}", "${infraVpnMacAddress}"].execute()
      proc.consumeProcessOutput(sout, serr)
      proc.waitFor()

      log.error("sout: ${sout}")
      log.error("serr: ${serr}")

      if (sout.contains("QR: ")) {
        // attach a qr image file
        String qrImgPath = sout.toString().substring(sout.toString().indexOf("QR: ")+4).trim() // trim 꼭 붙이자. 경로 끝에 \n 등이 들어가서 FileNotFoundException이 뜨는 경우가 있는데 디버깅하기 힘들다.
        def file = new File(qrImgPath)
        def builder = new CreateAttachmentParamsBean.Builder(file, file.name, "application/octet-stream", user, thisIssue)
        ComponentAccessor.attachmentManager.createAttachment(builder.build())

        // add a comment
        def commentManager = ComponentAccessor.getCommentManager()
        // 만약 true로 바꾸면 Event가 Dispatch되며 다시 이 스크립트가 실행되서 무한 루프가 일어날 수 있다.
        commentManager.create(thisIssue, user, "${sout}", false)
        commentManager.create(thisIssue, user, "!${file.name}!", false)
        commentManager.create(thisIssue, user, "[VPN Guide|http://vpn.com]를 참조하여 진행하시면 됩니다.", false)
      }

      return true
    }
  }
}
```



`/home/jira/create.py` 

```sh
pip install qrcode image certifi charset-normalizer python-freeipa requests urllib3 paramiko
```

```python
from python_freeipa import ClientMeta
import sys
import os
import paramiko
import time
import qrcode

# Static Global Parameters
ipa_url = "ldap.test.com"
ipa_id= "admin"
ipa_pw= "1234"
args = sys.argv

def con_freeipa():
    # ldap에 SSL인증서가 없음으로 verify_ssl=False
    client = ClientMeta(ipa_url, verify_ssl=False)
    client.login(ipa_id, ipa_pw)
    return client

def user_add (client,user_id,eng_name,email):
    #o_ipauserauthytpe = otp 로 해야 OTP으로 선택되어 2-Factor 인증 설정됩니다.
    user = client.user_add(a_uid=user_id,
                           o_givenname=eng_name.split(" ")[0],
                           o_random=True,
                           o_sn =eng_name.split(" ")[1],
                           o_cn=eng_name,
                           o_mail=email,
                           o_preferredlanguage='KR',
                           o_ipauserauthtype='otp')
    return user

def totp_create(client,user_id):
    #o_ipatokenowner 만 지정하면 사용자에게 적용됩니다. (OTP는 한계정당 복수개의 OTP를 가질수있습니다.
    otp = client.otptoken_add(o_ipatokenowner=user_id).get('result')
    otp_uri = otp.get('uri')
    otp_uri_split = otp_uri.split("&")
    #qrcode png저장 
    qr_img = qrcode.make(f"otpauth://totp/OpenVPN_test:{user_id}?{otp_uri_split[1]}")
    qr_img_path = f"/tmp/{user_id}.png"
    qr_img.save(qr_img_path)
    return qr_img_path

def mac_add(user_id,mac_address):
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    k = paramiko.RSAKey.from_private_key_file("/home/jira/admin")
    ssh.connect(hostname='1.1.1.1', username='admin', pkey=k)
    # 원격 서버에 명령 실행
    cmd = f"sudo /home/jira/create.sh {user_id} {mac_address}"
    (stdin, stdout, stderr) = ssh.exec_command(cmd)
    time.sleep(3)
    output = stdout.readlines()
    return output

if __name__ == "__main__":
    user_id = args[1]
    user_eng_name = args[2]
    user_mail = args[3]
    user_mac = args[4]
    
    # formatting
    user_id = re.sub(pattern="@.*", repl="", string=user_id)
    if " " not in user_eng_name:
        user_eng_name += " ."
    user_mac = re.sub(pattern="-", repl=":", string=user_mac).lower()

    # 계정에 대한 내용은 매개변수 또는 암호화 필요.
    client= con_freeipa()
    add_result = user_add(client, user_id, user_eng_name, user_mail)
    otp_result = totp_create(client, user_id)
    mac_result = mac_add(user_id, user_mac)

    print(f"ID: {user_id}")
    print(f"PW: {add_result.get('result').get('randompassword')}")
    print(f"QR: {otp_result}")
```



`/home/jira/create.sh` 

```sh
#/bin/bash
# jira scriptrunner에서 해당 script를 ssh를 통해 동작시킨다.
# $1 : username
# $2 : macaddress
# $3 : groupname
sacli --user $1 --key "conn_group" --value $3 UserPropPut
sacli --user $1 -k pvt_hw_addr -v $2 UserPropPut
```



# Automation

**담당자 지정 알림** 

> 이 방식보단  ScriptRunner를 사용하는 "Assignee 지정되면 Webhook 전송" 예제가 더 정확하고 유연하게 적용 가능하다.

- Jira > 프로젝트 > 프로젝트 설정 > 자동화 > 새 규칙
    - WHEN: `Status changed` 
    - IF: `이슈 일치` > `assignee was EMPTY AND assignee is not EMPTY` 
    - THEN: `WebHook` > URL: `webhook url`, 본문: `사용자 정의 페이로드 보내기`, `{"text": "[${issue.summary}](${request.url}) 건의 담당자가 지정되었습니다."}` 



# Issues

## 상태는 '종료됨'인데 해결책이 '미해결'이라 이슈가 계속 열려있을 때

해결책 필드를 굳이 사용할 필요가 없는 심플한 workflow에서도 이슈 상태는 `종료됨`으로 처리했는데 해결책이 `미해결`이라 이슈가 계속 열린 상태로 보일 수 있다. 

예를 들어 `Test` workflow 상 `진행중` 상태에서 `완료 처리` 전환을 거쳐 `완료` 상태로 가도록 설정되어 있다고 하면

- 프로젝트 설정 > 업무흐름 > `Test` workflow 우측 `편집` 버튼 클릭 > 단계 이름 `진행중` 우측 `완료 처리` 전환 링크 클릭 > 후속 조치 > 후속 조치 추가 > 이슈 필드 업데이트 선택 > 추가
    - 이슈 필드: 해결책
    - 필드 값: 완료



## OOME

- Jira 의 OOME 가 간혹 발생 함: 기존 Machine type은 GCP n1-standard-4 이고 OOME 겪고 8192m로 Heap Memory를 올림
- JVM Memory는 아래와 같이 설정되어 있었다.

```sh
# /data/atlassian/jira/bin/setenv.sh

JVM_MINIMUM_MEMORY="8192m"
JVM_MAXIMUM_MEMORY="8192m"

JAVA_OPTS="-Xms${JVM_MINIMUM_MEMORY} -Xmx${JVM_MAXIMUM_MEMORY} ${JVM_CODE_CACHE_ARGS} ${JAVA_OPTS} ${JVM_REQUIRED_ARGS} ${DISABLE_NOTIFICATIONS} ${JVM_SUPPORT_RECOMMENDED_ARGS} ${JVM_EXTRA_ARGS} ${JIRA_HOME_MINUSD} ${START_JIRA_JAVA_OPTS}"
```

- 이에 Machine type을 Custom(vCPU: 4, Memory: 24G)로 변경하고, MetaSpace 사이즈 옵션도 추가함.

```sh
# /data/atlassian/jira/bin/setenv.sh

JVM_MINIMUM_MEMORY="8192m"
JVM_MAXIMUM_MEMORY="8192m"
JVM_METASPACE_MIN_MEM="2048m"
JVM_METASPACE_MAX_MEM="2048m"

JAVA_OPTS="-XX:MetaspaceSize=${JVM_METASPACE_MIN_MEM} -XX:MaxMetaspaceSize=${JVM_METASPACE_MAX_MEM} -Xms${JVM_MINIMUM_MEMORY} -Xmx${JVM_MAXIMUM_MEMORY} ${JVM_CODE_CACHE_ARGS} ${JAVA_OPTS} ${JVM_REQUIRED_ARGS} ${DISABLE_NOTIFICATIONS} ${JVM_SUPPORT_RECOMMENDED_ARGS} ${JVM_EXTRA_ARGS} ${JIRA_HOME_MINUSD} ${START_JIRA_JAVA_OPTS}"
```

- startup, shutdown은 아래와 같이 진행

```sh
sudo su -
/data/atlassian/jira/bin/startup.sh
/data/atlassian/jira/bin/shutdown.sh
```

- VM Stop > VM Resize > VM Start > Jira Start 진행 완료