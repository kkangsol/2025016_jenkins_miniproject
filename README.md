# Java CI/CD Pipeline with Jenkins & Docker

#### 📝 프로젝트 개요

이 프로젝트는 Gradle로 빌드된 Java 애플리케이션을 **Jenkins와 Docker를 사용하여 자동으로 빌드하고 배포하는 CI/CD 파이프라인을 구축하는 것**을 목표로 합니다. GitHub Webhook을 통해 소스 코드 변경을 감지하고, Jenkins 파이프라인이 이를 자동으로 빌드하여 Docker 컨테이너에서 실행합니다.

---

## 🚀 Jenkins 사용 목적
| logo |
| ------- | 
| <img width="400" alt="image" src="https://github.com/user-attachments/assets/24e73b53-03cd-42cc-bd85-725a51b713bd" /> | 
- **빌드 자동화**: GitHub에 소스 코드가 푸시(Push)될 때마다 Gradle 빌드를 자동으로 실행하여 `.jar` 파일을 생성합니다.
- **배포 자동화**: 빌드된 `.jar` 파일을 Docker 컨테이너에 자동으로 배포하고 실행하여 수동 배포 과정을 없애고 휴먼 에러를 줄입니다.
- **CI/CD 구현**: 소스 코드 통합부터 빌드, 테스트, 배포까지 이어지는 지속적인 통합 및 배포(CI/CD) 환경을 구축하여 개발 생산성을 향상시킵니다. |

## 🚞 ngrok 사용 목적
<img width="1893" height="162" alt="image" src="https://github.com/user-attachments/assets/324daf39-5ffc-4330-a55e-7b7b8a967d53" />

- `ngrok`은 방화벽 뒤에 있는 로컬 서버(내 PC)를 안전한 터널을 통해 **외부 인터넷에 임시로 노출**시켜주는 툴입니다.
- 이번 프로젝트에서는 GitHub Webhook이 Jenkins 서버와 통신할 수 있도록 외부 접속 URL을 생성하기 위해 사용했습니다.

