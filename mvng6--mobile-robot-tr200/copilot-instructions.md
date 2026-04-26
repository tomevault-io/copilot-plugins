## mobile-robot-tr200

> Documentation and commenting standards for robot development


# 문서화 및 주석 작성 규칙

## 한국어 주석 및 문서화 원칙

### Docstring 작성 규칙
모든 클래스, 함수, 메서드에는 한국어 docstring을 작성합니다.

```python
class WooshRobot:
    """
    Woosh 로봇 제어를 위한 메인 클래스입니다.
    
    WebSocket을 통해 로봇과 통신하며, 이동, 작업 실행, 
    상태 모니터링 등의 기능을 제공합니다.
    
    Attributes:
        settings: 로봇 연결 설정 정보
        is_connected: 로봇 연결 상태
    """
    
    async def move_to_goal(self, x: float, y: float, theta: float) -> bool:
        """
        로봇을 지정된 목표 위치로 이동시킵니다.
        
        좌표계는 로봇의 초기 위치를 원점(0,0)으로 하는 
        데카르트 좌표계를 사용합니다.
        
        Args:
            x: 목표 X 좌표 (미터 단위)
            y: 목표 Y 좌표 (미터 단위)
            theta: 목표 방향각 (라디안, -π ~ π)
            
        Returns:
            bool: 이동 명령 전송 성공 여부
            
        Raises:
            ConnectionError: 로봇과의 연결이 끊어진 경우
            ValueError: 잘못된 좌표값이 입력된 경우
            
        Example:
            >>> robot = WooshRobot(settings)
            >>> await robot.run()
            >>> success = await robot.move_to_goal(1.0, 2.0, 0.0)
        """
```

### 인라인 주석 규칙

#### 복잡한 로직 설명
```python
# 로봇 상태를 주기적으로 확인하여 연결 상태 유지
async def _heartbeat_loop(self):
    while self.is_running:
        try:
            # 5초마다 로봇 상태 요청 전송
            await self._send_heartbeat()
            await asyncio.sleep(5.0)
        except Exception as e:
            # 연결 오류 발생 시 재연결 시도
            logger.warning(f"하트비트 오류: {e}")
            await self._reconnect()
```

#### 프로토콜 메시지 처리
```python
# Protocol Buffer 메시지 생성 및 직렬화
move_request = RobotMoveRequest()
move_request.target.x = x  # 목표 X 좌표 설정
move_request.target.y = y  # 목표 Y 좌표 설정
move_request.target.theta = theta  # 목표 방향각 설정

# 메시지를 바이트로 직렬화하여 전송
serialized_data = move_request.SerializeToString()
await self._send_message(serialized_data)
```

#### 에러 처리 로직
```python
try:
    # 로봇 연결 시도
    await self.websocket.connect()
    self.is_connected = True
except ConnectionRefusedError:
    # 로봇이 응답하지 않는 경우
    logger.error("로봇이 연결을 거부했습니다. IP 주소를 확인하세요.")
    return False
except asyncio.TimeoutError:
    # 연결 시간 초과
    logger.error("로봇 연결 시간이 초과되었습니다.")
    return False
```

## README 및 마크다운 문서 규칙

### README.md 구조
```markdown
# 프로젝트 제목

간단한 프로젝트 설명 (1-2문장)

## 📋 주요 기능

- 기능 1: 상세 설명
- 기능 2: 상세 설명

## 🚀 빠른 시작

### 설치 방법
\`\`\`bash
pip install -r requirements.txt
\`\`\`

### 기본 사용법
\`\`\`python
# 코드 예제
\`\`\`

## 📖 상세 문서

- [API 문서](docs/api.md)
- [예제 모음](examples/)

## 🔧 문제 해결

일반적인 문제와 해결 방법
```

### 코드 예제 문서화
```python
"""
예제: 로봇 기본 이동 제어

이 예제는 TR-200 로봇을 특정 위치로 이동시키는 
기본적인 방법을 보여줍니다.

실행 전 확인사항:
1. 로봇 전원 및 네트워크 연결 확인
2. 로봇 IP 주소 확인 (기본: 172.20.254.63)
3. 로봇 비상정지 해제 확인

실행 방법:
    python tr200_move_to_goal.py --ip 로봇IP --x 1.0 --y 2.0
"""

import asyncio
import argparse
from woosh_robot import WooshRobot
from woosh_interface import CommuSettings

async def main():
    # 명령행 인수 파싱
    parser = argparse.ArgumentParser(description="TR-200 로봇 이동 제어")
    parser.add_argument("--ip", required=True, help="로봇 IP 주소")
    parser.add_argument("--x", type=float, required=True, help="목표 X 좌표")
    parser.add_argument("--y", type=float, required=True, help="목표 Y 좌표")
    args = parser.parse_args()
    
    # 로봇 연결 설정
    loop = asyncio.get_event_loop()
    settings = CommuSettings(addr=args.ip, port=10003, loop=loop)
    robot = WooshRobot(settings)
    
    try:
        # 로봇 연결
        if not await robot.run():
            print("❌ 로봇 연결 실패")
            return
        
        print("✅ 로봇 연결 성공")
        
        # 목표 위치로 이동
        success = await robot.move_to_goal(args.x, args.y, 0.0)
        if success:
            print(f"🎯 목표 위치 ({args.x}, {args.y})로 이동 시작")
        else:
            print("❌ 이동 명령 전송 실패")
            
    except Exception as e:
        print(f"❌ 오류 발생: {e}")
    finally:
        # 연결 정리
        await robot.stop()

if __name__ == "__main__":
    asyncio.run(main())
```

## 로깅 및 디버그 메시지

### 로그 레벨별 메시지 작성
```python
import logging

logger = logging.getLogger(__name__)

# DEBUG: 개발 시 상세 정보
logger.debug(f"WebSocket 메시지 수신: {message_type}")

# INFO: 일반적인 동작 정보
logger.info("로봇 연결이 성공적으로 설정되었습니다")

# WARNING: 주의가 필요한 상황
logger.warning("로봇 배터리가 20% 이하입니다")

# ERROR: 오류 상황
logger.error(f"로봇 통신 오류: {error_message}")

# CRITICAL: 심각한 오류
logger.critical("로봇과의 연결이 완전히 끊어졌습니다")
```

## 문서 유지보수

### 버전 업데이트 시 확인사항
1. API 변경사항을 README에 반영
2. 예제 코드의 동작 확인 및 업데이트
3. 새로운 기능에 대한 사용법 추가
4. 알려진 문제점 및 해결방법 업데이트

### 주석 품질 체크리스트
- [ ] 복잡한 로직에 한국어 설명 추가
- [ ] 함수/클래스의 목적과 사용법 명시
- [ ] 매개변수와 반환값의 의미 설명
- [ ] 예외 상황과 에러 처리 방법 기술
- [ ] 실제 사용 예제 코드 포함

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mvng6) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
