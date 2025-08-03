# GitHub Fork

> GitHub Fork는 오픈소스 프로젝트에 기여하거나 다른 개발자와 협업할 때 사용하는 핵심 기능입니다.  
> 원본 저장소를 자신의 계정으로 복사하여 독립적으로 작업할 수 있게 해주며,  
> 변경사항을 원본 저장소에 Pull Request를 통해 제안할 수 있습니다.  

## 1. Fork란?

`Fork`는 **원본 저장소의 복사본을 자신의 GitHub 계정에 생성하는 것**입니다.  
이를 통해 원본 저장소에 직접적인 쓰기 권한이 없어도 프로젝트에 기여할 수 있습니다.

### Fork의 장점

* 원본 저장소에 영향을 주지 않고 자유롭게 실험 가능
* 자신만의 독립적인 작업 공간 확보
* 원본 저장소의 변경사항을 언제든지 동기화 가능
* Pull Request를 통한 체계적인 기여 프로세스

### 언제 사용하는가?

* 오픈소스 프로젝트에 기여하고 싶을 때
* 다른 개발자의 코드를 학습하고 싶을 때
* 팀 프로젝트에서 독립적인 작업 공간이 필요할 때
* 원본 저장소에 쓰기 권한이 없을 때

## 2. Fork 작업 흐름

### 2.1 Fork 생성

1. GitHub에서 원하는 저장소로 이동
2. 우측 상단의 "Fork" 버튼 클릭
3. 자신의 계정으로 저장소가 복사됨

### 2.2 로컬 저장소 설정

```bash
# Fork한 저장소를 로컬에 클론
git clone https://github.com/YOUR-USERNAME/FORKED-REPOSITORY.git

# 원격 저장소 확인
git remote -v
# origin  https://github.com/YOUR-USERNAME/FORKED-REPOSITORY.git (fetch)
# origin  https://github.com/YOUR-USERNAME/FORKED-REPOSITORY.git (push)
```

### 2.3 Upstream 원격 저장소 추가

```bash
# 원본 저장소를 upstream으로 추가
git remote add upstream https://github.com/ORIGINAL-OWNER/ORIGINAL-REPOSITORY.git

# 원격 저장소 확인
git remote -v
# origin    https://github.com/YOUR-USERNAME/FORKED-REPOSITORY.git (fetch)
# origin    https://github.com/YOUR-USERNAME/FORKED-REPOSITORY.git (push)
# upstream  https://github.com/ORIGINAL-OWNER/ORIGINAL-REPOSITORY.git (fetch)
# upstream  https://github.com/ORIGINAL-OWNER/ORIGINAL-REPOSITORY.git (push)
```

> 여기서 **Upstream**이란?  
> Upstream은 원본 저장소를 가리키는 원격 저장소의 별칭입니다.  
> Fork한 저장소에서 원본 저장소의 최신 변경사항을 가져오기 위해 사용하며,  
> `origin`은 자신의 Fork 저장소, `upstream`은 원본 저장소를 의미합니다.  
> `local, remote` 와는 다른 개념

## 3. Fork 동기화 (Sync)

### 3.1 웹 UI에서 동기화

1. Fork한 저장소 페이지로 이동
2. "Sync fork" 버튼 클릭
3. "Update branch" 클릭하여 동기화

### 3.2 명령줄에서 동기화

```bash
# upstream에서 최신 변경사항 가져오기
git fetch upstream

# 로컬 main 브랜치로 전환
git checkout main

# upstream의 변경사항을 로컬에 병합
git merge upstream/main

# 변경사항을 원격 저장소에 푸시
git push origin main
```

### 3.3 GitHub CLI 사용

```bash
# GitHub CLI로 동기화
gh repo sync owner/repository-name -b main

# 충돌이 있을 경우 강제 동기화
gh repo sync owner/repository-name -b main --force
```

## 4. 작업 및 기여 프로세스

### 4.1 기능 브랜치 생성

```bash
# 새로운 기능을 위한 브랜치 생성
git checkout -b feature/new-feature

# 작업 후 커밋
git add .
git commit -m "Add new feature"

# 원격 저장소에 푸시
git push origin feature/new-feature
```

### 4.2 Pull Request 생성

1. GitHub에서 Fork한 저장소로 이동
2. "Compare & pull request" 버튼 클릭
3. 변경사항 설명 작성
4. Pull Request 생성

### 4.3 코드 리뷰 및 병합

* 원본 저장소 관리자가 코드를 리뷰
* 필요시 수정 요청
* 승인 후 원본 저장소에 병합

## 5. 주의사항

### 5.1 동기화 시 주의사항

* 로컬에 커밋하지 않은 변경사항이 있다면 먼저 커밋하거나 스태시
* 충돌이 발생할 경우 수동으로 해결 필요
* 강제 동기화는 로컬 변경사항을 덮어쓸 수 있음

### 5.2 Pull Request 작성 시

* 명확한 제목과 설명 작성
* 변경사항의 목적과 이유 명시
* 관련 이슈나 버그 번호 참조
* 테스트 결과 포함

## 6. 유용한 명령어 정리

```bash
# 원격 저장소 확인
git remote -v

# upstream 추가
git remote add upstream <URL>

# upstream 제거
git remote remove upstream

# 모든 원격 브랜치 확인
git branch -r

# upstream에서 특정 브랜치 가져오기
git fetch upstream <branch-name>
git checkout -b <local-branch> upstream/<branch-name>
```

## 참고자료

* [GitHub 공식 문서 - Fork](https://docs.github.com/ko/pull-requests/collaborating-with-pull-requests/working-with-forks)
* [GitHub 공식 문서 - 동기화](https://docs.github.com/ko/pull-requests/collaborating-with-pull-requests/working-with-forks/syncing-a-fork) 
