---
name: foodtapa-setup
description: 음처기넷 스킬의 접속 정보 파일(foodtapa-api.env)을 맥·윈도우에 맞게 만들고 편집기로 열어, 사람이 아이디와 애플리케이션 비밀번호를 대화창이 아닌 파일에 직접 넣게 한 뒤 연결을 확인한다. 애플리케이션 비밀번호 발급 화면과 사이트 AI 키 등록도 안내한다. "음처기넷 접속 설정해 주세요", "애플리케이션 비밀번호 어떻게 만들어요?", "비밀번호 바꿨어요", "AI 키 등록해 주세요", "사이트 주소 바꿔 주세요"라고 할 때와 다른 음처기넷 스킬에서 접속 정보가 없거나 비었거나 401·403이 났을 때 사용한다.
---

# 음처기넷 접속 설정

아이디와 애플리케이션 비밀번호는 **사람이 파일에 직접** 넣는다. AI는 파일을 만들고 열어 주고, 넣은 뒤 연결만 확인한다. 값을 대화로 받지 않고, 파일 내용을 화면에 출력하지 않는다.

## 1. 경고를 먼저 보여 준다

어떤 안내보다 먼저 아래 경고를 답변 맨 위에 그대로 보여 준다.

```markdown
> ## ⚠️ 주의
>
> **아이디와 비밀번호를 이 대화창에 입력하거나 붙여 넣지 마세요.**
>
> 대화 내용은 AI 서비스로 전송되고 기록에 남습니다. 지금 열어 드리는 파일에 직접 붙여 넣고 저장만 해 주세요. 저장한 뒤에는 대화창에 **저장했어요**라고만 보내 주세요.
```

## 2. 파일을 만들고 연다

작업 폴더는 맥 `~/foodtapa`, 윈도우 `%USERPROFILE%\foodtapa`다. 파일이 없을 때만 틀을 만든다. 이미 있으면 내용을 바꾸지 않고 열기만 한다. 다만 `FOODTAPA_SITE`가 위 주소와 다르면 아래 "사이트 주소 바꾸기"를 따른다. 틀에는 비밀값을 넣지 않는다.

**맥**

```sh
mkdir -p ~/foodtapa && cd ~/foodtapa
[ -f foodtapa-api.env ] || printf 'FOODTAPA_SITE=https://xn--ok0bu1tj8ag2pewfs6i.net\nFOODTAPA_USER=\nFOODTAPA_APP_PASSWORD=\nNOTION_TOKEN=\n' > foodtapa-api.env
chmod 600 foodtapa-api.env
open -e foodtapa-api.env || open -R foodtapa-api.env
```

**윈도우(PowerShell)**

```powershell
$dir = Join-Path $env:USERPROFILE 'foodtapa'; New-Item -ItemType Directory -Force $dir | Out-Null
$f = Join-Path $dir 'foodtapa-api.env'
if (-not (Test-Path $f)) { [IO.File]::WriteAllLines($f, @('FOODTAPA_SITE=https://xn--ok0bu1tj8ag2pewfs6i.net','FOODTAPA_USER=','FOODTAPA_APP_PASSWORD=','NOTION_TOKEN=')) }
notepad $f
```

- 맥은 텍스트 편집기, 윈도우는 메모장으로 열린다. 열리지 않으면 맥은 Finder에서 파일 위치를 보여 주고(`open -R`), 윈도우는 `explorer /select,"$f"`로 위치를 보여 준다.
- 사람에게 파일의 전체 경로를 함께 알린다. 맥은 `echo ~/foodtapa/foodtapa-api.env`, 윈도우는 `$f`로 확인한 실제 경로를 적는다.

## 3. 넣을 값을 안내한다

아래 안내를 보여 주고 멈춘다. **저장했어요** 답을 받기 전에는 다음 단계로 가지 않는다.

