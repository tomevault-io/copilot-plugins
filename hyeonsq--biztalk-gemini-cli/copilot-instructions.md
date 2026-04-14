## biztalk-gemini-cli

> 이 문서는 BizTone Converter 프로젝트에서 Gemini 모델을 어떻게 활용하는지에 대한 정보를 제공합니다.

# Gemini 모델 활용 - BizTone Converter

이 문서는 BizTone Converter 프로젝트에서 Gemini 모델을 어떻게 활용하는지에 대한 정보를 제공합니다.

## 프로젝트 개요

BizTone Converter는 일상적인 표현을 상사, 동료, 고객 등 특정 대상에 맞춰 전문적인 비즈니스 말투로 자동 변환해 주는 AI 기반 웹 솔루션입니다. 이 서비스의 핵심 기능은 강력한 언어 모델인 Gemini를 통해 구현됩니다.

## Gemini 모델 사용

저희 프로젝트는 Groq AI API를 통해 `moonshotai/kimi-k2-instruct-0905` 모델을 사용하여 텍스트 변환을 수행합니다. 이 모델은 다양한 스타일과 톤의 텍스트를 생성하는 데 최적화되어 있어, 비즈니스 커뮤니케이션의 복잡한 요구사항을 충족시킵니다.

## API 통합

`backend/app.py` 파일에서 Groq AI API와의 통합 로직을 관리합니다. 사용자 요청이 들어오면, 지정된 대상(`audience`)에 따라 적절한 프롬프트를 생성하고 이를 Gemini 모델에 전달하여 변환된 텍스트를 받습니다.

### 프롬프트 엔지니어링

각 대상(`audience`)에 맞춰 최적화된 결과물을 얻기 위해 다음과 같은 프롬프트 전략을 사용합니다:

*   **상사 (Upward)**: 보고의 명확성, 격식, 신뢰성을 강조하는 정중하고 명확한 비즈니스 말투를 생성하도록 지시합니다. 결론부터 제시하고 신뢰성을 높이는 방향으로 프롬프트가 구성됩니다.
*   **타팀 동료 (Lateral)**: 협업의 원활함과 요청의 명확성을 중시하는 친절하고 상호 존중하는 어조를 요청합니다. 요청 사항과 마감 기한을 명확히 전달하도록 안내합니다.
*   **고객 (External)**: 서비스의 신뢰도, 친절함, 문제 해결을 중시하는 극존칭을 사용하며 전문성과 서비스 마인드를 강조하는 말투를 생성하도록 유도합니다.

### 구성 및 보안

Gemini 모델을 사용하기 위한 Groq AI API 키는 환경 변수(`GROQ_API_KEY`)를 통해 관리됩니다. 이는 `python-dotenv` 라이브러리를 사용하여 로드되며, 보안을 위해 클라이언트 측에 노출되지 않도록 엄격하게 관리됩니다.

## 주요 코드 스니펫 (backend/app.py)

```python
# Groq 클라이언트 초기화
from groq import Groq
from dotenv import load_dotenv
import os

load_dotenv()
try:
    groq_client = Groq(api_key=os.environ.get("GROQ_API_KEY"))
    print("Groq client initialized successfully.")
except Exception as e:
    groq_client = None
    print(f"Error initializing Groq client: {e}")

# ... (생략) ...

@app.route('/api/convert', methods=['POST'])
def convert_text():
    data = request.json
    original_text = data.get('text')
    audience = data.get('audience')

    if not original_text or not audience:
        return jsonify({"error": "텍스트와 변환 대상은 필수입니다."}), 400

    if groq_client is None:
        return jsonify({"error": "AI 서비스가 초기화되지 않았습니다. API 키를 확인해주세요."}), 500

    prompts = {
        "상사": f"다음 텍스트를 상사에게 보고하기에 적합한, 정중하고 명확하며 격식 있는 비즈니스 말투로 변환해 주세요. 결론부터 제시하고 신뢰성을 강조해 주세요. 원문: '{original_text}'",
        "타팀 동료": f"다음 텍스트를 타팀 동료에게 협조를 요청하거나 정보를 공유하기에 적합한, 친절하고 상호 존중하는 비즈니스 말투로 변환해 주세요. 요청 사항과 마감 기한을 명확히 전달해 주세요. 원문: '{original_text}'",
        "고객": f"다음 텍스트를 고객에게 안내하거나 응대하기에 적합한, 극존칭을 사용하며 전문성과 서비스 마인드를 강조하는 비즈니스 말투로 변환해 주세요. 원문: '{original_text}'"
    }

    system_message = prompts.get(audience)

    if not system_message:
        return jsonify({"error": "유효하지 않은 변환 대상입니다."}), 400

    try:
        chat_completion = groq_client.chat.completions.create(
            messages=[
                {
                    "role": "system",
                    "content": system_message,
                },
                {
                    "role": "user",
                    "content": original_text,
                }
            ],
            model="moonshotai/kimi-k2-instruct-0905",
            temperature=0.7,
            max_tokens=1024,
        )
        converted_text = chat_completion.choices[0].message.content
        
        response_data = {
            "original_text": original_text,
            "converted_text": converted_text,
            "audience": audience
        }
        
        return jsonify(response_data)

    except Exception as e:
        print(f"Error during Groq API call: {e}")
        return jsonify({"error": f"텍스트 변환 중 오류가 발생했습니다: {str(e)}. 잠시 후 다시 시도해주세요."}), 500
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hyeonsq)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/hyeonsq)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
