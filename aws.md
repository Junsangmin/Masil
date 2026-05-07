# 클라우드 / AWS 담당 역할 정리

## 한 줄 정의

> 로컬에서만 동작하는 프로젝트를 실제 사용자가 접속 가능한 AWS 기반 서비스로 배포하고, 서버·DB·보안·로그·운영 구조를 구성하는 역할

AWS 담당은 단순히 “EC2 하나 띄우는 역할”이 아니다.  
이 프로젝트가 AWS 수업 과제인 만큼, **서비스가 클라우드 환경에서 실제로 동작한다는 점을 증명하는 역할**을 맡는다.

즉, AWS 담당은 아래 흐름을 담당한다.

```text
프론트엔드 빌드
 ↓
백엔드 서버 배포
 ↓
DB 인프라 구성
 ↓
환경변수 / API Key 관리
 ↓
보안그룹 / IAM / 접근권한 설정
 ↓
로그 확인
 ↓
AWS 아키텍처 정리
```

---

# 1. AWS 담당의 핵심 역할

AWS 담당은 프론트엔드, 백엔드, DB가 로컬 개발 환경이 아니라  
AWS 클라우드 위에서 실제로 동작할 수 있도록 만든다.

## 핵심 목표

```text
1. 사용자가 웹사이트에 접속할 수 있어야 한다.
2. 프론트엔드가 백엔드 API를 호출할 수 있어야 한다.
3. 백엔드가 외부 API와 DB에 접근할 수 있어야 한다.
4. API Key와 DB 정보가 안전하게 관리되어야 한다.
5. 서버 로그와 오류를 확인할 수 있어야 한다.
6. 발표 때 AWS 구조를 명확히 설명할 수 있어야 한다.
```

---

# 2. 담당 범위

## 1) 전체 AWS 아키텍처 설계

AWS 담당은 프로젝트가 어떤 구조로 배포되는지 설계한다.

## 기본 아키텍처 예시

```text
사용자
 ↓
Frontend
S3 또는 EC2 배포
 ↓
Backend API Server
EC2 배포
 ↓
External APIs
ODsay / TMAP / Naver Maps
 ↓
Database
RDS 또는 DynamoDB
 ↓
Monitoring
CloudWatch Logs
```

---

## 발표용 아키텍처 예시

```text
[User]
   ↓
[Frontend - React]
   ↓
[Backend API Server - EC2]
   ↓
[AI Recommendation Module]
   ↓
[External Transit API]
   - ODsay / TMAP / Naver Maps
   ↓
[Database]
   - RDS or DynamoDB
   ↓
[CloudWatch]
   - Server Logs / Error Logs
```

---

# 3. 프론트엔드 배포

프론트엔드 담당자가 만든 웹 화면을 실제 접속 가능한 주소로 배포한다.

## 선택지 1: S3 정적 웹 호스팅

React/Vite 같은 정적 프론트엔드라면 S3에 빌드 파일을 올려서 배포할 수 있다.

### AWS 담당이 해야 할 일

```text
1. 프론트엔드 빌드 파일 생성 확인
2. S3 버킷 생성
3. 정적 웹사이트 호스팅 설정
4. 빌드 결과물 업로드
5. 퍼블릭 접근 정책 설정
6. 접속 URL 확인
```

---

## 선택지 2: EC2에서 프론트엔드 실행

프론트엔드를 백엔드와 같은 EC2에서 실행할 수도 있다.

### AWS 담당이 해야 할 일

```text
1. EC2 인스턴스 생성
2. Node.js 설치
3. 프론트엔드 코드 배포
4. npm install / npm run build
5. 정적 파일 서빙 설정
6. 백엔드 API 주소 연결 확인
```

---

## 추천

시간이 부족하면 아래 구조가 가장 단순하다.

```text
프론트엔드: S3 정적 호스팅
백엔드: EC2 배포
DB: RDS 또는 DynamoDB
로그: CloudWatch
```

---

# 4. 백엔드 서버 배포

백엔드 담당자가 만든 API 서버를 AWS EC2에 배포한다.

## AWS 담당이 해야 할 일

```text
1. EC2 인스턴스 생성
2. 보안그룹 설정
3. SSH 접속 설정
4. Node.js 또는 Python 실행 환경 설치
5. 백엔드 코드 업로드
6. .env 환경변수 설정
7. 서버 실행
8. 외부에서 API 접근 가능한지 확인
```