| 줄 | 넣을 값 | 구하는 곳 |
| --- | --- | --- |
| `FOODTAPA_SITE=` | 이미 적혀 있다. 바꾸지 않는다 | |
| `FOODTAPA_USER=` | 사이트 관리자 아이디(로그인할 때 쓰는 아이디) | |
| `FOODTAPA_APP_PASSWORD=` | 애플리케이션 비밀번호 | 아래 "애플리케이션 비밀번호 발급" |
| `NOTION_TOKEN=` | 노션 커넥터를 쓰면 비워 둔다 | |

- `=` 바로 뒤에 붙여 넣는다. 따옴표는 넣지 않는다. 비밀번호의 띄어쓰기는 그대로 붙여 넣어도 된다(다음 단계에서 정리한다).
- 다 넣었으면 저장(맥 `⌘S`, 윈도우 `Ctrl+S`)하고 대화창에 **저장했어요**라고만 보내 달라고 한다.

### 애플리케이션 비밀번호 발급

애플리케이션 비밀번호는 로그인 비밀번호와 다른, AI 앱 전용 비밀번호다. 사이트 관리자 화면에서 사람이 직접 만든다. AI가 대신 만들지 않는다.

1. 사이트 관리자로 **먼저 로그인한다.** 로그인 주소는 사이트 관리자가 알고 있다. 로그인하지 않은 상태로 아래 화면을 열면 404 화면으로 넘어간다.
2. AI가 프로필 화면의 애플리케이션 비밀번호 칸을 열어 준다. 링크 주소도 함께 보여 준다.

   ```sh
   cd ~/foodtapa; set -a; . ./foodtapa-api.env; set +a
   open "$FOODTAPA_SITE/wp-admin/profile.php#application-passwords-section"            # 맥
   # 윈도우: Start-Process "$($cfg.FOODTAPA_SITE)/wp-admin/profile.php#application-passwords-section"
   ```

   윈도우의 `$cfg`는 4단계 윈도우 명령의 앞 네 줄로 만든다.
3. **새 애플리케이션 비밀번호 이름**에 알아보기 쉬운 이름(예: `AI 스킬 - 내 PC`)을 적고 **새 애플리케이션 비밀번호 추가**를 누른다.
4. 화면에 나온 비밀번호(예: `abcd efgh ijkl mnop qrst uvwx`)를 복사해 열려 있는 `foodtapa-api.env`의 `FOODTAPA_APP_PASSWORD=` 뒤에 붙여 넣는다. **이 값은 이 화면에서 한 번만 보인다.** 대화창에는 붙여 넣지 않는다.

- **관리자 계정에서 만든다.** 다른 사람 글 편집과 댓글 관리 권한이 있어야 하고, AI 키 등록(아래 "AI 키 등록" 방법 2)까지 하려면 사이트 관리자여야 한다. `FOODTAPA_USER`에는 이 비밀번호를 만든 계정의 아이디를 넣는다.
- **PC마다 따로 만든다.** 같은 비밀번호를 여러 PC에 복사해 쓰지 않는다. 그래야 PC 하나를 잃어버렸을 때 그 비밀번호만 취소할 수 있다.
- **잃어버렸거나 대화창·메신저에 올라갔으면** 같은 화면 목록에서 그 이름의 **취소**를 누르고 새로 만든다. 취소하면 그 비밀번호로는 바로 접속되지 않는다.
- **애플리케이션 비밀번호 칸이 보이지 않으면** 사이트가 이 기능을 막아 둔 것이다. 사이트 관리자에게 알리라고 한다.

## 4. 저장한 파일을 정리하고 확인한다

**저장했어요**를 받으면 아래를 실행한다. 정리는 윈도우 메모장의 줄바꿈과 사이트 주소·비밀번호 줄의 띄어쓰기를 지운다. 워드프레스는 애플리케이션 비밀번호의 띄어쓰기를 무시하므로 지워도 같은 비밀번호다. 띄어쓰기가 남아 있으면 맥에서 파일을 읽지 못한다.

**맥**

