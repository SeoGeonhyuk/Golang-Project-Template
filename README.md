# Go 프로젝트 템플릿 사용 가이드

이 템플릿은 Go 프로젝트를 체계적으로 시작하고 Claude Code와 효과적으로 협업하기 위한 문서 모음입니다.

## 📋 템플릿 개요

### 목적
- **개발자**: Go 프로젝트 시작 시 참고할 체계적인 가이드 제공
- **Claude Code**: AI가 Go 프로젝트를 효과적으로 구현할 수 있도록 돕는 지침서

### 핵심 철학
- **TDD 중심**: 요구사항 → 테스트 → 구현 순서
- **Uber Go Style Guide 준수**: 일관된 코딩 스타일
- **실용적 접근**: 바로 사용 가능한 예제와 템플릿 제공

## 🚀 빠른 시작 가이드

### 1. 새 프로젝트 시작 시

```bash
# 1. 이 템플릿을 새 프로젝트로 복사
cp -r golang-project-template my-new-project
cd my-new-project

# 2. Git 초기화
git init
git add .
git commit -m "Initial commit with project template"

# 3. 프로젝트별 README 작성
mv README.md TEMPLATE_README.md
# 새로운 README.md 작성
```

### 2. Claude Code와 협업 시

Claude Code에게 다음과 같이 요청하세요:

```
"이 프로젝트는 documents/ 폴더의 Go 프로젝트 템플릿을 따릅니다. 
CLAUDE_INSTRUCTIONS.md를 먼저 읽고 TDD 방식으로 [기능명]을 구현해주세요."
```

## 📁 문서 구조 및 사용법

