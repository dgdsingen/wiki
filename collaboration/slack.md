# API

## Commands API

> https://api.slack.com/interactivity/slash-commands 참조



### Verifying requests from slack

> https://api.slack.com/authentication/verifying-requests-from-slack 참조

- 유의할 점은 slack의 경우 request를 verifying하는데 request body를 raw로 사용한다는 점이다. 이는 FastAPI 사용상의 제약을 가져온다. FastAPI에서는 function에 parameter로 아래와 같은 것들을 사용할 수 있는데,
    - `user_agent = Header(...)` : request header 중 name이 일치하는 것을 가져온다. ex: `request.headers.get("User-Agent")`
    - `payload = Body(...)` : request body의 raw 값을 그대로 가져온다. ex: `request.body()` b"text=1&param2=2"와 같이 나온다.
    - `text = Form(...)` : request body에 x-www-form-urlencoded 등 form encoding된 값 중 name이 일치하는 것을 가져온다. ex: `request.form().get('text')`
- request body의 raw 값을 얻기 위해 `payload = Body(...)`를 사용하는 순간 request.body()의 stream이 닫혀 버리면서 `text = Form(...)`과 공존할 수 없게 된다.
- Middleware를 사용해서 call_next(request)를 수행해도 마찬가지 에러가 난다. 하여 아래와 같이 request 자체를 parameter로 받아서 slack verifying에서는 request.body()를 쓰고, 나머지 부분에서는 request.form()을 쓰니 문제없이 잘 수행된다. 

```python
import os, time
import hmac, hashlib
from fastapi import APIRouter
from starlette.requests import Request

router = APIRouter(
    prefix='/slack',
    tags=['slack']
)


async def valid_from_slack(request: Request):
    signing_secret = os.environ['SLACK_SIGNING_SECRET']
    requeset_headers = request.headers
    payload = await request.body()
    x_slack_request_timestamp = requeset_headers.get('X-Slack-Request-Timestamp')
    if time.time() - float(x_slack_request_timestamp) > 60 * 5:
        return False

    sig_basestring = f'v0:{str(x_slack_request_timestamp)}:{payload.decode()}'
    request_hash = f'v0={hmac.new(str.encode(signing_secret), str.encode(sig_basestring), hashlib.sha256).hexdigest()}'
    x_slack_signature = request.headers.get('X-Slack-Signature')
    return hmac.compare_digest(request_hash, x_slack_signature)


@router.post('/test')
async def test(request: Request):
    if await valid_from_slack(request):
        form = await request.form()
        return form.get('text')
    else:
        return 'Invalid request'
```



## Webhook API

Slack app directory에서 Incoming/Outgoing Webhook 앱을 설치하는 방법은 이제 권장되지 않는다. 

