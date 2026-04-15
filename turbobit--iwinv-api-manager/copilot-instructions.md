## iwinv-api-manager

> - 서버컴포넌트, 크라이언트컴포넌트 구분

# Cursor Rules

## next.js 13 이상
- src/app
- app 라우터 사용
- 서버컴포넌트, 크라이언트컴포넌트 구분

## Project
- https://api.iwinv.kr/doc-637859 에 있는 기능을 next.js 프로젝트로 구현
- 프로젝트 디렉토리 구조는 최대한 유사하게 구성
- 최대한 타입스크립트로 구현
- 최대한 함수형 프로그래밍 방식으로 구현
- 최대한 모듈화하여 구현
- 최대한 컴포넌트 기반으로 구현
- 최대한 비동기 방식으로 구현
- 최대한 테스트 코드 작성

## 컨벤션
- 컴포넌트 이름은 대문자로 시작하고 중간에 대문자가 들어가는 것은 소문자로 변경
- 컴포넌트 파일은 대문자로 시작하고 중간에 대문자가 들어가는 것은 소문자로 변경
- 컴포넌트 파일은 대문자로 시작하고 중간에 대문자가 들어가는 것은 소문자로 변경

## api end
- https://api-krb.iwinv.kr

## api 연결시 auth 필요한 헤더 규칙
- X-iwinv-Timestamp
- X-iwinv-Credential
- X-iwinv-Signature

## api 연결시 auth 헤더 구현 예제
- There should not be / end of the $path. Query parameters should only be included when making a request and not when signing.
- ```$timestamp = time();
    $path = "/v1/example/param1/param2"; // Path Parameter
    $query_param = "?fields=1023"; // Query Parameter
    $signature = hash_hmac('sha256', $timestamp.$path, $secretKey);
```
- ```curl -i -X GET \
    -H "X-iwinv-Timestamp:$timestamp" \
    -H "X-iwinv-Credential:$accessKey" \
    -H "X-iwinv-Signature:$signature" \
    "https://api-kr.iwinv.kr$path$query_param"
```

## HTTPS Status Code
Status Code	Description
200	Completed
201	Creation Complete
202	Accepted, not done
400	Invalid request
401	Not Logged in
403	Logged in, but autherzation
404	Resource not found
405	Not allowed Method
429	Rate Limit Exceeded
500	Our server-side problem

## Standard Output
Normal
{
    "code": 0x00,
    "error_code": "SUCCESS",
    "message": "",
    "result": [
        {
            "data1": "...",
            "data2": "...",
            "data3": "...",
            "data4": "..."
        },
        {
            "data1": "...",
            "data2": "...",
            "data3": "...",
            "data4": "..."
        }
    ],
    "count": 2
}
Error
{
  "code": "0x1",
  "error_code": "NOT_FOUND",
  "message": "Check request.",
  "result": "error"
}

## 구현할 메뉴 리스트

### Zone
- 존리스트 : https://api.iwinv.kr/api-9479439
### Flavors
- Flavor List : https://api.iwinv.kr/api-9486560
- Flavor Detail : https://api.iwinv.kr/api-9489477
### images
- https://api.iwinv.kr/api-9557500
- https://api.iwinv.kr/api-9558632
### instances
- https://api.iwinv.kr/api-9553506
### block Storage
### Bill
### Authentication

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/turbobit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
