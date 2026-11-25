---
name: foodtapa-review-status
description: 노션 원고나 원고 ID·접수 번호로 푸드타파 사이트의 외부 원고 검수 결과를 조회해 항목별로 보고한다. "검수 결과 확인해 주세요", "검수 끝났어요?", "이 원고 지금 어떤 상태예요?"처럼 결과·상태만 물을 때 사용한다. 원고를 다시 보내지 않는다.
---

# 검수 결과 확인

원고를 보내지 않고, 이미 접수된 원고의 최신 검수 결과와 발행 상태를 알려 준다.

## 시작 전에

- 이 스킬은 푸드타파 플러그인의 세부 스킬이다. 작업 폴더와 접속 정보, 보낼 사이트, 지켜야 할 규칙, 실행 환경, 전송 방법(인증)은 메인 스킬 `foodtapa-notion-review`의 같은 이름 절을 따른다. 이 대화에서 아직 읽지 않았으면 이 스킬 폴더 옆의 `../foodtapa-notion-review/SKILL.md`에서 그 절을 먼저 읽는다.
- 사이트 API 판단 기준은 `../foodtapa-notion-review/references/api.md`를 따른다. 사이트 API는 바뀔 수 있으므로 판단은 사이트 응답, 사이트가 알려 주는 목록, 이 지침 순서로 한다.
- 사이트가 `200`·`201`·`202`가 아닌 응답을 주면 `foodtapa-review-errors` 스킬을 불러 따른다. 부를 수 없으면 `../foodtapa-review-errors/SKILL.md`를 읽는다.
- 아래 명령의 `$api`는 메인 스킬 "전송 방법"과 같이 정한다.

  ```sh
  set -a; . ./foodtapa-api.env; set +a
  api="$FOODTAPA_SITE/wp-json/fdt/v1/workflows/external-reviews"
  ```

## 최신 접수 찾기

1. 찾을 원고 ID를 정한다. 노션 링크를 받았으면 그 페이지 하나의 관리 정보에서 원고 ID를 읽는다. 원고 ID나 접수 번호를 직접 받았으면 그대로 쓴다. 셋 다 없으면 그것 하나만 묻는다.
2. 접수 번호를 받았으면 바로 "상태 조회"로 간다. 원고 ID면 아래로 조회한다.

   ```sh
   printf 'user = "%s:%s"\n' "$FOODTAPA_USER" "$FOODTAPA_APP_PASSWORD" | curl -sS -K - \
     -H 'Content-Type: application/json' --data '{"search":"원고 ID","orderby":"id","order":"desc"}' \
     "$api/query"
   ```

3. 결과에서 `manuscript_id`가 정확히 같은 첫 항목이 최신 접수다. 검색은 부분 일치라 다른 원고가 섞일 수 있으니 반드시 같은지 본다. 같은 항목이 없으면 "이 원고 ID로 접수된 원고가 없습니다"라고 알린다.

## 상태 조회

`"$api/{접수 id}/status"`에 `{}`를 POST로 보낸다.

- `queued`·`running`이면 검수 중이다. 사람이 기다려 달라고 하면 1분 간격으로 최대 10번 다시 조회한다. 그 안에 끝나지 않으면 `foodtapa-review-errors` 스킬의 "검수가 오류로 끝남" 마지막 문단대로 안내한다.
- `error`면 `foodtapa-review-errors` 스킬의 "검수가 오류로 끝남"으로 원인을 찾는다.

## 보고

답변 맨 위에 아래 제목을 단다.

```markdown
---

# 🔎 검수 결과 확인

---
```

| 항목 | 값 |
| --- | --- |
| 보낼 사이트 | |
| 원고 ID / 버전 | |
| 접수 번호 | |
| 검수 상태 | 통과·수정 필요·근거 부족·검수 중·오류 |
| 발행 상태 | 조회 결과의 `publish.state`. `ready` 승인 전, `stale` 새 버전 있음, `approval` 승인 대기, `hold` 보류, `discarded` 폐기, `draft` 임시저장, `pending` 검토 대기, `future` 예약(`publish.date`), `publish` 공개. 비어 있으면 "아직 없음" |
| 관리자 허용 | `override`가 있으면 허용한 사람·시각·사유 |

상태 조회 응답의 `checks`를 표로 보여 준다. `state`는 `pass` 충족, `fail` 미충족, `na` 해당 없음, `skipped` 검사 전으로 옮긴다. `issues`가 있으면 위치(`target`), 이유(`reason`), 고칠 방향(`suggestion`)을 표로 덧붙인다.

마지막에 상태에 맞는 다음 요청을 한 줄로 안내한다.

| 상태 | 안내 |
| --- | --- |
| 수정 필요·근거 부족 | "**노션 원고 수정 반영해 주세요**로 고칠 수 있습니다. 판정이 맞지 않으면 사유와 함께 **관리자 허용**을 요청할 수 있습니다." |
| 통과, 발행 상태 없음 | "노션 페이지에 예약 일시를 적고 **발행해 주세요**로 요청할 수 있습니다." |
| 오류 | 위 "상태 조회"의 오류 진단 결과 |
