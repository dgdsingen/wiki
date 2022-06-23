# Server

- License: 각 1EA 씩
    - Jira Software (Server) 10 user: New Commercial License Includes 12 months Support and Maintenance
    - Confluence (Server) 10 user: New Commercial License Includes 12 months Support and Maintenance
    - The Pivot Gadget (Server) 10 user: New Commercial License Includes 12 months Support and Maintenance
    - WBS Gantt-Chart for JIRA (Server) 10 user: New Commercial License Includes 12 months Support and Maintenance
    - JSU Automation Suite For Jira Workflows (Server) 10 user: New Commercial License Includes 12 months Support and Maintenance
    - Scriptrunner for JIRA (Server) 10 user: New Commercial License Includes 12 months Support and Maintenance
    - Draw.io Diagrams for Confluence (Server) 10 user: New Commercial License Includes 12 months Support and Maintenance
- Install
    - atlassian-jira-software-8.14.0-x64.bin
    - atlassian-confluence-7.10.0-x64.bin
    - Plugins
        - groovyrunner-6.16.0.jar
        - jira-suite-utilities-2.24.3.jar
        - pivotgadget-1.8.2-jira67.jar
        - wbsgantt-for-jira-9.13.1.1.obr
        - drawio-confluence-plugin-9.5.0.obr
- Server
    - mv-jira: 172.31.1.2:8080
        - /data/jira
            - /data/jira/bin/startup.sh
            - /data/jira/bin/shutdown.sh
        - `{JIRA_HOME}/bin/setenv.sh` 에서 JVM 메모리 설정
        - Jira가 사용자/그룹 데이터 마스터 역할을 한다. Jira에 사용자/그룹 추가 후 Conf > 설정 > 사용자 디렉토리 > Jira Server 동기화 클릭하면 Conf에 사용자/그룹이 싱크된다.
        - admin1/admin1234
    - mv-jira-conf: 172.31.69.3:8080
        - /data/confluence
            - /data/confluence/bin/startup.sh
            - /data/confluence/bin/shutdown.sh
        - `{CONFLUENCE_HOME}/bin/setenv.sh` 에서 JVM 메모리 설정
        - admin/admin
    - mv-jira-db: 172.31.69.4:5432
        - yum으로 설치했고 Service 등록된 상태: `sudo systemctl start/stop/status postgresql` 
        - 접속 방법

```sh
sudo su - postgres

psql

# JIRA Database: jiradb
# JIRA DB User : jirauser
# JIRA DB Password : jirapass
# Confluence Database : confdb
# Confluence DB User : confuser
# Confluence DB Password : confpass

create user jirauser with password 'jirapass';
CREATE DATABASE jiradb WITH ENCODING 'UNICODE' LC_COLLATE 'C' LC_CTYPE 'C' TEMPLATE template0;
GRANT ALL PRIVILEGES ON DATABASE jiradb TO jirauser;

create user confuser with password 'confpass';
CREATE DATABASE confdb WITH ENCODING 'UTF8';
GRANT ALL PRIVILEGES ON DATABASE confdb TO confuser;

# backup: DB backup하는 경우 home directory(data저장위치)는 별도로 backup 필요
sudo pg_dump --host=localhost --port=5432 --username=jirauser --password --dbname=jiradb > /home/jira/jiradb_backup.sql
Password: jirapass

sudo pg_dump --host=localhost --port=5432 --username=confuser --password --dbname=confdb > /home/jira/confdb_backup.sql
Password: confpass

## Xml backup (서비스에 등록되어 있습니다): 매일 주기로 xml backup이 이루어지며 아래 경로에 저장됩니다.
JIRA Backup File Location : /data/jira-data/export
Confluence Backup File Location : /data/confluence-data/backups

## Xml backup 실행 주기 및 시간은 따로 설정할 수 있습니다. Xml backup의 경우에도 마찬가지로 data는 따로 백업이 필요합니다.
JIRA : admin menu – System – Services – Backup Service
Confluence : admin menu – General – Scheduled jobs

## 수동 XML Backup :
JIRA : System – Backup System 에서 시행 가능
Confluence : General – Backup & Restore 에서 시행 가능

## Data backup: Data home directory 전체를 주기적으로 backup 해주시면 됩니다.
```



