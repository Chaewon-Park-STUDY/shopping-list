# 쇼핑 리스트

바닐라 HTML/CSS/JS로 만든 단일 파일 쇼핑 리스트 웹앱. 빌드 과정이 없고,
목록은 Supabase(Postgres)에 저장되어 기기 간에 공유됩니다.

**[▶ 바로 실행해 보기](https://chaewon-park-study.github.io/shopping-list/)**

![스크린샷](shopping-list-verified.png)

## 기능

- **추가** — 입력창에 물건 이름을 쓰고 Enter 또는 "추가" 버튼 (빈 입력은 거부)
- **체크** — 체크박스로 완료 표시, 취소선 처리와 함께 "N개 중 M개 완료" 카운터 갱신
- **삭제** — 항목별 `×` 버튼, 그리고 체크된 항목만 골라 지우는 일괄 삭제
- **저장** — Supabase의 `shopping_items` 테이블에 저장되어 어느 기기에서 열어도 같은 목록
- **실시간 동기화** — 다른 탭이나 다른 기기에서 바뀐 내용이 Supabase Realtime으로 즉시 반영
- **다크 모드** — `prefers-color-scheme`로 시스템 설정을 따름

## 실행 방법

로컬 서버로 띄운 뒤 열면 됩니다. 빌드 단계는 없습니다.

```bash
python -m http.server 8137
# http://127.0.0.1:8137/
```

> ES 모듈로 `supabase-js`를 CDN에서 불러오므로 `file://`로 직접 열면 브라우저가
> 스크립트를 막을 수 있습니다. 위처럼 `http://`로 여는 편이 확실합니다.

## 데이터베이스

Supabase 프로젝트의 `shopping_items` 테이블 하나를 씁니다.

| 컬럼 | 타입 | 비고 |
| --- | --- | --- |
| `id` | `uuid` | 기본키, `gen_random_uuid()` |
| `name` | `text` | 1~100자 `check` 제약 |
| `done` | `boolean` | 기본값 `false` |
| `created_at` | `timestamptz` | 기본값 `now()`, 정렬 기준 |

RLS를 켜고 `anon`/`authenticated`에 select·insert·update·delete를 모두 열어 둔
**공개 공유 목록**입니다. 링크를 아는 사람은 누구나 같은 목록을 고칠 수 있으니
개인 데이터를 담으려면 정책부터 좁혀야 합니다.

소스에 들어 있는 publishable key는 브라우저에 노출되는 것이 정상이며,
실제 접근 통제는 위 RLS 정책이 담당합니다.

## 구현 메모

- **XSS 안전** — 항목 이름을 `textContent`로 넣어 HTML이 실행되지 않습니다
- **ID 기반 갱신** — 배열 인덱스가 아닌 DB가 발급한 UUID로 항목을 다뤄 중간 항목 삭제에도 안전합니다
- **낙관적 갱신** — 체크·삭제는 화면을 먼저 바꾸고, 서버가 거부하면 이전 상태로 되돌립니다
- **오류 표시** — 네트워크나 권한 문제로 요청이 실패하면 목록 위에 사유를 띄웁니다

## 테스트

Playwright로 실제 Supabase 프로젝트에 붙여 불러오기·추가·체크·삭제와 카운터 갱신을 확인했습니다.
publishable key로 select·insert·update·delete가 RLS를 통과하는지, 빈 이름이 `check` 제약에
막히는지도 함께 검증했습니다. 자동 재실행 가능한 스펙 파일은 아직 없습니다.

## 라이선스

MIT