---

## 보안그룹 설정 예시

```text
22번 포트: SSH 접속용
80번 포트: HTTP 접속용
443번 포트: HTTPS 접속용
백엔드 포트: 예: 3000, 5000, 8000
```

단, 개발 단계에서는 백엔드 포트를 직접 열 수 있지만,  
최종 발표용으로는 가능하면 80번 포트 또는 리버스 프록시 구조를 쓰는 것이 좋다.

---

## 서버 실행 관리

단순히 터미널에서 서버를 실행하면 SSH 접속이 끊겼을 때 서버도 종료될 수 있다.

가능하면 아래 중 하나를 사용한다.

```text
pm2
systemd
nohup
screen / tmux
```

Node.js 백엔드라면 `pm2`를 사용하는 것이 비교적 간단하다.

```bash
pm2 start server.js
pm2 status
pm2 logs
```

---

# 5. DB 인프라 구성

DB는 AWS 담당과 백엔드 담당이 함께 다루지만, 역할이 다르다.

## 역할 분리

```text
AWS 담당: DB 인프라를 만든다.
백엔드 담당: DB를 코드에서 사용한다.
AI 담당: 어떤 데이터를 저장하면 좋은지 정의한다.
```

---

## AWS 담당이 하는 일

```text
1. RDS 또는 DynamoDB 선택
2. DB 인스턴스 또는 테이블 생성
3. DB 이름 / 유저 / 비밀번호 설정
4. 보안그룹 설정
5. EC2에서 DB 접속 가능하도록 허용
6. DB 접속 정보 정리
7. .env에 들어갈 정보 백엔드 담당에게 전달
```

---

## 백엔드 담당이 하는 일

```text
1. DB 연결 코드 작성
2. 검색 기록 저장
3. 사용자 요청 저장
4. 추천 결과 저장
5. 저장된 기록 조회 API 구현
```

---

## 저장할 데이터 예시

```text
출발지
도착지
사용자 자연어 요청
분석된 선호도
추천된 경로 ID
추천 이유
검색 시간
```

---

# 6. RDS vs DynamoDB 선택

## RDS를 쓰는 경우

RDS는 MySQL, PostgreSQL 같은 관계형 DB를 사용할 때 적합하다.

### 장점

```text
SQL 사용 가능
테이블 구조가 직관적
수업 과제에서 설명하기 쉬움
검색 기록 저장에 적합
```

### 단점

```text
초기 설정이 DynamoDB보다 번거로울 수 있음
보안그룹과 접속 설정을 신경 써야 함
```

---

## DynamoDB를 쓰는 경우

DynamoDB는 AWS의 NoSQL 데이터베이스다.

### 장점

```text
서버리스 형태로 사용 가능
AWS 서비스 활용도가 높아 보임
간단한 검색 기록 저장에 적합
스키마 변경이 유연함
```

### 단점

```text
처음 쓰면 개념이 어색할 수 있음
복잡한 조회에는 RDS보다 불편할 수 있음
```

---

## 추천

이번 프로젝트에서는 둘 중 하나만 쓰면 충분하다.

```text
SQL에 익숙하면 → RDS
AWS 서비스 활용도를 더 보여주고 싶으면 → DynamoDB
```

검색 기록 저장 정도만 한다면 DynamoDB도 괜찮다.  
하지만 팀원이 SQL에 익숙하다면 RDS가 더 설명하기 쉽다.

---

# 7. 환경변수 / API Key 관리

AWS 담당은 API Key와 DB 접속 정보가 코드에 직접 노출되지 않도록 관리해야 한다.

## 관리해야 할 값

```text
ODsay API Key
TMAP API Key
Naver Maps API Key
DB Host
DB User
DB Password
DB Name
Server Port
```

---

## 잘못된 방식

```js
const API_KEY = "실제_API_KEY_값";
```

이렇게 코드에 직접 넣으면 안 된다.

---

## 권장 방식

`.env` 파일을 사용한다.

```env
ODsay_API_KEY=your_key_here
NAVER_CLIENT_ID=your_client_id_here
NAVER_CLIENT_SECRET=your_client_secret_here
DB_HOST=your_db_host
DB_USER=your_db_user
DB_PASSWORD=your_db_password
DB_NAME=your_db_name
PORT=3000
```

---

## AWS 담당의 역할

