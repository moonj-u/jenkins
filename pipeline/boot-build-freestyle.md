# Freestyle Job을 이용한 Spring Boot Build

## Jenkins Job 구성

#### 1. Jenkins Job 생성

- Jenkins 대시보드에서 `새로운 Item`을 클릭하고, `Freestyle project`를 선택하여 새로운 Job을 생성합니다.

#### 2. General

- Git Connection에서 Git을 선택합니다.

![Freestyle Job-1](images/freestyle-boot-1.png)

#### 3. 소스 코드 관리

- 소스 코드 관리 섹션에서 Git을 선택합니다.

- GitLab 저장소 URL과 인증 정보를 선택합니다.

- Branches to build 섹션에서는 소스 코드 관리를 위한 브랜치명을 입력합니다.

![Freestyle Job-1](images/freestyle-boot-2.png)

#### 4. 빌드 환경

- 빌드 환경 섹션에서 `Provide Node & npm bin/ folder to PATH`을 선택합니다.

- Jenkins 관리 >> Tools 에서 설정한 Node.js Version을 선택합니다.

![Freestyle Job-1](images/freestyle-boot-3.png)

#### 5. Build Steps

- Build Steps 섹션에서 Execute shell을 선택합니다.

- 아래의 명령어를 입력하여 의존성 패키지를 설치하고, Vite를 개발 의존성으로 설치한 후, 프로젝트를 빌드합니다.

```
npm install
npm install vite --save-dev
npm run build
```

![Freestyle Job-1](images/freestyle-boot-4.png)