# JSM

## Create New Project

- Jira 관리 > 이슈 > 이슈 유형 > 이슈 유형 추가
    - 이슈 유형 만들고, (이슈 유형 계획 = 템플릿)임. 이 것을 예를 들어 "팀장승인"이라고 만들어 놓고
    - 해당 프로젝트 > 프로젝트 설정 > 이슈 유형 > 조치 > 다른계획사용 > 여기에서 불러와서 사용
    - 기존 것을 그대로 가져다 쓰면 내용 변경시 다른 프로젝트에 영향을 줄 수 있으므로 기존 것을 복사해서 오브젝트를 새로 만들어 사용하자
- Jira 관리 > 이슈 > 업무 흐름
    - 업무 흐름 만들고, 업무 흐름 계획에 매핑해서 사용
- Jira 관리 > 이슈 > 화면
    - 화면: Jira Task 들어가면 나오는 설명, 요약, 담당자 등의 필드들이 나오는 화면을 의미함
    - 화면추가 > 해당 화면에 어느 필드를 쓸지를 선택해서 넣어줌
    - 이슈 유형 화면 계획: 이슈마다 사용하는 필드들이 다르므로 각각 설정하기 위함
    - 화면 만들고 > 화면 계획 만들고 > 이 화면은 어느 이슈 유형에 쓸건지 설정. 해당 프로젝트마다 쓰는 방식으로 해야한다. 전체 확장하면 프로젝트 구분없이 모두 보여서 관리하기 어렵다.



- Jira 관리 > 이슈 > 이슈 유형 > 이슈 유형 추가
    - 이름 : Infra_PM_Prd_Issue
    - 설명 : PRD 환경 인프라 PM 이슈유형
    - 유형 : 표준 이슈 유형
- Jira 관리 > 이슈 > 이슈 유형 계획 > 이슈 유형 스키마 추가
    - 구성표이름: Infra_PM_Prd_Issue_Type
    - 설명: Test 인프라 PM에 사용하는 이슈 유형 계획
    - 기본 이슈 유형: Infra_PM_Prd_Issue
    - (프로젝트 > Test Service Request (TESTSR) > 프로젝트 설정 > 이슈 유형 > 이슈유형수정 > 현재 사용중인 이슈 유형에 Infra_PM_Prd_Issue 끌어다 놓음)

- Jira 관리 > 화면 > 화면 추가
    - 이름: Infra_Test_PM_Prd_View
- Jira 관리 > 사용자정의 필드 > 사용자 정의 필드 추가 > 사용자 선택기 (복수 선택) 선택
    - 이름: `Infra_Test_PM_승인자`
    - 이슈 유형 선택: `Infra_PM_Prd_Issue` (위에서 만든 이슈에서만 사용할 것임)
    - 컨텍스트 선택: 선택한 프로젝트의 이슈에 적용
    - 프로젝트 선택: Test Service Request (이 프로젝트에서만 사용)
    - 화면으로 전환됨. 화면 > `Infra_Test_PM_Prd_View` 선택 (`Infra_Test_PM_승인자` 필드를 화면과 연동)
