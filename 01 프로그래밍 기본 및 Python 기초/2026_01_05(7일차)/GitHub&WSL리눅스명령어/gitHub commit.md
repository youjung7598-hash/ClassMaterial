### 🔹 1. Git 초기 설정 (최초 1회만)
```bash
git config --global user.name "Your Name" 
git config --global user.email "your_email@example.com"
```

---
### 🔹 2. Git 저장소 초기화
```bash
git init  # 현재 폴더를 Git 저장소로 초기화
```

---
### 🔹 3. 상태 확인
```bash
git status  # 추적 중인 파일, 변경된 파일 등을 확인
```

---
### 🔹 4. 파일을 스테이지에 추가
```bash
git add filename        # 특정 파일 하나만 추가 git add .               # 현재 디렉토리 이하 모든 변경 파일 추가 git add -A              # 삭제된 파일까지 포함해서 모든 변경사항 추가
```

---
### 🔹 5. 커밋하기
```bash
git commit -m "커밋 메시지 작성"
```

---
### 🔹 6. 원격 저장소 연결 (GitHub)
```bash
git remote add origin https://github.com/username/repository.git
```

> `origin`은 원격 저장소 이름 (기본값), URL은 본인의 GitHub 저장소 주소로 바꿔주세요.

---
### 🔹 7. 브랜치 확인 및 생성
```bash
git branch              # 현재 브랜치 확인 git branch -M main      # 브랜치 이름을 main으로 변경 (필요한 경우)
```

---
### 🔹 8. GitHub로 푸시
```bash
git push -u origin main  # 최초 푸시 (브랜치 연결까지 함께) git push                 # 이후엔 그냥 push만 해도 됨
```

---
### 🔹 9. 변경사항 추적 후 다시 커밋 & 푸시
```bash
git add . git commit -m "변경사항 설명" git push
```

---
### 🔹 10. 로그 확인
```bash
git log                 # 전체 커밋 로그 확인 git log --oneline       # 간단한 로그 보기
```

---
### 🔹 11. 원격 저장소 주소 확인 및 변경
```bash
git remote -v                # 원격 저장소 확인 git remote set-url origin <새URL>  # URL 수정
```

---
### 🔹 12. 파일 삭제 및 커밋
```bash
git rm filename              # Git에서 파일 제거 git rm -r foldername         # 폴더 제거 git commit -m "파일 제거" git push
```

---
### 🔹 13. .git 폴더 제거 (Git 연결 해제)
```bash
rm -rf .git
```

> 해당 폴더에서 Git 연결을 완전히 해제하고 싶을 때 사용

---
### 🔹 14. 클론 (다른 사람의 GitHub 저장소 가져오기)
```bash
git clone https://github.com/username/repository.git
```