# How to Use VSCode's Trigger Suggest Shortcut (cmd+i) in Cursor Editor

Cursor 에디터를 사용하다 보면 VSCode의 기본 자동완성 단축키인 cmd+i가 작동하지 않는 것을 발견할 수 있습니다. Cursor에서 해당 단축키를 AI 채팅/컴포저 기능에 바인딩했기 때문입니다.

간단한 설정으로 이 문제를 해결할 수 있습니다.

Cursor의 키바인딩 설정 파일을 엽니다. shift+cmd+p를 눌러 Command Palette 에서 "Open Keyboard Shortcuts (JSON)" 검색합니다.

다음 코드를 추가합니다:

```json
{
  "key": "cmd+'",
  "command": "editor.action.triggerSuggest",
  "when": "editorHasCompletionItemProvider && textInputFocus && !editorReadonly && !suggestWidgetVisible"
}
```

이렇게 하면 `cmd+'` 단축키로 자동완성 기능을 사용할 수 있습니다. 다른 키 조합으로도 변경이 가능합니다.
