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
