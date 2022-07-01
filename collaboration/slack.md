# Commands API

> https://api.slack.com/interactivity/slash-commands 참조



## Verifying requests from slack

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



# Webhook API

Slack app directory에서 Incoming/Outgoing Webhook 앱을 설치하는 방법은 이제 권장되지 않는다. 

> Please note, this is a legacy custom integration - an outdated way for teams to integrate with Slack. These integrations lack newer features and they will be deprecated and possibly removed in the future. We do not recommend their use. Instead, we suggest that you check out their replacement: [Slack apps](https://api.slack.com/start).



- [Slack API > Your Apps](https://api.slack.com/apps) > Create New App으로 우선 앱을 만든다. 
- 생성된 앱을 클릭 > Basic Information > Building Apps for Slack > Add features and functionality > Incoming Webhooks > On
    - 화면 하단 Webhook URLs for Your Workspace에 Webhook URL과 테스트 코드가 생성된다. 
- Outgoing Webhook은 Event Subscriptions > Enable Events 로 켜주면 된다.



## with Jira

- 위와 같이 Slack app을 만들어 Incoming webhook을 enable 설정하는 표준 방법으로 Jira 알림을 받으려면 Jira automation이 필요한 듯 하다. 
- 그러나 Jira automation은 Jira cloud나 기타 Premium 버전에서만 지원되는 것으로 보이므로 다른 방법이 필요하다.
-  Jira에서 Slack용 app을 설치해주자. 이어서 Slack에서도 "Jira Server Alerts(레거시)"와 같은 Jira 전용 앱을 사용하여 Webhook URL을 생성하면 된다. 



# SCIM API

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



# Workflow

## Cloud 작업 요청

```json
{"source_id":"413413781876588037","version":"1","workflow":{"name":"Cloud 작업 요청","blueprint":{"version":"1","trigger":{"type":"channel_action","id":"9d5c43c9-b15c-4ef6-9a91-39e86057c2f1","config":{"name":"Cloud 작업 요청","channels":["C03MUNK0M5J"],"callback_id":"5552f8fa-ef7b-46fe-9f94-b6dc2e78c46d","description":"Cloud 작업 요청"}},"steps":[{"type":"dialog","id":"731d1852-ed1c-4209-9918-96b3aaba3757","config":{"dialog_title":"Cloud 작업 요청","dialog_elements":[{"name":"a07c9533-5451-4eab-b246-db3210489ca7","type":"select","label":"어느 Cloud 인가요?","value":"GCP","options":[{"label":"GCP","value":"GCP"},{"label":"AWS","value":"AWS"},{"label":"ETC","value":"ETC"}],"optional":false,"data_source":"static"},{"name":"5351a595-a417-42ca-8a7e-0a4b5d38c149","type":"select","label":"어떤 환경인가요?","options":[{"label":"ALL","value":"ALL"},{"label":"DEV","value":"DEV"},{"label":"QA","value":"QA"},{"label":"PRE","value":"PRE"},{"label":"PROD","value":"PROD"},{"label":"ETC","value":"ETC"}],"optional":false,"data_source":"static"},{"name":"6bf568fd-d815-4e27-949a-a0bb21b65f33","type":"select","label":"어떤 종류의 작업인가요?","value":"일반 문의","options":[{"label":"일반 문의","value":"일반 문의"},{"label":"계정/권한 요청","value":"계정/권한 요청"},{"label":"방화벽 요청","value":"방화벽 요청"},{"label":"리소스 생성/수정/삭제 요청","value":"리소스 생성/수정/삭제 요청"},{"label":"DB 생성/수정/삭제 요청","value":"DB 생성/수정/삭제 요청"},{"label":"기타","value":"기타"}],"optional":false,"data_source":"static"},{"name":"9d3ee147-4fb8-4418-a3a1-ed3451c7f295","type":"textarea","label":"요청 내용을 써주세요.","optional":false}],"dialog_submit_label":"","delivery_button_label":"양식 열기","delivery_message_text":"안녕하세요! 시작하려면 이 양식을 작성해주세요."}},{"type":"dialog","id":"0fa11628-c336-4511-8bad-34e24a333f3a","config":{"channel":{"value":"C03M06FU2GG"},"dialog_title":"Cloud 작업 담당자 지정","dialog_elements":[{"name":"030cd53d-df82-4f85-b777-098d89d6aa78","type":"select","label":"담당자 지정","optional":false,"data_source":"users"},{"name":"36e05cfd-8916-4bd8-811a-13a31553e494","type":"select","label":"승인자 지정","optional":false,"data_source":"users"}],"dialog_submit_label":"","delivery_button_label":"담당자 지정","delivery_message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}","delivery_dialog_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]}]}]}},{"type":"message","id":"7e0118ec-f928-4494-88bc-b0606c87aa5f","config":{"channel":{"value":"C03M06FU2GG"},"has_button":false,"message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}\n\n담당자: {{0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user}}\n승인자: {{0fa11628-c336-4511-8bad-34e24a333f3a==36e05cfd-8916-4bd8-811a-13a31553e494==user}}","message_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]},{"type":"rich_text_section","elements":[{"text":"\n","type":"text"}]},{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"담당자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"승인자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==36e05cfd-8916-4bd8-811a-13a31553e494==user","type":"workflowtoken","property":"","data_type":"user"}]}]},{"type":"rich_text_section","elements":[]}]}]}},{"type":"dialog","id":"e7339bd1-50e6-4b42-824d-1dbaa8f76dd3","config":{"user":{"ref":"0fa11628-c336-4511-8bad-34e24a333f3a==36e05cfd-8916-4bd8-811a-13a31553e494==user"},"dialog_title":"Cloud 작업 승인","dialog_elements":[{"name":"cd9fdc94-276b-48b3-8a2a-29bf07a546bc","type":"select","label":"승인 여부","value":"승인","options":[{"label":"승인","value":"승인"},{"label":"반려","value":"반려"}],"optional":false,"data_source":"static"},{"name":"9d3ea6c0-f94c-4b90-9400-54e06601c71b","type":"textarea","label":"설명","optional":true}],"dialog_submit_label":"","delivery_button_label":"승인 여부","delivery_message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}\n\n담당자: {{0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user}}\n승인자: {{0fa11628-c336-4511-8bad-34e24a333f3a==36e05cfd-8916-4bd8-811a-13a31553e494==user}}","delivery_dialog_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]},{"type":"rich_text_section","elements":[{"text":"\n","type":"text"}]},{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"담당자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"승인자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==36e05cfd-8916-4bd8-811a-13a31553e494==user","type":"workflowtoken","property":"","data_type":"user"}]}]}]}]}},{"type":"dialog","id":"a1179654-a1f4-4aba-88b2-e09919b3291b","config":{"user":{"ref":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user"},"dialog_title":"Cloud 작업 상태 변경","dialog_elements":[{"name":"5167afed-7b16-4c5a-9a4a-dc86c5081031","type":"select","label":"작업 진행 여부","value":"진행중","options":[{"label":"진행중","value":"진행중"},{"label":"진행하지 않음","value":"진행하지 않음"}],"optional":false,"data_source":"static"},{"name":"0910b4d0-75b9-498f-8abc-487ac6ab69e5","type":"textarea","label":"설명","optional":true}],"dialog_submit_label":"","delivery_button_label":"작업 진행 여부","delivery_message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}\n\n담당자: {{0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user}}\n승인자: {{0fa11628-c336-4511-8bad-34e24a333f3a==36e05cfd-8916-4bd8-811a-13a31553e494==user}}\n승인 여부: {{e7339bd1-50e6-4b42-824d-1dbaa8f76dd3==cd9fdc94-276b-48b3-8a2a-29bf07a546bc==text}}\n승인 설명: {{e7339bd1-50e6-4b42-824d-1dbaa8f76dd3==9d3ea6c0-f94c-4b90-9400-54e06601c71b==text}}","delivery_dialog_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]},{"type":"rich_text_section","elements":[{"text":"\n","type":"text"}]},{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"담당자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"승인자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==36e05cfd-8916-4bd8-811a-13a31553e494==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"승인 여부: ","type":"text"},{"id":"e7339bd1-50e6-4b42-824d-1dbaa8f76dd3==cd9fdc94-276b-48b3-8a2a-29bf07a546bc==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"승인 설명: ","type":"text"},{"id":"e7339bd1-50e6-4b42-824d-1dbaa8f76dd3==9d3ea6c0-f94c-4b90-9400-54e06601c71b==text","type":"workflowtoken","property":"","data_type":"text"}]}]}]}]}},{"type":"message","id":"95e2b410-db38-4fc4-b6b9-7be6e52efb22","config":{"channel":{"value":"C03M06FU2GG"},"has_button":false,"message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}\n\n담당자: {{0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user}}\n진행 여부: {{a1179654-a1f4-4aba-88b2-e09919b3291b==5167afed-7b16-4c5a-9a4a-dc86c5081031==text}}\n설명: {{a1179654-a1f4-4aba-88b2-e09919b3291b==0910b4d0-75b9-498f-8abc-487ac6ab69e5==text}}","message_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]},{"type":"rich_text_section","elements":[{"text":"\n","type":"text"}]},{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"담당자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"진행 여부: ","type":"text"},{"id":"a1179654-a1f4-4aba-88b2-e09919b3291b==5167afed-7b16-4c5a-9a4a-dc86c5081031==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"설명: ","type":"text"},{"id":"a1179654-a1f4-4aba-88b2-e09919b3291b==0910b4d0-75b9-498f-8abc-487ac6ab69e5==text","type":"workflowtoken","property":"","data_type":"text"}]}]}]}]}},{"type":"message","id":"3f7dc928-d885-46ad-862a-870cb6a4f040","config":{"user":{"ref":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted"},"has_button":false,"message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}\n\n담당자: {{0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user}}\n진행 여부: {{a1179654-a1f4-4aba-88b2-e09919b3291b==5167afed-7b16-4c5a-9a4a-dc86c5081031==text}}\n설명: {{a1179654-a1f4-4aba-88b2-e09919b3291b==0910b4d0-75b9-498f-8abc-487ac6ab69e5==text}}","message_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]},{"type":"rich_text_section","elements":[{"text":"\n","type":"text"}]},{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"담당자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"진행 여부: ","type":"text"},{"id":"a1179654-a1f4-4aba-88b2-e09919b3291b==5167afed-7b16-4c5a-9a4a-dc86c5081031==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"설명: ","type":"text"},{"id":"a1179654-a1f4-4aba-88b2-e09919b3291b==0910b4d0-75b9-498f-8abc-487ac6ab69e5==text","type":"workflowtoken","property":"","data_type":"text"}]}]}]}]}},{"type":"dialog","id":"4160bf5c-6a72-4d0e-8070-13537e70fbd0","config":{"user":{"ref":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user"},"dialog_title":"Cloud 작업 종료","dialog_elements":[{"name":"2cedba1f-439e-40f9-9540-2dbc0e8802e0","type":"select","label":"종료 여부","value":"종료","options":[{"label":"종료","value":"종료"},{"label":"진행 불가능","value":"진행 불가능"}],"optional":false,"data_source":"static"},{"name":"bd9096a0-252e-4ed2-8419-1aeaab9719ca","type":"textarea","label":"설명","optional":true}],"dialog_submit_label":"","delivery_button_label":"종료 여부","delivery_message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}\n\n담당자: {{0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user}}\n진행 여부: {{a1179654-a1f4-4aba-88b2-e09919b3291b==5167afed-7b16-4c5a-9a4a-dc86c5081031==text}}\n설명: {{a1179654-a1f4-4aba-88b2-e09919b3291b==0910b4d0-75b9-498f-8abc-487ac6ab69e5==text}}","delivery_dialog_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]},{"type":"rich_text_section","elements":[{"text":"\n","type":"text"}]},{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"담당자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"진행 여부: ","type":"text"},{"id":"a1179654-a1f4-4aba-88b2-e09919b3291b==5167afed-7b16-4c5a-9a4a-dc86c5081031==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"설명: ","type":"text"},{"id":"a1179654-a1f4-4aba-88b2-e09919b3291b==0910b4d0-75b9-498f-8abc-487ac6ab69e5==text","type":"workflowtoken","property":"","data_type":"text"}]}]}]}]}},{"type":"message","id":"0646ff6b-7757-4044-baea-092aa4c8afac","config":{"channel":{"value":"C03M06FU2GG"},"has_button":false,"message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}\n\n담당자: {{0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user}}\n종료 여부: {{4160bf5c-6a72-4d0e-8070-13537e70fbd0==2cedba1f-439e-40f9-9540-2dbc0e8802e0==text}}\n설명: {{4160bf5c-6a72-4d0e-8070-13537e70fbd0==bd9096a0-252e-4ed2-8419-1aeaab9719ca==text}}","message_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]},{"type":"rich_text_section","elements":[{"text":"\n","type":"text"}]},{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"담당자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"종료 여부: ","type":"text"},{"id":"4160bf5c-6a72-4d0e-8070-13537e70fbd0==2cedba1f-439e-40f9-9540-2dbc0e8802e0==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"설명: ","type":"text"},{"id":"4160bf5c-6a72-4d0e-8070-13537e70fbd0==bd9096a0-252e-4ed2-8419-1aeaab9719ca==text","type":"workflowtoken","property":"","data_type":"text"}]}]}]}]}},{"type":"message","id":"96ab4c3e-45fd-458a-be9e-a9dd5e68e3a1","config":{"user":{"ref":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted"},"has_button":false,"message_text":"요청자: {{731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted}}\nCloud: {{731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text}}\n환경: {{731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text}}\n작업 종류: {{731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text}}\n요청 내용: {{731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text}}\n\n담당자: {{0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user}}\n종료 여부: {{4160bf5c-6a72-4d0e-8070-13537e70fbd0==2cedba1f-439e-40f9-9540-2dbc0e8802e0==text}}\n설명: {{4160bf5c-6a72-4d0e-8070-13537e70fbd0==bd9096a0-252e-4ed2-8419-1aeaab9719ca==text}}","message_blocks":[{"type":"rich_text","elements":[{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"요청자: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==user_submitted","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"Cloud: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==a07c9533-5451-4eab-b246-db3210489ca7==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"환경: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==5351a595-a417-42ca-8a7e-0a4b5d38c149==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"작업 종류: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==6bf568fd-d815-4e27-949a-a0bb21b65f33==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"요청 내용: ","type":"text"},{"id":"731d1852-ed1c-4209-9918-96b3aaba3757==9d3ee147-4fb8-4418-a3a1-ed3451c7f295==text","type":"workflowtoken","property":"","data_type":"text"}]}]},{"type":"rich_text_section","elements":[{"text":"\n","type":"text"}]},{"type":"rich_text_list","style":"bullet","indent":0,"elements":[{"type":"rich_text_section","elements":[{"text":"담당자: ","type":"text"},{"id":"0fa11628-c336-4511-8bad-34e24a333f3a==030cd53d-df82-4f85-b777-098d89d6aa78==user","type":"workflowtoken","property":"","data_type":"user"}]},{"type":"rich_text_section","elements":[{"text":"종료 여부: ","type":"text"},{"id":"4160bf5c-6a72-4d0e-8070-13537e70fbd0==2cedba1f-439e-40f9-9540-2dbc0e8802e0==text","type":"workflowtoken","property":"","data_type":"text"}]},{"type":"rich_text_section","elements":[{"text":"설명: ","type":"text"},{"id":"4160bf5c-6a72-4d0e-8070-13537e70fbd0==bd9096a0-252e-4ed2-8419-1aeaab9719ca==text","type":"workflowtoken","property":"","data_type":"text"}]}]}]}]}}]}}}
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

