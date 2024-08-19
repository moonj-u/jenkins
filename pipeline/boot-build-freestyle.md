# Freestyle Job을 이용한 Spring Boot Build

## Jenkins Job 구성

#### 1. Jenkins Job 생성

- Jenkins 대시보드에서 `새로운 Item`을 클릭하고, `Freestyle project`를 선택하여 새로운 Job을 생성합니다.

<br/>

#### 2. General

- Git Connection에서 Git을 선택합니다.

![Freestyle Job-1](images/freestyle-boot-1.png)

<br/>

#### 3. 소스 코드 관리

- 소스 코드 관리 섹션에서 Git을 선택합니다.

- GitLab 저장소 URL과 인증 정보를 선택합니다.

- Branches to build 섹션에서는 소스 코드 관리를 위한 브랜치명을 입력합니다.

![Freestyle Job-1](images/freestyle-boot-2.png)

<br/>

#### 4. Build Steps

- Build Steps 섹션에서 `Invoke Gradle`을 선택합니다.

- Jenkins 관리 >> Tools에서 설정한 Gradle Version을 설정합니다.

- 아래의 명령어를 입력하여 프로젝트를 Clean Build를 진행합니다.

```
clean build
```

![Freestyle Job-1](images/freestyle-boot-3.png)
