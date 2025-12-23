###### ✅ 1. 기본적인 Linux 명령어 (파일/디렉토리 관련)
| 명령어         | 설명                                 |
| ----------- | ---------------------------------- |
| `ls`        | 현재 디렉토리의 파일 목록 보기 (`ls -l`은 상세 정보) |
| `cd 디렉토리명`  | 디렉토리 이동                            |
| `cd ..`     | 상위 디렉토리로 이동                        |
| `pwd`       | 현재 디렉토리 위치 출력                      |
| `mkdir 폴더명` | 새 디렉토리 생성                          |
| `rm 파일명`    | 파일 삭제 (`-rf` 옵션은 폴더 포함 강제 삭제)      |
| `mv 원본 대상`  | 파일 또는 폴더 이동/이름 변경                  |
| `cp 원본 대상`  | 파일 복사 (`-r` 옵션으로 폴더 복사)            |
| `touch 파일명` | 새 파일 생성                            |
| `cat 파일명`   | 파일 내용 출력                           |
| `nano 파일명`  | 간단한 텍스트 에디터로 파일 열기                 |
| `clear`     | 터미널 화면 깨끗하게 정리                     |

###### ✅ 2. Python 및 Django 관련
| 명령어                                 | 설명                                                                    |
| ----------------------------------- | --------------------------------------------------------------------- |
| `python3 --version`                 | 설치된 Python 버전 확인                                                      |
| `python3 manage.py runserver`       | Django 개발 서버 실행 (기본: [http://127.0.0.1:8000](http://127.0.0.1:8000/)) |
| `python3 manage.py startapp 앱이름`    | 새 Django 앱 생성                                                         |
| `python3 manage.py makemigrations`  | 모델 변경 사항 감지하여 마이그레이션 파일 생성                                            |
| `python3 manage.py migrate`         | DB에 실제 반영                                                             |
| `python3 manage.py createsuperuser` | 관리자 계정 생성                                                             |
| `python3 manage.py shell`           | Django 환경에서 파이썬 쉘 실행                                                  |
| `python3 manage.py collectstatic`   | static 파일들을 모아서 배포용 폴더로 복사                                            |
| `python3 manage.py test`            | Django 테스트 실행                                                         |
⚠️ python3 대신 python으로 써도 되지만, 혼동 방지를 위해 python3 권장

###### ✅ 3. 가상환경 관련 명령어 (venv)
| 명령어                               | 설명                            |
| --------------------------------- | ----------------------------- |
| `python3 -m venv venv`            | 가상환경 생성                       |
| `source venv/bin/activate`        | 가상환경 활성화                      |
| `deactivate`                      | 가상환경 비활성화                     |
| `pip install 패키지명`                | 패키지 설치 (`pip install django`) |
| `pip freeze > requirements.txt`   | 설치된 패키지 목록 저장                 |
| `pip install -r requirements.txt` | requirements.txt 기반 설치        |

###### ✅ 4. Git 관련 (버전 관리)
| 명령어                   | 설명                |
| --------------------- | ----------------- |
| `git init`            | Git 저장소 초기화       |
| `git status`          | 현재 Git 상태 확인      |
| `git add .`           | 모든 변경 파일 스테이징     |
| `git commit -m "메시지"` | 커밋                |
| `git log`             | 커밋 히스토리 확인        |
| `git push`            | 변경 사항 원격 저장소로 업로드 |
| `git pull`            | 원격 저장소에서 내려받기     |
| `git clone URL`       | Git 프로젝트 복제       |
| `git commit --amend`  | 로컬에서 커밋 메시지 수정하기  |

###### ✅ 5. 시스템/환경 확인 및 패키지 관리
| 명령어                     | 설명                   |
| ----------------------- | -------------------- |
| `htop`                  | 시스템 리소스 모니터링 (설치 필요) |
| `top`                   | 리눅스 기본 리소스 모니터       |
| `df -h`                 | 디스크 사용량 확인           |
| `free -m`               | 메모리 사용량 확인           |
| `sudo apt update`       | 패키지 목록 최신화           |
| `sudo apt upgrade`      | 패키지 전체 업그레이드         |
| `sudo apt install 패키지명` | 새 패키지 설치             |

###### ✅ 6. VSCode 연동 팁 (WSL)
| 명령어              | 설명                               |
| ---------------- | -------------------------------- |
| `code .`         | 현재 폴더를 VSCode로 열기 (WSL 확장 설치 필요) |
| `explorer.exe .` | 윈도우 파일 탐색기로 현재 폴더 열기             |







