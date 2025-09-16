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
<br>

**inotify**는 Linux 커널 2.6.13부터 제공하는 이벤트 모니터링 메커니즘입니다.파일이나 디렉토리를 개별적으로 모니터링 할 수 있도록 도와줍니다. <br><br>
이 프로젝트에서는 inotifywait 명령을 사용해 .jar 파일이 변경될 때마다 자동으로 기존 서버를 종료하고, 변경된 서버를 재시작하도록 구성합니다. <br>

<br>

### 2. 감시 스크립트 생성 (호스트)

`~/scripts/watch-jar.sh` 파일 생성

- inotify-tools 기반의 자동 재시작 로직을 담은 실행 스크립트입니다.
- 지정한 .jar 파일을 지속적으로 감시하다가 파일이 변경되면 기존 서버를 종료하고 새로운 .jar를 자동 실행합니다.

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

- **`inotifywait -m -e close_write "$TARGET_JAR"`**  
  지정한 `.jar` 파일에서 `close_write` 이벤트(파일 수정 완료)를 **지속적으로 감시**합니다.  
  파일이 새로 빌드될 때마다 이벤트가 발생하여 이후 로직이 실행됩니다.

- **`lsof -ti tcp:$TARGET_PORT`**  
  :contentReference[oaicite:1]{index=1} 명령으로 현재 지정한 포트(`$TARGET_PORT`)를 사용 중인 **프로세스 PID를 조회**합니다.  
  기존 서버를 정확히 찾아 종료하기 위해 사용됩니다.

- **`kill -15 $OLD_PID` → `sleep 5` → `kill -9 $OLD_PID`**  
  조회된 PID에 먼저 `SIGTERM`(정상 종료 요청)을 보내고 5초 대기한 뒤, 여전히 종료되지 않았다면 `SIGKILL`(강제 종료)로 **완전 종료**시킵니다.

- **`nohup java -jar "$TARGET_JAR" > server.log 2>&1 &`**  
  새로운 `.jar` 파일을 **백그라운드에서 실행**하며, 실행 로그를 `server.log`에 저장합니다.  
  `nohup`을 사용해 터미널 종료와 무관하게 계속 실행되도록 합니다.




```bash
chmod +x ~/scripts/watch-jar.sh
~/scripts/watch-jar.sh &

```
- 이후 chmod 명령어로 실행 권한을 부여하고, 스크립트를 실행해 감시를 시작합니다.

<br><br>



### 3. systemd 서비스 등록

`/etc/systemd/system/watch-jar.service` 생성:

- systemd는 Linux에서 서비스(데몬)들을 관리·자동 실행하는 초기화 시스템입니다.
- /etc/systemd/system 경로는 사용자가 직접 만든 서비스 유닛 파일을 등록하는 공식 경로입니다.
- watch-jar.service 파일을 여기에 만들면, watch-jar.sh를 일반 명령이 아니라 시스템 서비스로 취급할 수 있게 됩니다.

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

> systemd가 watch-jar.sh 스크립트를 ubuntu 사용자 권한으로 실행·관리하도록 설정한 서비스 정의로,
부팅 후 자동 실행되며 종료 시 자동 재시작되도록 구성되어 있습니다.

<br><br>

### 4. 서비스 활성화 및 시작

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now watch-jar.service

```

>systemd에 새 서비스 설정을 반영하고, 부팅 시 자동 실행되도록 등록한 뒤 즉시 실행하는 명령어입니다.

<br><br>


### 5. 상태 및 로그 확인

```bash
systemctl status watch-jar.service
journalctl -u watch-jar.service -f
```

<img width="1013" height="254" alt="image" src="https://github.com/user-attachments/assets/a2f7e06c-2433-462c-9069-38a9c1de880b" />

<br><br>



## ⚡ 동작 흐름

1. GitHub에 코드 푸시합니다.
2. Jenkins(컨테이너)이 `.jar`를 자동으로 빌드해줍니다.
3. 컨테이너 내부 경로(`/var/jenkins_home/...`)에서 호스트 바인드 마운트 경로로 프로젝트 파일이 실시간 반영됩니다.
4. 호스트의 `watch-jar.sh`(systemd 서비스)가 `.jar` 변경 감지합니다.
5. 기존 서버 종료 후 새 `.jar`로 자동 실행합니다.

---

## 트러블슈팅 🔥
### ⚠️ 문제 증상

- `.jar` 변경 시 새 서버가 실행되는데, 이 때 `Port already in use` 에러가 발생합니다.
- 기존 서버가 여전히 포트 점유 중이기 때문에 포트 충돌이 발생합니다.

### ⚡ 원인

- 초기에 사용한 방식은 PID 추적 기반을 사용했습니다:

```bash
CURRENT_PID=""
...
java -jar "$TARGET_JAR" &
CURRENT_PID=$!

```

- `$!`는 **마지막 실행한 백그라운드 프로세스 하나만 추적** 하도록 구성됩니다.
- 하지만 Spring Boot 앱은 내부적으로 여러 스레드/자식 프로세스를 생성하기 때문에 `kill $CURRENT_PID`만으로는 완전 종료가 불가합니다.
- 기존 인스턴스가 그대로 살아있어 포트 충돌이 발생합니다.

### ✅ 해결

- PID 추적 대신 **포트 기반 종료 방식**으로 변경합니다.:

```bash
OLD_PID=$(lsof -ti tcp:$TARGET_PORT)
kill -15 "$OLD_PID" && sleep 5
kill -9 "$OLD_PID"  # 필요 시 강제종료

```

- 항상 포트 점유 프로세스를 확실하게 먼저 종료하고, 새 `.jar`파일을 실행합니다.

---

## 📈 발전 방향 (Improvement Plan)

현재 시스템은 `.jar` 파일 변경 시 자동으로 서버를 재시작하는 데 초점을 맞추고 있습니다.  
실제 운영 환경에서도 안정적으로 사용할 수 있도록, 다음과 같은 방향으로 발전시킬 계획입니다.

- **로그 관리 고도화**  
  현재는 실행 로그가 `server.log` 파일에만 단순히 누적되고 있어, 장기간 운영 시 용량 관리가 어렵습니다.  
  이를 개선하기 위해 :contentReference[oaicite:0]{index=0}를 적용하여 일정 용량 이상이 되면 자동으로 로그를 분리·압축·삭제하도록 구성할 예정입니다.  
  또한 :contentReference[oaicite:1]{index=1}와 :contentReference[oaicite:2]{index=2}를 연동하여 서버 상태, 재시작 이력, 리소스 사용량 등을 시각화함으로써 모니터링 체계를 구축하고자 합니다.

- **무중단 배포 지원**  
  현재는 기존 서버를 종료한 후 새 서버를 실행하는 방식이므로, 재시작 시점에 짧은 서비스 중단이 발생합니다.  
  이를 해결하기 위해 `java -jar` 실행 방식 대신 :contentReference[oaicite:3]{index=3} 컨테이너로 `.jar`를 패키징하고,  
  `docker run`을 활용하여 **Blue-Green 배포 전략**을 적용함으로써 다운타임 없이 새로운 버전으로 전환되도록 개선할 계획입니다.

- **PID 파일 기반 종료로 변경**  
  현재는 포트를 기준으로 실행 중인 프로세스를 찾아 종료하고 있어, 동일 포트를 사용하는 다른 프로세스까지 종료할 위험이 존재합니다.  
  이를 방지하기 위해 서버 실행 시 고유한 `PID 파일`을 생성하고, 스크립트가 이 PID를 직접 참조해 종료하도록 변경하여 **더 안전한 종료 방식**을 구현할 예정입니다.

