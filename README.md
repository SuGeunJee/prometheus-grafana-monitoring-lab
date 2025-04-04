# prometheus-grafana-monitoring-lab

<br>

## 🤝 Team Members
| <img src="https://github.com/SuGeunJee.png" width="200px"> | <img src="https://github.com/HyunDooBoo.png" width="200px"> | <img src="https://github.com/wild-turkey.png" width="200px"> | <img src="https://github.com/unoYoon.png" width="200px"> |
| :---: | :---: | :---: | :---: |
| [지수근](https://github.com/SuGeunJee) | [오현두](https://github.com/HyunDooBoo) | [김지훈](https://github.com/wild-turkey) | [윤원호](https://github.com/unoYoon) |

<br>

## 📝프로젝트 개요
인프라 가용성과 시스템 안정성을 검증하기 위한 모니터링 환경을 구축하고, 

실제 부하 테스트를 통해 Prometheus-Grafana 기반의 실시간 트래픽 분석 체계를 구현하였습니다.

<br>

## 🎯프로젝트 목표

모니터링 환경 구축과 부하 테스트를 통해, 장애 발생 전 조기 감지 체계를 마련하고 

예기치 못한 시스템 중단을 방지함으로써 운영 안정성을 확보하고, 

리소스 낭비 없는 효율적 인프라 운영으로 비용 절감을 도모합니다.

<br>

## 🛠️프로젝트 과정

### ✅ 프로젝트 수행 순서



### 1-1. 서버 재부팅

    
```
    sudo reboot
```
    

    
### 1-2. 재부팅 후 접속해서 상태 확인

    - `mysqld_exporter`, `node_exporter`, `prometheus`, `grafana` 모두 **자동 실행되었는지 확인**
        

```
    systemctl status mysqld_exporter
    systemctl status prometheus
    systemctl status grafana-server
```
        

### 1-3. Grafana 대시보드 접속

    - http://localhost:3000 으로 접속
    - MySQL, Node, Prometheus 관련 대시보드가 **정상 표시되는지 확인**


    ![image](https://github.com/user-attachments/assets/96b52683-dd86-40c3-9784-3e8633161b30)


 - 재부팅하였는데 서비스를 주지 않은 9104 mysql-exporter가 비정상된 것을 확인하였습니다.


 ### 1-4. mysqld_exporter가 systemd 서비스로 등록되어 있는지 확인하세요.


```
sqld_exporter
```


만약 inactive라면 서비스를 다시 시작합니다.


```
sudo systemctl restart mysqld_exporter
```


만약 서비스가 아예 없다면, 새로 등록해야 합니다.

/etc/systemd/system/mysqld_exporter.service 파일을 만들고 다음을 추가하세요.


```
Description=MySQL Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=mysql
Group=mysql
Type=simple
ExecStart=/usr/local/bin/mysqld_exporter --config.my-cnf=/etc/.mysqld_exporter.cnf
Restart=always

[Install]
WantedBy=multi-user.target
```


이후 활성화 및 실행:



```
sudo systemctl daemon-reload
sudo systemctl enable --now mysqld_exporter
```


 5. Prometheus 설정 다시 확인


-   prometheus.yml에서 mysqld_exporter가 올바르게 설정되었는지 확인합니다.


```
scrape_configs:
  - job_name: 'mysql-exporter'
    static_configs:
      - targets: ['localhost:9104']
```

이후 Prometheus를 재시작합니다.



```
sudo systemctl restart prometheus
```



![image](https://github.com/user-attachments/assets/3f459d00-2496-4209-9960-9e18a916fd25)

Restart=always와 WantedBy=multi-user.target 덕분에 자동 재시작 및 부팅 시 자동 실행 가능하였습니다.

![image](https://github.com/user-attachments/assets/0cfdfaff-a794-4a8d-a6ac-f5bbb4c02f32)

service 설정을 통해 살아남을 확인하였습니다.

---

### 2-1  Linux CPU 부하 테스트


---


### 2-2. MySQL 부하 테스트


---


### 2-3. Spring App 부하 테스트

<br>

#### 1. Spring 애플리케이션을 서버에 배포
   
    - WAR or JAR 형태 실행

    ```
    java -jar my-spring-app.jar
    ```
    

#### 2. Prometheus에서 Spring App 데이터 수집 연동


```
management:
  endpoints:
    web:
      exposure:
        include: "prometheus,health,metrics"
  prometheus:
    metrics:
      export:
        enabled: true   
```


✅ Spring Actuator의 엔드포인트 노출 설정

    Prometheus가 수집할 수 있도록 /actuator/prometheus, /actuator/health, /actuator/metrics 엔드포인트를 외부에 노출 가능하게 합니다.


✅ Prometheus용 메트릭 export 설정

    Prometheus가 메트릭 데이터를 수집할 수 있도록 활성화합니다.



#### 3. Spring Boot 앱이 노출한 메트릭을 수집

```
scrape_configs:
  - job_name: 'spring-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8089']  # or 'IP:PORT' if remote
```


Prometheus가 localhost:8089/actuator/prometheus에 주기적으로 접속해서, 

Spring Boot 앱이 노출한 메트릭을 수집하도록 설정합니다.


---

#### 4. localhost:8089에 계속 HTTP 요청을 보내서 트래픽(부하)를 발생


```
while true; do curl -s http://localhost:8089 > /dev/null; done
```



![image](https://github.com/user-attachments/assets/da7be7c7-6a7a-421d-8ac7-f58a0107065c)



그라파나 대시보드를 통해 부하 증가 상황을 실시간으로 시각화하여 모니터링할 수 있습니다.
<br>


## 💥TrobleShooting

### SpringApp을 프로메테우스에서 확인할 때 404 Error 발생

#### 문제 상황

![image](https://github.com/user-attachments/assets/ac086ce7-ff40-4987-86c6-5240de9799ae)



Spring 애플리케이션을 Prometheus에서 모니터링하려고 시도했으나, /actuator/prometheus 엔드포인트에 접근 시 404 오류가 발생했습니다.

#### 해결 방법

```
  - job_name: 'spring-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8089/fisa1']
```

위와 같이 targets을 localhost:8089/fisa1로 설정하였습니다. 

하지만 targets는 IP:PORT 또는 호스트:PORT만 써야 하는 것을 확인하였습니다.

이를 해결하기에 jar 파일에 CicdController.java 부분을 수정하고, jar 파일을 다시 빌드하고 배포하였습니다. 

- 문제 해결 전

```
package edu.fisa.ce.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
public class CicdController {

    // http://localhost:8089/fisa
    @GetMapping("/fisa")
    public String reqRes() {
        System.out.println("reqRes() *******");
        log.info("***** 요청 & 응답 *****");
        return "요청 응답 성공";
    }
}

```

- 문제 해결 후

```
package edu.fisa.ce.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import lombok.extern.slf4j.Slf4j;

@Slf4j
@RestController
public class CicdController {

    // http://localhost:8089/
    @GetMapping("/")
    public String reqRes() {
        System.out.println("reqRes() *******");
        log.info("***** 요청 & 응답 *****");
        return "요청 응답 성공";
    }
}
```


```
  - job_name: 'spring-app'
    metrics_path: '/actuator/prometheus'
    static_configs:
      - targets: ['localhost:8089']
```


![image](https://github.com/user-attachments/assets/339b1983-a5a2-4755-9ebd-f34f427aa3e4)


spring-app이 프로메테우스에서 정상적으로 up 상태임을 확인하고, 그라파나 대시보드로 모니터링하였습니다.