- Jira 관리 > 사용자정의 필드 > 사용자 정의 필드 추가 > URL 항목란 선택
    - 이름: `Infra_Test_PM_작업계획서`
    - 설명: Confluence에 있는 PM 계획 템플릿, 변경 작업서, 영향도 분석서, 테크니컬 아키텍처, 비상조직연락망 링크 등록  입력
    - 이슈유형선택: `Infra_PM_Prd_Issue` (위에서 만든 이슈에서만 사용할 것임)
    - 컨텍스트선택: 선택한 프로젝트의 이슈에 적용
    - 프로젝트 선택: Test Service Request (이 프로젝트에서만 사용)
    - 화면으로 전환됨. 화면 > `Infra_Test_PM_Prd_View` 선택 (`Infra_Test_PM_작업계획서` 필드를 화면과 연동)
        - 화면: `Infra_Test_PM_전파대상할당` (화면 만들어야 함)
            - 필드: `Infra_Test_PM_전파대상자` (사용자 필드임)
            - 타입: 사용자 선택기 (복수 선택)
        - 화면: `Infra_Test_PM_작업자할당` (화면 만들어야 함)
            - 필드: 담당자
            - 타입: 시스템필드
        - 화면: `Infra_Test_PM_작업완료검증` (화면 만들어야 함)
            - 필드: End date
            - 타입: 날짜 선택기
            - 필드: `Infra_Test_PM작업완료 증빙` (사용자 필드임)
            - 타입: URL 항목란
- Jira 관리 > 업무흐름 > 업무흐름 추가
    - 이름: `Infra_Test_PM_Prd_WorkFlow`
    - 설명: Test 인프라 PM 워크플로우
    - 워크플로우 생성 (상태 추가)
        - 이름: `Infra_Test_PM_Prd_작업요청`
        - 설명: PM 요청시 아래 템플릿 링크를 달아주세요. 컨플루언스에 있는 [PM 계획 템플릿, 구성/변경 작업서, 영향도 분석서, 테크니컬 아키텍처, 비상조직 연락망]
        - 분류: 할 일
        - 이름: `Infra_Test_PM_Prd_작업승인`
        - 설명: 내부회계감사에 반드시 필요한 절차입니다. 요청에 대한 요청자의 팀장 승인, 요청에 대한 작업 진행전 구성/변경이 발생하는 곳 (인프라운영팀) 작업자의 팀장 승인
        - 분류: 할 일
        - 이름: `Infra_Test_PM_Prd_작업공지`
        - 설명: 해당 작업으로 영향을 받는 팀 또는 서비스를 담당하고 있는 담당자 또는 상위 직책자들에게 작업 공지. 영향도 분석서, 인지하고 있는 시스템간 테크니컬 아키텍처
        - 분류: 할 일
        - 이름: Infra_Test_PM_Prd_작업자할당
        - 설명: 해당 작업을 수행할 작업자 할당. CMS 담당자, 고객 아키텍트 또는 운영자
        - 분류: 진행중
        - 이름: Infra_Test_PM_Prd_작업수행
        - 설명: 작업 할당자가 요청사항에 대한 인프라 구성/변경 작업 수행
        - 분류: 진행중
        - 이름: Infra_Test_PM_Prd_수행내역점검
        - 설명: 감사에 반드시 필요한 작업. 인프라 구성/변경 작업 후 요청자가 원한 상태로 구성/변경되었는지에 대한 결과 & 테스트 수행내역 검증
        - 분류: 진행중
        - 이름: Infra_Test_PM_Prd_작업종료
        - 분류: 완료
    - 워크플로우 연결/전환
        - 이 상태에서: `Infra_Test_PM_Prd_작업요청`
        - 상태로: `Infra_Test_PM_Prd_작업승인`
        - 이름: PM작업승인
        - 설명:
        - 화면: 없음
        - 이 상태에서: `Infra_Test_PM_Prd_작업승인`
        - 상태로: `Infra_Test_PM_Prd_작업요청`
        - 이름: 반려
        - 설명:
        - 화면: 없음
        - 이 상태에서: `Infra_Test_PM_Prd_작업승인`
        - 상태로: `Infra_Test_PM_Prd_작업공지`
        - 이름: PM작업공지
        - 설명:
        - 화면: `Infra_Test_PM_전파대상할당` (화면 만들어야 함)
            - 필드: `Infra_Test_PM_전파대상자`
            - 타입: 사용자 선택기 (복수 선택)
        - 이 상태에서: `Infra_Test_PM_Prd_작업공지`
        - 상태로: `Infra_Test_PM_Prd_작업자할당`
        - 이름: `PM_작업자할당`
        - 설명:
        - 화면: `Infra_Test_PM_작업자할당` (화면 만들어야 함)
            - 필드: 담당자
            - 타입: 시스템필드
        - 이 상태에서: `Infra_Test_PM_Prd_작업자할당`
        - 상태로: `Infra_Test_PM_Prd_작업수행`
        - 이름: `PM_작업진행`
        - 설명:
        - 화면: 
        - 이 상태에서: `Infra_Test_PM_Prd_작업수행`
        - 상태로: `Infra_Test_PM_Prd_수행내역점검`
        - 이름: `PM_작업완료검증`
        - 설명:
        - 화면: `Infra_Test_PM_작업완료검증` (화면 만들어야 함)
            - 필드: End date
            - 타입: 날짜 선택기
            - 필드: `Infra_Test_PM작업완료 증빙`
            - 타입: URL 항목란
        - 이 상태에서: `Infra_Test_PM_Prd_수행내역점검`
        - 상태로: `Infra_Test_PM_Prd_작업수행`
        - 이름: `Infra_Test_PM_작업오류재수정`
        - 설명:
        - 화면:
        - 이 상태에서: `Infra_Test_PM_Prd_수행내역점검`
        - 상태로: `Infra_Test_PM_Prd_작업종료`
        - 이름: `PM_작업완료`
        - 설명:
        - 화면: 



