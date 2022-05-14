# JVM

> [GraalVM](https://www.graalvm.org/) : High Performanсe. Cloud Native. Polyglot.
>
> [JVM 메모리 구조와 JAVA OPTION 값 정리](http://javaslave.tistory.com/23) 
>
> [JAVA VM Heap Size and Garbage Collection Tuning](https://17billion.github.io/java/2017/06/25/java_heap_size__gc_tuning.html) 
>
> [In Java is Permanent Generation space garbage collected? - Stack Overflow](https://stackoverflow.com/questions/3796427/in-java-is-permanent-generation-space-garbage-collected) 
>
> [Java Garbage Collection](https://d2.naver.com/helloworld/1329) 
>
> [Garbage Collection 모니터링 방법](https://d2.naver.com/helloworld/6043) 
>
> [Garbage Collection 튜닝](https://d2.naver.com/helloworld/37111) 
>
> [Java Reference와 GC](https://d2.naver.com/helloworld/329631) 
>
> [LINE의 OpenJDK 적용기: 호환성 확인부터 주의 사항까지 - LINE ENGINEERING](https://engineering.linecorp.com/ko/blog/line-open-jdk/) 

## Memory Size

보통 Heap 사이즈만 설정하는데, 총 메모리 사용량은 OS + Heap + MetaSpace 이므로 OS 메모리를 고려한 뒤 Heap과 MetaSpace 영역을 모두 명시해주는 것이 좋다. 

Heap:MetaSpace 비율은 4:1 정도로 설정하지만 이는 애플리케이션 특성에 따라 다르니 최적화해 나가도록 한다.

```sh
JAVA_OPTS="-Xms8192m -Xmx8192m -XX:MetaspaceSize=2048m -XX:MaxMetaspaceSize=2048m"
```



# Spring

## Spring Cloud Gateway

> [Spring Cloud Gateway](https://cloud.spring.io/spring-cloud-gateway/) 
>
> [Spring Cloud Gateway - Configuration Properties](https://spring.getdocs.org/en-US/spring-cloud-docs/spring-cloud-gateway/configuration-properties/configuration-properties.html) 

아래와 같이 설정하면 order 때문에 `/test/**` 를 호출하더라도 `/**` 이 먼저 적용되어 버린다. default routes는 맨 밑에 두자.

```yaml
# application.yml

- id: all
  uri: http://all-service.default.svc.cluster.local
  predicates:
    - Path=/**
- id: test
  uri: http://test-service.default.svc.cluster.local
  predicates:
    - Path=/test/**
```

SCG에서 target 호출시 forwarded, x-forwarded 헤더 보내지 않도록 설정

```yaml
spring:
  cloud:
    gateway:
      forwarded:
        enabled: false
      x-forwarded:
        enabled: false
```

troubleshooting을 위한 로깅 설정

> https://cloud.spring.io/spring-cloud-gateway/2.1.x/multi/multi_troubleshooting.html

```yaml
spring:
  cloud:
    gateway:
      httpclient:
        wiretap: true
logging:
  level:
    root: TRACE
    org.springframework.cloud.gateway: TRACE
    org.springframework.http.server.reactive: TRACE
    org.springframework.web.reactive: TRACE
    org.springframework.boot.autoconfigure.web: TRACE
    reactor.netty: TRACE
```



## Actuator

> https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html

Spring Cloud Gateway의 routes 정보를 actuator로 보고 싶다면 아래와 같이 설정한다.

```yaml
# application.yml

management:
  endpoints:
    web:
      exposure:
        include: 
          - "health"
          - "gateway"
  endpoint:
    gateway:
      enabled: true
```

이후 `/actuator/gateway/routes`로 접속하면 된다.



# Maven

## Copy Resources


```xml
<build>
    <resources>
        <!-- 여기에 넣은 resource는 jar의 최상위에 복사된다. 즉 jar/resources로 복사됨 -->
        <resource>
            <directory>src/main/java/resources</directory>
        </resource>
    </resources>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <configuration>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
    <pluginManagement>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-resources-plugin</artifactId>
                <version>2.5</version>
                <executions>
                    <execution>
                        <id>copy-resources</id>
                        <phase>validate</phase>
                        <goals>
                            <goal>copy-resources</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>${basedir}/target/classes/com/test/mybatis/mapper</outputDirectory>
                            <resources>
                                <resource>
                                    <directory>src/main/java/com/test/mybatis/mapper</directory>
                                    <includes>
                                        <include>**/*.xml</include>
                                    </includes>
                                </resource>
                            </resources>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </pluginManagement>
</build>
```



## 로컬 라이브러리 추가

pom.xml에 dependency를 걸어준 로컬 라이브러리의 경우,

.m2 하위에 해당 groupId와 version으로 디렉토리가 생기면 그 밑에 그냥 jar를 넣어주면 된다.

그리고 나서 eclipse - preference - maven - ... - update user setting 및 reindex 하자

다만 jar 이름이 maven 표준에 맞는지 확인하자. 특히 artifactId와 version 사이에 언더바(_)가 아니라 마이너스(-)다


## 의존적 jar를 찾지 못하는 경우

pom.xml에 repository를 추가해준다.


```xml
...
</build>
 
<repositories>
    <repository>
        <id>mesir-repo</id>
        <url>http://mesir.googlecode.com/svn/trunk/mavenrepo</url>
    </repository>
    <repository>
        <id>ops4j-releases</id>
        <name>Ops4j Releases</name>
        <url>https://oss.sonatype.org/content/repositories/ops4j-releases</url>
    </repository>
    <repository>
        <id>adobe</id>
        <name>Adobe Public Repository</name>
        <url>http://repo.adobe.com/nexus/content/groups/public/</url>
        <layout>default</layout>
    </repository>
</repositories>
 
<dependencies>
...
```



## Repository를 추가했음에도 jar가 추가되지 않으면

web browser로 직접 repository 경로로 들어가 jar, jar.sha1, pom, pom.sha1 파일을 모두 로컬 .m2 repository 경로로 내려받아 수동 추가.


## 모든 jar가 추가되었음에도 일부 jar를 못찾으면

jar가 있는데도 못찾으면 .m2 repository 해당 경로를 지우고 다시 받는다.

jar가 없으면 pom.xml dependency를 추가해주자.


## Compile 중 클래스 syntax error

project encoding과 workspace encoding이 맞는지 확인해보자. encoding이 맞지 않을 경우 한글 주석 등에서 syntax error를 뿜을 수 있다.


## Compile 중 클래스 타입 T error

jdk Compile 세팅을 1.6으로 맞춰주자


## Javadoc Plugin


```xml
<build>
    <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-javadoc-plugin</artifactId>
        <executions>
            <execution>
                <phase>package</phase>
                <goals>
                    <goal>javadoc</goal>
                </goals>
            </execution>
        </executions>
        <configuration>
            <!-- private 속성도 javadoc에 포함할 경우 -->
            <show>private</show>
            <nohelp>true</nohelp>
            <!-- Skip warnings -->
            <additionalparam>-Xdoclint:none</additionalparam>
            <additionalOptions>-Xdoclint:none</additionalOptions>
            <additionalJOption>-Xdoclint:none</additionalJOption>
        </configuration>
    </plugin>
</build>
```



## jar-with-dependencies


```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-assembly-plugin</artifactId>
            <executions>
                <execution>
                    <phase>package</phase>
                    <goals>
                        <goal>single</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <descriptorRefs>
                    <descriptorRef>jar-with-dependencies</descriptorRef>
                </descriptorRefs>
            </configuration>
        </plugin>
    </plugins>
</build>
```



## Copy dependencies to directory


```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <version>3.1.1</version>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>${project.build.directory}/dependencies</outputDirectory>
                        <overWriteReleases>false</overWriteReleases>
                        <overWriteSnapshots>false</overWriteSnapshots>
                        <overWriteIfNewer>true</overWriteIfNewer>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



## mvn test 시 Parameter 사용하기


```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-surefire-plugin</artifactId>
            <configuration>
                <systemProperties>
                    <property>
                        <name>filepath</name>
                        <value>${filepath}</value>
                    </property>
                </systemProperties>
            </configuration>
        </plugin>
    </plugins>
</build>
```

```bash
mvn test -Dfilepath=src/main/resources/test.txt
```

```java
String filepath = System.getProperty("filepath");
```



# Maven + AspectJ + Lombok

Maven을 사용하여 빌드하는 과정에서 Compile 시점에 AspectJ와 Lombok이 충돌하는 경우가 있다. 이에 대한 원인 분석과 해결책을 기술한다.



- Weaving: AOP로 정의한 Cross-Cutting 로직을 타겟에 적용하는 과정을 의미한다. Weaving 적용 방식은 아래와 같이 분류된다.
    - CTW (Compile Time Weaving): AJC(AspectJ Compiler)를 사용하여 Compile 시점에 타겟의 바이트 코드를 조작해 Weaving을 수행한다. Lombok과 같이 Compile 시점에 바이트 코드 조작을 가하는 다른 라이브러리와 충돌하게 된다.
    - PCW (Post Compile Weaving): Compile 직후 생성된 class 혹은 jar 파일에 Weaving을 수행한다.
    - LTW (Load Time Weaving): ClassLoader가 Class를 JVM으로 로딩하는 시점에 Weaving을 수행한다. 
    - RTW (Runtime Weaving): Spring AOP에서 사용하는 방식. 타겟을 직접 변형하지 않고 Proxy를 사용하여 Weaving을 수행한다.
- Weaving 방식 성능 비교
    - Compile 성능: `LTW = RTW > CTW > PCW` 
    - Runtime 성능: `CTW = PCW > LTW > RTW` 



Runtime 성능을 위해 CTW를 사용한다면 Compile 시점에 AspectJ와 Lombok이 서로 충돌하게 된다.

Runtime 성능을 그대로 유지하면서 AspectJ와 Lombok의 충돌을 피하고 싶다면 PCW를 사용하면 된다.

- AS-IS: java source를 AJC로 Compile 하여 AspectJ CTW 적용
- TO-BE: java source를 javac로 Compile 하여 Lombok 적용하고, 그 class 파일을 AJC로 다시 Compile하여 AspectJ PCW 적용

> https://stackoverflow.com/questions/41910007/lombok-and-aspectj
>
> https://www.mojohaus.org/aspectj-maven-plugin/examples/weaveDirectories.html

이에 대한 `pom.xml` 설정 사항은 아래와 같다. (주석으로 표기)

```xml
<project>
    <dependencies>
        <!-- add dependency for ajc -->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>1.9.5</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjtools</artifactId>
            <version>1.9.5</version>
        </dependency>

        <!-- add dependency for lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <scope>provided</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>aspectj-maven-plugin</artifactId>
                <version>1.11</version>
                <configuration>
                    <complianceLevel>1.8</complianceLevel>
                    <source>1.8</source>
                    <target>1.8</target>
                    <showWeaveInfo>true</showWeaveInfo>
                    <verbose>true</verbose>
                    <Xlint>ignore</Xlint>
                    <encoding>UTF-8</encoding>

                    <!-- compile java sources with javac for lombok, and compile classes with ajc for aspectj pcw -->
                    <excludes>
                        <exclude>**/*.java</exclude>
                    </excludes>
                    <forceAjcCompile>true</forceAjcCompile>
                    <sources/>

                    <aspectLibraries>
                        <aspectLibrary>
                            <groupId>org.springframework</groupId>
                            <artifactId>spring-aspects</artifactId>
                        </aspectLibrary>
                    </aspectLibraries>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <!-- use this goal to weave all your main classes -->
                            <goal>compile</goal>
                            <!-- use this goal to weave all your test classes -->
                            <goal>test-compile</goal>
                        </goals>

                        <!-- specifying a path that compiled java sources with javac for lombok -->
                        <configuration>
                            <weaveDirectories>
                                <weaveDirectory>${project.build.directory}/classes</weaveDirectory>
                            </weaveDirectories>
                        </configuration>

                    </execution>
                </executions>
            </plugin>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>pl.project13.maven</groupId>
                <artifactId>git-commit-id-plugin</artifactId>
                <version>4.0.3</version>
            </plugin>
        </plugins>
    </build>
</project>
```



# Ant

## Only using xml

build.xml

```xml
<?xml version = "1.0" encoding = "UTF-8"?>
<project name = 'bin' default = 'execute' basedir = '.'>
    <target name="clean">
        <delete dir="bin"/>
        <delete file="${ant.project.name}.jar"/>
    </target>
 
    <target name='compile'>
        <mkdir dir = "bin"/>
        <manifest file = "bin/MANIFEST.MF">
            <attribute name = "Built-By" value = "dgdsingen"/>
            <attribute name = "Main-Class" value = "com.test.Main"/>
        </manifest>
        <javac srcdir    = 'src'
               destdir   = 'bin'
               includes  = '**/*.java'
               excludes  = '**/*.class'>
            <classpath>
                <fileset dir="bin"/>
                <fileset dir="lib">
                    <include name="**/*.jar"/>
                </fileset>
            </classpath>
        </javac>
    </target>
 
    <target name = "jar" depends = "compile">
        <jar destfile = "${ant.project.name}.jar"
             basedir  = "bin"
             includes = "**/*.class"
             excludes = "**/*.java"
             compress = "true"
             index    = "true"
             manifest = "bin/MANIFEST.MF">
            <fileset dir = "bin"/>
        </jar>
    </target>
 
    <target name = "run" depends="compile">
        <java dir         = "bin"
              classname   = "com.test.Main"
              fork        = "true"
              failonerror = "true"
              maxmemory   = "128m">
            <classpath>
                <fileset dir="bin"/>
                <fileset dir="lib">
                    <include name="**/*.jar"/>
                </fileset>
            </classpath>
        </java>
    </target>
</project>
```

## Using properties

build.properties

```properties
project.name=${ant.project.name}
project.version=1.0
main.class=com.test.Main
apps.name=${ant.project.name}
jar.name=${apps.name}.jar
user.name=dgdsingen
src.dir=./src
bin.dir=./bin
lib.dir=./lib
classpath=./bin
backup.dir=bak
```

build.xml

```xml
<?xml version = "1.0" encoding = "UTF-8"?>
<project name = 'bin' default = 'execute' basedir = '.'>
    <property file = "./build.properties"/>
 
    <target name="clean">
        <delete dir="${bin.dir}"/>
    </target>
 
    <target name='compile'>
        <mkdir dir = "${bin.dir}"/>
        <javac srcdir    = '${src.dir}'
               destdir   = '${bin.dir}'
               includes  = '**/*.java'
               excludes  = '**/*.class'>
            <classpath>
                <fileset dir="bin"/>
                <fileset dir="lib">
                    <include name="**/*.jar"/>
                </fileset>
            </classpath>
        </javac>
    </target>
 
    <target name="create_manifest" depends="compile">
        <manifest file = "${src.dir}/MANIFEST.MF">
            <attribute name = "Built-By" value = "${user.name}"/>
            <attribute name = "Main-Class" value = "${main.class}"/>
        </manifest>
    </target>
 
    <target name = "jar" depends = "create_manifest">
        <jar destfile = "${basedir}/${jar.name}"
             basedir  = "${bin.dir}"
             includes = "**/*.class"
             excludes = "**/*.java"
             compress = "true"
             index    = "true"
             manifest = "${src.dir}/MANIFEST.MF">
            <fileset dir = "${bin.dir}"/>
        </jar>
    </target>
 
    <target name = "execute" depends = "jar">
        <java jar        = "${basedir}/${jar.name}"
              fork        = "true"
              failonerror = "true"
              maxmemory   = "128m"/>
    </target>
</project>

```



# NumberFormat

```java
import java.text.NumberFormat; 
double test = 123123;
String formatted = NumberFormat.getInstance().format(test);
```

# SHA-512

```java
String newPasswd = "";

try {
    MessageDigest md = MessageDigest.getInstance("SHA-512");
    md.update(bean.getPasswd().getBytes());
    
    byte[] mb = md.digest();
    for (int i = 0; i < mb.length; i++) {
        byte temp = mb[i];
        String s = Integer.toHexString(new Byte(temp));
        while (s.length() < 2) {
            s = "0" + s;
        }
        s = s.substring(s.length() - 2);
        newPasswd += s;
    }
} catch (NoSuchAlgorithmException e) {
    e.printStackTrace();
}
```



# MyBatis

## insert의 return 값

```java
public Integer insert(Bean bean) {
	return (Integer) getSqlMapClientTemplate().insert("namespace.insert", bean);
}
```

```xml
<insert id="id" parameterClass="class">
    <selectKey resultClass="int" keyProperty="seq">
        select #seq# as seq from dual
    </selectKey>
</insert>
```

위 코드에서 dao.insert()가 int 값을 return 하려면 반드시 insert에 selectKey를 넣어주어야 한다. 그렇지 않으면, return 값은 null이 된다. 중요한 것은, return되는 int값은 insert된 개수가 아니라 selectKey의 결과값이다.

## #var#과 $var$의 차이

#을 쓰면 preparedStatement처럼 쿼리를 Compile 해놓고 변수 부분만 바꿔서 쓴다. 즉 쿼리 풀을 조금 차지하면서 파싱과정을 지나치기 때문에 작고 빠르다.

$를 쓸 경우 매우 동적인 쿼리(서브쿼리 자체를 아예 바꾼다던가 하는)를 만들 수 있다는 장점은 있으나 쿼리를 날릴 때마다 파싱하고 쿼리 풀을 채우기 때문에 메모리, 속도면에서 #에 밀린다고 보면 된다. 또 $를 쓰면 SQL Injection과 같은 보안 문제도 일으킬 수 있다.

```
#var#
'$var$%'

'#var#%' 이건 불가능하다. ORA-01006 (바인딩 변수가 없음) 에러가 난다.
```

## xml tag 문자 텍스트로 삽입하기

```xml
<![CDATA[>=]]>
```

# NIO

```java
// Channel을 이용한 File 처리 by NIO

import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.io.FileOutputStream;
import java.io.IOException;
import java.nio.ByteBuffer;
import java.nio.channels.FileChannel;

public class Test {
	public static void main(String[] args) {
	
		try {
			FileInputStream fin = new FileInputStream("input.txt");
			FileOutputStream fout = new FileOutputStream("output.txt");
			ByteBuffer bf = ByteBuffer.allocateDirect(4096);
			
			FileChannel cin = fin.getChannel();
			FileChannel cout = fout.getChannel();

//			간단한 방법
//			cin.transferTo(0, cin.size(), cout);

//			좀더 복잡하지만, customizable
			for ( int pos = 0; (pos = cin.read(bf)) >= 0 ; ) {
				bf.flip();
				cout.write(bf);
				bf.clear();
			}

		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}		
		
	}

}
// 객체 다루기 by NIO

####The sending end 

Message outgoingMessage; 
SocketChannel socketChannel; 
//we open the channel and connect 
ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream(); 
ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream); 
objectOutputStream.writeObject(outgoingMessage); 
objectOutputStream.flush(); 
socketChannel.write(ByteBuffer.wrap(byteArrayOutputStream.toByteArray())); 

####The receiving end 

SocketChannel socketChannel; 
ByteBuffer data; 
//we open the channel and connect 
socketChannel.read(data); 
ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(data.array()); 
ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream); 
Message message = (Message)objectInputStream.readObject(); 

// 위에서 receiving 쪽 error가 날 경우 ByteBuffer를 allocateDirect() 대신 allocate()로 할당한다.
```



# Reflection

## Object의 Field값 출력

```java
import java.lang.reflect.Field;
import java.util.function.Function;

public interface Stringifiable {
    // Example
    // System.out.println(Stringifiable.stringify.apply(new Object()));
    Function<Object, Object> stringify = (Object o) -> {
        StringBuffer sb = new StringBuffer();
        sb.append("\n\n").append(o).append("={").append("\n");
        for (Field f : o.getClass().getDeclaredFields()) {
            f.setAccessible(true);

            try {
                sb.append("\t").append(f.getName()).append(" : ").append(f.get(o)).append(",\n");
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        sb.append("}\n");

        return sb.toString();
    };
}
```

## Object의 method 수행

```java
Method[] methods = null;			
methods = OrderBean.class.getDeclaredMethods();
for (Method method : methods) {
	if (method.getName().startsWith("get")) 
		System.out.println(method.getName() + " : " + method.invoke(orderBean));
}
```

## Reflection을 이용한 shallow copy

```java
public Object copyObject(Object o) {
    Object result = null;
    
    try {
        Class cls1 	= o.getClass();
        result 		= cls1.newInstance();
        Class cls2 	= result.getClass();

        Method[] methods1 = cls1.getDeclaredMethods();
        Method[] methods2 = cls2.getDeclaredMethods();
        
        // [method1 -> method2]로 복사
        for (Method method1 : methods1) {
            for (Method method2 : methods2) {
                if (method1.getName().startsWith("get")) {
                    if (method1.getName().substring(method1.getName().indexOf("get")+3)
                            .equals(method2.getName().substring(method2.getName().indexOf("set")+3)))
                    method2.invoke(result, method1.invoke(o));
                }
            }
        }
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    } catch (IllegalArgumentException e) {
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        e.printStackTrace();
    }
    
    return result;		
}
```

# Singleton

> 참조: [Multi Thread 환경에서의 올바른 Singleton](https://medium.com/@joongwon/multi-thread-환경에서의-올바른-singleton-578d9511fd42) 

일반적으로 하나의 인스턴스만 존재해야 할 경우 Singleton 패턴을 사용하게 된다. 물론 Single Thread에서 사용되는 경우에는 문제가 되지 않지만 Multi Thread 환경에서 Singleton 객체에 접근 시 초기화 관련하여 문제가 있다. 보통 Singleton 객체를 얻는 static 메서드는 getInstance()로 작명하는 게 일반적이다. 그렇다면 어떻게 코드를 작성해야 Singleton 객체를 생성하는 로직을 thread-safe 하게 적용할 수 있을까? 정말 단순하게 별로 신경 쓰고 싶지 않다면 메서드에 synchronized 키워드만 추가해도 무방할 것이다. 그렇지만 좋은 개발자가 되기 위해선 효율적인 코드에 대해서 고민을 해 봐야 한다. 메서드에 Singleton 클래스의 getInstance() 메서드에 synchronized 키워드를 추가하는 건 역할에 비해서 동기화 오버헤드가 심하다고 생각한다. 그래서 개발자들 사이에서 Singleton 초기화 관련하여 많은 이디엄들이 연구되었고 몇몇 이디엄들을 소개하려고 한다.

## Double Checked Locking

일명 DCL이라고 불리는 이 기법은 현재 broken 이디엄이고 사용을 권고하지 않지만 이러한 기법이 있었다는 걸 설명하고 싶다. 코드는 다음과 같다.

```java
public class Singleton {
    private volatile static Singleton instance;
    private Singeton() {}

    public static Singleton getInstance() {
        if (instance == null)  {
            synchronized(Singleton.class) {
                if (instance == null) {
                	instance == new Singleton();  
                }
            }
        }

        return instance;
    }
}
```

메서드에 synchronized를 빼면서 동기화 오버헤드를 줄여보고자 하는 의도로 설계된 이디엄이다. instance가 null 인지 체크하고 null 일 경우 동기화 블럭에 진입하게 된다. 그래서 최초 객체가 생성된 이후로는 동기화 블럭에 진입하지 않기 때문에 효율적인 이디엄이라고 생각할 수 있다. 하지만 아주 안 좋은 케이스로 정상 동작하지 않을 수 있다. 그래서 broken 이디엄이라고 하는 것이다.Thread A와 Thread B가 있다고 하자. Thread A가 instance의 생성을 완료하기 전에 메모리 공간에 할당이 가능하기 때문에 Thread B가 할당된 것을 보고 instance를 사용하려고 하나 생성과정이 모두 끝난 상태가 아니기 때문에 오동작할 수 있다는 것이다. 물론 이러할 확률은 적겠지만 혹시 모를 문제를 생각하여 쓰지 않는 것이 좋다.

## Enum 사용

Java에 지대한 공헌을 한 Joshua Bloch가 언급한 이디엄이다. 그냥 간단하게 class가 아닌 enum으로 정의하는 것이다.

```java
public enum Singleton {
	INSTANCE;  
}
```

Enum은 인스턴스가 여러 개 생기지 않도록 확실하게 보장해주고 복잡한 직렬화나 리플렉션 상황에서도 직렬화가 자동으로 지원된다는 이점이 있다. 하지만 Enum에도 한계는 있다. 보통 Android 개발을 하게 될 경우 보통 Singleton의 초기화 과정에 Context라는 의존성이 끼어들 가능성이 높다. Enum의 초기화는 Compile 타임에 결정이 되므로 매번 메서드 등을 호출할 때 Context 정보를 넘겨야 하는 비효율적인 상황이 발생할 수 있다. 결론은 Enum은 효율적인 이디엄이지만 상황에 따라 사용이 어려울 수도 있다는 점이다.

## LazyHolder

현재까지 가장 완벽하다고 평가받고 있는 이디엄이다. 이 이디엄은 synchronized도 필요 없고 Java 버전도 상관없고 성능도 뛰어나다. 일단 코드를 보면 아래와 같다.

```java
public class Singleton {
  private Singleton() {}
  public static Singleton getInstance() {
    return LazyHolder.INSTANCE;
  }
  
  private static class LazyHolder {
    private static final Singleton INSTANCE = new Singleton();  
  }
}
```

이 이디엄에 대해서 설명을 하자면 객체가 필요할 때로 초기화를 미루는 것이다. Lazy Initialization이라고도 한다. Singleton 클래스에는 LazyHolder 클래스의 변수가 없기 때문에 Singleton 클래스 로딩 시 LazyHolder 클래스를 초기화하지 않는다. LazyHolder 클래스는 Singleton 클래스의 getInstance() 메서드에서 LazyHolder.INSTANCE를 참조하는 순간 Class가 로딩되며 초기화가 진행된다. Class를 로딩하고 초기화하는 시점은 thread-safe를 보장하기 때문에 volatile이나 synchronized 같은 키워드가 없어도 thread-safe 하면서 성능도 보장하는 아주 훌륭한 이디엄이라고 할 수 있다.현재까지 LazyHolder에 대해서 문제점은 나타나지 않고 있다. 혹시나 Multi Thread 환경에서 Singleton을 고려하고 있다면 LazyHolder 기법을 이용하자.



# String

## Character Encoding 변경

```java
// 아래 코드는 인코딩을 '변경'하는 코드가 아니다. 
// oldStr을 ISO8859-1로 인코딩한 바이트 코드를 KSC5601로 디코딩해서 읽는 것 뿐이다. 
// 그래서 만약 oldStr이 ISO8859-1, KSC5601과 호환되지 않는다면, 이상한 문자만 나오게 된다. 
String newStr = new String(oldStr.getBytes("ISO8859-1"),"KSC5601");

// 진짜 인코딩을 변경하는 코드는 다음과 같다. 
public class CharEncoding {
    public String encode(String str, String charset) {
        StringBuilder sb = new StringBuilder();
        try {
            byte[] key_source = str.getBytes(charset);
            for(byte b : key_source) {
                String hex = String.format("%02x", b).toUpperCase();
                sb.append("%");
                sb.append(hex);
            }
        } catch(UnsupportedEncodingException e) { }//Exception
        return sb.toString();
    }
     
    public String decode(String hex, String charset) {
        byte[] bytes = new byte[hex.length()/3];
        int len = hex.length();
        for(int i = 0 ; i < len ;) {
            int pos = hex.substring(i).indexOf("%");
            if(pos == 0) {
                String hex_code = hex.substring(i+1, i+3);
                bytes[i/3] = (byte)Integer.parseInt(hex_code, 16);
                i += 3;
            } else {
                i += pos;
            }
        }
        try {
            return new String(bytes, charset);
        } catch(UnsupportedEncodingException e) { }//Exception
        return "";
    }
     
    public String changeCharset(String str, String charset) {
        try {
            byte[] bytes = str.getBytes(charset);
            return new String(bytes, charset);
        } catch(UnsupportedEncodingException e) { }//Exception
        return "";
    }
}

CharEncoding charEncoding = new CharEncoding();
String newStr = charEncoding.encode(oldStr, "utf-8");
```



# Enum

> 참조: [java - enum](https://linuxism.ustd.ip.or.kr/977) 

## 기본 형태

원소들간의 구분은 콤마(,)로 합니다. 그리고 enum 객체 역시 클래스이기 때문에 기본생성자가 있어야합니다. 물론, 아래와 같은 경우에는 생성자를 생략해도 Java가 디폴트 생성자를 만들어줍니다.

```java
public enum FontStyle {
    NORMAL, BOLD, ITALIC, UNDERLINE;

    FontStyle() {
    }
}
```

// 간단하게 인라인 클래스로 배열 선언하듯 사용할 수 있습니다. public으로 설정되어 있기 때문에 SampleClass의 인스턴스를 sample이라고 할 때 sample.FontStyle.NORMAL 처럼 접근할 수 있습니다.

```java
public enum SampleClass {
    public enum FontStyle { NORMAL, BOLD, ITALIC, UNDERLINE }
}
```

## 추가 속성을 부여한 형태

아래는 enum 클래스의 각 원소에 별도 설명을 부여한 형태입니다. 사람이 이해하기 좋은 추가 속성을 지정했습니다. 각 원소 뒤에 괄호를 열고 primitive 속성으로 값을 지정하면 되고, 한개 이상의 경우 콤마로 구분하면됩니다. 단, 이경우에는 설명을 저자할 수 있는 필드가 하나 추가되고 생성자가 이 필드에 값을 설정할 수 있도록 인자가 있습니다.

```java
public enum FontStyle {
    NORMAL("This font has normal style."),
    BOLD("This font has bold style."),
    ITALIC("This font has italic style."),
    UNDERLINE("This font has underline style.");
  
    // 원소 설명
    private String description;

    FontStyle(String description) {
        this.description = description;
    }
    
    public String getDescription() {
        return this.description;
    }
}
```


## 에러코드/에러설명을 정의한 사례

단순히 에러코드만 반환하는 환경이 아니라 사용자 친숙한 코드/설명 쌍으로 결과를 반환하는경우 또는 로깅을 조금더 보기 좋게 하는 경우 사용법2를 응용해서 속성이 2개인 원소들로 구성된 enum을 만들 수 있습니다.


```java
public enum Errors {
    CLIENT_ERROR("ERR_1001", "This is client error."),
    IO_ERROR("ERR_1002", "This is In/Out error."),
    PARSE_ERROR("ERR_1003", "This is parse error."),
    CONTEXT_ERROR("ERR_1004", "This is context error.");


    private String code;
    private String desc;


    Errors(String code, String desc) {
        this.code = code;
        this.desc = desc;
    }
    public String getCode() {
        return this.code;
    }
    public String getDesc() {
        return this.desc;
    }
}
```

실제로 사용할 때는 그때 그때 다르지만 MYException으로 래핑하기 위해 아래와 같이 throw new를 이용하고 MYException 객체는 인자로 enum 클래스를 받게 했습니다.

```java
try {
    ...
} catch (XXException e) {
    throw new MYException(Errors.CLIENT_ERROR);
}
```


## enum을 사용하지 않는 경우

enum을 사용하지 않고 위의 폰트 스타일 예를 분기하는 경우, 가장 쉬운 방법은 아래와 같을 겁니다.


```java
if ("NORMAL".equals(fontStyle)) {
    ...
}
```

이걸 조금 더 구조화해서 개발을 한다면 아래와 같이 별도의 static 클래스를 만들고 int 값으로 비교를 하는 것도 좋은 방법이 되겠죠. 실제로 가장 일반적인 방법이기도 하고요. SWT에서도 UI위젯의 style 속성 정의를 이런 식으로 합니다.

```java
class FontStyle {
    public static final int NORMAL = 0;
    public static final int BOLD = 1;
    public static final int ITALIC = 2;
    public static final int UNDERLINE = 3;
}

if (fontStyle == FontStyle.NORMAL) {
    ...
}
```


## 기존 방법의 문제점

enum은 아래와 같은 문제점들을 해결하기 위해 만들어졌다고 합니다. 형검사를 하지 않으므로 오류발생 소지가 있다. 네임스페이스를 따르지 않으므로 중복의 소지가 있다. 순서 변경, 새로운 원소 추가에 취약합니다. 알아보기가 어렵습니다.

제가 생각하는 enum의 가장 큰 장점은 소스코드 속에서 사용자가 직접 입력한 String을 제거할 수 있다는 것과, 자동 형 검사를 통해 오류를 줄일 수 있다는데 있는것 같습니다. 게다가 enum 클래스는 Java VM 1.5에서 지원하지 시작한 기능으로 아무 추가 작업없이 그냥 쓸 수 있죠.

C로 개발을 할 때도 코드에서 사용할 데이터 구조체와 상수값을 별도의 헤더파일로 뽑아내어 소스코드의 하드코딩 요소들을 제거 했었습니다. enum 클래스 헤더파일을 별도의 클래스로 분리한 겁니다. 형검사라는 강력한 기능이 추가되어서 말이죠.


## enum도 class다


```java
public enum MyColor {
    RED (0xff0000),//객체
    GREEN (0x00ff00),
    private final int rgb,
 
    MyColor(int rgb) { //생성자
    this.rgb = rgb;
    }
 
    public int getRGB() { //일반method
    return rgb;
    }
}
 
enum Element {
    EARTH, WIND,
    FIRE {
    public String info() {
    return "f";
    }
    };
 
    public String info() {
    return "i";
    }
}
 
MyColor mc = new MyColor(); // 단, enum은 일반적인 객체 생성이 불가능하다.
```

# LocalDateTime


## JSON → VO using GSON


```java
package io.kiesnet.sdk;

import java.io.Serializable;
import java.lang.reflect.Type;
import java.time.LocalDateTime;
import java.time.ZonedDateTime;
import java.util.List;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonDeserializationContext;
import com.google.gson.JsonDeserializer;
import com.google.gson.JsonElement;
import com.google.gson.JsonParseException;

public class Account implements Serializable {
    private String s;
    private Number n;
    private boolean b;
    private LocalDateTime t;
    private List<Account> a;

    public static void main(String[] args) {
        Gson gson = new GsonBuilder().registerTypeAdapter(LocalDateTime.class, new JsonDeserializer<LocalDateTime>() {
            @Override
            public LocalDateTime deserialize(JsonElement json, Type type, JsonDeserializationContext jsonDeserializationContext) throws JsonParseException {
                return ZonedDateTime.parse(json.getAsJsonPrimitive().getAsString()).toLocalDateTime();
            }
        }).create();

        Account acc = gson.fromJson("{s:'s', n:123.24, b:true, t:'2002-10-02T15:00:00.05Z', a:[{s:'s1', n:123.24, b:true, t:'2002-10-02T15:00:00.05Z', a:[{s:'s2', n:123.24, b:true, t:'2002-10-02T15:00:00.05Z', a:[]}]}]}", Account.class);

    }
}
```

# Calendar


```java
Calendar cal = Calendar.getInstance();

int year   = cal.get(Calendar.YEAR);
int month  = cal.get(Calendar.MONTH) + 1;
int day    = cal.get(Calendar.DAY_OF_MONTH);
int hour   = cal.get(Calendar.HOUR_OF_DAY);
int minute = cal.get(Calendar.MINUTE);

String sYear   = "" + year;
String sMonth  = month  < 10 ? "0" + month  : "" + month;
String sDay    = day    < 10 ? "0" + day    : "" + day;
String sHour   = hour   < 10 ? "0" + hour   : "" + hour;
String sMinute = minute < 10 ? "0" + minute : "" + minute;
String sNow    = sYear + sMonth + sDay + sHour + sMinute;

String start = "201404170100";
String end   = "201404170300";

if (sNow.compareTo(start) >= 0 && sNow.compareTo(end) <= 0) {
    PrintWriter pw = response.getWriter();
    pw.println("<script>window.location.href = '/'; alert('읭?'); </script>");
    pw.flush();
}
```



## set(MONTH, 1) 시 2월 -> 3월로 넘어감


```java
// 오늘이 1월 30일이라고 가정해보자. 
Calendar cal = Calendar.getInstance();

// 2월 28일자의 Calendar 객체를 만들고 싶다.  
cal.set(Calendar.YEAR, 2015);
cal.set(Calendar.MONTH, 1);

// Month를 2월로 set한 후에 date를 set하기 전이라면 2월 30일인 상태다. 
// 이 상태에서 get()을 한번이라도 호출하면 2월 30일이 자동 계산되어 3월 2일로 변환된다. 
cal.get(Calendar.MONTH);

// 3월 2일이 된 후 setDate(28)을 실행하므로 최종 날짜는 3월 28일이 된다. 
cal.set(Calendar.DATE, 28);

// 그러므로 임의의 시간을 set하려면 그냥 아래와 같이 하자. 
cal.set(2015, 1, 28);
```

# Collections

- Collection : 오브젝트 집합을 나타내는 가장 기본적인 인터페이스
    - Set : 중복 요소 없는 오브젝트 집합
    - SortedSet : 요소가 자동 올림순으로 정렬된다. 삽입되는 요소는 Comparable 인터페이스 속성을 갖고 있지만 지정된 Comparator에 의해 받아들여진다.
    - List : 순서있는 오브젝트 집합, 중복허가, 리스트내의 특정위치에 요소를 넣을 수 있다.

유저는 위치(인덱스)를 지정하여 각요소에 접근할 수 있다.

- Map : 키-값으로 나타내는 오브젝트 집합. 키는 중복될 수 없다.
    - SortedMap : 요소가 키로서 자동 올림순 정렬되는 Map.삽입된 키는 Comparable 인터페이스 속성을 갖고 있지만 지정된 Comparator에 의해 받아들여진다.

| 구분 | 해쉬테이블 | 가변배열  | 밸런스트리 | 링크리스트 |
| ---- | ---------- | --------- | ---------- | ---------- |
| Set  | HashSet    |           | TreeSet    |            |
| List |            | ArrayList |            | LinkedList |
| Map  | HashMap    |           | TreeMap    |            |

## Set & Map & List

- HashSet : HashMap 인터페이스를 기본으로 Set인터페이스를 구현한 것. 순서를 보장할 수 없다.
- TreeSet : TreeMap 인터페이스를 기본으로 Set인터페이스를 구현한 것. 올림순으로 소트된다.
- LinkedHashSet : 삽입된 순서를 기억한다.

같은 요소를 다시 삽입하면 이전 순서를 그대로 기억한다.

- ArrayList : List 인터페이스 속성을 갖고 배열크기를 변경할 수 있다.

배열크기를 조작하는 메소드를 제공한다.

- LinkedList : 리스트의 처음과 마지막 요소를 삽입,삭제할 수 있다.

그 관련 메소드 제공. 동기화되어 있지 않다는 것을 제외하면 Vector와 같다.

- HashMap : 동기화되어 있지않다는 것과 Null을 허용한다는 것 이외에는 HashTable과 같다.
- TreeMap : Red-Black Tree. 키의 올림순으로 소트된다.

LinkedHashMap : 키가 맵에 삽입된 순서를 기억한다. 같은 키로 삽입해도 이전의 순서를 기억한다.

## HashSet,TreeSet,LinkedHashSet 비교

중복요소없는 오브젝트 집합을 관리하는 클래스에는 HashSet,TreeSet,LinkedHashSet가 있다.

- Set기능만 필요한 경우에는 HashSet.
- 요소를 올림차순으로 정렬하는 경우에는 TreeSet.
- 요소의 삽입순서를 기억할 필요가 있을 때에는 LinkedHashSet.


## ArrayList와 LinkedList

순서있는 오브젝트 집합을 관리하는 클래스로는 ArrayList와 LinkedList가 있다.

| 구분       | index access | Iterator access | 추가 | 삽입 |
| ---------- | ------------ | --------------- | ---- | ---- |
| ArrayList  | ○            | ○               | ○    | 느림 |
| LinkedList | 느림         | ○               | ○    | ○    |


- ArrayList는 인덱스에 의한 랜덤엑세스 성능이 좋지만 요소의 삽입, 삭제에는 좋지않다.
- LinkedList는 거꾸로 요소의 삽입, 삭제에는 성능이 좋지만 인덱스에 의한 랜덤엑세스는 좋지 않다.


## HashMap, TreeMap, LinkedHashMap

키-값 관계로 맵핑하는 경우에 사용하는 클래스에는 HashMap, TreeMap, LinkedHashMap이 있다.

- Map 기능만 필요한 경우는 HashMap
- 키를 올림차순으로 소트할 필요가 있을 때는 TreeMap
- 키의 삽입순서를 기억할 필요가 있을 때에는 LinkedHashMap

## Collections.sort(List, Comparator)


```java
import java.util.List;
import java.util.Collections;
import java.util.Comparator;
 
Collections.sort(ExampleList, new Comparator<String>() {
    public int compare(String o1, String o2) {
        return o1.compareTo(o2);
    };
});
 
// 다만 위처럼 String을 비교할 경우 숫자값 비교가 틀어진다. 
// 예를 들어 "1", "2", "11"은 "1", "11", "2"로 sort된다. 
// 숫자값으로 sort하려면 아래와 같이 구현.
 
Collections.sort(ExampleList, new Comparator<ItemEntry>() {
    public int compare(String o1, String o2) {
        int i1 = Integer.valueOf(o1);
        int i2 = Integer.valueOf(o2);
        return i1 - i2;
    };
});
```

## merge(Map baseMap, Map overrideMap)


```java
private static Map merge(Map baseMap, Map overrideMap) {
    for (Object key : overrideMap.keySet()) {
        Object baseObj = baseMap.get(key);
        Object newObj = overrideMap.get(key);

        if (baseObj instanceof Map && newObj instanceof Map) {
            baseMap.put(key, merge((Map) baseObj, (Map) newObj));
        } else if (baseObj instanceof List && newObj instanceof List) {
            List baseList = (List) baseObj;
            for (Object newInnerObj : (List) newObj)
                if (!baseList.contains(newInnerObj))
                    baseList.add(newInnerObj);
        } else {
            baseMap.put(key, newObj);
        }
    }

    return baseMap;
}
```

## deepMerge(Map baseMap, Map overrideMap)


```java
private static Map deepMerge(Map baseMap, Map overrideMap) {
    Map newMap = new HashMap(baseMap);

    for (Object key : overrideMap.keySet()) {
        Object baseObj = baseMap.get(key);
        Object overrideObj = overrideMap.get(key);

        if (baseObj instanceof Map && overrideObj instanceof Map) {
            newMap.put(key, deepMerge((Map) baseObj, (Map) overrideObj));
        } else if (baseObj instanceof List && overrideObj instanceof List) {
            List newList = new ArrayList((List) baseObj);
            for (Object overrideInnerObj : (List) overrideObj)
                if (!newList.contains(overrideInnerObj))
                    newList.add(overrideInnerObj);

            newMap.put(key, newList);
        } else {
            newMap.put(key, overrideObj);
        }
    }

    return newMap;
}
```

# Mail

```java
import java.io.UnsupportedEncodingException;
import java.util.ArrayList;
import java.util.Date;
import java.util.Properties;

import javax.mail.Message;
import javax.mail.MessagingException;
import javax.mail.Multipart;
import javax.mail.PasswordAuthentication;
import javax.mail.Session;
import javax.mail.Transport;
import javax.mail.internet.InternetAddress;
import javax.mail.internet.MimeBodyPart;
import javax.mail.internet.MimeMessage;
import javax.mail.internet.MimeMultipart;

import org.apache.commons.lang.StringUtils;
import org.apache.commons.mail.HtmlEmail;
import org.apache.felix.scr.annotations.Component;
import org.apache.felix.scr.annotations.Reference;
import org.apache.poi.util.StringUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import com.day.cq.mailer.MessageGateway;
import com.day.cq.mailer.MessageGatewayService;
import com.day.cq.workflow.WorkflowException;
import com.day.cq.workflow.WorkflowSession;
import com.day.cq.workflow.exec.WorkItem;
import com.day.cq.workflow.metadata.MetaDataMap;

@Component(label       = "EmailService",
           description = "Email Send Service",
           immediate   = true, enabled = true, metatype = true)
public class EmailService {
    private static final Logger log = LoggerFactory.getLogger(EmailService.class);
    protected String m_smtpHost;
    protected String m_smtpUser;
    protected String m_smtpPassword;
    
    // 특정 smtp 사용을 위한 method
    public boolean send(String host, String from, String fromMail, String to, 
                        String subject, String msg, boolean istext ) {
        Properties props = new Properties();
        props.put("mail.smtp.host", host);

        Session sess = Session.getDefaultInstance(props, null);

        try {
            // create a message
            Message message = new MimeMessage(sess);
            message.setFrom(new InternetAddress(fromMail, from, "utf-8"));
            InternetAddress[] address = { new InternetAddress(to) };

            log.debug("address=======>>>" + address.toString());

            message.setRecipients(Message.RecipientType.TO, address);
            message.setSubject(subject);
            message.setSentDate(new Date());

            // create and fill the first message part
            MimeBodyPart mbp1 = new MimeBodyPart();

            if (istext) {
                mbp1.setContent(msg, "text/plain; charset=utf-8");
            } else {
                mbp1.setContent(msg, "text/html; charset=utf-8");
            }

            // create the Multipart and its parts to it
            Multipart mp = new MimeMultipart();
            mp.addBodyPart(mbp1);

            message.setContent(mp);

            log.debug("[to] : " + to + "  [subject] : " + subject);

            Transport.send(message);
        } catch (MessagingException mex) {
            log.debug("error msg :" + mex.toString());
            return false;
        } catch (UnsupportedEncodingException e) {
            log.debug(e.getMessage());
            return false;
        }
        return true;
    }

    // 일반적인 메일 전송을 위한 default method
    public void send(String aHost, String fromAddress, String toAddress, String userAddress,
            String aSubject, String aMessage, String username, String password,
            String port) {
        try {
            log.info("send() method starts " + EmailService.class.getName());
            
            if( !StringUtils.isEmpty(username) ) {
                log.info("send() user name exists " + EmailService.class.getName());
                
                System.out.println("send() user name exists ");
                
                Properties properties = new Properties();
                
                properties.setProperty("mail.transport.protocol", "smtp");
                if (aHost.contains("gmail"))
                    properties.setProperty("mail.smtp.starttls.enable", "true");
                properties.setProperty("mail.smtp.auth", "true");
                properties.setProperty("mail.smtp.host", aHost);
                properties.setProperty("mail.smtp.port", port);
                
                Authenticator authenticator = new Authenticator(username, password);    
                properties.setProperty("mail.smtp.submitter", authenticator.getPasswordAuthentication().getUserName());
                
                Session session = Session.getInstance(properties, authenticator);
                MimeMessage message = new MimeMessage(session);
                message.setFrom(new InternetAddress(fromAddress));
                message.addRecipient(Message.RecipientType.TO, new InternetAddress(
                        toAddress));
                message.setSubject(aSubject, "UTF-8");
                message.setContent(aMessage, "text/html; charset=utf-8");

                Transport.send(message);
            } else {
                log.info("send() user name not exists" + EmailService.class.getName());
                
                Properties properties = System.getProperties();
                
                properties.setProperty("mail.smtp.host", aHost);
                properties.setProperty("mail.smtp.port", port);
                                
                properties.setProperty("mail.smtp.auth", "false");
                
                Session session = Session.getDefaultInstance(properties);
                MimeMessage message = new MimeMessage(session);
                message.setFrom(new InternetAddress(fromAddress));
                message.addRecipient(Message.RecipientType.TO, new InternetAddress(
                        toAddress));
                message.setSubject(aSubject, "UTF-8");
                message.setContent(aMessage, "text/html; charset=utf-8");
                
                Transport.send(message);
            }
            
            log.info("send() method ends " + EmailService.class.getName());
            
        } catch (Exception ex) {
            log.error("send():Exception---", ex);
        }
    }
    
    private class Authenticator extends javax.mail.Authenticator {
        private PasswordAuthentication authentication;
        
        public Authenticator(String username, String password) {
            log.info("Authenticator() username : " + username);
            log.info("Authenticator() password : " + password);
            authentication = new PasswordAuthentication(username, password);
        }
        
        protected PasswordAuthentication getPasswordAuthentication() {
            return authentication;
        }
    }

    public static boolean isEmpty(String aValue) {
        return (aValue == null || aValue.trim().length() == 0);
    }
}
```



# JSTL

## Config

jakarta-taglibs-standard-1.1.2.zip

jstl.jar과 standard.jar을 WEB-INF/lib 폴더에 넣는다.

tld 폴더를 WEB-INF 폴더에 넣고 web.xml에 다음과 같은 구문을 추가한다.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	<jsp-config>
		<taglib>
			<taglib-uri>http://java.sun.com/jstl/core</taglib-uri>
			<taglib-location>/WEB-INF/tld/c.tld</taglib-location>
		</taglib>
 
		<taglib>
			<taglib-uri>http://java.sun.com/jstl/xml</taglib-uri>
			<taglib-location>/WEB-INF/tld/x.tld</taglib-location>
		</taglib>
 
		<taglib>
			<taglib-uri>http://java.sun.com/jstl/fmt</taglib-uri>
			<taglib-location>/WEB-INF/tld/fmt.tld</taglib-location>
		</taglib>
	</jsp-config>
</web-app>
```



# Web Service

## CXF

- WSDL에서 Client를 만들 경우 : Eclipse에서 WSDL 파일 우클릭 - Make Client Code
- WSDL에서 Server를 만들 경우 : Eclipse에서 WSDL 파일 우클릭 - Make Skeleton Code - runtime을 CXF로 설정
- 라이브러리에서 주의할 점은 cxf-bundle.jar로 동작하지 않는다는 점이다. cxf full runtime을 받아서 lib 폴더에 있는 것들을 넣도록 한다 (jetty의 경우 사용 여부에 따라 생략가능)
- 이후 설정들은 아래와 같이 진행한다.

```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
        <param-value>
            classpath:spring/context-*.xml
        </param-value>
</context-param>
 
  <servlet>
    <servlet-name>CXFServlet</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>CXFServlet</servlet-name>
    <url-pattern>/services/*</url-pattern>
  </servlet-mapping>
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-2.5.xsd http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
    <import resource="classpath:META-INF/cxf/cxf.xml" />
	<import resource="classpath:META-INF/cxf/cxf-extension-soap.xml" />
	<import resource="classpath:META-INF/cxf/cxf-servlet.xml" />
 
<!-- 이 부분은 Implementation class를 명시한다 -->
    <bean id="ServiceImpl" 
    	  class="com.ck.ServiceImpl" />
 
<!-- web service를 만들기 위해서 필요한 factory -->
	<bean id="wsfactory" 
		  class="org.apache.cxf.jaxws.support.JaxWsServiceFactoryBean" 
		  scope="prototype">
		<property name="wrapped" value="true" />
	</bean>
 
<!-- 여기서 주의할 것은 implementor는 implementation class이고, implementorClass는 interface class라는 점이다. 
     implementorClass에 implementation class를 넣을 경우 server가 구동되지 않는다 -->
    <jaxws:endpoint id="ServiceImplService" 
					implementor="#ServiceImpl"
 					implementorClass="com.ck.Service"
					address="/ServiceImplService">
		<jaxws:serviceFactory>
			<ref bean="wsfactory"/>
		</jaxws:serviceFactory>
	</jaxws:endpoint>
 
 </beans>
```

## AXIS2

- install axis2 runtime : need to set axis2 runtime in eclipse
- add axis2 to project facets
- set WEB-INF/web.xml
- add WEB-INF/applicationContext.xml
- make Bean, Service, Client sources
- add META-INF/services.xml
- install a plug-in to make aar(axis archive) : axis2-eclipse-service-plugin-1.6.2.zip
- make & add aar file to services/
- add aar filename to services/services.list

# OSGi

## Bundle


### create by Maven

Maven으로 OSGi Bundle 생성 가능.


### create by Bnd tool

eclipse에서 bnd plugin을 설치하자. bnd 설정파일에 Imported, Exported 등을 명시하고 Release bundle을 돌려주면 된다.


### create by jar cvfm

SimpleCaptcha-1.2.1.jar를 OSGI Bundle로 변환

유의할 점은 manifest.txt에서 Export-Package 작성시 필요한 모든 패키지를 명시해야 한다. (예를 들어, nl.captcha, nl.captcha.audio, nl.captcha.audio.noise...)

manifest.txt


```
Manifest-Version: 1.0
Created-By: dgdsingen
Bundle-ManifestVersion: 2
Bundle-Name: SimpleCaptcha 1.2.1 OSGI Bundle
Bundle-Description: SimpleCaptcha 1.2.1 OSGI Bundle
Bundle-Version: 1.2.1
Bundle-ClassPath: .,simplecaptcha-1.2.1.jar
Bundle-SymbolicName: ln.captcha
Export-Package: com.jhlabs.image,com.jhlabs.math,sounds.en.numbers,sounds.noises,nl.captcha.audio,nl.captcha.audio.noise,nl.captcha.audio.producer,nl.captcha.backgrounds,nl.captcha.gimpy,nl.captcha.noise,nl.captcha.servlet,nl.captcha.text.producer,nl.captcha.text.renderer,nl.captcha.util,nl.captcha
jar cvfm simplecaptcha-1.2.1-bundle.jar manifest.txt simplecaptcha-1.2.1.jar
```

### Issue : too long line in manifest file while trying to create jar

the classpath is too long due to the number of jar files in it. The maximum characters in the classpath list per line should not exceed 72. 그러므로 아래와 같이 길이에 따라 각 라인을 잘라주면 된다.

```
Export-Package: com.jhlabs.image,com.jhlabs.math,sounds.en.numbers,soun
ds.noises,nl.captcha.audio,nl.captcha.audio.noise,nl.captcha.audio.prod
ucer,nl.captcha.backgrounds,nl.captcha.gimpy,nl.captcha.noise,nl.captch
a.servlet,nl.captcha.text.producer,nl.captcha.text.renderer,nl.captcha.
util,nl.captcha
```

# Quartz

## Spring + Quartz Config

- web.xml
- struts-config/struts-config.xml (struts + spring 사용할 경우)
- config/applicationContext_cronJob.xml
- com.ck.* (Job Class extends QuartzJobBean)

[ApplicationContext_cronJob.xml](http://docs.google.com/quartz.html)

```xml
<!-- quartz job class (실제 로직을 담은 클래스) 설정 -->
<bean id="testJob" class="org.springframework.scheduling.quartz.JobDetailBean">
    <property name="jobClass" value="com.ck.TestJob"/>
</bean>
 
<!-- quartz trigger (cron 시간) 설정-->
<bean id="testJobTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
    <property name="jobDetail" ref="testJob"/>
    <property name="cronExpression" value="0/5 * * * * ?"/>
</bean>
 
<!-- quartz 실행 환경 설정 : 첫번째 방법 -->
<bean class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
    <property name="triggers">
        <list>
            <ref bean="testJobTrigger"/>
        </list>
    </property>
    <property name="quartzProperties">
        <props>
            <prop key="org.quartz.threadPool.class">org.quartz.simpl.SimpleThreadPool</prop>
            <prop key="org.quartz.threadPool.threadCount">3</prop>
            <prop key="org.quartz.threadPool.threadPriority">4</prop>
            <prop key="org.quartz.jobStore.class">org.quartz.simpl.RAMJobStore</prop>
            <prop key="org.quartz.jobStore.misfireThreshold">60000</prop>
        </props>
    </property>
</bean>
 
<!-- quartz 실행 환경 설정 : 두번째 방법 -->
<bean id="scheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean" lazy-init="false" autowire="byName">
    <property name="triggers">
        <list>
            <ref bean="TestJobTrigger"/>
        </list>
    </property>
    <property name="dataSource" ref="dataSource"/>
    <property name="configLocation" value="classpath:quartz.properties"/>
</bean>
```

[quartz.properties](http://docs.google.com/quartz.codeblock.1.html)

```properties
# ApplicationContext_cronJob.xml 설정시 
# quartz 실행 환경 설정 : 두번째 방법을 사용했으면 
# classpath 아래 (ex : WEB-INF/classes/) quartz.properties를 설정해준다. 
#============================================================================
# Configure Main Scheduler Properties  
#============================================================================
org.quartz.scheduler.instanceName = DefaultQuartzScheduler
org.quartz.scheduler.instanceId = AUTO
 
#============================================================================
# Configure ThreadPool  
#============================================================================
org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
org.quartz.threadPool.threadCount = 10
org.quartz.threadPool.threadPriority = 5
org.quartz.threadPool.threadsInheritContextClassLoaderOfInitializingThread = true
 
#============================================================================
# Configure JobStore  
#============================================================================
org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
org.quartz.jobStore.driverDelegateClass = org.quartz.impl.jdbcjobstore.oracle.OracleDelegate
org.quartz.jobStore.misfireThreshold = 60000
org.quartz.jobStore.tablePrefix = QRTZ_
org.quartz.jobStore.maxMisfiresToHandleAtATime=10
org.quartz.jobStore.isClustered = true
org.quartz.jobStore.clusterCheckinInterval = 15000
```

Bean

```java
package com.ck;
 
import java.text.SimpleDateFormat;
import org.quartz.JobExecutionContext;
import org.quartz.JobExecutionException;
import org.springframework.scheduling.quartz.QuartzJobBean;
 
public class TestJob extends QuartzJobBean{
    @Override
    protected void executeInternal(JobExecutionContext arg0) throws JobExecutionException {
        long time = System.currentTimeMillis();
        SimpleDateFormat sdf = new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");
        System.out.println("TestJob : " + sdf.format(time));
    } 
}
```

## WAS Cluster 환경에서 Quartz 중복실행

- 방안 1. 각 서버에 실행할 스케줄을 분배
- 방안 2. Quartz의 JDBC-Jobstore 를 활용하여 Clustering 한다.아래 URL을 참조. 해당 방법은 위 quartz.properties에 선언되어 있다.
- 원문 : http://www.quartz-scheduler.org/documentation/quartz-1.x/tutorials/TutorialLesson11
- 번역페이지 : http://blog.naver.com/dreamy_alice/100054710317
- 이 경우 먼저 DB에 테이블을 생성해야 한다. 그 후 DB에서 Trigger 정보는 아래와 같이 조회한다.

```sql
-- Trigger 정보
SELECT ROWID, a.*
  FROM qrtz_triggers a
 
-- Trigger cron 정보  
SELECT ROWID, a.*
  FROM qrtz_cron_triggers a
 
-- fired된 Trigger 조회
SELECT ROWID, a.*
  FROM qrtz_fired_triggers a
 
-- Trigger가 실행할 job 정보 
SELECT ROWID, a.*
  FROM qrtz_job_details a
```

# Issues

## JSP에서 EL이 적용되지 않는 경우

```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```

위와 같은 소스가 있다면 아래와 같이 바꿔주자. uri 때문에 EL을 인식하지 못하는 경우가 있음.

```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jstl/core" %>
```

## Get Client's IP

```java
String ip = request.getHeader("X-Forwarded-For");
if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip))
    ip = request.getHeader("Proxy-Client-IP");
if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip))
    ip = request.getHeader("WL-Proxy-Client-IP");
if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip))
    ip = request.getHeader("HTTP_CLIENT_IP");
if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip))
    ip = request.getHeader("HTTP_X_FORWARDED_FOR");
if (ip == null || ip.length() == 0 || "unknown".equalsIgnoreCase(ip))
    ip = request.getRemoteAddr();
```

## JSP Chracter Encoding

```java
<%@ page contentType="text/html; charset=UTF-8"  language="java" %>
```

## Response no-cache

```java
response.setHeader("Cache-Control","no-store");
response.setHeader("Pragma","no-cache");
response.setDateHeader("Expires",0);
if (request.getProtocol().equals("HTTP/1.1"))
	response.setHeader("Cache-Control", "no-cache");
```

## c:out 결과값에 태그 넣어서 동작시키기

test.desc를 c:out으로 출력하면 그냥 문자열이 나온다. fn:replace로 `<br>`을 박아도 그냥 문자열로 인식된다. 그러므로 javascript를 이용해 후처리를 해주면 태그를 동적으로 변경할 수 있다.

```html
<c:forEach var="test" items="${testList}" varStatus="stat">        
    <li>
        <p id="test_${stat.count}"><c:out value="${test.desc}" /></p>
        <script>
            var txt = $("#test_${stat.count}");
            txt.html( jQuery.trim(txt.html()).replace(' ', '<br>') );
        </script>
    </li>
</c:forEach>
```

## HTML Tag 제거 정규식


```java
<(/)?([a-zA-Z]*)(\\s[a-zA-Z]*=[^>]*)?(\\s)*(/)?>

public String removeTag(String html) throws Exception {
	return html.replaceAll("<(/)?([a-zA-Z]*)(\\s[a-zA-Z]*=[^>]*)?(\\s)*(/)?>", "")
}
```



## Deep copy와 Shallow copy


### clone 메소드 override를 이용한 deep-copy


```java
class Test {
    Object inner;
}

// 위 구조에서 Test.clone()을 실행하면 Test 객체는 deep copy가 되나 inner는 shallow copy가 된다. 
// 자바에서 객체를 만드는 과정이 성능에 큰 영향을 미치기 때문에 clone 메소드가 이렇게 동작하도록 되어있다. 
// 그래서 Test.clone()을 다음과 같이 override 해준다.

class Test {
    Inner inner;
    
    @Override
    protected Object clone() throws CloneNotSupportedException {
        Test t = (Test) super.clone();
        t.inner = (Inner) t.inner.clone();
        return t;
    }
}
```



### Serialized를 이용한 deep-copy


```java
class Fruit implements Serializable {
    public Fruit(String arg_strName) {
        m_strName = arg_strName;
    }

    String m_strName;
}

class CloneTester implements Serializable {

    public CloneTester(int arg_nVal) {
        m_nVal = arg_nVal;
    }
   
    public Object customClone() throws IOException, ClassNotFoundException {
        // serializing
        ByteArrayOutputStream byteArrOs = new ByteArrayOutputStream();
        ObjectOutputStream objOs = new ObjectOutputStream(byteArrOs);
        objOs.writeObject(this);
        
        // de-serializing
        ByteArrayInputStream byteArrIs = new ByteArrayInputStream(byteArrOs.toByteArray());
        ObjectInputStream objIs = new ObjectInputStream(byteArrIs);
        Object deepCopy = objIs.readObject();
       
        return deepCopy;
    }
   
    int m_nVal;
    Fruit fruit = new Fruit("apple");
}
```



### Reflection을 이용한 copy


```java
public Object copyObject(Object o) {
    Object result = null;
    
    try {
        Class cls1 	= o.getClass();
        result 		= cls1.newInstance();
        Class cls2 	= result.getClass();

        Method[] methods1 = cls1.getDeclaredMethods();
        Method[] methods2 = cls2.getDeclaredMethods();
        
        // [method1 -> method2]로 복사
        for (Method method1 : methods1) {
            for (Method method2 : methods2) {
                if (method1.getName().startsWith("get")) {
                    if (method1.getName().substring(method1.getName().indexOf("get")+3)
                            .equals(method2.getName().substring(method2.getName().indexOf("set")+3)))
                    method2.invoke(result, method1.invoke(o));
                }
            }
        }
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    } catch (IllegalArgumentException e) {
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        e.printStackTrace();
    }
    
    return result;		
}
```



## shell 실행


```java
// 아래 코드에는 문제가 있다. 
private static int shellFtpUpload(String filename) {
    String s = null;
	Process process = null;

	try {
        process = new ProcessBuilder("/usr/bin/sh", "/home/as/bd01/GWM/JAVA/f001.sh " + filename).start();

	BufferedReader stdOut = new BufferedReader(new InputStreamReader(process.getInputStream()));
	BufferedReader stdError = new BufferedReader(new InputStreamReader(process.getErrorStream()));

	while ((s = stdOut.readLine()) != null)
	    System.out.println(s);
	while ((s = stdError.readLine()) != null)
	    System.err.println(s);

	} catch (Exception e) {
		System.err.println("Exception : " + e.getMessage());
	} finally {
        process.destroy();
        System.err.println("Shell exit value : " + process.exitValue());
        return process.exitValue();
    }
}

//waitFor()는 자식 프로세스가 죽을 때까지 기다린다. destroy()는 강제로 죽이는 것이다. 
//만약 프로세스가 아직 도는데 출력할 것이 없다고 해서 destroy()를 치고 exitValue()를 받아낸다면 안된다.
//그러면 무조건 에러가 나게 된다. 그래서 프로세스가 다 돌때까지 기다린다.
//그러나 어떤 예외(타임아웃 등)가 나면 그때는 프로세스를 강제로 죽이고 반환값을 -1 그대로 내보낸다. 
//만약 예외는 나지 않았으나 프로세스가 정상적으로 종료되지 않아도 그 반환값을 제대로 받을 수 있다. 
@SuppressWarnings("finally")
public static int exec(String... statements) {
    Process process = null;
    
    String[] commands = new String[statements.length + 1];
    commands[0] = "/bin/sh";
    for (int i = 0; i < statements.length; i++)
        commands[i+1] = statements[i];            
    
    // process가 종료하면서 반환하는 값
    int exitValue = -1;

    try {
        process = new ProcessBuilder(commands).start();

        BufferedReader stdOut = new BufferedReader(new InputStreamReader(process.getInputStream()));
        BufferedReader stdError = new BufferedReader(new InputStreamReader(process.getErrorStream()));

        String s = null;
        while ((s = stdOut.readLine()) != null)
            System.out.println(s);
        while ((s = stdError.readLine()) != null)
        System.err.println(s);

        exitValue = process.waitFor();
    } catch (Exception e) {
        process.destroy();
        System.err.println("Exception : " + e.getMessage());
    } finally {
        System.err.println("Shell exit value : " + exitValue);
        return exitValue;
    }
}
```



## String Encoding 변경


```java
// 문자 인코딩이 ISO-8859-1인 문자열을 EUC_KR로 변경한다.
String newStr = new String( oldStr.getBytes("ISO-8859-1"), "EUC_KR" );
```



## for-each 문에서 변수의 길이 변경 불가

arrayList에 대해 for-each 안에서 요소 추가나 삭제시 에러가 난다. (당연하다. for-each문은 arrayList의 모든 요소를 하나씩 모두 방문하는 loop문이다. 하나가 추가되거나 삭제되는 과정이 있다면 arrayList의 index와 그에 대응되는 값은 완전히 달라지게 된다.)


## 가변 인자


```java
public void go(String... s)
public void go(String...s)
public void go(String[]... s)
```


위 3개 모두 가능하다. 그러나 가변 인자는 반드시 인자 목록 중 마지막에 정의되어야 한다


## 생성자

Child extends Parent 에서 Child()가 Parent()를 명시적으로 호출하지 않는다면 (super(), super(...)를 호출하지 않는다면) 자동적으로 super()가 추가된다. 이 상태에서 Parent에 Parent()가 없을 때는 자동으로 Parent()가 만들어져 정상적으로 구동된다. 하지만 Parent(...)를 구현했다면 Parent()도 반드시 구현해야만 한다.


## 접근 제한자

- public > protected > default > private
- public : package를 벗어나 어떤 class에서든 접근 가능
- protected : 상속 관계만 접근 가능
- default : 같은 package 안에서만 접근 가능
- private : 자신만 접근 가능


## overriding vs overloading

- overriding : parameter type 동일해야 함 return type 호환 가능해야 함 (호환 가능이란, override한 method가 더 상위 type을 return할 수 없다) 접근 제한자는 더 좁아지면 안됨
- overloading : parameter type이 반드시 달라야 한다 return type은 달라도 된다 그러나 parameter type이 같은데 return type만 다른 것은 당연히 불가능 새로운 method 선언이므로 접근 제한자 여부는 상관 없음


## File 읽고 쓰기


```java
new BufferedWriter(new FileWriter(aFile));
new BufferedReader(new FileReader(aFile));
File aFile = new File("test.txt");
reader.read() : character 단위 읽음
reader.readLine() : String, 줄 단위 읽음
```



## Generic


```java
public <T extends Animal> void go(ArrayList<T> list)
```


Animal을 상속한 T 타입은 모두 사용 가능하다.


```java
public void go(ArrayList<Animal> list)
```


Animal 타입만 사용 가능하다.


```java
public void go(ArrayList<? extends Animal> list)
```


Animal을 상속한 타입은 모두 사용 가능하다. 그러나 ?(와일드카드)사용시 list.add() 같은 목록 추가는 불가능하다.

```java
<? super T> : T의 상위 타입
```


## abstract vs interface

abstract method가 하나라도 있다면 그 class는 반드시 abstract class가 되어야 한다. 하지만 abstract class는 concrete field와 method를 가질 수 있다. 여기서 abstract variable이란 존재하지 않으며, abstract method는 반드시 상속이나 구현을 통해 몸체를 가질 때에만 그 의미가 있다. 그래서 abstract method는 private이 될 수 없다.

interface는 100% abstract class이다. interface의 내부 모든 것은 public이며, abstract method와 public static final var만이 존재한다. interface를 extends 할 수 있는 것은 interface 뿐이므로 interface의 method는 public abstract 일 수 밖에 없다. (class가 아닌 interface인 이상 method를 override 받아 구현할 수가 없다)

abstract class의 객체는 생성이 불가능한가? 명시적으로 생성은 불가능하다. new 키워드를 사용할 수 없다. 그러나 생성자를 가질 수 있으며, 암시적으로 객체도 생성된다. abstract class의 자식 class가 만들어질 때 그 생성자가 실행된다. (객체가 생성된다)


## Thread

object.wait() : wait()을 실행하기 위해서는 현재 thread가 락을 가지고 있어야 한다. wait()을 호출한 thread는 (락을 가진 thread) 락을 풀고 wait-set에 들어가서 notify()나 notifyAll()을 기다린다. (해당 객체에 하나의 wait-set이 존재한다)

object.notifyAll() : wait-set에서 기다리는 모든 thread들을 깨운다. notifyAll()을 호출한 thread는 synchronized의 notifyAll() 이후 코드를 모두 수행하고 나서 (synchronized 블록을 끝내고) 락을 해제한다. 락은 wait-set에서 일어나 대기하던 thread 중 하나에 주어진다. (wait()으로 락을 건네주고 들어간 thread의 실행이 보장된다. 락을 다시 받은 thread는 wait() 이후부터 수행한다)

wait / notify는 synchronized 블락 부분을 둘 이상의 thread가 사용할 때 일정 단위별로 수행하기 위한 method이다. wait / notify를 synchronized가 아닌 곳에 쓸 수는 있으나 반드시 synchronized 블락 안에서 사용하도록 하자

만약 락을 가지지 않은 thread가 wait()이나 notify()를 호출한다면 IllegalMonitorStateException이 발생한다.


```java
synchronized(Thread.currentThread()) {
    object.wait();
    object.notify();
}
```


위의 예에서 락은 Thread.currentThread()로 반환되는 객체에 걸려있다. 락은 object에 걸린 것이 아니다. 그래서 object 객체에 참여하는 thread들에게는 object에 대한 락이 존재하지 않는다. 만약 파라미터를 object로 바꾼다면 예외가 발생하지 않는다.

InterruptedException : object.wait()이나 Thread.sleep(), join()은 thread를 잠시 또는 영원히 중단할 수 있다. thread가 대기 상태로 들어갔다가 깨어나지 못할 때 발생하는 예외이다. (interrupt()로 thread를 중단시킬 때 발생하기도 한다) 그러므로 wait(), sleep(), join()은 기본적으로 예외 가능성을 지닌다.

여기서 실행 중인 thread는 interrupt 신호를 받아도 isInterrupt()로 확인하기 전까지는 이를 알지 못한다. block 상태의 thread는 interrupt 신호를 받으면 바로 깨어난다. 하지만 깨어났을 때는 이미 interrupt 신호가 해제된 상태이다.

모든 Thread는 Runnable이다. 그래서 new Thread(new Runnable)와 같은 형태로 thread 객체를 생성해 돌려도 Runnable로 구현한 run()보다 Thread를 상속받아 구현한 run()이 우선 적용된다. 하지만 이것이 Thread에만 적용되는 것은 아니다. class A extends B implements C 일 경우 B와 C에 같은 이름의 method가 있다면 B의 method를 구현하게 된다.


## hashCode()와 equals()

두 객체가 같으면 equals()는 true를 반환한다. 또한 같은 hashCode를 가진다.

두 객체의 hashCode가 같다고 두 객체가 반드시 같은 것은 아니다. 그러나 두 객체가 같다면 두 hashCode는 반드시 같아야 한다. 

(hashMap같은 경우 같은 hashCode 안에 여러 객체가 들어갈 수 있다. 하나의 hashCode는 객체를 담는 바구니 역할을 한다)

equals()를 override한다면, hashCode()도 반드시 override해야 한다.

hashCode()는 heap의 각 객체가 가지는 유일한 정수를 return한다. class에서 hashCode()를 override하지 않으면 두 객체는 절대 같은 것으로 간주될 수 없다.


## Equality

equals() : 객체의 값이 같은가? == : 가리키는 객체가 같은가?

레퍼런스 동치 : o1 == o2. 두 레퍼런스가 heap에 있는 하나의 객체를 참조한다. 객체 동치 : o1.equals(o2) && o1.hashCode() == o2.hashCode()). 두 레퍼런스가 한 객체를 참조할 수도 있고, 같은 것으로 간주되는 서로 다른 객체를 참조할 수도 있음.


```java
String s1 = "asd";
String s2 = new String("asd");
s1 == "asd" // true
s2 == "asd" // false
s1.equals("asd") // true
s2.equals("asd") // true
```

## Casting


```java
Parent p = new Parent();
Child c = new Child();
c = (Child) p;// Compile 에러는 없으나 ClassCastException 발생.
 
Parent p = new Child();
Child c = new Child();
c = (Child) p;// 캐스팅 해야 한다.
p = c; // 그냥 가능하다.
```


즉, 상위 타입의 레퍼런스에 하위 타입 레퍼런스를 할당하는 경우는 문제가 없다. 반대의 경우는 캐스팅이 필요하다. 그러나 캐스팅을 하더라도 하위 타입 레퍼런스가 상위 타입 객체를 가리키는 것은 불가능하다.


## 상속시 Exception 문제


```java
class Parent {
void go() throws...
}
class Child {
void go() ...
}
Parent p = new Child();
p.go(); //Compile 에러가 난다. 예외 처리를 해주어야 한다.
Child c = new Child();
c.go(); //이 코드는 문제 없이 실행된다.
```


비슷하지만 다른 예로 Parent에 없던 예외를 Child가 override받은 method에서 던지는 경우는 어떻게 되는가? 상위에 없던 예외를 던지는 경우는 허용되지 않는다. 또, 둘 다 예외를 던지더라도 상위보다 하위 메소드가 더 상위 타입의 예외를 던지는 것도 허용되지 않는다.

static field, method는 override되지 않는다. 대신 hiding 된다.


```java
A a = new B();
a.staticMethod(); //A.staticMethod()
a.instanceMethod(); //B.instanceMethod()
((B)a).staticMethod(); //B.staticMethod()
```

## Error와 Exception, Iterable과 Iterator

Error와 Exception 둘 다 Throwable로부터 갈라져 나왔지만 다르다. Iterable과 Iterator 역시 다르다. for-each에 들어가는 것은 Iterable이다.


## Type 호환에 관해


```java
void go(Short s)
void go(int s)
short s = 6;
go(s); // go(int s)가 실행된다.
 
void go(Long s)
void go(int s)
long s = 6;
go(s); // go(Long s)가 실행된다.
```


왜 첫번째 예에서 go(int s)가 실행되는가? go(Short s)의 파라미터 타입은 Short, 즉 primitive가 아니라 class다. 

호환 가능하다면 class보다 primitive 타입이 앞서게 된다. 두번째 예에서는 int가 long을 담을 수 없으므로 go(Long s)가 실행된다.


## Swing 구성시

JPanel과 그 안의 Component들이 size 변경 때마가 깨지는 이유는 repaint() 될 때마다 setBounds() 등으로 size를 재지정 해주지 않기 때문이다.Component의 size를 repaint() 때마다 다시 지정해주지 않으면 size가 초기화된다. (JFrame 및 JPanel의 size가 변경되므로 당연히 그 내부의 Component들은 재조정된다)

JPanel 안에 JButton이 있을 경우, JButton.repaint(); _ JButton을 다시 그린다. JPanel.repaint(); _ JPanel을 다시 그린다.




# References
- [Java EE Docs](https://www.oracle.com/technetwork/java/javaee/documentation/index.html) 
- [Java SE Docs](https://www.oracle.com/technetwork/java/javase/documentation/api-jsp-136079.html) 
- [Java SE 8 Docs](https://docs.oracle.com/javase/8/docs/) 
- [JSR](https://www.jcp.org/en/jsr/stage?listBy=final) 
- [ojdkbuild/ojdkbuild: OpenJDK Community builds](https://github.com/ojdkbuild/ojdkbuild) 
- [GITNE/icedtea-web: IcedTea-Web for Windows](https://github.com/GITNE/icedtea-web) 
- [iluwatar/java-design-patterns](https://github.com/iluwatar/java-design-patterns) 
- [Multi Thread 환경에서의 올바른 Singleton](https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42) 
- [What Do WebLogic, WebSphere, JBoss, Jenkins, OpenNMS, and Your Application Have in Common? This Vulnerability.](https://foxglovesecurity.com/2015/11/06/what-do-weblogic-websphere-jboss-jenkins-opennms-and-your-application-have-in-common-this-vulnerability/) 
- [Java Connector Using Gradle - MariaDB Knowledge Base](https://mariadb.com/kb/en/library/java-connector-using-gradle/) 
- [Lambda Expressions in Java](https://jdm.kr/blog/181) 
- [SLF4J 로깅 처리](https://www.google.co.kr/amp/s/sonegy.wordpress.com/2014/05/23/how-to-slf4j/amp/) 
- [Simple Json 오브젝트 생성과 파싱 (Create JSonObject and Parsing)](http://www.dante2k.com/458) 
- [json을 map으로, map을 json으로 변환하는 예제들](http://zzznara2.tistory.com/687) 
- [Java HashMap은 어떻게 동작하는가?](https://d2.naver.com/helloworld/831311) 
- [Java Enum 활용기](http://woowabros.github.io/tools/2017/07/10/java-enum-uses.html) 
- [tomcat(톰켓) 튜닝-2 | I am not okay](https://jeedyblog.wordpress.com/2016/08/11/tomcat%ED%86%B0%EC%BC%93-%ED%8A%9C%EB%8B%9D-2/) 
- [CPU 사용률이 높은 프로세스에서 쓰레드 찾는 명령어](http://soul0.tistory.com/450) 
- [PermGen OutOfMemoryError](http://egloos.zum.com/aploit/v/5014643) 
- [Java Is Still Free – Java Champions – Medium](https://medium.com/@javachampions/java-is-still-free-c02aef8c9e04) 
- [Java 스트림 Stream (1) 총정리 | Writer, IT Blog](https://futurecreator.github.io/2018/08/26/java-8-streams/) 
- [Java 스트림 Stream (2) 고급 | Writer, IT Blog](https://futurecreator.github.io/2018/08/26/java-8-streams-advanced/) 
- [Java 8 개선 사항 관련 글 모음](https://blog.fupfin.com/?p=27) 
- [java keytool 사용법 - Keystore 생성, 키쌍 생성, 인증서 등록 및 관리](https://www.lesstif.com/pages/viewpage.action?pageId=20775436) 

## Spring
- [Spring.io](https://spring.io/) 
- [Spring 5.0.2 Docs](https://docs.spring.io/spring/docs/5.0.2.RELEASE/spring-framework-reference/pdf/) 
- [Baeldung | Java, Spring and Web Development tutorials](https://www.baeldung.com/) 
- [Spring @Async 비동기처리](http://dveamer.github.io/java/SpringAsync.html) 
- [spring transaction을 활용해 async로 데이터를 처리할 때의 이슈](https://www.slipp.net/questions/123) 
- [SpringBoot + Ehcache 기본 예제 및 소개](http://jojoldu.tistory.com/57) 
- [Spring MVC - HTTP Streaming using ResponseBodyEmitter](https://www.logicbig.com/tutorials/spring-framework/spring-web-mvc/http_streaming.html) 
- [Building an Application with Spring Boot](https://spring.io/guides/gs/spring-boot/) 
- [Project Reactor](https://projectreactor.io/) 
- [Creating an Asynchronous, Event-Driven Application with Reactor](https://spring.io/guides/gs/messaging-reactor/) 
- [Spring Cloud](http://projects.spring.io/spring-cloud/) 
- [Routing and Filtering using Netflix Zuul](https://spring.io/guides/gs/routing-and-filtering/) 
- [Handling REST API using Spring RestTemplate](http://blog.saltfactory.net/using-resttemplate-in-spring/) 
- [Spring MVC REST API](https://rebeccacho.gitbooks.io/spring-study-group/content/chapter16.html) 
- [Spring Initializr](https://start.spring.io/) 
- [19. Running Your Application](https://docs.spring.io/spring-boot/docs/current/reference/html/using-boot-running-your-application.html) 
- [java - SpringApplication.run main method](https://stackoverflow.com/questions/24271705/springapplication-run-main-method) 
- [Create API using SpringBoot &amp; JPA](https://jojoldu.tistory.com/251?category=635883) 
- [Configuring Spring Boot for MariaDB](https://springframework.guru/configuring-spring-boot-for-mariadb/) 
- [RestTemplate (정의, 특징, URLConnection, HttpClient, 동작원리, 사용법, connection pool 적용)](http://sjh836.tistory.com/141) 
- [benelog/spring-jdbc-tips: Spring JDBC 활용팁, SQL 관리 방안 등](https://github.com/benelog/spring-jdbc-tips) 
- [Spring + MyBatis 에서 여러 개의 DataSource Routing 하는 방법](http://sidnancy.kr/2014/02/04/DataSource-Routing-in-several-ways-to-Spring-MyBatis.html) 
- [분산 DB 환경 RoutingDataSource &amp; JTA 트랜잭션 처리](https://d2.naver.com/helloworld/5812258) 
- [1) 스프링부트로 웹 서비스 출시하기 - 1. SpringBoot &amp; Gradle &amp; Github 프로젝트 생성하기](https://jojoldu.tistory.com/250?category=635883) 
- [SpringBoot에서 날짜 타입 JSON 변환에 대한 오해 풀기](https://jojoldu.tistory.com/361) 
- [Spring Boot Actuator 소개](https://supawer0728.github.io/2018/05/12/spring-actuator/) 
- [Spring boot Actuator 사용해보자 (1)](http://wonwoo.ml/index.php/post/1787) 
- [Spring boot Actuator 사용해보자 (2)](http://wonwoo.ml/index.php/post/1796) 
- [Spring boot Actuator 사용해보자 (3)](http://wonwoo.ml/index.php/post/1803) 
- [Gradle Multi Module에서 Spring Rest Docs 사용하기](https://jojoldu.tistory.com/294?category=635883) 
- [How to Create an Executable JAR with Maven | Baeldung](https://www.baeldung.com/executable-jar-with-maven) 
- [Spring Boot Memory Performance](https://spring.io/blog/2015/12/10/spring-boot-memory-performance) 

