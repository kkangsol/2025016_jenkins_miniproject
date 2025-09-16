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


