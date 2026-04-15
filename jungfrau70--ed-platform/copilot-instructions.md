## ed-platform

> 교육 교구 시스템의 콘텐츠 생성 및 관리 규칙


# 교육 교구 시스템 콘텐츠 생성 규칙

## 📚 교육 자료 작성 원칙

### **교육 자료 구조화**
- **명확한 학습 목표**: 각 교육 자료는 구체적인 학습 목표를 명시
- **단계별 진행**: 복잡한 내용을 단계별로 분해하여 설명
- **실습 중심**: 이론과 실습을 균형있게 배치
- **피드백 반영**: 학습자 피드백을 지속적으로 반영

### **교육 자료 작성 가이드라인**
```markdown
<!-- ✅ DO: 교육 자료 구조 -->
# 🎯 [과정명] - [일차]: [주제]

## 📚 학습 목표
- 구체적이고 측정 가능한 목표
- 학습 후 달성할 수 있는 능력 명시

## 🔧 실습 환경 준비
- 필요한 도구 및 환경 설정
- 단계별 설치 가이드

## 💻 단계별 실습
### 1단계: [실습 제목]
- 명확한 실습 지침
- 예상 결과 및 확인 방법

## 📊 학습 정리
- 핵심 개념 요약
- 다음 단계 안내
```

## 🎨 UI/UX 디자인 원칙 (교육 특화)

### **학습자 중심 디자인**
- **직관적 네비게이션**: 학습자가 쉽게 찾을 수 있는 구조
- **진도 표시**: 현재 학습 위치와 진행 상황 명확히 표시
- **접근성 고려**: 다양한 학습자 접근성 보장
- **반응형 디자인**: 다양한 디바이스에서 최적화

### **교육 도구 UI 컴포넌트**
```vue
<!-- ✅ DO: 교육 과정 카드 컴포넌트 -->
<template>
  <div class="course-card">
    <div class="course-header">
      <span class="course-icon">{{ courseIcon }}</span>
      <h3 class="course-title">{{ courseTitle }}</h3>
    </div>
    <div class="course-meta">
      <span class="course-level">{{ courseLevel }}</span>
      <span class="course-duration">{{ courseDuration }}</span>
    </div>
    <div class="course-progress">
      <div class="progress-bar" :style="{ width: progress + '%' }"></div>
    </div>
  </div>
</template>
```

## 🤖 AI 활용 콘텐츠 생성

### **AI 도구 활용 규칙**
- **콘텐츠 생성**: AI를 활용한 교육 자료 초안 작성
- **품질 검증**: AI 생성 콘텐츠의 정확성 검증 필수
- **개인화**: 학습자 수준에 맞는 맞춤형 콘텐츠 생성
- **지속적 학습**: AI 모델의 성능 지속적 개선

### **AI 콘텐츠 생성 프로세스**
```python
# ✅ DO: AI 콘텐츠 생성 프로세스
def generate_educational_content(topic: str, level: str, duration: int):
    # 교육 콘텐츠 AI 생성
    
    # 1. 학습 목표 생성
    learning_objectives = ai_generate_objectives(topic, level)
    
    # 2. 실습 가이드 생성
    practice_guide = ai_generate_practice_guide(topic, duration)
    
    # 3. 평가 기준 생성
    assessment_criteria = ai_generate_assessment(topic, level)
    
    # 4. 품질 검증
    validated_content = validate_educational_content({
        'objectives': learning_objectives,
        'practice': practice_guide,
        'assessment': assessment_criteria
    })
    
    return validated_content
```

## 📝 문서 작성 표준

### **마크다운 작성 규칙**
- **헤딩 구조**: H1(과정명) → H2(섹션) → H3(실습) → H4(세부사항)
- **코드 블록**: 언어별 문법 하이라이팅 필수
- **이미지**: 교육 자료에 적합한 이미지 사용, alt 텍스트 필수
- **링크**: 모든 링크는 유효하고 관련성 있는 내용

### **코드 예시 작성 규칙**
```typescript
// ✅ DO: 교육용 코드 예시
/**
 * 교육 과정: Cloud Basic - AWS S3 실습
 * 목적: S3 버킷 생성 및 파일 업로드 학습
 * 난이도: 초급
 */
export class S3PracticeGuide {
  private bucketName: string;
  
  constructor(bucketName: string) {
    this.bucketName = bucketName;
  }
  
  /**
   * S3 버킷에 파일 업로드
   * @param file - 업로드할 파일
   * @returns 업로드 결과
   */
  async uploadFile(file: File): Promise<UploadResult> {
    // 실습 단계별 주석 포함
    const uploadParams = {
      Bucket: this.bucketName,
      Key: file.name,
      Body: file
    };
    
    return await s3.upload(uploadParams).promise();
  }
}
```

