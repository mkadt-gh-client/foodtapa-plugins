# 푸드타파 스킬

노션 원고 한 편을 푸드타파 사이트의 외부 원고 검수로 보내는 스킬입니다. Claude 데스크탑 앱과 Codex에서 함께 씁니다.

## Claude 데스크탑 앱 (Code 탭)

1. Code 탭 입력창에 아래를 차례로 보냅니다.

   ```text
   /plugin marketplace add mkadt-gh-client/foodtapa-plugins
   /plugin install foodtapa@foodtapa-plugins
   ```

2. 자동 업데이트를 켭니다. 입력창에 `/plugin`을 보내고 **Marketplaces**에서 `foodtapa-plugins`를 골라 자동 업데이트를 켭니다.

## Codex

터미널에서 아래를 실행합니다.

```sh
codex plugin marketplace add mkadt-gh-client/foodtapa-plugins
codex plugin add foodtapa@foodtapa-plugins
```

업데이트는 아래 두 줄을 다시 실행합니다.

```sh
codex plugin marketplace upgrade foodtapa-plugins
codex plugin add foodtapa@foodtapa-plugins
```

## 설치 확인

새 대화에서 **푸드타파 스킬 설치 확인해줘**라고 요청합니다.

## 처음 써 보기 (튜토리얼)

설치한 뒤 새 대화 입력창에 아래를 보냅니다. 노션 없이 샘플 원고로 접속 설정부터 검수 보내기, 결과 확인, 발행 연습(임시저장)까지 따라 해 볼 수 있습니다.

```text
/foodtapa:foodtapa-tutorial
```

Codex에서는 **푸드타파 튜토리얼 시작해 주세요**라고 요청합니다.

## 처음 설정

새 대화에서 **푸드타파 접속 설정해 주세요**라고 요청합니다. AI가 접속 정보 파일을 열어 주면 아이디와 애플리케이션 비밀번호를 그 파일에 직접 붙여 넣고 저장합니다. 아이디와 비밀번호는 대화창에 입력하지 않습니다.
