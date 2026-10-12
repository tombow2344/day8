# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 현재 상태

아직 코드가 없다. 저장소에는 기획 문서 두 개만 있다.

- [PRD.md](PRD.md): 무엇을 만드는지 (요구사항, 페이지 구성, 미정 목록)
- [STEPS.md](STEPS.md): 만드는 순서와 단계별 확인 체크리스트

구현을 시작하기 전에 두 문서를 읽는다. STEPS.md의 단계 순서대로 **한 번에 한 페이지씩** 만들고, 그 단계의 확인 항목을 통과한 뒤에 다음 단계로 간다.

빌드·린트·테스트 명령은 없다. npm을 쓰지 않는다. 확인은 브라우저에서 `index.html`을 열어서 한다.

## 아키텍처

골목카페 사장님용 운영 대시보드. 정적 HTML 5개와 Supabase로 이루어진다.

- **iframe 틀**: `index.html`이 왼쪽 메뉴와 가운데 iframe을 가진 틀이면서 홈 화면(숫자 카드, 7일 매출 그래프)이다. `sales.html`, `menu.html`, `reviews.html`, `promo.html`은 이 iframe 안에서 열린다. 그래서 각 페이지는 iframe 안에서도, 단독으로 열어도 동작해야 한다.
- **파일 하나에 페이지 하나**: CSS와 JS를 HTML 안에 넣는다. 모든 페이지가 같은 색·글꼴을 쓰도록 각 파일 맨 위에 같은 CSS 변수 블록을 둔다. 기준은 `index.html`이고, 값을 바꾸면 5개 파일을 모두 맞춘다.
- **데이터**: 사장님 본인의 Supabase 프로젝트를 브라우저에서 anon key로 직접 호출한다. 테이블 초안은 `menu_items`, `sales`, `reviews`이고, 페이지 사이의 연결은 아래와 같다.
  - `menu_items` → `sales.html`의 판매 입력 선택지
  - `sales`, `menu_items`, `reviews` → 홈 숫자 카드와 7일 그래프
- **AI**: Edge Function `cafe-ai` 하나가 리뷰 답글과 홍보문구를 모두 만든다. 요청에 종류를 같이 보내 구분한다. AI 기능을 더할 때도 새 함수를 만들지 말고 이 함수에 종류를 추가한다.

## 지켜야 할 것

- anon key는 HTML에 들어가므로 누구나 볼 수 있다. 데이터 보호는 Supabase RLS에 달려 있다. AI 키와 service role key는 Edge Function secret에만 두고 HTML·문서·채팅에 적지 않는다.
- 「오늘」과 날짜별 집계는 기기 시간대가 아니라 한국 시간(Asia/Seoul) 기준이다. `index.html`의 `kstDay()` 같은 헬퍼를 쓰고, 서버 쪽 집계 함수(`sales_by_day`)에는 `p_tz`로 `Asia/Seoul`을 넘긴다.
- npm, 번들러, 별도 빌드 단계를 추가하지 않는다.
- PRD.md의 「정해야 할 것」은 임의로 정하지 말고 사용자에게 확인한다. 특히 로그인 여부와 판매 입력 시 재고 자동 차감 여부는 데이터 구조에 영향을 준다.