- Jira 관리 > 이슈 > 업무 흐름 계획 > Test_ServiceDesk 편집 > 업무흐름 추가 > 기존 추가 > `Infra_Test_PM_Prd_WorkFlow` 선택 > `Infra_PM_Prd_Issue` 선택
- 이슈 유형 > 이슈 유형 수정 > 



- 프로젝트 > Test Service Request (TESTSR) > 프로젝트 설정 > 업무흐름 > `Infra_Test_PM_Prd_WorkFlow` 편집 > `INFRA_Test_PM_PRD_작업승인` 클릭 > 승인추가: 설정 클릭
    - 승인자를찾을곳: `Infra_Test_PM_승인자`
    - 다음 기간의 경과 후 승인 완료로 간주 : 모두 승인 (모두 승인해야 다음 단계 진입)  # 1 승인 (승인자들중 1명이라도 승인하면 다음 단계 진입)
    - 승인되었을때 이행: PM작업공지
    - 거절되었을때 이행: 반려
- 프로젝트 > Test Service Request (TESTSR) > 프로젝트 설정 > 이슈유형 > Infra_PM_Prd_Issue 선택 > 필드 > 아래 두개 추가
    - Infra_Test_PM_승인자
    - Infra_Test_PM_작업계획서
- 프로젝트 > Test Service Request (TESTSR) > 프로젝트 설정 > 요청유형 > 그룹추가 >
    - 그룹이름: 예방정기점검(Prevention Maintenance)
    - 요청이름 : PM작업요청 (최종사용자가 서비스포털 화면에 볼 이름)
    - 이슈유형 : `Infra_PM_Prd_Issue`
    - 설명: PM 작업 요청시 해당 Task를 이용해주세요.
    - 필드편집 클릭 > 필드 추가 클릭 > 아래 필드들 선택해서 추가
        - 기한 : 필수
            - 필드도움말: 해당 작업 완료 기한을 넣어주세요.
        - 요약 : 필수
            - 필드도움말: 해당 작업의 제목(요약)을 해주세요. (예, LB SSL 갱신 작업)
        - 설명 : 필수
            - 필드도움말: 해당 작업의 상세 내용을 적어주세요. 하단 "Infra_Test_PM_작업계획서" 에 URL 꼭 입력 바랍니다.
        - Infra_Test_PM_승인자 : 필수, 숨기기
            - Infra_Test_PM_승인자에 들어갈 사람들 계정을 넣는다. hsk4004
        - Infra_Test_PM_작업계획서 : 필수
            - 필드도움말: 작업계획서 URL 기입해주세요. (컨플루언스에 있는 PM 계획 템플릿, 변경 작업서, 영향도 분석서, 테크니컬 아키텍처, 비상조직연락망 링크 등록)
        - 첨부파일 : 필수아님
            - 해당 작업에 대한 별도 문서가 있다면 첨부바랍니다. 