```sh
cd ~/foodtapa
sed -i '' -e $'s/\r$//' -e '/^FOODTAPA_SITE=/s/[[:space:]]//g' -e '/^FOODTAPA_APP_PASSWORD=/s/[[:space:]]//g' foodtapa-api.env
for k in FOODTAPA_SITE FOODTAPA_USER FOODTAPA_APP_PASSWORD; do grep -q "^$k=." foodtapa-api.env && echo "$k 있음" || echo "$k 비어 있음"; done
set -a; . ./foodtapa-api.env; set +a
w="$FOODTAPA_SITE/wp-json/fdt/v1/workflows"
printf 'user = "%s:%s"\n' "$FOODTAPA_USER" "$FOODTAPA_APP_PASSWORD" | curl -sS -K - -o /dev/null -w 'HTTP %{http_code}\n' \
  -H 'Content-Type: application/json' --data '{"per_page":1}' "$w/external-reviews/query"
printf 'user = "%s:%s"\n' "$FOODTAPA_USER" "$FOODTAPA_APP_PASSWORD" | curl -sS -K - \
  -H 'Content-Type: application/json' --data '{}' "$w/ai-status/check" |
  jq -r '.providers | to_entries[] | select((.value.models|length)>0) | "\(.value.label): \(.value.health.label)"'
```

첫 줄의 `HTTP` 코드로 접속을 판정하고, 둘째 결과로 사이트의 AI 연결 상태를 본다. 접속이 `200`이 아니면 둘째 결과는 보지 않는다.

**윈도우(PowerShell)**

```powershell
$f = Join-Path $env:USERPROFILE 'foodtapa\foodtapa-api.env'
$lines = Get-Content $f | ForEach-Object { if ($_ -match '^FOODTAPA_(SITE|APP_PASSWORD)=') { $_ -replace '\s','' } else { $_ } }
[IO.File]::WriteAllLines($f, $lines)
$cfg = @{}; $lines | ForEach-Object { if ($_ -match '^([A-Z_]+)=(.*)$') { $cfg[$Matches[1]] = $Matches[2] } }
'FOODTAPA_SITE','FOODTAPA_USER','FOODTAPA_APP_PASSWORD' | ForEach-Object { if ($cfg[$_]) { "$_ 있음" } else { "$_ 비어 있음" } }
$w = "$($cfg.FOODTAPA_SITE)/wp-json/fdt/v1/workflows"; $auth = "user = `"$($cfg.FOODTAPA_USER):$($cfg.FOODTAPA_APP_PASSWORD)`""
'{"per_page":1}' | Set-Content -NoNewline q.json; '{}' | Set-Content -NoNewline e.json
$auth | curl.exe -sS -K - -o NUL -w 'HTTP %{http_code}\n' -H 'Content-Type: application/json' --data-binary '@q.json' "$w/external-reviews/query"
($auth | curl.exe -sS -K - -H 'Content-Type: application/json' --data-binary '@e.json' "$w/ai-status/check" | ConvertFrom-Json).providers.PSObject.Properties |
  Where-Object { @($_.Value.models.PSObject.Properties).Count -gt 0 } | ForEach-Object { "$($_.Value.label): $($_.Value.health.label)" }
Remove-Item q.json, e.json
```

윈도우는 PowerShell 따옴표가 JSON을 깨뜨리지 않게 요청 본문을 파일로 넘긴다. 어느 쪽이든 아이디·비밀번호 값은 화면에 출력하지 않는다.

| 결과 | 보고 |
| --- | --- |
| 비어 있는 줄이 있음 | 그 줄 이름만 알리고 2단계처럼 파일을 다시 열어 준다 |
| `HTTP 200` | "접속 설정 완료"와 함께 AI 상태를 알린다. 이 사이트가 쓰는 제공사가 `미등록`이면 "검수에 필요한 AI 키가 사이트에 없습니다"라고 알리고 아래 "AI 키 등록"으로 이어 간다 |
| `HTTP 401` | 아이디나 애플리케이션 비밀번호가 틀렸다. 로그인 비밀번호를 넣지 않았는지, 복사가 잘리지 않았는지 확인하고 새로 만들어 넣어 달라고 하며 파일을 다시 열어 준다 |
| `HTTP 403` | 이 계정에 관리자 권한이 없다. 관리자 계정의 아이디와 그 계정에서 만든 애플리케이션 비밀번호로 바꿔 달라고 한다 |
| 그 밖 | 응답 코드를 그대로 알리고 멈춘다 |

## 사이트 주소 바꾸기

파일의 `FOODTAPA_SITE`가 `https://xn--ok0bu1tj8ag2pewfs6i.net`(음식물처리기.net)이 아니면(예전 주소 `https://kincomm.kinsta.cloud` 등) 이 절을 따른다. 사이트 주소는 비밀값이 아니라 AI가 바꾼다. 아이디와 애플리케이션 비밀번호는 사이트마다 따로라 예전 값은 비우고, 사람이 새 사이트의 값을 직접 넣게 한다. `NOTION_TOKEN`은 그대로 둔다.

