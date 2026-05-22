# GitHub Pages 사용법 및 배포 가이드

이 문서는 현재 구축된 Docsify 기반의 스터디 위키 사이트를 **GitHub Pages**에 무료로 호스팅하고, 웹상에서 누구나 접속할 수 있도록 배포하는 단계를 단계별로 설명합니다.

---

## 1. 사전 준비 사항
* **GitHub 계정**이 필요합니다.
* 컴퓨터에 **Git**이 설치되어 있고 기본 설정이 되어 있어야 합니다.
* 본 워크스페이스에 설치된 파일들 (`index.html`, `README.md`, `_sidebar.md`, `.nojekyll`, `study/` 등)이 온전히 있어야 합니다.

---

## 2. 1단계: GitHub 저장소(Repository) 생성
1. [GitHub](https://github.com)에 로그인합니다.
2. 우측 상단의 **[+]** 버튼을 누르고 **New repository**를 클릭합니다.
3. 다음과 같이 설정합니다:
   * **Repository name:** 원하는 이름 입력 (예: `my-study-wiki`)
   * **Public/Private:** 무료 계정인 경우 **Public**으로 설정해야만 Pages 배포가 가능합니다. (유료 계정은 Private도 가능)
   * **Initialize this repository with:** 모두 선택 해제 상태로 둡니다 (Readme, Gitignore 추가 안 함).
4. **Create repository** 버튼을 누릅니다.

---

## 3. 2단계: 로컬 코드를 GitHub에 업로드 (Git Push)
현재 작업 공간인 `/mnt/e/ai` 폴더로 터미널을 열고 다음 명령어를 순서대로 실행합니다.

```bash
# 1. Git 저장소로 초기화
git init

# 2. 모든 파일 상태 추적 추가 (.nojekyll, index.html 등이 포함됨)
git add .

# 3. 첫 번째 커밋 생성
git commit -m "Initialize Docsify Wiki"

# 4. 기본 브랜치 이름을 main으로 변경
git branch -M main

# 5. 내 GitHub 저장소와 로컬 저장소 연결 (username과 repository-name에 본인 값 대입)
git remote add origin https://github.com/<your-username>/<repository-name>.git

# 6. GitHub에 소스 코드 푸시
git push -u origin main
```

---

## 4. 3단계: GitHub Pages 기능 활성화
코드가 성공적으로 업로드되었다면, 웹 브라우저에서 방금 만든 GitHub 저장소 페이지로 이동하여 설정합니다.

1. 저장소 상단 메뉴에서 **Settings (설정)** 톱니바퀴 아이콘을 클릭합니다.
2. 좌측 사이드바 메뉴에서 **Code and automation** 섹션 아래의 **Pages**를 클릭합니다.
3. **Build and deployment** 섹션의 **Source** 설정을 확인합니다:
   * **Source:** `Deploy from a branch` (기본값)
4. **Branch** 설정에서 배포할 브랜치와 폴더를 지정합니다:
   * 브랜치를 **`main`** (또는 코드 푸시한 기본 브랜치)으로 설정합니다.
   * 폴더는 **`/ (root)`**로 설정합니다. (Docsify 설정 파일들이 최상위 루트에 있으므로)
5. 오른쪽에 있는 **Save (저장)** 버튼을 누릅니다.

---

## 5. 4단계: 배포 확인 및 접속
1. 배포 설정이 완료되면 GitHub Actions가 배포를 시작합니다. (보통 1~2분 소요)
2. 페이지 상단에 녹색 체크 표시와 함께 웹사이트 주소가 나타납니다:
   > **Your site is live at:** `https://<your-username>.github.io/<repository-name>/`
3. 해당 URL을 클릭하면 웹 브라우저에서 나만의 위키 사이트가 전 세계로 서비스 중인 것을 확인할 수 있습니다.

---

## 6. 실시간 업데이트 워크플로우 (이후 작업 흐름)
위키에 새로운 글을 추가하거나 내용을 수정한 후 웹사이트에 반영하려면, 터미널에서 다음 세 줄만 실행하면 됩니다.

```bash
git add .
git commit -m "docs: 신규 스터디 문서 추가"
git push origin main
```
*푸시가 완료되면 GitHub Pages가 자동으로 업데이트를 감지해 수 분 내로 사이트에 반영해 줍니다.*

---

> [!IMPORTANT]
> **Jekyll 빌드 우회 확인 (.nojekyll)**
> GitHub Pages는 기본적으로 Jekyll이라는 정적 사이트 빌더를 구동해 렌더링을 시도합니다. 이 때문에 Docsify처럼 프론트엔드 단에서 마크다운을 해석하는 도구들은 밑줄(`_`)로 시작하는 파일(예: `_sidebar.md`)을 읽지 못하는 문제가 발생할 수 있습니다.
> 이를 방지하기 위해 생성해 둔 [**.nojekyll**](file:///mnt/e/ai/.nojekyll) 파일이 반드시 루트 경로에 함께 push되어야 정상 작동합니다.