- 워크플로우 다이어그램 만듬
    - PM요청
    - PM계획수립 (컨플루언스에 있는 PM 계획 템플릿, 변경 작업서, 영향도 분석서, 테크니컬 아키텍처, 비상조직연락망 링크 등록)
    - 검토/승인
    - 요청자 팀장
    - 적용할 인프라 관리 팀장
    - 상위 승인권자
    - 사전공지
    - 긴급 유무 > 긴급이면 종합 상황반 구성 > PM 전파 > PM 전파
    - PM 수행
    - 인프라/어플리케이션 자원 변경 필요에 따라 인프라/어플리케이션 변경관리 프로세스 적용
    - 변경 완료 통보 > 아키텍트, 서비스 PL
    - 변경사항 테스트 > 이상이 있으면 다시 인프라/어플리케이션 자원 변경
    - 종료 전파
        - 아키텍트, 서비스 PL, 승인자

    - PM완료



## Create 일반문의

- 이슈 > 이슈 유형 > 이슈 유형 추가

    - 이름: `Infra_일반문의_Issue`
    - 유형: 표준 이슈 유형

- 이슈 > 이슈 유형 계획 > 이슈 유형 스키마 추가

    - 구성표 이름: `Infra_일반문의_Issue_Type`
    - 사용 가능한 이슈 유형에서 `Infra_일반문의_Issue` 찾아서 현재 사용중인 이슈 유형에 끌어다 놓음
    - 기본 이슈 유형: `Infra_일반문의_Issue 선택`

- 일반문의 절차

    - 일반문의 요청 > 아키텍트팀 인지 > 아키텍트팀 1명이 회신

- 이슈 > 화면 > 화면추가

    - 이름: `인프라_일반문의_답변완료_화면`
    - 필드: 해결책
    - 타입: 시스템 필드

- 이슈 > 업무흐름 > 업무흐름 추가

    - 이름: `Infra_General_Inquiry_Workflow`

- 상태추가 >

    - `인프라_일반문의_요청`
    - `인프라_일반문의_확인`
    - `인프라_일반문의_답변`
    - `인프라_일반문의_답변완료` (꼭 `인프라_일반문의_답변완료_화면` 필드로 해결책 필드를 넣어서 해당 작업이 "작업완료" 바꿔줘야 JSM 대기열-"모두열림" 에서 "최근 해결됨"으로 해당 이슈가 넘어간다)

    - 전환추가
        - 이 상태에서: `인프라_일반문의_답변`
        - 상태로: `인프라_일반문의_답변완료`
        - 이름: `답변완료`
        - 설명: `인프라일반문의답변완료`
        - 화면: `인프라_일반문의_답변완료_화면`



- 이슈 > 업무흐름계획 > Test_ServiceDesk 찾음 > 편집 > 기존 추가 >
    - `Infra_General_Inquiry_Workflow` 찾음/선택 > 다음
    - `Infra_일반문의_Issue` 찾음/선택 > 게시 > 연계
- 프로젝트 > Service > Test Service Request 선택 > 프로젝트 설정 > 이슈 유형 > 이슈 유형 수정
    - 사용 가능한 이슈 유형에서 "Infra_일반문의_Issue" 찾아서 현재 사용중인 이슈 유형에 끌어다 놓음