## 🧪 교육 자료 테스트 규칙

### **콘텐츠 품질 검증**
- **정확성 검증**: 기술적 내용의 정확성 확인
- **완성도 검증**: 모든 실습 단계가 완료 가능한지 확인
- **일관성 검증**: 교육 과정 전반의 일관성 유지
- **접근성 검증**: 다양한 학습자 접근성 보장

### **자동화된 콘텐츠 검증**
```python
# ✅ DO: 교육 콘텐츠 자동 검증
def validate_educational_content(content: dict) -> ValidationResult:
    # 교육 콘텐츠 자동 검증
    
    validation_result = {
        'is_valid': True,
        'errors': [],
        'warnings': []
    }
    
    # 1. 학습 목표 검증
    if not content.get('objectives') or len(content['objectives']) < 2:
        validation_result['errors'].append('학습 목표가 부족합니다.')
        validation_result['is_valid'] = False
    
    # 2. 실습 가이드 검증
    if not content.get('practice') or len(content['practice']['steps']) < 3:
        validation_result['warnings'].append('실습 단계가 부족할 수 있습니다.')
    
    # 3. 코드 예시 검증
    if content.get('code_examples'):
        for example in content['code_examples']:
            if not validate_code_syntax(example['code'], example['language']):
                validation_result['errors'].append(f'코드 문법 오류: {example["title"]}')
                validation_result['is_valid'] = False
    
    return validation_result
```

## 📊 콘텐츠 성과 측정

### **학습 효과 지표**
- **완료율**: 교육 과정 완료 비율
- **이해도**: 학습자 이해도 평가 점수
- **만족도**: 학습자 만족도 설문 결과
- **실습 성공률**: 실습 단계별 성공률

### **콘텐츠 개선 프로세스**
```python
# ✅ DO: 콘텐츠 성과 분석
def analyze_content_performance(content_id: str) -> PerformanceReport:
    # 콘텐츠 성과 분석 및 개선 제안
    
    # 학습자 데이터 수집
    learner_data = get_learner_data(content_id)
    
    # 성과 지표 계산
    completion_rate = calculate_completion_rate(learner_data)
    satisfaction_score = calculate_satisfaction(learner_data)
    difficulty_score = calculate_difficulty(learner_data)
    
    # 개선 제안 생성
    improvement_suggestions = generate_improvement_suggestions({
        'completion_rate': completion_rate,
        'satisfaction': satisfaction_score,
        'difficulty': difficulty_score
    })
    
    return PerformanceReport(
        content_id=content_id,
        metrics={
            'completion_rate': completion_rate,
            'satisfaction': satisfaction_score,
            'difficulty': difficulty_score
        },
        suggestions=improvement_suggestions
    )
```

## 🔄 콘텐츠 업데이트 및 유지보수

### **정기 업데이트 규칙**
- **주간 검토**: 학습자 피드백 기반 콘텐츠 검토
- **월간 업데이트**: 기술 트렌드 반영 및 내용 보완
- **분기별 개편**: 교육 과정 구조 전면 검토
- **연간 혁신**: 새로운 교육 방법론 도입

### **버전 관리**
```python
# ✅ DO: 콘텐츠 버전 관리
class ContentVersionManager:
    def __init__(self):
        self.current_version = "1.0.0"
    
    def create_new_version(self, content_id: str, changes: dict) -> str:
        # 새 버전 생성
        version = self.increment_version()
        
        # 변경사항 기록
        self.record_changes(content_id, version, changes)
        
        # 이전 버전 백업
        self.backup_previous_version(content_id, version)
        
        return version
    
    def rollback_version(self, content_id: str, target_version: str):
        # 이전 버전으로 롤백
        if self.is_valid_version(target_version):
            self.restore_version(content_id, target_version)
        else:
            raise ValueError(f"유효하지 않은 버전: {target_version}")
```

## 📚 관련 파일

- [ContentView.vue](mdc:frontend/components/ContentView.vue) - 교육 콘텐츠 표시
- [SyllabusExplorer.vue](mdc:frontend/components/SyllabusExplorer.vue) - 교육 과정 탐색
- [curriculum.py](mdc:backend/app/api/routes/curriculum.py) - 교육 과정 API
- [content_validator.py](mdc:backend/app/utils/content_validator.py) - 콘텐츠 검증

이 규칙을 통해 고품질의 교육 콘텐츠를 지속적으로 생성하고 관리할 수 있습니다.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jungfrau70)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jungfrau70)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
