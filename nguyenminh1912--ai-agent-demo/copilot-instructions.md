## restcontroller-conventions

> Specifies standards for RestController classes, including API route mappings, HTTP method annotations, dependency injection, and error handling with ApiResponse and GlobalExceptionHandler.

- Must annotate controller classes with @RestController.
- Must specify class-level API routes with @RequestMapping, e.g. ("/api/user").
- Class methods must use best practice HTTP method annotations, e.g, create = @postMapping("/create"), etc.
- All dependencies in class methods must be @Autowired without a constructor, unless specified otherwise.
- Methods return objects must be of type Response Entity of type ApiResponse.
- All class method logic must be implemented in a try..catch block(s).
- Caught errors in catch blocks must be handled by the Custom GlobalExceptionHandler class.

---
> Source: [NguyenMinh1912/ai-agent-demo](https://github.com/NguyenMinh1912/ai-agent-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