- 기존 2개에서 
    - [이슈 유형](http://jira.test.com/plugins/servlet/project-config/TESTSR/issuetypes)
    - [CTO최종승인_생성요청(계정,정책)](http://jira.test.com/plugins/servlet/project-config/TESTSR/issuetypes/13108)
            - [Infra_PM_Prd_Issue](http://jira.test.com/plugins/servlet/project-config/TESTSR/issuetypes/13111)
    - 아래처럼 `Infra_일반문의_Issue` 가 추가됨
    - [이슈 유형](http://jira.test.com/plugins/servlet/project-config/TESTSR/issuetypes)
    - [CTO최종승인_생성요청(계정,정책)](http://jira.test.com/plugins/servlet/project-config/TESTSR/issuetypes/13108)
        - [Infra_PM_Prd_Issue](http://jira.test.com/plugins/servlet/project-config/TESTSR/issuetypes/13111)
            - [Infra_일반문의_Issue](http://jira.test.com/plugins/servlet/project-config/TESTSR/issuetypes/13114)

- 일반문의 받을 담당자들 지정을 위한 사용자필드 생성 > 이슈 > 사용자정의필드 > 사용자정의필드추가 > 사용자 선택기 (복수 선택) > 다음 >
    - 이름: `Infra_일반문의_담당자` > 컨텍스트 구성 >
    - 이슈유형선택: `Infra_일반문의_Issue`
    - 컨텍스트선택: 선택한 프로젝트의 이슈에 적용
    - 프로젝트선택: Test Service Request
- 프로젝트 > Test Service Request (TESTSR) > 프로젝트 설정 > 이슈유형 > `Infra_일반문의_Issue` 선택 > 필드 > 아래 찾아서 추가
    - `Infra_일반문의_담당자`
- 프로젝트 > Service > Test Service Request 선택 > 프로젝트 설정 > 요청 유형 > 그룹추가
    - 그룹이름: 일반문의
    - 요청이름: DB일반문의
    - 이슈유형: `Infra_일반문의_Issue`
    - 설명: DB관련 일반적인 문의를 요청해주세요.
    - 필드 편집 > 필드추가 > 아래 필드 선택
        - 기한:
        - 설명:
        - 담당자:
    - 요청이름: GCP/GKE 일반 문의
    - 이슈유형: `Infra_일반문의_Issue`
    - 설명: GCP/GKE 관련 일반적인 문의를 요청해주세요.
    - 필드 편집 > 필드추가 > 아래 필드 선택
        - 기한:
        - 설명:
        - 담당자:
    - 사전설정값
        - 담당자: 



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
# /data/jira/bin/setenv.sh

JVM_MINIMUM_MEMORY="8192m"
JVM_MAXIMUM_MEMORY="8192m"

JAVA_OPTS="-Xms${JVM_MINIMUM_MEMORY} -Xmx${JVM_MAXIMUM_MEMORY} ${JVM_CODE_CACHE_ARGS} ${JAVA_OPTS} ${JVM_REQUIRED_ARGS} ${DISABLE_NOTIFICATIONS} ${JVM_SUPPORT_RECOMMENDED_ARGS} ${JVM_EXTRA_ARGS} ${JIRA_HOME_MINUSD} ${START_JIRA_JAVA_OPTS}"
```

- 이에 Machine type을 Custom(vCPU: 4, Memory: 24G)로 변경하고, MetaSpace 사이즈 옵션도 추가함.

```sh
# /data/jira/bin/setenv.sh

JVM_MINIMUM_MEMORY="8192m"
JVM_MAXIMUM_MEMORY="8192m"
JVM_METASPACE_MIN_MEM="2048m"
JVM_METASPACE_MAX_MEM="2048m"

JAVA_OPTS="-XX:MetaspaceSize=${JVM_METASPACE_MIN_MEM} -XX:MaxMetaspaceSize=${JVM_METASPACE_MAX_MEM} -Xms${JVM_MINIMUM_MEMORY} -Xmx${JVM_MAXIMUM_MEMORY} ${JVM_CODE_CACHE_ARGS} ${JAVA_OPTS} ${JVM_REQUIRED_ARGS} ${DISABLE_NOTIFICATIONS} ${JVM_SUPPORT_RECOMMENDED_ARGS} ${JVM_EXTRA_ARGS} ${JIRA_HOME_MINUSD} ${START_JIRA_JAVA_OPTS}"
```

- startup, shutdown은 아래와 같이 진행

```sh
sudo su -
/data/jira/bin/startup.sh
/data/jira/bin/shutdown.sh
```

- VM Stop > VM Resize > VM Start > Jira Start 진행 완료