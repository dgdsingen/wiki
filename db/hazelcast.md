# Hazelcast

In-Memory Data Grid. 분산된 시스템들에 인메모리로 데이터를 저장하고 동기화한다. 또한 Master 서버가 없이 모두 Active로 동작하므로 확장하기에 좋다. 모든 WAS에 분산해서 설치해두고 Local Cache로 사용하는 방법, 혹은 Hazelcast 서버들만 모아두고 Global Cache로 사용하는 방법 모두 가능하다. 



## Config

pom.xml에 아래와 같이 추가해주자. 바로 사용할 수 있다. 

```xml
<dependencies>
    <dependency>
        <groupId>com.hazelcast</groupId>
        <artifactId>hazelcast</artifactId>
        <version>4.0</version>
    </dependency>
</dependencies>
```



## Test

간단한 테스트를 위해 https://start.spring.io/ 에서 Web만 추가하여 다운로드 받는다. 이후 Controller를 작성한다. 

```java
package com.example.demo.controller;

import java.util.Arrays;
import java.util.Map;
import java.util.Queue;

import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.HazelcastInstance;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/cache")
class CacheController {

    public HazelcastInstance getHazelcastInstance() {
        return LazyHolder.HAZELCAST_INSTANCE;
    }

    private static class LazyHolder {
        private static final HazelcastInstance HAZELCAST_INSTANCE = 
	    Hazelcast.getHazelcastInstanceByName("hzc-instance-1");
    }

    @GetMapping("/set")
    public void set() {
        HazelcastInstance hazelcastInstance = this.getHazelcastInstance();

        Map<Integer, String> mapCustomers = hazelcastInstance.getMap("customers");
        mapCustomers.put(1, "Joe");
        mapCustomers.put(2, "Ali");
        mapCustomers.put(3, "Avi");

        Map<Integer, String> mapTests = hazelcastInstance.getMap("tests");
        mapTests.put(1, "1");
        mapTests.put(2, "2");
        mapTests.put(3, "3");

        Queue<String> queueCustomers = hazelcastInstance.getQueue("customers");
        queueCustomers.offer("Tom");
        queueCustomers.offer("Mary");
        queueCustomers.offer("Jane");
    }

    @GetMapping("/get")
    public String get() {
        HazelcastInstance hazelcastInstance = this.getHazelcastInstance();

        Map<Integer, String> mapCustomers = hazelcastInstance.getMap("customers");
        Map<Integer, String> mapTests = hazelcastInstance.getMap("tests");
        Queue<String> queueCustomers = hazelcastInstance.getQueue("customers");

        StringBuilder sb = new StringBuilder();
        sb.append(mapCustomers.values()).append("\n")
        .append(mapTests.values()).append("\n")
        .append(Arrays.toString(queueCustomers.toArray())).append("\n");

        return sb.toString();
    }

}
```



그냥 포트만 달리해서 hazelcast를 사용하는 Tomcat을 2개 띄울수도 있겠지만, 같은 JVM 내에서 hazelcast 노드를 여러개 실행할 경우 ip:port를 지정해주지 않아도 자동으로 클러스터링될 수 있으므로 환경 분리를 위해 docker container로 각각 2개를 실행한다. 클러스터링 테스트를 위해 우선 docker network를 만들어준다. 

```sh
docker network create --gateway 172.20.0.1 --subnet 172.19.0.0/21 my-net
```



위에서 만든 network를 사용하는 container를 2개 만들어준다. 

```sh
docker run -it -v /home/dgdsingen/repo/me:/data --network my-net --ip 172.20.0.2 -p 5701:5701 -p 8080:8080 ubuntu
docker run -it -v /home/dgdsingen/repo/me:/data --network my-net --ip 172.20.0.3 -p 5702:5702 -p 8081:8081 ubuntu
```



src/main/resources/application.properties에 각 컨테이너의 8080, 8081 포트를 넣어준다. 

```properties
server.port=8080
```

```properties
server.port=8081
```



src/main/resources/hazelcast.yaml에 각 컨테이너의 5701, 5702 포트를 넣어준다. 

```yaml
hazelcast:
  cluster-name: hzc-cluster-1
  instance-name: hzc-instance-1
  management-center:
    scripting-enabled: false
    
  network:
    public-address: 172.20.0.2:5701
    port:
      auto-increment: false
      port: 5701
    join:
      multicast:
        enabled: false
      tcp-ip:
        enabled: true
        member-list:
          - 172.20.0.3:5702

  map:
    customers:
      time-to-live-seconds: 0
      eviction:
        eviction-policy: LFU
        max-size-policy: PER_NODE
        size: 1000
```