1. 1단계 경고를 보여 준다.
2. 주소를 바꾸고 아이디·비밀번호 줄을 비운 뒤 파일을 연다. 값은 출력하지 않는다.

   ```sh
   cd ~/foodtapa
   sed -i '' -e $'s/\r$//' -e 's#^FOODTAPA_SITE=.*#FOODTAPA_SITE=https://xn--ok0bu1tj8ag2pewfs6i.net#' -e 's/^FOODTAPA_USER=.*/FOODTAPA_USER=/' -e 's/^FOODTAPA_APP_PASSWORD=.*/FOODTAPA_APP_PASSWORD=/' foodtapa-api.env
   grep '^FOODTAPA_SITE=' foodtapa-api.env
   open -e foodtapa-api.env || open -R foodtapa-api.env
   ```

   윈도우(PowerShell):

   ```powershell
   $f = Join-Path $env:USERPROFILE 'foodtapa\foodtapa-api.env'
   $lines = Get-Content $f | ForEach-Object { $_ -replace '^FOODTAPA_SITE=.*','FOODTAPA_SITE=https://xn--ok0bu1tj8ag2pewfs6i.net' -replace '^FOODTAPA_USER=.*','FOODTAPA_USER=' -replace '^FOODTAPA_APP_PASSWORD=.*','FOODTAPA_APP_PASSWORD=' }
   [IO.File]::WriteAllLines($f, $lines); Select-String '^FOODTAPA_SITE=' $f; notepad $f
   ```

3. "사이트 주소를 음식물처리기.net으로 바꿨습니다. 예전 사이트의 아이디와 애플리케이션 비밀번호는 새 사이트에서 쓸 수 없어 비웠습니다"라고 알리고 3단계 안내(새 사이트에서 애플리케이션 비밀번호 발급 포함)로 이어 간다. 이후 4단계 확인까지 마친다.

## AI 키 등록

사이트에 AI 키가 없으면(AI 상태가 `미등록`) 검수가 모두 오류로 끝난다. "AI 키 등록해 주세요"라고 하거나 위 확인에서 `미등록`이 나오면 이 절을 따른다. AI 키도 아이디·비밀번호처럼 **대화창으로 받지 않는다.** 1단계 경고를 다시 보여 준다.

**어느 회사 키인지**는 위 확인 결과에서 이름이 나온 제공사(Anthropic 또는 OpenAI)다. 둘 다 나오면 둘 다 필요하다.

### 방법 1. 관리자 화면에서 직접 (기본)

사이트 관리자로 **먼저 로그인한 뒤** 아래 링크를 연다. 로그인하지 않은 상태로 열면 404 화면으로 넘어간다. 로그인 주소는 사이트 관리자가 알고 있다.

```sh
set -a; . ./foodtapa-api.env; set +a
open "$FOODTAPA_SITE/wp-admin/admin.php?page=fdtw-ai-connection"            # 맥
# 윈도우: Start-Process "$($cfg.FOODTAPA_SITE)/wp-admin/admin.php?page=fdtw-ai-connection"
```

사람에게 링크 주소도 함께 보여 준다. 화면의 해당 회사 칸에 키를 붙여 넣고 저장한 뒤 **저장했어요**라고 보내 달라고 한다. 받으면 위 4단계의 AI 연결 확인만 다시 실행해 `연결됨`인지 본다.