```text
1. EC2 서버에 .env 파일 생성
2. 백엔드 담당이 필요한 환경변수 목록 정리
3. API Key가 GitHub에 올라가지 않도록 확인
4. .gitignore에 .env 포함 여부 확인
5. 배포 환경에서 환경변수 정상 적용 확인
```

---

# 8. 보안그룹 설정

AWS 담당은 외부 접근이 필요한 포트만 열어야 한다.

## 기본 원칙

```text
필요한 포트만 연다.
DB는 외부 전체에 열지 않는다.
API Key와 DB 비밀번호는 코드에 노출하지 않는다.
```

---

## 보안그룹 예시

### EC2 보안그룹

```text
SSH: 22번 포트
HTTP: 80번 포트
HTTPS: 443번 포트
Backend: 3000 또는 8000번 포트
```

---

### RDS 보안그룹

```text
DB 포트: 3306 또는 5432
허용 대상: EC2 보안그룹
```

즉, RDS는 모든 인터넷에서 접속 가능하게 열지 않고,  
백엔드 서버가 있는 EC2에서만 접속 가능하게 만드는 것이 좋다.

---

# 9. CloudWatch 로그

AWS 담당은 서버 상태와 에러를 확인할 수 있도록 로그 구조를 잡는다.

## 최소 목표

```text
서버가 정상 실행 중인지 확인
API 요청이 들어오는지 확인
에러 로그 확인
외부 API 호출 실패 여부 확인
```

---

## 가능한 방식

### 간단한 방식

```text
EC2에서 pm2 logs 또는 서버 로그 확인
```

### 조금 더 AWS다운 방식

```text
CloudWatch Logs로 서버 로그 전송
```

---

## 발표에서 설명할 수 있는 내용

```text
CloudWatch를 통해 서버 실행 로그와 에러 로그를 확인할 수 있도록 구성했습니다.
이를 통해 API 장애나 외부 API 호출 실패를 추적할 수 있습니다.
```

---

# 10. 백엔드와 AWS의 역할 경계

AWS 담당과 백엔드 담당은 반드시 역할을 나눠야 한다.

## 백엔드 담당

```text
서버 코드 작성
API 엔드포인트 구현
외부 API 호출 코드 작성
DB 저장/조회 코드 작성
AI 알고리즘 모듈 연결
```

---

## AWS 담당

```text
EC2 생성
서버 실행 환경 구성
보안그룹 설정
RDS/DynamoDB 생성
환경변수 설정
배포 주소 관리
로그 확인
AWS 아키텍처 작성
```

---

## 한 문장 정리

```text
백엔드는 서버를 만든다.
AWS는 서버가 실제 클라우드에서 돌아가게 만든다.
```

---

# 11. 프론트엔드와 AWS의 역할 경계

## 프론트엔드 담당

```text
React 화면 구현
입력 폼 구현
결과 카드 UI 구현
백엔드 API 호출 코드 작성
```

---

## AWS 담당

```text
프론트엔드 빌드 파일 배포
S3 또는 EC2에 업로드
접속 가능한 URL 제공
CORS 문제 확인 지원
```

---

## 한 문장 정리

```text
프론트엔드는 화면을 만든다.
AWS는 그 화면을 실제 접속 가능한 주소에 올린다.
```

---

# 12. AI & 알고리즘과 AWS의 역할 경계

## AI & 알고리즘 담당

```text
자연어 분석
가중치 변환
경로 점수화
추천 이유 생성
```

---

## AWS 담당

```text
AI 로직이 포함된 백엔드가 배포 환경에서 정상 실행되도록 지원
필요한 API Key와 환경변수 관리
배포 후 오류 로그 확인
```

---

## 한 문장 정리

```text
AI 담당은 추천 로직을 만든다.
AWS 담당은 그 로직이 서버에서 안정적으로 실행되게 배포 환경을 만든다.
```

---

# 13. AWS 최소 구현 범위

시간이 부족하면 아래 기능은 반드시 구현해야 한다.

## 필수 구현

```text
1. EC2 인스턴스 생성
2. 백엔드 서버 배포
3. 보안그룹 설정
4. 환경변수 설정
5. 프론트엔드 접속 주소 제공
6. 백엔드 API 외부 접근 확인
7. DB 사용 시 RDS 또는 DynamoDB 구성
8. 서버 로그 확인
9. AWS 아키텍처 다이어그램 작성
```

---

# 14. AWS 추가 구현 범위

시간이 남으면 아래 기능을 추가한다.

## 선택 구현

