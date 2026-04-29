## food-delivery-service

> 앞으로 모든 답변을 한국어로 대답합니다.

## setting gemini cli response
앞으로 모든 답변을 한국어로 대답합니다. 


## API SPEC 테스트 구현 사항 

아래 API문서에 따라, 가짜 응답을 제공하는 Controller를 작성하고, 작성된 내용을 만족하는지 테스트하는 통합 테스트를 구현한다.


### API 문서
# Food-Delivery-Serivce API 명세
--- 
1. **회원 가입**
   post  ```/auth/sign```
   **Request Body**
   ```json
   {
     "email": "user@example.com",
     "password": "Password123!",
     "name": "김예진",
     "signupType": "USER"
     }
   ```
   **Response Body**
   - **201 Created**
   ```json
   {
     "id": 1,
     "email": "user@example.com",
     "name": "김예진",
     "nickname": "yejin",
     "createdAt": "2026-01-05T12:30:00"
   }
   ```

2. **로그인 (session)**
   POST ```/auth/login```
   ```json
   {
     "email": "user@example.com",
     "password": "Password123!"
   }   
   ```
   **Success (200 OK)**
   ```json
   {
     "userId": 1,
     "userType": "USER",
     "name": "김예진"
   }   
   ```
3. **주문 생성** **API**
   ```json
   [USER]
     주문 생성
     할인 미리보기
     결제 완료
           ↓
   [OWNER]
     라이더 매칭 요청
           ↓
   [System]
     Route 생성
     라이더 매칭
   ```

   POST ```/orders```
   **Request Body**
   ```json
   {
     "storeId": 101,
     "orderItems": [
       {
         "menuId": 1,
         "quantity": 2
       },
       {
         "menuId": 3,
         "quantity": 1
       }
     ],
     "deliveryAddress": "서울시 강남구 테헤란로 123"
   }
   ```

   **Response Body (201 Created)**
   ```json
   {
     "orderId": 5001,
     "orderStatus": "CREATED",
     "totalPrice": 18000
   }
   ```

4. **할인 적용 결과 반환 API**
   POST ```/api/order/{orderId}/discounts/preview```

- 배민 클럽 할인
- 쿠폰은 최소 금액 15,000원 이상일 때만 사용 가능.
- 최대 할인 금액은 5,000원이다.
- 어떤 쿠폰은 배달료를 포함한 전체 금액에 적용, 어떤 쿠폰은 음식 가격에만 적용
- 특정 가게에서만 사용할 수 있는 쿠폰
- 쿠폰 우선순위 결정 알고리즘. 어떤 쿠폰을 선택했을 떄 최종 금액이 최소인가? (쿠폰은 1개만 적용 가능하다.)


**Request Body**
  ```json
  {
     "couponId": 10,
     "useBaeminClub": true
  }   
  ```

**ResponseBody**
  ```json
  {
    "orderId": 5001,
    "originalPrice": 18000,
    "discountDetails": {
      "couponDiscount": 3000,
      "baeminClubDiscount": 1500
    },
    "totalDiscountAmount": 4500,
    "finalPrice": 13500
  }   
  ```

5. **주문 결제**
   POST ```/payment/{orderId}/
   **Request Body**
   ```json
   {
     "paymentMethod": "CARD",
     "couponId": 10,
     "useBaeminClub": true
   }
   ```

   **Response Body**
   ```json
   {
     "orderId": 5001,
     "orderStatus": "PAID",
     "paymentStatus": "SUCCESS",
     "originalPrice": 18000,
     "discountAmount": 4500,
     "paidAmount": 13500,
     "paymentMethod": "CARD",
     "paidAt": "2026-01-05T15:10:00"
   }
   ```

6. **Owner - 라이더 매칭 요청** (O)
   **post**  ```/deliveries/{orderId}/dispatch```
   한집 배달 / 알뜰 배달  선택 가능

   **Request Body**
   ```json
   {
     "deliveryType": "BUNDLE"
   }   
   ```

   **Response Body**
   ```json
   {
     "orderId": 5001,
     "dispatchStatus": "REQUESTED",
     "deliveryType": "BUNDLE"
   }
   ```

7. **라이더의 배달 상태 변경**
   POST ```/deliveries/{deliveryId}/status```
   **Request Body**
```json
{
  "status": "PICKED_UP"
}
```

Status : 200 OK
Response Body
```json
{
	"deliveryId" : 1,
	"deliveryStatus" : "PICKED_UP"
}
```

8. **라이더의 현재 배달 Route 조회 API**
   GET ```/routes/me/current-route```

**Status :** 200 ok
Response Body
```json
{
  "stops": [
    {
      "sequence": 1,
      "type": "PICKUP",
      "orderId": 5001,
      "address": "가게 A"
    },
    {
      "sequence": 2,
      "type": "PICKUP",
      "orderId": 5002,
      "address": "가게 B"
    },
    {
      "sequence": 3,
      "type": "DELIVERY",
      "orderId": 5001,
      "address": "고객 A"
    },
    {
      "sequence": 4,
      "type": "DELIVERY",
      "orderId": 5002,
      "address": "고객 B"
    }
  ]
}

```

9. **라이더/주문자/owner 의 주문 단위 상세 조회 API**
   **Role-based View Model**
   GET ```/orders/{orderId}```
```json
{
  "orderId": 5001,
  "store": {
    "name": "김밥천국",
    "address": "서울시 강남구 ..."
  },
  "deliveryAddress": "서울시 강남구 테헤란로 123",
  "orderStatus": "PICKED_UP",
  "items": [
    { "name": "김밥", "quantity": 2 },
    { "name": "라면", "quantity": 1 }
  ],
  "customerNote": "문 앞에 놔주세요"
}

```
10. 알림 이벤트 - websocket?
```
- ORDER_PICKED_UP
- ORDER_NEARBY
- ORDER_DELIVERED
```

11. **라이더의 위치 전송 API**
```
[RIDER]
  GPS 위치 획득
    ↓
POST /routes/{routeId}/location
    ↓
  [SERVER]
    ↓
GET /orders/{orderId}/rider-location
    ↓
[USER]
  지도에 표시 (클라이언트)

```

POST ```/riders/{routeId}/location```
**Request Body**
```json
{
  "latitude": 37.498095,
  "longitude": 127.027610
}
```

12. **사용자의 라이더 위치 조회 API**
    GET ```/riders/{riderId}/location```
    **Response Body**
```json
{
  "riderId": 5001,
  "latitude": 37.498095,
  "longitude": 127.027610,
  "lastUpdatedAt": "2026-01-05T16:15:00"
}
```

8. **라이더 위치 업데이트 API**
   POST ```/riders/{riderId}/location```
   **RequestBody**
```
{
  "latitude": 37.498095,
  "longitude": 127.027610
}
```

**Response**
**201 created**
```json
{
  "riderId": 5001,
  "latitude": 37.498095,
  "longitude": 127.027610,
  "lastUpdatedAt": "2026-01-05T16:15:00"
}
```

---
> Source: [dPwls0125/Food-Delivery-Service](https://github.com/dPwls0125/Food-Delivery-Service) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
