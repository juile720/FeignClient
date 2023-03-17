### Feign Client 사용


1. build.gradle<br/>
- 아래 내용을 dependencies에 추가
```
implementation 'org.springframework.cloud:spring-cloud-starter-openfeign'
implementation 'org.springframework.cloud:spring-cloud-starter-config' 
implementation 'org.springframework.cloud:spring-cloud-starter-kubernetes-all:1.1.7.RELEASE'
```
- 아래 내용을 추가
```
dependencyManagement {
	imports {
		mavenBom "org.springframework.cloud:spring-cloud-dependencies:2021.0.1"
	}
}
```

<br/>

2. App Application.java 파일에서  class 상단에 아래를 추가
```java
@EnableFeignClients
@EnableDiscoveryClient
```

<br/>

3. feign폴더를 생성하여 그 안에 feign을 구현할 interface를 생성

```java
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

@FeignClient(name = "demo-app02")
public interface UserServiceFeignClient {
    @PostMapping("/hello")
    String hello();
}

```
<br/>

4. 아래처럼 service파일에서 feign을 호출함
```java
public String hello() {
        return userServiceFeignClient.hello();
}
```
<br/>

5. application.yaml 파일에 아래 내용 추가
```yaml
spring:
  cloud:
    kubernetes:
      loadbalancer:
        mode: service
```


### Forbidden!Configured service account doesn't have access. Service account may have been revoked 오류 발생 시
: 해당 오류는 어플리케이션의 Service Account가 openshift 자원 조회 서비스에 접근 권한이 없기 때문에 발생하는 문제이므로,  <br/>
  어플리케이션의 Service account(default)에 관련 Role을 추가해 주어야 함 (6. 번 내용 진행)
<br/>

6. k8s 폴더 안에 아래 두 파일을 추가 시킴
- role.yaml파일을 생성한 후 아래 내용을 기입

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  # Reference to upper's `metadata.name`
  name: default #service account name로, 기본값은 default
  # Reference to upper's `metadata.namespace`
  namespace: default #k8s namespace
rules:
  - apiGroups: [ "" ,"apps" ]
    resources: [ "nodes","deployments","configmaps", "pods", "services", "endpoints", "secrets" ]
    verbs: [ "get", "list", "watch" ]
```
- 터미널에서 아래 명령어를 실행시켜 파일을 apply 시킴(해당 파일이 위치한 경로에서)

```
kubectl apply -f role.yaml
```

- role-binding.yaml파일을 생성한 후 아래 내용을 기입 <br/>
 : 생성한 Service Account와 Role을 서로 Binding하는 작업
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: default
  namespace: default
subjects:
  - kind: ServiceAccount
    name: default #service account name로, 기본값은 default
    namespace: default #k8s namespace
roleRef:
  kind: Role
  name: default
  apiGroup: rbac.authorization.k8s.io
```

- 터미널에서 아래 명령어를 실행시켜 파일을 apply 시킴(해당 파일이 위치한 경로에서)

```
kubectl apply -f role-binding.yaml
```

<br/>


### 참조 사이트
- https://peterconrey.medium.com/spring-boot-microservice-communication-on-kubernetes-with-feign-clients-69e2cb267c35


