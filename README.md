# Git 자주 쓰는 명령어 정리

실무에서 반복적으로 쓰게 되는 git 명령어를 사용 예시와 함께 정리했다.

---

## 목차

1. [최초 설정](#1-최초-설정)
2. [저장소 시작하기](#2-저장소-시작하기)
3. [상태 확인](#3-상태-확인)
4. [스테이징과 커밋](#4-스테이징과-커밋)
5. [브랜치](#5-브랜치) · [checkout vs switch vs restore](#checkout-vs-switch-vs-restore)
6. [원격 저장소](#6-원격-저장소)
7. [되돌리기](#7-되돌리기)
8. [임시 저장 (stash)](#8-임시-저장-stash)
9. [추적 제외 (.gitignore)](#9-추적-제외-gitignore)
10. [협업 흐름 (PR)](#10-협업-흐름-pr)
11. [자주 만나는 에러](#11-자주-만나는-에러)

---

## 1. 최초 설정

```bash
# 커밋에 기록될 이름과 이메일 (--global은 이 PC 전체 기본값)
git config --global user.name "mgkim-developer"
git config --global user.email "mgkim.developer@gmail.com"

# 특정 저장소에서만 다른 계정을 쓰고 싶을 때 (--global 빼고 해당 폴더에서 실행)
git config user.email "work@company.com"

# 현재 적용된 설정 확인 (어느 파일에서 온 설정인지까지 표시)
git config --list --show-origin

# 자주 쓰는 명령어에 별칭 달기
git config --global alias.st status
git config --global alias.lg "log --oneline --graph --all"
```

---

## 2. 저장소 시작하기

```bash
# 새 저장소 생성 (기본 브랜치를 main으로)
git init -b main

# 원격 저장소 복제
git clone https://github.com/mgyokim/99_git.git

# 폴더명을 다르게 지정
git clone https://github.com/mgyokim/99_git.git my-project

# 히스토리 없이 최신 1개 커밋만 (용량 큰 저장소에 유용)
git clone --depth 1 https://github.com/mgyokim/99_git.git
```

---

## 3. 상태 확인

```bash
git status              # 현재 변경 상태
git status -s           # 짧게 (M=수정, A=추가, D=삭제, ??=미추적)
git status -sb          # 짧게 + 브랜치 정보

git log                             # 커밋 히스토리
git log --oneline                   # 한 줄씩
git log --oneline --graph --all     # 브랜치 그래프까지 (제일 많이 씀)
git log -5                          # 최근 5개만
git log -p README.md                # 특정 파일의 변경 내역까지
git log --author="mgyokim"          # 작성자로 필터
git log --since="2 weeks ago"       # 기간으로 필터

git diff                    # 워킹 디렉토리 vs 스테이징
git diff --staged           # 스테이징 vs 마지막 커밋 (커밋 직전 확인용)
git diff main..feature      # 브랜치 간 차이
git diff --stat             # 파일별 변경 줄 수 요약

git show e519f92            # 특정 커밋의 내용
git blame README.md         # 각 줄을 마지막으로 수정한 사람/커밋
```

---

## 4. 스테이징과 커밋

```bash
git add README.md       # 특정 파일
git add src/            # 디렉토리 전체
git add .               # 현재 경로 아래 전부
git add -p              # 변경 덩어리(hunk) 단위로 골라서 추가 ★유용

git reset README.md     # 스테이징 취소 (변경 내용은 유지)

git commit -m "feat: 로그인 API 추가"
git commit                      # 에디터에서 여러 줄 메시지 작성
git commit -am "fix: 오타 수정"  # add + commit (이미 추적 중인 파일만)

# 방금 한 커밋 수정 (메시지 변경 / 파일 빠뜨렸을 때)
git commit --amend -m "새 메시지"
git commit --amend --no-edit    # 메시지는 그대로, 파일만 추가
```

> ⚠️ `--amend`는 커밋 해시를 바꾼다. **이미 push한 커밋에는 쓰지 말 것** (강제 push가 필요해짐).

### 커밋 메시지 컨벤션

| 접두사 | 용도 |
|---|---|
| `feat:` | 새 기능 |
| `fix:` | 버그 수정 |
| `docs:` | 문서 |
| `refactor:` | 리팩토링 (동작 변화 없음) |
| `test:` | 테스트 코드 |
| `chore:` | 빌드/설정 등 잡무 |

---

## 5. 브랜치

```bash
git branch                  # 로컬 브랜치 목록
git branch -a               # 원격 포함 전체
git branch -vv              # 각 브랜치의 upstream까지 표시 ★문제 파악에 유용

git checkout -b feature/login   # 브랜치 생성 + 이동 (가장 많이 씀)
git checkout main               # 브랜치 이동
git checkout -                  # 직전 브랜치로 (cd - 와 같은 개념)

git branch -m old new       # 브랜치 이름 변경
git branch -d feature/login # 삭제 (머지된 것만, 안전)
git branch -D feature/login # 강제 삭제 (머지 안 됐어도)

git merge feature/login     # 현재 브랜치로 병합
git merge --no-ff feature/login   # 머지 커밋을 항상 남김 (이력 추적 용이)
```

### checkout vs switch vs restore

Git 2.23(2019)에서 `checkout`의 역할이 `switch`(브랜치 이동)와 `restore`(파일 복원)로
분리됐다. `checkout`이 성격이 전혀 다른 세 가지 일을 한꺼번에 맡고 있었기 때문이다.

```bash
git checkout main          # ① 브랜치 이동
git checkout abc123        # ② HEAD 분리 (detached HEAD)
git checkout -- file.txt   # ③ 파일 되돌리기 — 작업 내용이 삭제된다
```

#### 대응표

| 하고 싶은 일 | 기존 (checkout) | 신규 |
|---|---|---|
| 브랜치 이동 | `git checkout main` | `git switch main` |
| 브랜치 생성 + 이동 | `git checkout -b feat` | `git switch -c feat` |
| 직전 브랜치로 | `git checkout -` | `git switch -` |
| 커밋으로 이동 | `git checkout abc123` | `git switch --detach abc123` |
| 파일 되돌리기 | `git checkout -- file` | `git restore file` |
| 스테이징 취소 | `git reset file` | `git restore --staged file` |
| 다른 브랜치의 파일 가져오기 | `git checkout main -- file` | `git restore --source=main file` |

#### 차이 ① — 커밋 해시로 이동할 때

`checkout`은 경고 없이 detached HEAD 상태로 들어간다. 이 상태에서 커밋하면
어느 브랜치에도 속하지 않아 나중에 찾기 어려워진다.

```bash
$ git checkout 215613c
$ git status -sb
## HEAD (no branch)          # 조용히 분리됨
```

`switch`는 아예 거부하고, 의도한 것이라면 `--detach`를 명시하게 한다.

```bash
$ git switch 215613c
fatal: a branch is expected, got commit '215613c'
hint: If you want to detach HEAD at the commit, try again with the --detach option.
```

#### 차이 ② — 브랜치명과 파일명이 겹칠 때

`checkout`은 인자가 브랜치인지 파일 경로인지 스스로 추측한다.

```bash
$ ls
f.txt
$ git branch
  f.txt          # 공교롭게 같은 이름의 브랜치가 있는 상황
  main

$ git checkout f.txt        # f.txt 파일을 되돌리려던 의도였지만
Switched to branch 'f.txt'  # 브랜치로 해석됨
```

이래서 파일을 지정할 땐 `--`로 구분해야 했다.

```bash
git checkout -- f.txt       # 명시적으로 "이건 파일이다"
```

`switch`는 브랜치만, `restore`는 파일만 받으므로 이 모호함 자체가 없다.

#### 그래서 뭘 쓸까

- **브랜치 이동/생성** — `checkout -b`와 `switch -c`는 동작이 완전히 같다. 쓰던 걸 그대로 써도 무방하다.
- **파일 되돌리기** — `git restore`로 바꾸는 것을 권장한다.
  `git checkout -- file`은 **되돌릴 수 없는 몇 안 되는 일상 명령**이다.
  커밋하지 않은 변경은 reflog에도 남지 않아 그대로 사라진다.
  이름이 분리돼 있으면 "파일을 날리는 명령"이라는 게 눈에 보인다.

> ⚠️ `switch`와 `restore`는 아직 공식적으로 실험 단계다. man page에
> `THIS COMMAND IS EXPERIMENTAL. THE BEHAVIOR MAY CHANGE.` 문구가 남아 있다.
> 실무에서 깨진 사례는 없지만, CI 스크립트나 오래된 문서에는 여전히 `checkout`이
> 압도적으로 많이 쓰인다. 구버전 git 환경에서도 무조건 동작한다는 것도 장점이다.

### 브랜치 네이밍 예시

```
feature/login-api      기능 개발
fix/null-pointer       버그 수정
hotfix/payment-error   긴급 수정
0904_mgyo              날짜_작업자 (교육/실습용)
```

---

## 6. 원격 저장소

```bash
git remote -v                                   # 등록된 원격 목록
git remote add origin https://github.com/mgyokim/99_git.git
git remote add upstream https://github.com/WONIT-A/99_git.git   # 포크 원본

git push                            # upstream이 설정된 경우
git push -u origin feature/login    # 최초 push + upstream 설정 ★
git push origin HEAD:main           # 현재 HEAD를 원격 main으로
git push origin --delete feature/login  # 원격 브랜치 삭제

git fetch                   # 원격 변경사항 가져오기만 (병합 X)
git fetch --prune           # + 원격에서 삭제된 브랜치 참조 정리 ★
git pull                    # fetch + merge
git pull --rebase           # fetch + rebase (이력이 깔끔해짐)
```

### `-u` (`--set-upstream`) 란?

로컬 브랜치와 원격 브랜치를 연결해두는 옵션. 한 번 걸어두면 이후로는 인자 없이
`git push` / `git pull`만 쳐도 된다.

```bash
git push -u origin 0904_mgyo
# branch '0904_mgyo' set up to track 'origin/0904_mgyo'.

git branch -vv
# * 0904_mgyo e519f92 [origin/0904_mgyo] 커밋 메시지
#                     └─ 연결된 upstream
```

---

## 7. 되돌리기

가장 헷갈리는 영역. **어디까지 되돌릴 것인가**로 구분하면 쉽다.

```bash
# 파일 수정 취소 (커밋 안 한 변경 폐기) — 복구 불가! 주의
git restore README.md
git checkout -- README.md       # 구버전 표기, 동일 동작

# 스테이징만 취소 (수정 내용은 유지)
git restore --staged README.md
git reset README.md             # 구버전 표기, 동일 동작

# 커밋 되돌리기
git reset --soft HEAD~1    # 커밋만 취소, 변경은 스테이징에 남김
git reset --mixed HEAD~1   # 커밋+스테이징 취소, 파일은 유지 (기본값)
git reset --hard HEAD~1    # 전부 삭제 ★위험

# 이미 push한 커밋 되돌리기 → revert 사용 (되돌리는 커밋을 새로 추가)
git revert e519f92
```

| 상황 | 명령어 |
|---|---|
| push 전, 내 로컬에서만 | `reset` |
| push 후, 남과 공유된 이력 | **`revert`** (reset 쓰면 이력 충돌) |

### 실수했을 때 — reflog

```bash
git reflog          # HEAD가 거쳐온 모든 이력 (reset/rebase 이전 상태 포함)
git reset --hard HEAD@{3}   # 그 시점으로 복구
```

> `reset --hard`로 날린 **커밋**은 reflog로 복구 가능하다.
> 하지만 **커밋하지 않은 변경**은 reflog에도 없어서 복구할 수 없다.

---

## 8. 임시 저장 (stash)

작업 중인데 급하게 다른 브랜치로 가야 할 때.

```bash
git stash                       # 현재 변경사항 치워두기
git stash -u                    # 미추적 파일(untracked)까지 포함 ★
git stash save "로그인 작업 중"   # 설명 붙여서 저장

git stash list                  # 목록 확인
# stash@{0}: On main: 로그인 작업 중

git stash pop                   # 최근 것 꺼내오기 + 목록에서 삭제
git stash apply stash@{1}       # 특정 것 꺼내오기 (목록엔 남김)
git stash show -p stash@{0}     # 내용 미리보기
git stash drop stash@{0}        # 삭제
git stash clear                 # 전부 삭제
```

> ⚠️ stash는 **완전히 로컬 전용**이다. `.git/refs/stash`에 저장되며
> push/clone으로 전달되지 않는다. 다른 PC나 동료와 공유할 수 없고,
> 저장소 폴더를 지우면 함께 사라진다.
> 하루 넘게 보관할 작업이라면 stash 대신 **WIP 커밋 후 브랜치 push**를 권장한다.

---

## 9. 추적 제외 (.gitignore)

```bash
# .gitignore 작성 예시
cat > .gitignore <<'EOF'
# IDE
.idea/
.vscode/

# OS
.DS_Store

# 빌드 산출물
build/
dist/
node_modules/

# 환경 변수
.env
EOF
```

### 이미 커밋된 파일을 추적 제외하려면

`.gitignore`는 **이미 추적 중인 파일에는 적용되지 않는다.** 인덱스에서 먼저 빼야 한다.

```bash
git rm -r --cached .idea     # 추적만 해제 (로컬 파일은 유지)
git commit -m "chore: .idea 디렉토리 git 추적 제외"
```

> `--cached`를 빼면 **실제 파일까지 삭제**되니 반드시 붙일 것.

---

## 10. 협업 흐름 (PR)

```bash
# 1. 최신 main 받아오기
git checkout main
git pull

# 2. 작업 브랜치 생성
git checkout -b feature/login

# 3. 작업 후 커밋
git add .
git commit -m "feat: 로그인 API 추가"

# 4. 원격에 push (최초엔 -u)
git push -u origin feature/login

# 5. PR 생성 (GitHub CLI)
gh pr create --base main --head feature/login \
  --title "feat: 로그인 API 추가" \
  --body "로그인 엔드포인트를 추가했습니다."

# 6. 머지 후 로컬 정리
git checkout main
git pull
git fetch --prune               # 삭제된 원격 브랜치 참조 정리
git branch -d feature/login     # 로컬 작업 브랜치 삭제
```

### 포크(fork) 저장소에서 주의할 점

포크에서 PR을 열면 GitHub이 **base를 원본 저장소로 기본 설정**한다.
내 포크 안에서만 테스트하려면 base를 명시해야 한다.

```bash
gh pr create --repo mgyokim/99_git --base main --head 0904_mgyo
#            └─ 원본(WONIT-A)이 아닌 내 포크로 고정
```

PR 페이지에서 확인하는 법:

```
wants to merge 2 commits into main from 0904_mgyo                          ← 내 저장소 안
wants to merge 2 commits into WONIT-A/99_git:main from mgyokim:0904_mgyo   ← 원본으로 가는 PR
```

---

## 11. 자주 만나는 에러

### `The upstream branch of your current branch does not match the name of your current branch`

```
fatal: The upstream branch of your current branch does not match
the name of your current branch. ...
    git push origin HEAD:main
```

**원인** — 로컬 브랜치 이름과 연결된 upstream 이름이 다름.
`git branch -m`으로 이름을 바꾸면 **upstream 설정이 그대로 승계**되기 때문에 자주 발생한다.

```bash
git branch -vv
# * 0904_mgyo e519f92 [origin/main: ahead 1]
#                      └─ 이름이 안 맞음
```

**해결**

```bash
git push -u origin 0904_mgyo     # 같은 이름으로 원격에 만들고 upstream 재설정
git branch --unset-upstream      # 또는 연결만 끊기
```

### `The current branch xxx has no upstream branch`

```bash
git push -u origin feature/login   # 안내대로 -u 붙여서 push
```

### `Updates were rejected because the remote contains work that you do not have locally`

```bash
git pull --rebase       # 원격 변경을 먼저 가져와 내 커밋을 그 위에 재배치
git push
```

### 머지 충돌 (conflict)

```bash
git status                  # 충돌 파일 확인
# 파일 열어서 <<<<<<< ======= >>>>>>> 마커 정리
git add 충돌파일
git commit                  # 머지 완료

git merge --abort           # 충돌 해결 포기, 머지 이전으로 복구
```

### detached HEAD 상태에 빠졌을 때

```bash
git status
# HEAD detached at e519f92

git checkout main           # 브랜치로 복귀 (여기서 한 커밋은 사라짐)
git checkout -b temp        # 여기서 한 작업을 살리려면 브랜치를 만들 것
```

---

## 참고

- [Pro Git (한국어)](https://git-scm.com/book/ko/v2)
- [Git 공식 문서](https://git-scm.com/docs)
- [GitHub CLI](https://cli.github.com/manual/)
09:21 여러분의 FORK 후 새로 데이터가 업데이트 되었습니다.
09:35 0904_seungyeon에서 작업한 내용입니다.

09:36 노서현의 0904_seohyun branch에서 작업한 내용입니다.
09:35 김예지의 0904_lecture branch에서 작업한 내용입니다.

# git stash: 다른 브랜치로 넘어갈 때 임시저장 (commit 권장)
09:35 이승태의 0904_seungtae branch에서 작업한 내용입니다.

09:35 김소정의 0904_sojung branch에서 작업한 내용입니다.

10:17 두 번째 Pull request 입니다.
