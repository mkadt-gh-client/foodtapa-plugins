# 외부 원고 검수 API 기준

이 문서는 2026-11-25에 사이트 코드로 확인한 내용이다. 사이트 API는 바뀔 수 있으므로 아래 "판단 기준"의 순서대로 판단한다.

## 판단 기준

1. **사이트 응답이 가장 우선이다.** 이 문서와 실제 응답이 다르면 응답을 따른다. 오류는 코드 이름을 외워 판단하지 않고 HTTP 상태, `message`, `data.field`(위치), `data.reason`(이유)을 읽어 판단한다.
2. **경로가 바뀌면 사이트가 알려 준다.** 이 문서의 경로에 보냈는데 `404`와 `rest_no_route`가 오면 경로가 없어진 것이다. 보내지 말고 "사이트 API가 바뀌어 스킬 업데이트가 필요합니다"라고 알린 뒤 멈춘다. 허용값이 틀리면 `400`과 `rest_invalid_param`이 오고 `message`에 허용값이 나온다. 그 값으로 고친다.
3. **GET 요청을 보내지 않는다.** 사이트 캐시가 인증한 GET 응답을 저장해 로그인하지 않은 사람에게 보여 줄 수 있다. 외부 원고 검수와 AI 상태 확인은 모두 POST로 보낸다. 예외는 아래 4의 분류 목록 조회뿐이다(공개된 분류 이름이고 사이트가 캐시하지 않는 것을 확인했다).
4. **원고 JSON 형식은 사이트가 받은 뒤 검사한다.** 그래서 422 응답의 위치·이유가 형식의 기준이다. 이 문서에 없는 필드를 요구받으면 위치·이유대로 고칠 방법을 정한다. 그 값이 노션 페이지에 없으면 만들지 않고 묻는다.
5. **분류 이름은 사이트에 등록된 목록으로 확인한다.** 축마다 아래로 조회한다. 100개가 넘으면 `page=2`로 이어서 읽는다.

   ```sh
   set -a; . ./foodtapa-api.env; set +a
   for t in product brand model topic situation; do
     printf '%s: ' "$t"
     printf 'user = "%s:%s"\n' "$FOODTAPA_USER" "$FOODTAPA_APP_PASSWORD" | curl -sS -K - \
       "$FOODTAPA_SITE/wp-json/wp/v2/fdt-$t?per_page=100&_fields=name" | jq -r 'map(.name)|join(", ")'
   done
   ```

6. **응답에 이 문서에 없는 항목이 더 있으면** 버리지 말고 보고 표에 함께 보여 준다. 검수 항목(`checks`)에 처음 보는 `key`가 있으면 그 `label`을 그대로 쓴다.
7. **처음 보는 상태나 오류**는 응답 내용을 그대로 알리고 멈춘다. 뜻을 짐작해 다음 단계로 넘어가지 않는다.
8. 같은 응답이 이 문서와 계속 다르면 스킬이 오래된 것일 수 있다. "음처기넷 스킬 설치 확인해줘"로 최신 버전인지 확인하라고 안내한다.

## 인증과 공통 사항

- 모든 요청은 `foodtapa-api.env`의 아이디와 애플리케이션 비밀번호로 인증한다. 인증은 메인 스킬 "전송 방법"처럼 표준 입력으로만 넘긴다.
- 아래 경로는 모두 **POST**다. 본문이 없는 요청도 `{}`를 보낸다.
- 기본 주소: `api="$FOODTAPA_SITE/wp-json/fdt/v1/workflows/external-reviews"`

## 경로

| 경로 | 하는 일 | 보내는 값 | 성공 응답 |
| --- | --- | --- | --- |
| `$api` | 원고 접수 | 원고 JSON | `202` 새 접수, `200` 같은 원고·버전 재전송(기존 결과) |
| `$api/images` | 사진 한 장 올리기 | `file` 폼 필드 | `201` 새로 올림, `200` 같은 사진 |
| `$api/query` | 접수 목록 조회 | 아래 "조회 조건" | 접수 목록 배열 |
| `$api/{id}/status` | 접수 한 건 상태·결과 | `{}` | 접수 한 건 |
| `$api/{id}/retry` | 오류 난 검수 다시 실행 | `{}` | `202`, `status`가 `queued` |
| `$api/{id}/override` | 관리자 허용 | `{"reason":"사유"}` | `200`, `status`가 `passed` |
| `$api/{id}/publish` | 발행 승인과 글 등록 | `{"status":"…","date":"…"}` | `200`, 글 번호와 상태 |
| `$FOODTAPA_SITE/wp-json/fdt/v1/workflows/ai-status/check` | AI 연결 확인 | `{}` | 제공사별 키·연결 상태 |

`$api/{id}/prepare`와 `$api/{id}/approval-drafts`는 관리자 화면이 쓰는 중간 단계다. 발행 요청이 이 둘을 함께 처리하므로 직접 부르지 않는다.

## 조회 조건 (`$api/query`)

