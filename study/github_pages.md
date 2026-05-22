# GitHub Pages의 구조와 작동 방식 이해하기

이 문서는 GitHub Pages의 아키텍처, 일반 Git 저장소와의 차이점, 그리고 비공개 설정(보안/공개 범위)에 대해 설명합니다.

---

## 1. GitHub Pages란?
GitHub Pages는 GitHub 저장소(Repository)에서 HTML, CSS, JavaScript 파일을 직접 호스팅하여 웹사이트로 제공하는 **정적 웹사이트 호스팅 서비스**입니다.

## 2. 일반 Git 저장소 vs GitHub Pages 저장소
일반 Git 저장소와 GitHub Pages 활성화 저장소는 근본적으로 동일한 Git 저장소이지만, **GitHub 측의 웹 서버 배포 엔진 연동 여부**에서 차이가 납니다.

| 구분 | 일반 Git 저장소 | GitHub Pages 저장소 |
| :--- | :--- | :--- |
| **목적** | 소스 코드 버전 관리 및 백업 | 정적 웹사이트 호스팅 및 배포 |
| **동작 방식** | 코드 변경 사항만 기록/저장 | 특정 브랜치에 코드가 푸시되면 자동으로 빌드하여 웹 서버에 배포 |
| **액세스 방식** | Git 클라이언트 또는 GitHub 웹 UI를 통한 접근 | 웹 브라우저를 통한 URL 접근 (`https://<username>.github.io/<repo>`) |
| **콘텐츠** | 임의의 소스 코드 전체 | 웹 서버가 인식할 수 있는 정적 파일 (`index.html`, 이미지, JS, CSS 등) |

---

## 3. GitHub Pages의 동작 구조
GitHub Pages는 크게 두 가지 방식으로 정적 웹사이트를 생성하고 서빙합니다.

### A. 도메인 구조
GitHub Pages의 주소는 저장소 이름과 사용자 이름에 따라 결정됩니다.
* **사용자/조직 페이지 (User/Organization Page):**
  * 저장소 이름이 반드시 `<username>.github.io` 형식이어야 합니다.
  * URL: `https://<username>.github.io`
* **프로젝트 페이지 (Project Page):**
  * 일반적인 임의의 저장소에서 활성화합니다.
  * URL: `https://<username>.github.io/<repository-name>`

### B. 빌드 및 배포 흐름
1. **소스 브랜치 설정:** 
   * 웹사이트로 배포할 소스 코드가 들어있는 브랜치(예: `main`, `master`, 또는 `gh-pages`)와 폴더(예: `/` 또는 `/docs`)를 설정합니다.
2. **자동 빌드 (Jekyll 또는 GitHub Actions):**
   * 코드가 해당 브랜치에 푸시(Push)되면 GitHub Pages 배포 워크플로우가 자동으로 트리거됩니다.
   * 기본적으로 Jekyll 빌드 엔진이 내장되어 있어 Markdown 파일을 HTML로 변환해 줍니다. 
   * 최신 방식은 GitHub Actions를 설정하여 React, Vue, Next.js 등의 프레임워크 빌드 결과물(정적 파일)을 직접 배포할 수 있습니다.
3. **정적 파일 서빙:**
   * 빌드가 완료되면 결과물이 GitHub의 Edge CDN을 통해 전 세계 사용자에게 서빙됩니다.

---

## 4. 공개 설정(Visibility)과 접근 제한
GitHub Pages의 가장 중요한 특징 중 하나는 **공개 범위**입니다.

### A. 기본 원칙: "저장소가 Public이면 Pages도 Public이다"
* **무료 계정(Free Plan)의 일반적인 경우:**
  * GitHub Pages를 활성화하려면 저장소(Repository) 자체가 **공개(Public)** 상태여야 합니다.
  * 저장소가 공개 상태이므로 소스 코드와 배포된 웹사이트 모두 인터넷상의 누구나 접근하여 볼 수 있습니다.

### B. 비공개(Private) 저장소에서의 GitHub Pages
* **GitHub Free (무료 계정):** 
  * 무료 계정의 비공개 저장소에서는 GitHub Pages 기능을 활성화할 수 없습니다. 활성화하려면 저장소를 공개(Public)로 전환하라는 안내가 뜹니다.
* **GitHub Pro / GitHub Team / GitHub Enterprise (유료 계정):**
  * 유료 플랜을 사용하는 경우, **비공개(Private) 저장소에서도 GitHub Pages를 활성화할 수 있습니다.**
  * 이때, 배포된 웹사이트의 공개 범위(Access Control)를 설정할 수 있습니다:
    * **Public:** 저장소 코드는 비공개이지만, 배포된 웹사이트 주소(`https://...`)는 전 세계 누구나 접속할 수 있습니다.
    * **Private (Access Control):** 웹사이트도 비공개로 유지됩니다. 이 경우, 해당 GitHub 조직(Organization)의 멤버나 권한이 있는 사용자만 GitHub 계정 로그인을 거쳐 웹사이트에 접속할 수 있습니다.

> [!WARNING]
> 무료 플랜을 사용 중인 상황에서 GitHub Pages를 통해 웹사이트를 서비스하려면 저장소를 **Public(공개)**으로 설정해야만 합니다. 이 경우 소스 코드(코드 내부의 API Key나 비밀번호 등 포함)가 외부에 모두 노출되므로 민감 정보가 포함되지 않도록 주의해야 합니다.
