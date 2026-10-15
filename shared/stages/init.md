# init: SpecSync 초기화

`.specs/principles.md`가 없을 때 한 번 수행한다.

1. 저장소를 훑어 기술 스택과 기존 검증 명령(테스트, 린트, 빌드)을 파악한다.
2. AskUserQuestion으로 Spec 키 접두어를 정한다. 저장소 이름에서 2~4자 대문자 후보를 제안한다.
3. `templates/principles.md`를 읽어 `.specs/principles.md`를 만들고, `{{prefix}}`, `{{date}}`(오늘)와 1에서 찾은 검증 명령을 채운다.
4. 만든 내용을 짧게 보여주고 원래 요청을 이어서 처리한다.
