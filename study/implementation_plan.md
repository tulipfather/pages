# 위키(Docsify) 구축 계획서

이 계획서는 워크스페이스 내의 마크다운(.md) 파일들을 웹 브라우저에서 아름다운 웹페이지(위키) 형태로 볼 수 있도록 Docsify를 설정하는 계획입니다.

Docsify는 빌드 단계가 필요 없는 초경량 정적 문서 사이트 발전기(Static Site Generator)로, 브라우저가 직접 마크다운 파일을 로드하여 렌더링하기 때문에 구조가 매우 단순하고 확장이 쉽습니다.

---

## 제안하는 변경 사항

프로젝트 루트 디렉토리 `/mnt/e/ai`에 다음 파일들을 생성합니다.

### 1. [NEW] [index.html](file:///mnt/e/ai/index.html)
Docsify의 설정과 메인 진입점 역할을 하는 파일입니다.
- **디자인 테마:** 모던하고 세련된 `docsify-themeable` 테마를 사용하여 프리미엄 느낌의 타이포그래피와 레이아웃을 구현합니다.
- **주요 플러그인 탑재:**
  - **검색(Search):** 한국어 검색이 잘 작동하도록 설정합니다.
  - **코드 복사(Copy to Clipboard):** 코드 블록의 우측 상단에 복사 버튼을 추가합니다.
  - **이미지 확대(Zoom Image):** 이미지 클릭 시 부드럽게 확대하는 기능을 탑재합니다.
  - **다크 모드(Dark Mode):** 라이트/다크 테마 전환 기능을 탑재합니다.

### 2. [NEW] [_sidebar.md](file:///mnt/e/ai/_sidebar.md)
좌측 사이드바 메뉴를 구성합니다. 작성한 마크다운 문서들(예: `study/github_pages.md`)로 바로 갈 수 있는 링크를 체계적으로 정리합니다.

### 3. [NEW] [README.md](file:///mnt/e/ai/README.md)
위키의 메인 홈 화면에 노출될 대시보드 문서입니다. 환영 메시지와 단축 링크 등을 배치합니다.

### 4. [NEW] [.nojekyll](file:///mnt/e/ai/.nojekyll)
GitHub Pages에 배포할 때 Jekyll 빌드 엔진을 건너뛰고 Docsify가 정상적으로 작동하도록 하는 빈 설정 파일입니다.

---

## 검증 계획

### 1. 로컬 실행 검증
- 터미널에서 다음 명령어를 실행하여 로컬 서버를 구동합니다:
  ```bash
  npx docsify-cli serve .
  ```
- 브라우저로 `http://localhost:3000`에 접속하여 다음 사항을 확인합니다:
  - 사이트 디자인 및 한글 폰트 적용 여부
  - 사이드바 메뉴 구성 및 [study/github_pages.md](file:///mnt/e/ai/study/github_pages.md) 문서 정상 렌더링 여부
  - 실시간 문서 검색 및 다크 모드 전환 작동 여부
