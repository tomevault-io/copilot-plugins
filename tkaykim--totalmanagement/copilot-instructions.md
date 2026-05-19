## rules

> 필수 : 0. 모든 기능, 페이지, api, 컴포넌트, 모달을 구현할 때에는 한 페이지에 와르르 집어넣지 말고, 적절히 파일을 분리하여 한 tsx파일이나 페이지, api문서가 너무 길어지지 않도록 적절히 분리하여 구성하세요.

필수 : 0. 모든 기능, 페이지, api, 컴포넌트, 모달을 구현할 때에는 한 페이지에 와르르 집어넣지 말고, 적절히 파일을 분리하여 한 tsx파일이나 페이지, api문서가 너무 길어지지 않도록 적절히 분리하여 구성하세요.

**배포는 꼭 필요할 때에만 하고 사용자가 테스트를 마치고 배포를 승인했거나, 본인이 Playwright mcp를 활용하여 직접 눈으로 보고, supbase mcp를 통해 DB에도 반영이 되었는지 체크를 한 후에 배포해야합니다.**

1. 특정 기능 구현이나 수정이 완료되면, 실제 동작하는지 확인할 때에는 'playwright MCP'를 활용하여 직접 확인하고 디버깅까지 마친 후 보고할 것. 이때, 실제로 브라우저를 띄워 사용자도 모니터링 과정을 눈으로 직접 볼 수 있게 작동할 것.

2. sql의 수정이 필요하거나, db와의 연동상태, CRUD 상태에 대한 결과를 확인할 때에는 'supabase mcp'를 통해 직접 수행하라. 현재 DB 스키마 구조를 이해하고자 할 때에도 MCP를 연결하여 현재 DB상태를 직접 확인하고 고려하라.

3. 배포를 명령받은 경우에만 npm run build를 직접 수행하고 오류가 없는지 확인 후, 디버깅 완벽하게 해서 오류가 없는 경우에 git에 add, commit, push까지 직접 진행하고 배포에 오류가 난다면 다시 build 수정해서 npm run build 후 완벽하게 작동하는지 체크 한 다음 다시 git push 반복하여 정상 빌드 될때까지 반복하라.

기능 구현을 마치면 0번을 점검하고, 1번까지 수행한 다음 보고할 것

---
> Source: [tkaykim/totalmanagement](https://github.com/tkaykim/totalmanagement) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
