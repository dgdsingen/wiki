# AEM 관련 API 혹은 코딩가이드 (공통)

- ex) SlingConstants, WCMMode, cq taglib 등에 대한 가이드
- [http://dev.day.com/docs/en/cq/current.html#Developing%20on%20Adobe%20Experience%20Manager](https://www.google.com/url?q=http://dev.day.com/docs/en/cq/current.html%23Developing%20on%20Adobe%20Experience%20Manager&sa=D&ust=1581424151183000)
- [http://dev.day.com/docs/en/cq/5-3/developing/sling-adapters.html](https://www.google.com/url?q=http://dev.day.com/docs/en/cq/5-3/developing/sling-adapters.html&sa=D&ust=1581424151183000)

# 신규 서블릿(service), 컴포넌트 생성시 관련한 추가 설정 가이드(공통)

- [http://sling.apache.org/documentation/the-sling-engine/servlets.html](https://www.google.com/url?q=http://sling.apache.org/documentation/the-sling-engine/servlets.html&sa=D&ust=1581424151184000)
- [http://felix.apache.org/documentation/subprojects/apache-felix-service-component-runtime.html](https://www.google.com/url?q=http://felix.apache.org/documentation/subprojects/apache-felix-service-component-runtime.html&sa=D&ust=1581424151184000)
- [http://felix.apache.org/documentation/subprojects/apache-felix-maven-scr-plugin/scr-annotations.html](https://www.google.com/url?q=http://felix.apache.org/documentation/subprojects/apache-felix-maven-scr-plugin/scr-annotations.html&sa=D&ust=1581424151185000)

# Clustering Publish

- 불가능하댄다. 더군다나 세션 공유 설정은.

# Activate & Replicate 시 반영이 안되는 문제

- Author -> Publish

- port가 막혀서 못나가거나 못받는 경우
- port는 열려있으나 pending 걸리는 경우
- 설정이 안되어 있는 경우

- Publish -> Dispatcher

- port가 막혀서 못나가거나 못받는 경우
- Dispatcher의 Cache가 update되지 않는 경우

# Replication Access Denied

- publisher의 queue에 pending된 작업이 많을 경우 access denied가 난다.
- tools - replication - by author에 가서 publisher의 상태를 보자.
- queue가 많이 찼다면, 비우면 진행된다.

# New Component가 Sidekick에 나오지 않을때

Each CQ5 template has a different set of components that can be defined to be available for use. This allows you to control what the authors will be allowed to use, and it makes it easier for them, because they will only see the relevant components, instead of the tons of components CQ5 offers.

When looking at a page, you can switch to something called the "design" mode (as opposed to the "edit" or "preview" modes where you spend most of your time authoring the page). This design mode allows to define the per-template specific settings. It is accessed through the yellow ruler icon on the very bottom of the sidekick.

When in design mode, click on the "Edit" button that is on the blue toolbar called "Design of par", there you'll be able to enable the components you want to be able to use.

When you'll be building components, keep in mind that the design mode and the corresponding design dialogs of the components is a convenient way to define global per-template settings that you don't want to be required to be set specifically on each component instance.

# system bundle을 /app/site/install로 내릴 때

- web console에서 해당 bundle을 uninstall하고 webDAV로 /app/site/install에 복사해 넣는다.
- 기존에 importing bundle 링크가 걸려있는 다른 bundle에 가서 refresh package를 해준다.
- 새로 교체한 번들의 정보에서 importing bundles 목록이 복원되었다면 성공이다.

# WebDAV 사용

> http://dev.day.com/docs/en/crx/current/how_to/webdav_access.html

1. cq-quickstart 를 run
2. 내 컴퓨터 - 네트워크 위치 추가
3. 해당 번들 폴더에 작업하신 jar를 업로드하면 됩니다



윈도우 7의 경우 레지스트리 수정 필요:

기본적으로 윈도우7은 SSL로만 webDav에 접속가능하다고 한다. 이를 바꿔서 일반 연결로도 접속가능하도록 해줘야 한다.

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters 를 찾은 다음

BasicAuthLevel 값이 기본 1로 되어있는데 2로 변경 (키 이름이 없다면 "DWORD Value"키를 새로 생성)

>  참고: http://719121812.blog.me/20176669284

그리고 파일전송의 최대 크기가 제한되어 있는데(Max 50메가), 이 제한을 풀어준다. 그렇지 않으면 50메가 이하의 파일들만 전송이 가능하다.

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters 를 찾은 다음

FileSizeLimitInBytes 키를 더블클릭하고 10진수를 선택한다. 여기서 최대값(4294967295)를 입력한다.

다 되었으면 컴퓨터를 재부팅한다.

>  출처: http://takuma99.tistory.com/222



# Customize Logging

- Log into Felix Console: http://host:port/system/console/configMgr
- From "Factory Configurations", create "Apache Sling Logging Writer Configuration"
- Set value of "Log File" to "logs/dam.log"
- Click on "Save"
- From "Factory Configurations", create "Apache Sling Logging Logger Configuration"
- Set value of "Log Level" to "Debug"
- Set value of "Log File" to "logs/dam.log"
- Add "Logger" => com.day.cq.dam.core
- Click on "Save"
- 만약 이전에 설정해둔 logger가 있다면 삭제 후 다시 설정해주면 적용된다. logger값을 제어함으로써 애플리케이션별 혹은 기능별 로깅이 가능하다.
- JSP에서 로깅을 사용하고 싶다면, 아래와 같이 사용하자.

```jsp
<%@page import="org.apache.log4j.*"%>
<%! static Logger logger = Logger.getLogger("com.test"); %>
<% logger.debug("write"); %>
```



# Dependency Injection

- DI로 주입되는 객체를 사용하려는 경우 사용되는 객체와 사용하는 객체 모두 컨테이너의 관리 하에 있어야 한다.
- 예를 들어 아래 코드로 MyService가 SqlSessionManager를 사용할 때
- MyService를 가져다 쓰는 곳에서 DI 방식이 아닌 MyService myService = new MyService()와 같이 임의로 객체를 생성하여 사용한다면
- 임의 생성된 myService 객체 안의 sqlSessionManager는 바인딩되지 않아 null로 떨어진다.

```java
@Component(immediate = true, metatype = false)
@Service(MyService.class)
public class MyService {
  @Reference(bind="bindSqlSessionManager",unbind="unbindSqlSessionManager")
  private SqlSessionManager sqlSessionManager;

  public void bindSqlSessionManager(SqlSessionManager sqlSessionManager){
    this.sqlSessionManager = sqlSessionManager;
  }

  public void unbindSqlSessionManager(SqlSessionManager sqlSessionManager){
    if(this.sqlSessionManager != sqlSessionManager) return;
    this.sqlSessionManager = null;
  }
}

@Component(metatype = true, immediate = true, label = "SqlSessionManager")
@Service
public class SqlSessionManager {
}
```



# DataSourcePool 사용 (with MyBatis)

```java
import java.io.IOException;
import java.io.Reader;
import java.sql.Connection;
 
import javax.sql.DataSource;
 
import org.apache.felix.scr.annotations.Activate;
import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Deactivate;
import org.apache.felix.scr.annotations.Modified;
import org.apache.felix.scr.annotations.Property;
import org.apache.felix.scr.annotations.Reference;
import org.apache.felix.scr.annotations.Service;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import org.osgi.service.component.ComponentContext;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
 
import com.day.commons.datasource.poolservice.DataSourcePool;
 
@Component(metatype = true, immediate = true, label = "SqlSessionManager")
@Service
public class SqlSessionManagerImpl implements SqlSessionManager{
  private Logger logger = LoggerFactory.getLogger(getClass());
  private SqlSessionFactory sqlSessionFactory = null;
 
  @Reference(bind="bindDataSourcePool", unbind="unbindDataSourcePool")
  private DataSourcePool dataSourcePool;
 
  public void bindDataSourcePool(DataSourcePool paramDataSourcePool) {
    if (logger.isDebugEnabled())
      logger.debug("datasourcePool is bounded");
    this.dataSourcePool = paramDataSourcePool;
    init();
  }
 
  public void unbindDataSourcePool(DataSourcePool paramDataSourcePool) {
    if (logger.isDebugEnabled())
      logger.debug("datasourcePool is unbounded");
    if (this.dataSourcePool != paramDataSourcePool)
      return;
    this.dataSourcePool = null;
    this.sqlSessionFactory = null;
  }   
 
  @Property(value = "_ojdbc/was", label = "DataSource Name", description = "DataSource Name")
  private static final String P_DATASOURCENAME = "com.my.common.SqlSessionManager.DataSourceName";
  private String DATASOURCENAME = "_ojdbc/was";
 
  @Activate
  public void activated(ComponentContext ctx) {
    modifyProperties(ctx);
  }
 
  @Deactivate
  public void deactivated(ComponentContext ctx) {
    this.dataSourcePool = null;
    this.sqlSessionFactory = null;
    if (logger.isDebugEnabled())
      logger.debug("component is deactivated");
  }
 
  @Modified
  public void modified(ComponentContext ctx) {
    modifyProperties(ctx);
  }
 
  private void modifyProperties(ComponentContext ctx ){
    DATASOURCENAME = (String) ctx.getProperties().get(P_DATASOURCENAME);
    init();
  }
 
  public void init(){
    try {
      Reader reader = null;
      try {
        reader = Resources.getResourceAsReader("com/my/common/mapper/mybatis-config.xml");
      } catch (IOException e) {
        throw new RuntimeException(e.getMessage());
      }
      sqlSessionFactory  = new SqlSessionFactoryBuilder().build(reader);
 
      if (logger.isDebugEnabled())
        logger.debug("sqlSessionFactory is initalized");
    } catch (Exception e) {
      logger.error("##ERROR## SqlSessionManagerImpl.init()", e);
    }     
  }
 
  @Override
  public SqlSession openSession() {
    Connection conn = null;
    try {
      DataSource ds = (DataSource) dataSourcePool.getDataSource(DATASOURCENAME);
      conn = ds.getConnection();
    } catch (Exception e) {
      logger.error("cannot get connection", e);
    }
    if (null == conn) {
      logger.warn("cannot acquire connection, so delegate to ds");
      return sqlSessionFactory.openSession();
    }
    return sqlSessionFactory.openSession(conn);
  }
}
 
@Component(immediate = true, metatype = false)
@Service(MyService.class)
public class MyService {
  @Reference(bind="bindSqlSessionManager",unbind="unbindSqlSessionManager")
  private SqlSessionManager sqlSessionManager;
 
  public void bindSqlSessionManager(SqlSessionManager sqlSessionManager){
    this.sqlSessionManager = sqlSessionManager;
  }
 
  public void unbindSqlSessionManager(SqlSessionManager sqlSessionManager){
    if (this.sqlSessionManager != sqlSessionManager) return;
    this.sqlSessionManager = null;
  }
}
```



# Dispatcher Health Check

```nginx
/farms
{
 /daycom
 {
   ...
 }
 /docdaycom
 {
   ...
 }
/health_check
 {
 /url
"/health_check.html" #로직이 없는 html로 선정 .
}
/retryDelay
"1"
/numberOfRetries
"5"
/unavailablePenalty
"1"
/failover
"1"
}
```

> https://docs.adobe.com/docs/en/dispatcher/disp-config.html#par_title_7