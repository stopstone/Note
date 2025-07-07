# GitHub에서 Merge 방식 선택: Merge Commit vs Squash vs Rebase

> GitHub에서 Pull Request를 머지할 때 보게 되는 세 가지 옵션이 있습니다.  
> `Create a merge commit`, `Squash and merge`, `Rebase and merge`는 각각 다르게 기록되고, 협업 및 히스토리 관리 방식에 큰 영향을 미칩니다.  
> 이 글에서는 각 방식의 차이를 정리하고, 어떤 상황에서 어떤 방식을 선택하면 좋을지 살펴보겠습니다.  

## 1. Create a merge commit

* **설명**: 별도의 merge 커밋을 생성하여 브랜치를 병합합니다.
* **기록 방식**: 모든 커밋 기록이 그대로 보존되며, 머지 지점에 새로운 커밋이 추가됩니다.
* **Git 로그 예시**:

  ```
  *   commit M (Merge commit)
  |\  
  | * commit C (feature 브랜치 커밋)
  | * commit B
  | * commit A
  * | commit Z (main 커밋)
  ```
* **장점**:

  * 커밋 히스토리를 그대로 보존하므로 개발 흐름을 명확히 파악할 수 있음
  * 큰 프로젝트에서 병합의 시점을 명확히 알 수 있음
* **단점**:

  * 히스토리가 복잡해짐 (특히 rebase를 사용하지 않은 여러 브랜치가 얽히는 경우)

## 2. Squash and merge

* **설명**: feature 브랜치의 커밋을 모두 하나로 합쳐(main에) 머지합니다.
* **기록 방식**: 모든 커밋을 하나로 압축하여 단일 커밋으로 main 브랜치에 추가됩니다.
* **Git 로그 예시**:

  ```
  * commit S (Squashed commit: A+B+C)
  * commit Z
  ```
* **장점**:

  * main 브랜치의 히스토리가 깔끔해짐
  * 커밋 메시지를 팀 컨벤션에 맞게 재작성 가능
* **단점**:

  * 세부적인 작업 기록이 사라짐 → 디버깅 시 어려움
  * 원 브랜치 커밋 해시가 사라짐 → 협업자와의 커밋 공유가 애매해짐

## 3. Rebase and merge

* **설명**: feature 브랜치의 커밋을 main 브랜치 위에 재배치한 후 병합합니다.
* **기록 방식**: 커밋 기록은 그대로지만, main 브랜치 위에 재정렬됨
* **Git 로그 예시**:

  ```
  * commit C (rebase된 커밋)
  * commit B
  * commit A
  * commit Z
  ```
* **장점**:

  * 커밋 히스토리가 일직선으로 깔끔함
  * 기능별 커밋 추적이 쉬움
* **단점**:

  * 원본 브랜치와 커밋 해시가 달라짐
  * 충돌이 발생하면 해결 과정이 복잡해질 수 있음

## 4. 어떤 방식을 선택해야 할까?

| 상황                       | 추천 방식                 | 이유                         |
| ------------------------ | --------------------- | -------------------------- |
| 소규모 팀, 리뷰 완료 후 단일 기능 머지  | Squash and merge      | 깔끔한 기록, 의미 있는 커밋 메시지 작성 가능 |
| 오픈소스 프로젝트, 커밋 기록을 보존해야 함 | Create a merge commit | 외부 기여자의 작업 흐름을 보존하는 게 중요함  |
| 한 명이 여러 작업 브랜치를 운영       | Rebase and merge      | 병합 시점이 명확하고, 커밋 단위 추적이 쉬움  |

---

### 참고자료

* [GitHub Docs: About pull request merges](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/working-with-pull-requests/about-pull-request-merges)
* [Atlassian Git Tutorials](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)