```text
1. S3 정적 웹 호스팅
2. CloudFront 연결
3. RDS 구성
4. DynamoDB 구성
5. CloudWatch Logs 연동
6. API Gateway 연결
7. Lambda 일부 기능 분리
8. Route 53 도메인 연결
9. HTTPS 적용
10. 배포 자동화 스크립트 작성
```

---

# 15. 가장 현실적인 AWS 구성

이번 프로젝트에서 가장 현실적인 구조는 아래와 같다.

```text
Frontend: S3 정적 호스팅
Backend: EC2
Database: RDS 또는 DynamoDB
Logs: CloudWatch
External API: ODsay / TMAP / Naver Maps
```

---

## 간단한 구조

```text
[User]
  ↓
[S3 Frontend]
  ↓
[EC2 Backend API]
  ↓
[ODsay / TMAP / Naver Maps API]
  ↓
[RDS or DynamoDB]
  ↓
[CloudWatch Logs]
```

---

# 16. 더 단순한 MVP 구조

시간이 부족하면 아래처럼 해도 된다.

```text
[User]
  ↓
[EC2]
  - Frontend
  - Backend
  ↓
[External Transit API]
  ↓
[Local JSON or Simple DB]
```

다만 AWS 과제라는 점을 고려하면, 가능하면 DB는 RDS 또는 DynamoDB를 쓰는 것이 좋다.

---

# 17. AWS 담당의 체크리스트

## 초기 세팅

```text
AWS 계정 접속 가능 여부 확인
사용할 리전 정하기
EC2 생성
키페어 생성 및 보관
보안그룹 생성
SSH 접속 확인
Node.js 또는 Python 설치
Git 설치
```

---

## 백엔드 배포

```text
GitHub에서 코드 clone
npm install 또는 pip install
.env 파일 생성
서버 실행
API 접속 테스트
프론트엔드에서 API 호출 테스트
```

---

## 프론트엔드 배포

```text
프론트엔드 빌드
S3 버킷 생성
정적 웹 호스팅 활성화
빌드 파일 업로드
백엔드 API 주소 연결
브라우저에서 접속 확인
```

---

## DB 연결

```text
RDS 또는 DynamoDB 생성
접속 정보 정리
EC2에서 DB 접근 확인
백엔드 DB 연결 확인
검색 기록 저장 테스트
```

---

## 로그 확인

```text
서버 실행 로그 확인
API 요청 로그 확인
에러 로그 확인
외부 API 실패 로그 확인
```

---

# 18. AWS 담당이 백엔드 담당에게 받아야 할 것

AWS 담당은 백엔드 담당에게 아래 정보를 받아야 한다.

```text
서버 실행 명령어
사용 포트
필요한 환경변수 목록
필요한 API Key 목록
DB 종류
DB 테이블 또는 컬렉션 구조
서버 시작 파일 이름
배포 후 테스트할 API 경로
```

---

## 예시

```text
서버 실행 명령어: npm start
사용 포트: 3000
헬스체크 API: GET /api/health
추천 API: POST /api/recommend
필요 환경변수: ODSAY_API_KEY, DB_HOST, DB_USER, DB_PASSWORD
```

---

# 19. AWS 담당이 프론트엔드 담당에게 받아야 할 것

```text
프론트엔드 실행 명령어
빌드 명령어
빌드 결과물 폴더명
백엔드 API 주소 설정 위치
환경변수 사용 여부
```

---

## 예시

```text
실행 명령어: npm run dev
빌드 명령어: npm run build
빌드 결과물: dist
백엔드 API 주소: VITE_API_BASE_URL
```

---

# 20. AWS 담당이 AI 담당에게 확인해야 할 것

```text
AI 알고리즘이 별도 API Key를 사용하는지
LLM API를 사용하는지
추천 로직이 백엔드 내부 함수인지
추가 Python 패키지가 필요한지
응답 시간이 오래 걸릴 가능성이 있는지
```

---

## 예시

```text
AI 로직: 백엔드 내부 모듈로 실행
LLM 사용 여부: 1차 MVP에서는 사용하지 않음
추가 패키지: 없음
응답 시간: 1초 이내 예상
```

---

# 21. 에러 대응

AWS 담당은 배포 후 발생할 수 있는 대표 오류를 확인해야 한다.

## 대표 오류