### 사용 방법
| 사진 | 설명 |
| ---- | ----- |
| <img width="1292" height="678" alt="image" src="https://github.com/user-attachments/assets/2d84489c-a708-42ec-a9fd-0b34d658eb5f" /> |1. **ngrok 가입 및 다운로드**<br> [ngrok 공식 홈페이지](https://ngrok.com/)에 접속하여 회원가입 후, 자신의 운영체제에 맞는 버전을 다운로드합니다. |
| <img width="2296" height="266" alt="image" src="https://github.com/user-attachments/assets/88c50b19-2ecb-454a-8bd6-e82fdd16f6b7" />|<br><br>2. **인증 토큰(Authtoken) 연결**<br> ngrok 대시보드에 있는 인증 토큰을 복사하여 아래 명령어를 터미널에 한 번만 실행합니다. <br> `ngrok config add-authtoken [인증토큰]` <br><br>|
|<img width="2914" height="940" alt="image" src="https://github.com/user-attachments/assets/f5a39f29-2e9d-4f00-b452-1d6dba6eade8" />|3. **터널 실행**<br> Jenkins가 8080 포트를 사용하고 있으므로, 아래 명령어를 실행하여 로컬 8080 포트에 대한 터널을 엽니다. <br>`ngrok http 8080` |

---
<br>

## 🛠️ 실행 방법

### 1. Java 프로젝트 생성 (with. Gradle)

Gradle 기반의 간단한 Spring Boot 웹 애플리케이션 프로젝트를 생성합니다.

- get, post 테스트가 가능한 간단한 컨트롤러 코드를 만들어줍니다.

```java
package edu.fisa.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("app")
public class Controller {

	@GetMapping("/get")
	public String getReqRes() {
		return "get 방식 요청의 응답 데이터 : get ";
		
	}
	
	//http://localhost:8282/app/post
	@PostMapping("/post")
	public String getReqRes2() {
		return "post 방식 요청의 응답 데이터 : post";
	}
}
 
```

### 2. Jenkins 컨테이너 빌드 (바인드 마운트)

Jenkins 서버를 Docker 컨테이너로 실행하고, 이때 **바인드 마운트**를 사용하여 Jenkins 내부와 호스트(우분투) 간의 폴더를 연결합니다.

이렇게 설정하는 주된 이유는, Jenkins 컨테이너 안에서 빌드된 `.jar` 파일을 **호스트(우분투)에서도 직접 접근하고 실행**할 수 있도록 하기 위함입니다. 또한, Jenkins의 주요 설정 데이터를 보존하기 위해 `jenkins_home` 볼륨도 함께 사용합니다.

```java
docker run -d   -p 8080:8080   
-v /home/ubuntu/jenkins_home:/var/jenkins_home
--name myjenkins2   jenkins/jenkins:lts-jdk17
```

### 3. Jenkins에서 Pipeline Job 생성

1. Jenkins에 접속하여 **새로운 Item**을 생성하고 **Pipeline**을 선택합니다. (예: `step03_teamArt`)
2. **Pipeline** 탭의 Definition 항목에서 **Pipeline script from SCM**을 선택합니다.
3. **SCM**에서 **Git**을 선택하고, 프로젝트의 GitHub 저장소 URL을 입력합니다.

| 1 | 2 | 3 |
| -- | -- | -- |
| <img width="1472" height="558" alt="image" src="https://github.com/user-attachments/assets/d723d5fd-3c74-45c9-87e4-56aac3a7ce76" />|<img width="2564" height="580" alt="image" src="https://github.com/user-attachments/assets/65d149c4-4dfa-4310-94ad-470400bf3d23" /> | <img width="922" height="333" alt="image" src="https://github.com/user-attachments/assets/f6c64476-0750-49a6-a4a8-a794f48c605b" /> |

4. pipeline 스크립트 파일 작성
```shell
pipeline {
    agent any
    environment {
        GITHUB_REPO = '{git repository url}'
        BRANCH_NAME = 'main'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: "${BRANCH_NAME}", url: "${GITHUB_REPO}"
                
                echo '**********'
                sh 'ls -al'  
                echo '**********'
            }
        }

        stage('Build') {
            steps {
                script {
                    if (fileExists('gradlew')) {
                        sh 'chmod +x gradlew'
                        sh './gradlew build'
                    } else if (fileExists('pom.xml')) {
                        sh 'mvn clean package'   
                    } else {
                        error 'Gradle 또는 Maven 프로젝트가 아님'
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ 빌드 성공!'
        }
        failure {
            echo '❌ 빌드 실패! 오류 확인 필요!'
        }
    }
}
```

### 4. 파이프라인 실행 및 JAR 파일 확인

1. GitHub 저장소에 코드를 푸시하여 빌드를 트리거하거나, Jenkins에서 **Build Now**를 클릭합니다.

| 빌드 시작 | 빌드 성공 |
| ------- | ------- |
|<img width="1871" height="722" alt="image" src="https://github.com/user-attachments/assets/6ba8e36f-e762-4b18-a5d4-af32ae8ce603" />|<img width="1432" height="558" alt="image" src="https://github.com/user-attachments/assets/bb6b3566-ecc6-49a3-879a-c76ef363fdef" />|
2. 파이프라인이 성공적으로 완료되면, `.jar` 파일이 Jenkins 서버(호스트)의 지정된 마운트 폴더에 생성됩니다.
3. 아래 명령어로 Jenkins를 실행하고 있는 컨테이너로 진입합니다.
- `docker exec -it {컨테이너 이름 or 컨테이너 ID} bash`
4. 정상적으로 자바 프로젝트가 동작하는지 최종 확인합니다.
- `java -jar {jar 파일 이름}`
- `curl http://localhost:8282/app/get`

| 결과 사진 |
| ------- |
| <img height="450" alt="image" src="https://github.com/user-attachments/assets/d8d74608-0ae5-4cf9-b9fe-e20004c5330c" />|
| <img height="40" alt="image" src="https://github.com/user-attachments/assets/d9cae6b3-d6e4-4d0d-bf32-650d9bac3e59" />|

---

# 자동실행 구현
### 🏗️ 전체 구조도

```
[GitHub Push]
     ↓ (Webhook)
[Jenkins 컨테이너]
 ──────────────┐
  └─ 코드 빌드 → /var/jenkins_home/workspace/.../build/libs/step04_gradleBuild-0.0.1-SNAPSHOT.jar
                 (호스트에 **바인드 마운트**로 공유됨)
                               │
                               ▼
             [호스트 머신 - systemd 서비스로 등록된 watch-jar.sh]
                               │
                ├─ 기존 서버 종료 (포트 기반)
                └─ 새로운 jar 자동 실행 (java -jar)

```

- Jenkins 컨테이너는 `.jar`를 컨테이너 내부에 생성
- 이 경로가 **호스트 디렉터리에 바인드 마운트**되어 있어서
    
    호스트에서도 동일 경로로 `.jar` 파일 접근 가능
    
- 호스트에서 `inotifywait` 기반 스크립트가 변경을 감지해 자동 실행



## 📦 설치 및 설정

### 1. inotify-tools 설치 (호스트)

```bash
sudo apt-get update && sudo apt-get install -y inotify-tools

```


### 2. 감시 스크립트 생성 (호스트)

`~/scripts/watch-jar.sh` 파일 생성:

```bash
#!/bin/bash
TARGET_JAR="/호스트/바인드경로/step03_teamArt/build/libs/step04_gradleBuild-0.0.1-SNAPSHOT.jar"
TARGET_PORT=8282

inotifywait -m -e close_write "$TARGET_JAR" | while read path action file; do
    echo "🔁 JAR 변경 감지: $file ($action)"

    # 1. 기존 서버 종료 (포트 기반)
    OLD_PID=$(lsof -ti tcp:$TARGET_PORT)
    if [ -n "$OLD_PID" ]; then
        echo "🛑 포트 $TARGET_PORT 사용 중인 PID: $OLD_PID → 종료 중"
        kill -15 "$OLD_PID"
        sleep 5
        if kill -0 "$OLD_PID" 2>/dev/null; then
            echo "⚠️ 정상 종료 실패 → 강제 종료"
            kill -9 "$OLD_PID"
        fi
        echo "✅ 기존 서버 종료 완료"
    fi

    # 2. 새 서버 실행
    echo "🚀 새로운 JAR 실행 시작"
    nohup java -jar "$TARGET_JAR" > server.log 2>&1 &
    echo "🌟 새 서버 PID: $!"
done

```

```bash
chmod +x ~/scripts/watch-jar.sh

```

- `TARGET_JAR` : 호스트 기준 바인드 마운트 경로
- `TARGET_PORT` : Spring Boot 애플리케이션 포트

---

### 3. systemd 서비스 등록

`/etc/systemd/system/watch-jar.service` 생성:

```bash
sudo nano /etc/systemd/system/watch-jar.service

```

```
[Unit]
Description=Watch JAR and restart on change
After=network.target

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu
ExecStart=/home/ubuntu/scripts/watch-jar.sh
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target

```

> User=ubuntu, ExecStart 경로는 실제 계정과 경로에 맞게 변경
> 


### 4. 서비스 활성화 및 시작

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now watch-jar.service

```

- 부팅 시 자동 시작
- 지금 즉시 실행됨

---

### 5. 상태 및 로그 확인

```bash
systemctl status watch-jar.service
journalctl -u watch-jar.service -f
```

## ⚡ 동작 흐름

1. GitHub에 코드 푸시
2. Jenkins(컨테이너)이 `.jar` 빌드
3. 컨테이너 내부 경로(`/var/jenkins_home/...`) → 호스트 바인드 마운트 경로로 실시간 반영
4. 호스트의 `watch-jar.sh`(systemd 서비스)가 `.jar` 변경 감지
5. 기존 서버 종료 후 새 `.jar`로 자동 실행

---

## 트러블슈팅 🔥
### ⚠️ 문제 증상

- `.jar` 변경 시 새 서버 실행 → `Port already in use` 에러 발생
- 기존 서버가 여전히 포트 점유 중

### ⚡ 원인

- 초기에 사용한 방식은 PID 추적 기반:

```bash
CURRENT_PID=""
...
java -jar "$TARGET_JAR" &
CURRENT_PID=$!

```

- `$!`는 **마지막 실행한 백그라운드 프로세스 하나만 추적**
- 하지만 Spring Boot 앱은 내부적으로 여러 스레드/자식 프로세스 생성 → `kill $CURRENT_PID`만으로는 완전 종료 안 됨
- 기존 인스턴스가 그대로 살아있어 포트 충돌 발생

### ✅ 해결

- PID 추적 대신 **포트 기반 종료 방식**으로 변경:

```bash
OLD_PID=$(lsof -ti tcp:$TARGET_PORT)
kill -15 "$OLD_PID" && sleep 5
kill -9 "$OLD_PID"  # 필요 시 강제종료

```

- 항상 포트 점유 프로세스를 먼저 종료 → 새 `.jar` 실행

---

## 📈 발전 방향 (Improvement Plan)

- **로그 관리 고도화**
    - logrotate로 `server.log` 용량 관리
    - Prometheus + Grafana 로 서버 상태 모니터링
- **무중단 배포 지원**
    - `docker run`으로 `.jar` 컨테이너화 → Blue-Green 배포 적용
- **PID 파일 기반 종료로 변경**
    - 포트가 아닌 PID 파일을 직접 관리해 더 안전한 종료 보장