### Phase 1: 요구사항 정의
📂 **1-requirements/**
- `REQUIREMENTS.md` - 요구사항 명세서 작성

**사용법:**
1. 템플릿을 복사하여 프로젝트 요구사항 작성
2. 기능별로 우선순위와 인수 조건 명시
3. Claude Code가 이해하기 쉽도록 구체적으로 작성

### Phase 2: 설계
📂 **2-design/**
- `ARCHITECTURE.md` - 아키텍처 설계 가이드
- `API_DESIGN.md` - API 설계 가이드

**사용법:**
1. 프로젝트 규모에 맞는 아키텍처 패턴 선택
2. ADR(Architecture Decision Record) 형식으로 설계 결정 기록
3. API가 필요한 경우 REST/gRPC 설계 지침 참고

### Phase 3: 구현
📂 **3-implementation/**
- `PROJECT_SETUP.md` - 프로젝트 초기 설정
- `TDD_GUIDE.md` - TDD 실천 가이드
- `CODE_STRUCTURE.md` - 코드 구조 가이드

**사용법:**
1. `PROJECT_SETUP.md` 따라 프로젝트 기반 구조 생성
2. TDD 사이클로 기능 구현
3. Uber Go Style Guide 준수

### Phase 4: 진행 관리
📂 **4-progress/**
- `DEVELOPMENT_LOG.md` - 개발 진행 로그

**사용법:**
1. 일별/주별 진행 상황 기록
2. 이슈 및 해결 방법 문서화
3. 성과 지표 추적

### Phase 5: 학습 기록
📂 **5-learning/**
- `LEARNING_NOTES.md` - 패키지 및 기능 학습 노트

**사용법:**
1. 새로운 Go 패키지 사용 시 내용 추가
2. Claude Code가 사용한 패키지 설명 요청
3. 팀 학습 자료로 활용

## 🤖 Claude Code 활용 팁

### 효과적인 요청 방법

#### ✅ 좋은 예시
```
"documents/1-requirements/REQUIREMENTS.md에 정의된 FR-001 기능을 구현해주세요.
TDD 방식으로 진행하고, 완료 후 5-learning/LEARNING_NOTES.md에 
사용한 패키지 설명을 추가해주세요."
```

#### ❌ 피해야 할 예시
```
"사용자 관리 기능 만들어줘"  // 너무 모호함
```

### 단계별 협업 가이드

#### 1단계: 요구사항 확인
```
"REQUIREMENTS.md의 [기능명] 요구사항을 검토하고 
테스트 케이스를 작성해주세요."
```

#### 2단계: 설계 검토
```
"이 기능에 적합한 아키텍처 패턴을 ARCHITECTURE.md를 참고해서 
제안해주세요."
```

#### 3단계: 구현
```
"TDD_GUIDE.md를 따라 [기능명]을 구현해주세요.
먼저 실패하는 테스트를 작성하고, 최소 구현 후 리팩토링까지 
진행해주세요."
```

#### 4단계: 문서화
```
"방금 구현한 기능을 DEVELOPMENT_LOG.md에 기록하고,
사용한 새로운 패키지들을 LEARNING_NOTES.md에 추가해주세요."
```

## 📚 권장 워크플로우

### 새 기능 개발 시

1. **요구사항 정의**
   - `REQUIREMENTS.md`에 기능 요구사항 추가
   - 인수 조건 명확히 정의

2. **설계**
   - 필요시 `ARCHITECTURE.md` 참고하여 설계 결정
   - API가 필요하면 `API_DESIGN.md` 참고

3. **TDD 구현**
   - Claude Code에게 TDD 방식 구현 요청
   - 테스트 → 구현 → 리팩토링 사이클 반복

4. **문서화**
   - 진행 상황 `DEVELOPMENT_LOG.md`에 기록
   - 새 패키지 사용 시 `LEARNING_NOTES.md`에 추가

### 코드 리뷰 시

`CLAUDE_INSTRUCTIONS.md`의 체크리스트 활용:
- [ ] Uber Go Style Guide 준수
- [ ] 적절한 에러 처리
- [ ] 테스트 커버리지 확인
- [ ] 문서화 완료

## 🔧 프로젝트별 커스터마이징

### 필수 수정 사항

1. **프로젝트 정보 업데이트**
   ```bash
   # REQUIREMENTS.md에서 프로젝트명, 목적 수정
   # DEVELOPMENT_LOG.md에서 팀 구성, 일정 수정
   ```

2. **기술 스택 선택**
   ```bash
   # ARCHITECTURE.md에서 사용할 데이터베이스, 프레임워크 결정
   # PROJECT_SETUP.md의 의존성 목록 수정
   ```

### 선택적 수정 사항

1. **문서 구조 조정**
   - 불필요한 문서 제거
   - 프로젝트 특성에 맞는 추가 문서 생성

2. **체크리스트 커스터마이징**
   - 팀 표준에 맞게 코딩 스타일 조정
   - 프로젝트별 품질 기준 추가

## 💡 활용 사례

### 웹 API 프로젝트
```
1. REQUIREMENTS.md → REST API 요구사항 정의
2. API_DESIGN.md → OpenAPI 스펙 작성
3. ARCHITECTURE.md → 레이어드 아키텍처 선택
4. TDD로 핸들러부터 구현
```

### CLI 도구 프로젝트
```
1. REQUIREMENTS.md → 명령어 인터페이스 정의
2. ARCHITECTURE.md → 단순한 구조 선택
3. 코어 로직부터 TDD로 구현
```

### 마이크로서비스
```
1. ARCHITECTURE.md → 서비스 분리 전략 수립
2. API_DESIGN.md → gRPC 인터페이스 설계
3. 서비스별로 독립적 개발
```

## 🎯 성공 기준

이 템플릿을 잘 활용하고 있다면:

- [ ] 요구사항이 테스트 케이스로 명확히 변환됨
- [ ] 모든 기능이 TDD로 구현됨
- [ ] 일관된 코드 스타일 유지됨
- [ ] 진행 상황이 체계적으로 기록됨
- [ ] 새로운 기술 학습이 문서화됨
- [ ] Claude Code와의 협업이 원활함

## 🔗 추가 자료

### 참고 문서
- [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)
- [Effective Go](https://go.dev/doc/effective_go)
- [Go Project Layout](https://github.com/golang-standards/project-layout)

### 권장 도구
- **IDE**: VS Code with Go extension
- **Linter**: golangci-lint
- **Testing**: testify
- **Documentation**: godoc

## 🤝 기여하기

이 템플릿을 개선하고 싶다면:

1. 사용 경험 피드백
2. 누락된 가이드 제안
3. 실제 프로젝트 적용 사례 공유
4. 새로운 Go 패턴 및 베스트 프랙티스 추가

---

**버전**: 1.0  
**최종 업데이트**: 2024년 1월  
**문의**: 이슈나 개선 사항이 있으면 언제든 말씀해주세요!

> 💡 **팁**: 이 README.md는 프로젝트 시작 후 프로젝트별 README로 교체하세요. 이 파일은 `TEMPLATE_README.md`로 백업해두는 것을 권장합니다.