| 값 | 뜻 |
| --- | --- |
| `search` | 제목·원자료 식별키 검색어(100자 이내). 원고 ID를 넣어 그 원고의 접수를 찾는다 |
| `source_id` | 원자료 ID로 거르기 |
| `view` | 상태 탭. `checking` 검수 중·오류, `revise` 수정·근거 보완, `ready` 승인 전, `approval` 승인 대기, `draft` 임시저장, `future` 예약, `publish` 공개. 비우면 전체 |
| `type` / `mode` | 글 종류 / 유형으로 거르기 |
| `orderby` / `order` | `id`·`updated_at`·`title` / `asc`·`desc`. 기본은 최근 접수 순 |
| `page` / `per_page` | 쪽 번호 / 쪽당 건수(최대 100) |

결과에서 `manuscript_id`가 정확히 같은 항목만 그 원고의 접수다. 검색어는 부분 일치이므로 다른 원고가 섞일 수 있다.

## 접수 한 건의 내용 (`status` 응답)

| 항목 | 뜻 |
| --- | --- |
| `id` | 접수 번호 |
| `source_id`, `manuscript_id`, `version` | 보낸 원자료 ID, 원고 ID, 버전 |
| `status` | `queued` 대기, `running` 검수 중, `passed` 통과, `needs_revision` 수정 필요, `insufficient_evidence` 근거 부족, `error` 오류 |
| `checks` | 항목별 결과. `key`·`label`·`state`. `state`는 `pass` 충족, `fail` 미충족, `na` 해당 없음, `skipped` 검사 전 |
| `issues` | 지적 목록. `check` 항목, `target` 위치(`post`, 댓글 `id`, 사진 `id`), `excerpt` 문제 문장, `reason` 이유, `suggestion` 고칠 방향 |
| `error` | `status`가 `error`일 때의 안내 문구 |
| `override` | 관리자 허용 기록(허용한 사람·시각·사유). 허용하지 않았으면 없음 |
| `payload` | 접수한 원고. 사진은 `payload.post.images`에 `sha256`이 있다 |

`checks`의 `key`는 지금 다음과 같다. 앞의 다섯은 서버 규칙 검사이고, 여기서 미충족이 나오면 뒤의 AI 검사는 `skipped`로 남는다.

| key | 항목 |
| --- | --- |
| `privacy` | 개인정보 |
| `copy` | 근거 원문 복사 |
| `length` | 본문 350자 |
| `keyword` | 제목 키워드 1회 |
| `guide` | 정보글 구조 |
| `flow` | 문장 연결 |
| `comment_repeat` | 댓글 반복 |
| `reply_context` | 대댓글 맥락 |
| `evidence` | 사실 근거 |
| `image` | 사진 개인정보 |
| `other` | 기타 기준 |

## 발행 (`$api/{id}/publish`)

| `status` | 뜻 | `date` |
| --- | --- | --- |
| `future` | 예약 | 필수. 시간대를 붙인 미래 시각(예: `2026-12-01T09:00:00+09:00`) |
| `publish` | 지금 공개 | 넣지 않는다 |
| `draft` | 임시저장 | 넣지 않는다 |
| `pending` | 검토 대기 | 넣지 않는다 |

- 응답: `post_id` 글 번호, `status` 글 상태, `date_gmt` UTC 시각(예약·공개일 때), `edit_url` 관리자 편집 주소, `comments_approved` 함께 승인한 댓글 수, `comments_skipped` 관리자가 보류·폐기해 건너뛴 댓글 수.
- 승인한 댓글은 글이 공개될 때 함께 등록된다. 예약 글에는 공개 전까지 댓글이 보이지 않는다.
- 같은 접수에 다시 보내면 같은 글의 상태와 시각만 바뀐다.
- 주제 묶음(비교·추천, 불만·고장, 실사용 후기)은 사이트가 글 내용을 보고 정한다. 보낸 주제와 다르게 붙을 수 있으며 오류가 아니다.

## AI 상태 (`ai-status/check`)

검수가 `error`로 끝났을 때 원인을 찾는 데 쓴다. 등록된 키로 모델 조회만 해 보고 결과를 돌려준다. 글을 만들지 않으므로 생성 비용이 들지 않는다. 키 값은 응답에 나오지 않는다. `GET .../ai-status`는 위 "판단 기준" 3 때문에 쓰지 않는다.

```sh
printf 'user = "%s:%s"\n' "$FOODTAPA_USER" "$FOODTAPA_APP_PASSWORD" | curl -sS -K - \
  -H 'Content-Type: application/json' --data '{}' "$FOODTAPA_SITE/wp-json/fdt/v1/workflows/ai-status/check" |
  jq '.providers | map_values({key_configured, health, models: (.models | map_values({state, error, last_request}))})'
```

- `providers`에 제공사(`anthropic`, `openai`)별로 `key_configured` 키 등록 여부, `health.label` 상태, `models` 이 사이트가 쓰는 모델별 연결 기록이 있다.
- `health.label`은 지금 `미등록`, `키를 읽지 못함`, `사용 중인 단계 없음`, `연결 실패`, `연결됨`, `연결 확인 필요` 중 하나다. 판단은 `foodtapa-review-errors` 스킬의 "검수가 오류로 끝남"을 따른다.
