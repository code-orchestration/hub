# Service Issue Handler Workflow Setup

## Overview
이 워크플로우는 'service' 레이블이 붙은 이슈가 생성되면 자동으로:
1. 새로운 서비스 레포지토리를 생성
2. 기본 프로젝트 구조를 설정
3. 원본 이슈를 새 레포지토리로 마이그레이션
4. 원본 이슈를 업데이트하고 닫기

## 필수 설정

### 1. Personal Access Token (PAT) 생성
조직 레포지토리를 생성하려면 적절한 권한이 있는 PAT가 필요합니다.

1. GitHub Settings > Developer settings > Personal access tokens > Tokens (classic)
2. "Generate new token" 클릭
3. 다음 권한 선택:
   - `repo` (전체 권한)
   - `admin:org` > `write:org` (조직 레포 생성용)
4. 토큰 생성 및 복사

### 2. Repository Secret 추가
1. Hub 레포지토리로 이동
2. Settings > Secrets and variables > Actions
3. "New repository secret" 클릭
4. Name: `ADMIN_PAT`
5. Value: 위에서 생성한 PAT 붙여넣기

## 사용 방법

### 이슈 생성 시 자동 트리거
1. 새 이슈 생성
2. 제목 형식: `[Service] 서비스-이름` (예: `[Service] user-auth`)
3. 'service' 레이블 추가
4. 이슈 생성

### 수동으로 기존 이슈에 적용
1. 기존 이슈 열기
2. 'service' 레이블 추가
3. 워크플로우가 자동으로 실행됨

## 워크플로우 동작 과정

1. **서비스 이름 추출**
   - 이슈 제목에서 `[Service]` 뒤의 텍스트 추출
   - 공백은 하이픈으로 변환, 소문자로 변환
   - 제목에 패턴이 없으면 `service-{issue-number}` 사용

2. **레포지토리 생성**
   - 이름: `{service-name}-service`
   - 자동으로 README, 기본 디렉토리 구조, CI/CD 설정 포함

3. **이슈 마이그레이션**
   - 새 레포지토리에 원본 이슈 내용 복사
   - 원본 작성자와 링크 정보 포함

4. **원본 이슈 업데이트**
   - 새 레포지토리 링크 댓글 추가
   - 'migrated' 레이블 추가
   - 이슈 자동 닫기

## 생성되는 레포지토리 구조
```
{service-name}-service/
├── README.md           # 서비스 설명 및 원본 이슈 정보
├── .gitignore         # 기본 ignore 패턴
├── src/               # 소스 코드 디렉토리
├── docs/              # 문서 디렉토리
├── tests/             # 테스트 디렉토리
└── .github/
    └── workflows/
        └── ci.yml     # 기본 CI/CD 파이프라인
```

## 문제 해결

### 워크플로우가 실행되지 않음
- 'service' 레이블이 제대로 추가되었는지 확인
- Actions 탭에서 워크플로우 실행 로그 확인

### 레포지토리 생성 실패
- `ADMIN_PAT` secret이 올바르게 설정되었는지 확인
- PAT에 필요한 권한이 있는지 확인
- 조직 설정에서 멤버가 레포지토리를 생성할 수 있는지 확인

### 이슈 마이그레이션 실패
- 새 레포지토리에 이슈가 활성화되어 있는지 확인
- PAT 권한 확인

## 커스터마이징

### 레포지토리 템플릿 변경
워크플로우 파일의 "Initialize service repository" 단계를 수정하여:
- 다른 디렉토리 구조 생성
- 추가 설정 파일 포함
- 특정 기술 스택에 맞는 템플릿 사용

### 레이블 변경
다른 레이블을 사용하려면:
```yaml
if: contains(github.event.issue.labels.*.name, 'your-label')
```

### 비공개 레포지토리 생성
```json
"private": true  // false를 true로 변경
```