> Please note, this is a legacy custom integration - an outdated way for teams to integrate with Slack. These integrations lack newer features and they will be deprecated and possibly removed in the future. We do not recommend their use. Instead, we suggest that you check out their replacement: [Slack apps](https://api.slack.com/start).



- [Slack API > Your Apps](https://api.slack.com/apps) > Create New App으로 우선 앱을 만든다. 
- 생성된 앱을 클릭 > Basic Information > Building Apps for Slack > Add features and functionality > Incoming Webhooks > On
    - 화면 하단 Webhook URLs for Your Workspace에 Webhook URL과 테스트 코드가 생성된다. 
- Outgoing Webhook은 Event Subscriptions > Enable Events 로 켜주면 된다.



### with Jira

- 위와 같이 Slack app을 만들어 Incoming webhook을 enable 설정하는 표준 방법으로 Jira 알림을 받으려면 Jira automation이 필요한 듯 하다. 
- 그러나 Jira automation은 Jira cloud나 기타 Premium 버전에서만 지원되는 것으로 보이므로 다른 방법이 필요하다.
-  Jira에서 Slack용 app을 설치해주자. 이어서 Slack에서도 "Jira Server Alerts(레거시)"와 같은 Jira 전용 앱을 사용하여 Webhook URL을 생성하면 된다. 



## SCIM API

SCIM API는 Slack에 사용자 등을 Provisioning하기 위해 사용되는 표준이다. 

예를 들어 Web API는 Workspace Level의 권한을 가진 API라 표시 이름(display_name)이 안된다.

표시 이름(display_name)을 수정하기 위해서는 Org Level 권한을 가진 SCIM API를 사용해야 한다. 



SCIM API를 사용하려면 우선 Org Level App을 만들어야 한다. 

- https://api.slack.com/apps > Features > OAuth & Permissions > Scopes
    - Bot Token Scopes: channels:join, users:read, users:write
    - User Token Scopes: admin, channels:read, groups:read, users.profile:read, users.profile:write, users:read, users:write
- https://api.slack.com/apps > Features > Org Level Apps > Org apps program > Opt-In
    - Opt-In 버튼 활성화를 위해선 위에서 Bot Token Scopes을 추가해줘야 가능하다. 그러나 꼭 위와 같이 Scope을 설정해야 하는 것은 아니다. 필요한 Scope만 넣자. 



이후 아래와 같이 SCIM API 호출 가능하다.

- https://api.slack.com/apps > Features > OAuth & Permissions > Scopes > OAuth Tokens for Your Workspace > User OAuth Token > Copy

```sh
host="https://api.slack.com"
token="xoxp-123456" # User OAuth Token

# Get Users
http $host/scim/v1/Users Authorization:"Bearer $token" count==10000 | jq -c ".Resources [] | select(.active==true) .id + \":::\" + .userName + \":::\" + .displayName"

# Get User's profile
http -v $host/scim/v1/Users/U027GAD3UJC Authorization:"Bearer $token"

# Set User's profile
http -v PATCH $host/scim/v1/Users/U027GAD3UJC Authorization:"Bearer $token" displayName=""
```



# Issues

## 사용자 이름 변경

Slack에서 사용되는 사용자 이름은 총 3가지가 존재한다. 

- 성명(real_name): 보통 사용자의 Full Name을 의미한다.
- 표시 이름(display_name): Slack에서 기본적으로 노출되는 사용자명
- 사용자 이름(name): SAML SSO 연동시 ID 역할을 한다.



Slack과 SAML로 SSO 연동하는 경우, Slack에서 제공하는 유저명 관련 SAML Attribute는 다음과 같다. 

- NameID
- User.Username
- first_name
- last_name



Slack에서 사용되는 사용자 이름과 SAML Attribute를 매핑하면 아래와 같다. 

- 성명(real_name): first_name + last_name
- 표시 이름(display_name): User.Username
- 사용자 이름(name): User.Username

여기서 User.Username가 문제가 된다. 사용자 이름(name)이 SAML SSO 연동시 ID 역할을 하기 때문에, IDP에서는 사번과 같은 Unique한 난수값을 전달할 수 있다. 

그런데 그 난수값이 표시 이름(display_name)으로도 설정되어 버리니 Slack 사용자들끼리 멘션하는 경우 서로를 식별하기 어렵다. 



이 때 아래와 같이 설정을 변경하면 표시 이름(display_name) 대신 성명(real_name)으로 표기할 수 있다.

- Slack > 설정 > 조직 정책 > 이름 표시 > 성명을 기본값으로 설정 (default = 표시 이름을 기본값으로 설정)

그러나 이것도 메시지 작성시에 유저 멘션할때는 성명(real_name)이 잘 뜨지만, 발송한 후 메시지의 유저 멘션 부분에는 다시 표시 이름(display_name)이 표기된다. 



이럴 때는 아래와 같이 설정해서 우선 사용자가 임의로 표시 이름(display_name)을 수정하지 못하게 막는다. 

- Slack > 조직 설정 > 보안 > SSO 설정 > 동기화 사용자 프로필 > 활성화
- Slack > 조직 설정 > 보안 > SSO 설정 > 사용자가 자신의 표시 이름을 변경하도록 허용 > 비활성화

이후 사용자의 표시 이름(display_name)을 비워버리면, 이제 메시지 작성시/작성후 모두 성명(real_name)이 표기된다. 



다만 이렇게 해도 IDP => Slack으로 SAML SSO 될 때마다 표시 이름(display_name)이 update 된다. 이에 다음과 같은 해결책을 적용할 수 있다.

- Slack SAML은 displayName 필드가 아예 없어서 수정 불가능. ID(사번)를 받으면 그 값을 자동으로 displayName에 넣는 로직임

- Slack Web API도 displayName 수정 불가능. (Web API는 Workspace Level 권한인데, displayName 수정은 Org Level 권한이 필요함)

- Slack SCIM API는 displayName 수정 가능함. (SCIM API는 Org Level 권한임)



즉 Slack SCIM API를 호출하여 표시 이름(display_name)을 수정해주는 방식이다. 아래와 같이 진행한다. 


- Slack에 Event Trigger(user_change)를 추가하고, 그 Event를 받기 위한 API 서버를 준비함

- User의 displayName 변경해봄

- Slack Event Trigger(user_change)가 발생하여 API 서버로 전달됨

- API 서버에서 Slack SCIM API 호출하여 해당 user의 displayName을 지움

```sequence
User --> IDP: Sign-in
IDP --> Slack: SAML SSO(update User's displayName='사번')
Slack --> API Server: Event Trigger(user_change)
API Server --> Slack: SCIM API(update displayName='')
```

SCIM API를 호출하기 위해선 [SCIM API](#SCIM-API)을 참조하여 Org Level App을 먼저 준비하고, 이후 Event Subscriptions를 추가한다.

- https://api.slack.com/apps > Features > Event Subscriptions
    - Enable Events: On
    - Request URL: API Endpoint 기입. 최초 기입시 API Endpoint로 validation을 위한 [Event(url_verification)](https://api.slack.com/events/url_verification)가 호출된다. 
    - Subscribe to events on behalf of users: user_change, team_join



이렇게 Event Trigger(user_change, member_joined_channel)에 동일하게 API 적용해주면 

- 사용자 계정이 처음 Slack에 Provisioning 되거나
- 이후 IDP를 통해 SAML SSO로 재로그인 하거나 

모든 경우에 표시 이름(display_name)을 자동으로 수정해주고, 사용자는 해당 속성을 변경할 수 없으니 문제가 해결된다. 



# References

- [Slack SAML SSO](https://slack.com/intl/ko-kr/help/articles/203772216-SAML-Single-Sign-On)
- [Slack Custom SAML SSO](https://slack.com/intl/ko-kr/help/articles/205168057-사용자-지정-SAML-Single-Sign-On)
- [Slack SAML SSO Errors](https://slack.com/intl/ko-kr/help/articles/360037402653-SAML-인증-오류-문제-해결)
- [Connect groups from your identity provider to Slack](https://slack.com/intl/ko-kr/help/articles/115001435788-Enterprise-Grid-조직에-ID-제공업체-그룹-연결)
- [Slack 접속 네트워크 제한](https://slack.com/intl/ko-kr/help/articles/360024821873-네트워크에-대해-Slack-워크스페이스-승인)

