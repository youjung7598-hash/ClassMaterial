### 🧩 전체 시나리오
	내가 만든 미니 프로젝트 하나를 실제 팀 프로젝트처럼 Git/GitHub로 관리해본다.

- 대상: Git 기본(add/commit/push), 브랜치, PR 개념을 한 번 배운 상태
- 준비물:
    - WSL + VSCode
    - Git 설치 완료
    - GitHub 계정
    - 이미 만들어둔 미니 프로젝트 폴더 하나
        (예: `mini-account-book`, `streamlit-dashboard`, `todo-api`, …)
        
---
### 🧪 미션 구조

- **미션 0** 준비 – 사용할 프로젝트 정하고 환경 점검
- **미션 1** 로컬 Git 초기화 + GitHub 레포 생성 + 첫 push
- **미션 2** dev 브랜치 생성 + Git 전략 문서화
- **미션 3** feature 브랜치에서 기능 수정 → PR 생성 → 코드 리뷰 & 병합
- **미션 4** 의도적인 충돌(conflict) 만들기 + 해결
- **미션 5** dev → main Release PR 만들기(최종 배포 시뮬레이션)

---
### ⏱ 미션 0. 준비

✅ **목표**
- 어떤 미니 프로젝트를 실습에 사용할지 고른다
- Git, GitHub 설정이 잘 되어 있는지 확인한다

💻 해야 할 일
1. 미니 프로젝트 폴더 선택
    
    예시:
```bash
cd ~
ls
cd mini-account-book # 또는 본인 프로젝트 이름
```
    
2. Git 설치 여부 확인
```bash
git --version
```
    
3. Git 사용자 정보 설정 확인
```bash
git config --global --list
```
    없으면 설정:
    
```bash
git config --global user.name"홍길동"
git config --global user.email"youremail@example.com"
```
    
체크리스트
- [ ] 사용할 프로젝트 폴더 하나를 정했다
- [ ] `git --version` 정상 출력
- [ ] `user.name`, `user.email`이 GitHub 계정과 맞춰져 있다

---
⏱ 미션 1. 로컬 Git 초기화 + GitHub 레포 생성 + 첫 push (약 20분)

✅ 목표
- 이미 만들어진 미니 프로젝트를 Git이 관리하도록 만든다
- GitHub에 원격 레포를 만들고 연결 + 첫 push 까지 완료한다

💻 해야 할 일

1. Git 초기화
```bash
git init
git status
```
    
2. 불필요 파일 무시할 `.gitignore` 추가 (선택)
- Python 예시:
```bash
echo"venv/" >> .gitignore
echo"__pycache__/" >> .gitignore
echo".DS_Store" >> .gitignore
```
        
3. **첫 커밋 만들기**
```bash
git add .
git commit -m"Initial commit: existing mini project"
```
    
4. GitHub에서 새 레포 만들기
    - 이름 예: `mini-account-book-git-practice`
    - README, .gitignore 자동 생성 ❌ (이미 로컬에 있음)
5. 원격 연결 + 첫 push
```bash
git remote add origin <https://github.com/><username>/mini-account-book-git-practice.git
git branch -M main
git push -u origin main
```
    

체크리스트
- [ ] 로컬 프로젝트에 `.git` 폴더가 생겼다
- [ ] `git log --oneline` 으로 첫 커밋이 보인다
- [ ] GitHub 레포 페이지에서 프로젝트 파일들이 보인다(main 브랜치)

---
⏱ 미션 2. dev 브랜치 만들기 + Git 전략 문서화 (약 20분)

✅ 목표
- main은 “완성본”, dev는 “통합 개발용”이라는 구조를 실제로 만든다
- `docs/git-strategy.md` 파일에 **자기 팀의 Git 전략을 글로 정리**해본다

💻 해야 할 일
1. dev 브랜치 생성 + 원격 push
```bash
git checkout main
git pull origin main# 혹시 모를 차이 정리
git checkout -b dev
git push -u origin dev
 ```
    
2. Git 전략 문서 작성 (`docs/git-strategy.md`)
    없으면 디렉토리/파일 생성:
```bash
mkdir -p docs
nano docs/git-strategy.md
```
    내용 예시(학생이 직접 작성하게):
    