```text
프론트엔드에서 백엔드 API 호출 실패
CORS 오류
EC2 서버 종료
보안그룹 포트 미개방
환경변수 누락
API Key 누락
DB 접속 실패
RDS 보안그룹 오류
외부 API 호출 실패
서버 포트 충돌
```

---

## 대응 예시

### API 호출 실패

```text
1. 백엔드 서버가 실행 중인지 확인
2. EC2 보안그룹에서 포트가 열려 있는지 확인
3. 프론트엔드 API 주소가 올바른지 확인
4. CORS 설정 확인
```

---

### DB 접속 실패

```text
1. RDS가 실행 중인지 확인
2. DB 주소와 포트 확인
3. 보안그룹에서 EC2 접근 허용 여부 확인
4. .env의 DB 정보 확인
5. 백엔드 DB 연결 코드 확인
```

---

### API Key 오류

```text
1. .env 파일에 API Key가 들어 있는지 확인
2. 서버가 환경변수를 제대로 읽는지 확인
3. API Key가 GitHub에 노출되지 않았는지 확인
4. 외부 API 콘솔에서 사용량 또는 권한 확인
```

---

# 22. 발표에서 AWS 담당이 설명할 내용

AWS 담당은 발표에서 아래 내용을 설명하면 좋다.

## 발표 포인트

```text
1. 본 서비스는 로컬 환경이 아니라 AWS 클라우드 환경에 배포되었다.
2. 프론트엔드는 S3 또는 EC2를 통해 사용자가 접속할 수 있도록 구성했다.
3. 백엔드 API 서버는 EC2에서 실행되며, 프론트엔드 요청을 처리한다.
4. 검색 기록과 추천 결과 저장을 위해 RDS 또는 DynamoDB를 사용했다.
5. 외부 대중교통 API Key와 DB 접속 정보는 환경변수로 분리했다.
6. 보안그룹을 통해 필요한 포트만 허용했다.
7. CloudWatch 또는 서버 로그를 통해 API 요청과 오류를 확인할 수 있도록 했다.
8. 전체 구조를 AWS 아키텍처 다이어그램으로 정리했다.
```

---

---

# 24. AWS 담당의 최종 산출물

AWS 담당이 최종적으로 제출하거나 시연할 수 있어야 하는 것은 다음과 같다.

```text
1. EC2 백엔드 배포 환경
2. 프론트엔드 접속 URL
3. 백엔드 API 접속 URL
4. RDS 또는 DynamoDB 구성
5. 환경변수 관리 구조
6. 보안그룹 설정
7. 서버 로그 확인 방식
8. AWS 아키텍처 다이어그램
9. 배포 과정 정리 문서
```

---

# 25. AWS 아키텍처 다이어그램에 들어갈 요소

발표 자료에 아래 요소를 포함하면 좋다.

```text
User
Frontend
Backend API Server
AI Recommendation Module
External Transit API
Database
CloudWatch
IAM / Security Group
```

---

## 다이어그램 예시

```text
[User]
   |
   v
[Frontend - S3 Hosting]
   |
   v
[Backend API - EC2]
   |
   +--> [External Transit API]
   |       - ODsay / TMAP / Naver Maps
   |
   +--> [AI Recommendation Module]
   |
   v
[Database - RDS or DynamoDB]
   |
   v
[CloudWatch Logs]
```

---

# 26. 정리

| 구분 | 내용 |
|---|---|
| 핵심 역할 | 서비스를 AWS 클라우드 환경에 배포하고 운영 구조 구성 |
| 프론트 관련 | S3 또는 EC2를 통한 프론트엔드 배포 |
| 백엔드 관련 | EC2에서 API 서버 실행 |
| DB 관련 | RDS 또는 DynamoDB 생성 및 연결 환경 구성 |
| 보안 관련 | 보안그룹, 환경변수, API Key 관리 |
| 로그 관련 | 서버 로그 또는 CloudWatch 로그 확인 |
| 최종 산출물 | 실제 접속 가능한 서비스 URL과 AWS 아키텍처 |

AWS 담당은 단순 배포 담당이 아니라,  
**프로젝트가 실제 서비스처럼 클라우드 위에서 동작한다는 것을 증명하는 역할**이다.

프론트엔드가 화면을 만들고, 백엔드가 API를 만들고, AI 알고리즘이 추천 로직을 만들더라도,  
AWS 담당이 클라우드 배포와 운영 구조를 완성해야 프로젝트가 “실제 서비스 가능한 형태”가 된다.
