# Config

## config SAML with Slack

https://www.okta.com/free-trial > 30일 Free Trial 등록



https://subdomain.okta.com > Admin > Applications > Applications > Add Application > Slack > Add

- Slack의 subdomain 입력 : https://test.slack.com 인 경우 test 입력 후 Next (만약 https://test.enterprise.slack.com이라면 test.enterprise 입력)
- 'View Setup Instruction' 버튼 클릭해 SAML 2.0 연동 가이드대로 값 입력. Okta 먼저 설정 완료하고, 아직 Slack에서 Test 버튼은 누르지 말자. 



https://subdomain.okta.com > Admin > Applications > Applications > Slack > Assign > Assign to People

- 테스트할 계정을 추가해준다. 여기까지 완료되면 Slack에서 Test 버튼 눌러 Assign된 계정으로 SAML SSO 연동되는지 확인한다. 



## config Policy

https://subdomain.okta.com > Admin > Security > Networks > LegacyIpZone > Edit

- Gateway IPs : CIDR로 IP 대역 입력



https://subdomain.okta.com > Admin > Applications > Applications > Slack > Sign On > Sign On Policy > Add Rule

- Rule Name : PC 외부망 접속 차단
    - Who does this rule apply to? : Users assigned this app
    - If the user is located : Not in Zone
    - Network Zones : LegacyIpZone
    - If the user's platform is any of these : 
        - iOS, Android, Other mobile (e.g. BlackBerry) 체크 해제
        - Windows, macOS, Other desktop (e.g. Linux) 체크
    - If the device is : Any
    - When all the conditions above are met, sign on to this application is : Denied
- Rule Name : Mobile 접속 허용
    - Who does this rule apply to? : Users assigned this app
    - If the user is located : Anywhere
    - If the user's platform is any of these : 
        - iOS, Android, Other mobile (e.g. BlackBerry) 체크
        - Windows, macOS, Other desktop (e.g. Linux) 체크 해제
    - If the device is : Any
    - When all the conditions above are met, sign on to this application is : Allowed



https://subdomain.okta.com > Admin > Applications > Applications > Slack > Sign On > Sign On Policy

- Policy 순서 조정 : 
    1. Mobile 접속 허용
    2. PC 외부망 접속 차단
    3. Default sign on rule



Okta Admin의 접속 Policy도 설정 가능하다. 

https://subdomain.okta.com > Admin > Security > Authentication > Sign On > Add Rule

- Rule Name : 외부망 접속 차단
    - IF User's IP is : Not in Zone
        - LegacyIpZone
    - AND Authenticates via : Any
    - THEN Access is : Denied



## Guest계정을 Okta를 통하여 로그인 하는 방법

Okta의 Conditional Access 사용으로 지정된 IP Range에서만 로그인이 가능한 정책 사용필요시

기본적으로 Okta를 통하여 유저를 provisioning 하는 경우 Full Member로만 사용이 가능합니다. 그러나 아래 방법을 사용할 수 있습니다.

1. Okta에 Guest로 사용할 계정 생성
    - 꼭 Okta에서 수기로 생성할 필요는 없고, Okta 로그인시 AD 통해서 JIT Provisioning되는 계정도 사용 가능하다.
    - 다만 이때는 사용자에게 Slack 게스트 초청 메일 발송 > 링크 클릭해서 Okta에 로그인 > Okta에 JIT Provisioning > Okta에서 해당 계정 Slack App에 Assign 해야 한다.
2. Slack Workspace 관리 페이지 => 멤버관리 => 사용자 초대하기 (게스트 지정 후 게스트가 참여할 채널지정)
3. Okta에서 Guest로 사용할 계정을 Slack App에 Assign 
4. 2를 실행하면 해당 유저에게 초대 이메일이 발송되며, "Join Now"를 선택하면 Slack 으로 연결되며 게스트로 로그인 가능 



Slack에서 유저타입 확인을 어떻게 하는가?

- 조직대시보드 => 멤버 => 계정유형 확인



## Issues

### VERIFICATION_ERROR when login

ID/PW를 잘못 입력하는 경우도 있지만, AD에 givenName(firstName), sn(lastName)이 비어있는 경우에도 에러가 발생할 수 있다. 

이 경우 Admin > Directory > Profile Editor 로 가서 아래와 같이 설정한다.

- Okta User (Default) > Profile > firstName/lastName > i > Uncheck Attribute required 'Yes' > Save Attribute
- AD User > Profile > firstName/lastName > i > Uncheck Attribute required 'Yes' > Save Attribute

만약 okta와 연동하는 application에서 firstName/lastName이 nullable하지 않다면 아래와 같이 임의의 값을 넣어주어도 된다.

- appuser.firstName != null ? appuser.firstName : 'unknown'
- appuser.lastName != null ? appuser.lastName : 'user'



## firstName, lastName 변경

Slack 등 외부 서비스와 연동하는 경우 firstName, lastName이 전달되는데 이 값의 내용을 바꾸어 노출하고 싶은 경우가 있다. 

예를 들어 "{회사명} {이름}"이나 "{부서명} {이름}"과 같이 노출해야 하는 경우 company, department와 같은 속성을 Okta > Slack으로 전달하여 노출시켜야 한다. 

그런데 Slack에서 company, department를 노출해줄 옵션이 없다면 Okta에서 firstName, lastName값을 변경해주는 수 밖에 없다. 이때는 아래와 같이 설정한다. 

- Okta Admin > Directory > Profile Editor > AD > Profile > Mapping
    - firstName : appuser.department != null ? String.substring(String.substringBefore(appuser.department, "("), 0, 50) : "부서명없음"
    - lastName : appuser.displayName != null ? String.substring(appuser.displayName, 0, 50) : appuser.firstName != null && appuser.lastName != null ? String.substring(appuser.firstName, 0, 24) + " " + String.substring(appuser.lastName, 0, 24) : "이름없음"



## Slack DisplayName 변경

Admin > Applications > Applications > Slack > Sign On > Settings > Edit

- Application username format : Custom
    - user.displayName

위 설정 적용 후 반드시 'Update Now' 버튼을 눌러주자.



# References

- [Okta - How to Configure SAML 2.0 for Slack](https://saml-doc.okta.com/SAML_Docs/How-to-Configure-SAML-2.0-for-Slack.html?baseAdminUrl=https://test-admin.okta.com&app=slack&instanceId=0oaoldkjhZrjCyo9k5d6)