```markdown
# Git 브랜치 전략 (Mini Project)
    
## 브랜치 역할
- main: 발표/배포용 최종 코드, 항상 동작해야 함
- dev: 여러 기능을 통합하는 개발 브랜치
- feature/*: 개인 기능 개발 브랜치 (예: feature/youjung-login)
    
## PR 규칙
- feature/* → dev 로 PR 생성 (기능 단위)
- dev → main 은 릴리즈 시점에만 PR 생성
- PR 템플릿:
  - 변경 내용
  - 테스트 방법
  - TODO/주의사항
    
## 충돌 대응
  - 브랜치 변경 전: 항상 git status, git pull 확인
  - 충돌 발생 시: 파일 열고 <<<<<<< 표시 기준으로 최종 버전 선택 후 commit
 ```
    
3. dev 브랜치에 커밋
```bash
git add docs/git-strategy.md
git commit -m"Add Git branch strategy document"
git push
```
    
체크리스트
- [ ] GitHub에서 `dev` 브랜치가 보인다
- [ ] `docs/git-strategy.md` 파일이 dev에 존재
- [ ] 문서 안에 main/dev/feature 전략이 본인 말로 정리되어 있다

---
⏱ 미션 3. feature 브랜치 → PR → 코드 리뷰 → dev 병합 (약 30분)

✅ 목표
- dev에서 feature 브랜치를 따서
- 코드/문서를 조금 수정해보고
- PR을 만들어 **코드리뷰 코멘트까지 남긴 뒤, Squash Merge로 dev에 병합**해본다

💻 해야 할 일
1. dev 기준으로 feature 브랜치 생성
```bash
git checkout dev
git pull origin dev
    
git checkout -b feature/readme-improve
```
    
2. README 개선 작업
    
    - `README.md` 에 아래와 같이 섹션 추가(학생 마음대로):
```markdown
## 프로젝트 소개
        
- 이 프로젝트는 미니 가계부 / API / 대시보드를 연습하기 위한 예제입니다.
- Git, GitHub, 브랜치, PR 실습을 위해 사용합니다.
        
## 실행 방법
        
- Python 버전, 가상환경 생성 방법
- 핵심 실행 명령어 (예:`python app.py`,`streamlit run app.py`)
```
        
3. 커밋 + feature 브랜치 push
```bash
git add README.md
git commit -m"Improve README with project intro and run guide"
git push -u origin feature/readme-improve
```
    
4. GitHub에서 PR 생성 (feature → dev)
    - base: `dev`
    - compare: `feature/readme-improve`
    - PR 제목: `Improve README with project introduction`
    - PR 본문에 아래 형식으로 작성하게 하기:
```markdown
### 변경 내용
- README에 프로젝트 소개 섹션 추가
- 실행 방법 섹션 추가
        
### 테스트 방법
- 로컬에서 README 렌더링 확인 (마크다운 미리보기)
        
### 기타
- 추가로 기여 규칙(CONTRIBUTING) 섹션을 나중에 작성 예정
```
        
5. 자기 PR에 “리뷰 코멘트” 남기기
    - Files changed 탭에서 README의 한 줄을 선택 → `Add comment`
        예: 프로젝트 소개에 사용 기술 스택도 추가하면 좋겠습니다.
        
6. Squash and merge로 dev에 병합
    - PR 페이지에서 `Squash and merge` 선택
    - 병합 후 `Delete branch` 버튼 눌러 feature 브랜치 삭제
	
7. 로컬 dev 최신화
```bash
git checkout dev
git pull origin dev
```
    
체크리스트
- [ ] GitHub에서 `feature/readme-improve` 브랜치가 생성되었다
- [ ] PR 화면에서 Files changed 탭에 변경 내용이 보였다
- [ ] 최소 1개 이상의 리뷰 코멘트를 남겨봤다
- [ ] Squash and merge로 병합했고, PR 상태가 `Merged`이다
- [ ] 로컬 dev에서 `git log --oneline`에 squash된 커밋이 보인다

---
⏱ 미션 4. 일부러 충돌 만들기 + 해결하기 (약 25분)

✅ 목표
- “같은 줄을 서로 다르게 수정해서 충돌 나게 하기”
- `<<<<<<< HEAD` 표시의 의미를 직접 눈으로 보고,
- 최종 버전을 고른 뒤 commit & push까지 해본다

💻 해야 할 일
4-1. 충돌 상황 만들기

1. 상태가 깨끗한지 먼저 확인
```bash
git checkout main
git pull origin main
git status
```
    - `nothing to commit, working tree clean` 이어야 시작
    
2. (GitHub 웹에서) README 한 줄 수정 – “원격 버전”
    GitHub main 브랜치에서:
    - README.md → ✏️ Edit
    - 예를 들어, 맨 위 줄을 이렇게 수정:
```markdown
# Mini Project with Git Practice (edited on GitHub)
```
    - Commit changes → main에 바로 반영
        
3. (로컬에서) 같은 줄을 다르게 수정 – “로컬 버전”
```bash
nano README.md
```
    또는 VSCode로 열어서,
    
```markdown
# Mini Project with Git Practice (edited on local)
```
    로 바꾸기 → 저장
    
4. 로컬에서 commit
```bash
git add README.md
git commit -m"Edit README title locally"
```
    
5. 이제 pull 시도 → 충돌 발생
```bash
git pull
```
    
예상 메세지:
```
 CONFLICT (content): Mergeconflictin README.md
Automatic merge failed; fix conflictsandthencommit the result.
 ```

---
4-2. 충돌 해결

1. 충돌 난 파일 열어서 확인
```bash
nano README.md
```
    
또는
```bash
code README.md
```
    
내용 예시:
```
<<<<<<< HEAD
# Mini Project with Git Practice (edited on local)
=======
# Mini Project with Git Practice (edited on GitHub)
>>>>>>> origin/main
```
    
2. 최종 버전 선택 (둘 중 하나 또는 합치기)
    
    예시 – 둘을 합쳐서 새 문장으로:
```markdown
# Mini Project with Git Practice (local + GitHub edits merged)
```
    
3. 가장 중요! 충돌 표시 삭제하기
    
    아래 줄들 모두 삭제되어야 한다:
    - `<<<<<<< HEAD`
    - `=======`
    - `>>>>>>> origin/main`
3. 해결 후 add → commit → push
```bash
git add README.md
git commit -m"Resolve README merge conflict"
git push
```

체크리스트
- [ ] `git pull` 시 실제로 conflict 에러를 한 번 봤다
- [ ] README 안에 `<<<<<<<`, `=======`, `>>>>>>>` 블록을 직접 확인했다
- [ ] 최종 버전을 스스로 결정해서 남겼다
- [ ] conflict 해결 후 commit & push까지 마무리했다
- [ ] GitHub main 브랜치의 README에 최종 결정한 문장이 반영되어 있다

---
⏱ 미션 5. dev → main Release PR 만들기 (약 15분)

✅ 목표
- 지금까지 dev에 모인 변경들을 **“릴리즈 PR”**로 main에 합치는 연습
- “기능 개발 PR”과 “릴리즈 PR”의 차이를 몸으로 이해하기

💻 해야 할 일
1. dev가 최신인지 확인
```bash
git checkout dev
git pull origin dev
```
    
2. GitHub에서 Release PR 생성
    - base: `main`
    - compare: `dev`
    - PR 제목 예: `Release: sync dev into main`
    - 설명 예:
```markdown
### 포함된 변경 사항
- README 개선 (프로젝트 소개/실행 방법 추가)
- Git 브랜치 전략 문서 추가
- README 충돌 해결 후 최종 문구 정리
        
### 테스트
- 로컬에서 main/dev 동기화 후 README와 docs 파일 존재 확인
```
        
3. Merge (Squash or Merge commit 중 택 1)
    - 실습에서는 **Squash and merge** 추천 (히스토리 깔끔)
    - 병합 후 main 브랜치 최신화:
```bash
git checkout main
git pull origin main
```

체크리스트
- [ ] GitHub에서 `dev → main` PR을 한 번 만들었다
- [ ] PR 설명에 “포함된 변경 사항 / 테스트”를 적어봤다
- [ ] PR 상태가 `Merged` 로 바뀌었다
- [ ] 로컬 main에서 변경 내용이 반영된 것을 확인했다

---
✅ 마지막: 자기 점검 질문

체크 질문
1. 지금 내 프로젝트의 브랜치는 어떤 구조(main/dev/feature)로 되어 있나요?
2. 새로운 기능을 개발하고 싶을 때, 어떤 순서로 브랜치를 만들고, 어디로 PR을 보내야 하나요?
3. `git pull`과 `git fetch`는 실제로 언제, 어떻게 다르게 쓸 수 있을까요?
4. 충돌이 났을 때, 어떤 파일에서 어떤 표시를 보고, 무엇을 기준으로 최종 버전을 선택해야 하나요?
5. 지금 이 미니 프로젝트는 “실제 팀 프로젝트”로 확장해도 Git 구조가 버틸 수 있을까요? (아니라면, git-strategy.md에 무엇을 더 보완해야 할까요?)