### 방법 2. AI에게 맡기기 (사람이 원할 때만)

사이트의 키 등록 API(`PUT $FOODTAPA_SITE/wp-json/fdt/v1/workflows/ai-keys/{anthropic|openai}`)로 보낸다. 키는 임시 파일로만 넘기고 끝나면 지운다.

1. 임시 파일을 만들어 연다. 사람에게 키 하나만 붙여 넣고 저장한 뒤 **저장했어요**라고 보내 달라고 한다.

   ```sh
   cd ~/foodtapa && : > ai-key.txt && chmod 600 ai-key.txt && open -e ai-key.txt
   # 윈도우: New-Item -Force ai-key.txt | Out-Null; notepad ai-key.txt
   ```

2. 받으면 보낸다. `p`는 위에서 정한 회사(`anthropic` 또는 `openai`)다. 키 값은 화면에 출력하지 않는다.

   ```sh
   cd ~/foodtapa; set -a; . ./foodtapa-api.env; set +a; p=anthropic
   (umask 077; jq -Rs '{api_key: (gsub("^\\s+|\\s+$"; ""))}' ai-key.txt > ai-key.json)
   printf 'user = "%s:%s"\n' "$FOODTAPA_USER" "$FOODTAPA_APP_PASSWORD" | curl -sS -K - -X PUT \
     -H 'Content-Type: application/json' --data-binary @ai-key.json -w '\nHTTP %{http_code}\n' \
     "$FOODTAPA_SITE/wp-json/fdt/v1/workflows/ai-keys/$p"
   rm -f ai-key.txt ai-key.json
   ```

   윈도우는 `$k = (Get-Content ai-key.txt -Raw).Trim(); @{api_key = $k} | ConvertTo-Json -Compress | Set-Content -NoNewline ai-key.json`로 본문 파일을 만들어 같은 요청을 `curl.exe`로 보내고 `Remove-Item ai-key.txt, ai-key.json`으로 지운다.

3. 응답을 확인한다.

   | 결과 | 보고 |
   | --- | --- |
   | `200`, `key_source`가 `admin` | 등록됨. 4단계의 AI 연결 확인을 다시 실행해 `연결됨`인지 본다. `연결 실패`면 키가 틀렸거나 그 회사 계정의 결제·한도 문제일 수 있다고 알린다 |
   | `200`, `key_source`가 `server` | 사이트 서버에 따로 설정된 키가 우선이라 방금 넣은 키는 쓰이지 않는다. 사이트 관리자에게 알리라고 한다 |
   | `422` | 키가 비었거나 공백이 섞였거나 너무 길다. 파일에 키만 다시 넣게 한다 |
   | `403` | 이 계정은 키를 등록할 권한(사이트 관리자)이 없다. 방법 1로 사이트 관리자가 직접 넣게 한다 |
   | `404` | 회사 이름(`p`)이 틀렸거나 사이트에 이 API가 없다. 방법 1로 안내한다 |
   | 그 밖 | 응답 코드와 `message`를 알리고 멈춘다 |

4. 임시 파일이 지워졌는지(`ls ai-key.*`에 아무것도 없는지) 확인하고 보고한다.

## 대화창에 비밀번호가 올라왔을 때

경고에도 사람이 대화창에 아이디·비밀번호를 붙여 넣으면 다음을 지킨다.

- 그 값을 파일에 옮기거나 명령에 쓰지 않는다. 다시 적거나 인용하지 않는다.
- "방금 대화에 비밀번호가 올라갔습니다. 사이트 관리자 **사용자 → 프로필 → 애플리케이션 비밀번호**에서 그 비밀번호를 **취소**하고, 새로 만든 값을 파일에만 넣어 주세요"라고 알린다.
- 2단계처럼 파일을 다시 열어 준다.

## 끝났을 때

답변 맨 위에 **✅ 접속 설정 완료** 또는 **⚠️ 접속 설정 미완료**를 단다. 완료면 마지막 줄에 "이제 **노션 원고 검수 보내 주세요. 링크: https://…**로 요청할 수 있습니다."를 붙인다.