```yaml
hazelcast:
  cluster-name: hzc-cluster-1
  instance-name: hzc-instance-2
  management-center:
    scripting-enabled: false
    
  network:
    public-address: 172.20.0.3:5702
    port:
      auto-increment: false
      port: 5702
    join:
      multicast:
        enabled: false
      tcp-ip:
        enabled: true
        member-list:
          - 172.20.0.2:5701

  map:
    customers:
      time-to-live-seconds: 0
      eviction:
        eviction-policy: LFU
        max-size-policy: PER_NODE
        size: 1000
```



각각의 서버로 들어가서

```sh
docker exec -it {서버명} /bin/bash
```

필요한 패키지를 설치하고

```sh
apt update && apt install -y maven
```

spring-boot를 가동한다. 

```sh
cd /data/hazelcast
mvn spring-boot:run
```



이후 아래 URL을 차례대로 들어가보면 캐시값이 각 서버간에 잘 연동되는 것을 확인할 수 있다. 

http://t.com:8080/cache/set

http://t.com:8080/cache/get

http://t.com:8081/cache/get




## with Spring

pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>
    <dependency>
        <groupId>com.hazelcast</groupId>
        <artifactId>hazelcast</artifactId>
    </dependency>
    <dependency>
        <groupId>com.hazelcast</groupId>
        <artifactId>hazelcast-spring</artifactId>
    </dependency>
</dependencies>
```



hazelcast.yaml

```yaml
hazelcast:
  cluster-name: hzc-cluster-1
  instance-name: hzc-instance-1
  management-center:
    scripting-enabled: false

  network:
    public-address: 172.0.0.1:5701
    port:
      auto-increment: false
      port: 5701
```



DemoApplication.java

```java
package com.example.demo;

import java.io.FileNotFoundException;

import com.hazelcast.config.ClasspathYamlConfig;
import com.hazelcast.config.Config;
import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.HazelcastInstance;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
@EnableCaching
public class DemoApplication {

	@Bean
	public Config hazelCastConfig() {
		return new ClasspathYamlConfig("hazelcast.yaml");
	}

	@Bean
	public HazelcastInstance hazelcastInstance() throws FileNotFoundException {
		return Hazelcast.newHazelcastInstance(hazelCastConfig());
	}

	public static void main(final String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}

}
```



CacheController.java

```java
package com.example.demo.controller;

import java.util.Collection;

import com.example.demo.service.CacheService;
import com.hazelcast.core.DistributedObject;
import com.hazelcast.core.Hazelcast;
import com.hazelcast.core.HazelcastInstance;
import com.hazelcast.core.IMap;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/cache")
public class CacheController {

    @Autowired
    CacheService cacheService;

    @GetMapping("/get/{id}")
    public String get(@PathVariable String id) {
        return cacheService.get(id);
    }

    @GetMapping("/info")
    public String info() {
        StringBuilder sb = new StringBuilder();

        HazelcastInstance hazelcastInstance = Hazelcast.getHazelcastInstanceByName("hzc-instance-1");

        Collection<DistributedObject> distributedObjects = hazelcastInstance.getDistributedObjects();
        sb.append(distributedObjects);
        for (DistributedObject object : distributedObjects) {
            if (object instanceof IMap) {
                IMap map = hazelcastInstance.getMap(object.getName());
                sb.append("Mapname=" + map.getName()).append("\n");
                map.entrySet().forEach(sb::append);
            }
        }

        return sb.toString();
    }

}
```



CacheService.java

```java
package com.example.demo.service;

import java.util.HashMap;
import java.util.Map;

import org.springframework.cache.annotation.CacheConfig;
import org.springframework.cache.annotation.Cacheable;
import org.springframework.stereotype.Service;

@Service
@CacheConfig(cacheNames = "customers")
public class CacheService {

    private Map<String, String> mapCustomers;

    public CacheService() {
        mapCustomers = new HashMap<String, String>();
        mapCustomers.put("1", "Joe");
        mapCustomers.put("2", "Ali");
        mapCustomers.put("3", "Avi");
    }

    @Cacheable(key="#id")
    public String get(String id) {
        return mapCustomers.get(id);
    }

}
```



## Issues

서버를 Scale-out하게 되면 기존 서버들이 새로운 서버를 알아야 하고, 반대로 새로운 서버도 기존 서버들을 알아야 서로 통신을 할 수 있다. 그러나 서로의 IP를 알 수 있는 방법이 없다면 Scale-out이 힘들어진다. 이 경우 https://github.com/hazelcast/hazelcast-eureka와 같은 Service Discovery 연동이 필요하다. 



# References
- [이제 필요한 것은 In Memory Data Grid](https://d2.naver.com/helloworld/106824) 
- [Hazelcast?](https://supawer0728.github.io/2018/03/11/hazelcast/) 
- [Hazelcast - Local Cache 와 Invalidation Message Propagation 전략을 활용하여 API 성능 튜닝하기](https://pkgonan.github.io/2018/10/hazelcast-hibernate-second-level-cache) 

