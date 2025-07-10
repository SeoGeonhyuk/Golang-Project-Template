# Claude Code 작업 가이드

이 문서는 Claude Code가 Go 프로젝트를 효과적으로 진행할 수 있도록 돕는 가이드입니다.

## 📋 프로젝트 시작 체크리스트

### 1. 요구사항 분석 단계
- [ ] `1-requirements/REQUIREMENTS.md`를 읽고 요구사항 정의
- [ ] 각 요구사항을 테스트 가능한 단위로 분해
- [ ] 우선순위와 의존성 파악
- [ ] 불명확한 부분은 사용자에게 질문

### 2. 설계 단계
- [ ] `2-design/ARCHITECTURE.md`를 참고하여 전체 구조 설계
- [ ] 필요한 인터페이스 정의
- [ ] `2-design/API_DESIGN.md`를 참고하여 API 설계 (해당되는 경우)
- [ ] 외부 의존성 최소화

### 3. 구현 단계
- [ ] `3-implementation/PROJECT_SETUP.md`를 따라 프로젝트 초기화
- [ ] `3-implementation/TDD_GUIDE.md`를 따라 테스트 먼저 작성
- [ ] 테스트 실패 확인 (Red)
- [ ] 최소한의 코드로 테스트 통과 (Green)
- [ ] 리팩토링 (Refactor)
- [ ] `3-implementation/CODE_STRUCTURE.md`를 참고하여 코드 구조화

### 4. 문서화 단계
- [ ] `4-progress/DEVELOPMENT_LOG.md`에 진행 사항 기록
- [ ] `5-learning/LEARNING_NOTES.md`에 새로운 패키지/기능 설명 추가
- [ ] 코드 주석 작성 (godoc 형식)
- [ ] README 업데이트

## 📚 코딩 스타일 가이드

이 프로젝트는 **Uber Go Style Guide**를 따릅니다:
- 📄 **상세 문서**: `code-guide/DEV_STYLE.md` 참조
- 🔗 **원본**: [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md)

주요 체크 포인트:
- [ ] 네이밍 컨벤션 (PascalCase, camelCase)
- [ ] 에러 처리 패턴 (`fmt.Errorf` with `%w`)
- [ ] 인터페이스 설계 (작고 구체적으로)
- [ ] 구조체 초기화 (필드명 명시)

## 🚀 자주 사용하는 Go 패턴

### 1. 에러 처리
```go
// 항상 에러를 즉시 처리
if err != nil {
    return fmt.Errorf("context description: %w", err)
}

// 여러 에러 가능성이 있을 때
switch {
case errors.Is(err, ErrNotFound):
    // 특정 처리
case errors.Is(err, ErrInvalid):
    // 다른 처리
default:
    return err
}
```

### 2. 인터페이스 정의
```go
// 작은 인터페이스를 선호
type Reader interface {
    Read(ctx context.Context, id string) (*Entity, error)
}

// 인터페이스 만족 확인
var _ Reader = (*ConcreteReader)(nil)
```

### 3. 테스트 작성
```go
// Table-driven 테스트
func TestFunction(t *testing.T) {
    tests := []struct {
        name    string
        input   string
        want    string
        wantErr bool
    }{
        {
            name:  "valid input",
            input: "test",
            want:  "TEST",
        },
        {
            name:    "empty input",
            input:   "",
            wantErr: true,
        },
    }
    
    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            got, err := Function(tt.input)
            if (err != nil) != tt.wantErr {
                t.Errorf("Function() error = %v, wantErr %v", err, tt.wantErr)
                return
            }
            if got != tt.want {
                t.Errorf("Function() = %v, want %v", got, tt.want)
            }
        })
    }
}
```

### 4. 컨텍스트 사용
```go
// 항상 컨텍스트를 첫 번째 파라미터로
func DoSomething(ctx context.Context, param string) error {
    // 컨텍스트 취소 확인
    select {
    case <-ctx.Done():
        return ctx.Err()
    default:
    }
    
    // 실제 로직
    return nil
}
```

## ⚠️ 피해야 할 안티패턴

### 1. 전역 변수 남용
```go
// ❌ 나쁜 예
var db *sql.DB

// ✅ 좋은 예
type Service struct {
    db *sql.DB
}
```

### 2. 큰 인터페이스
```go
// ❌ 나쁜 예
type DataStore interface {
    Create(...)
    Read(...)
    Update(...)
    Delete(...)
    List(...)
    Count(...)
}

// ✅ 좋은 예
type Reader interface {
    Read(ctx context.Context, id string) (*Entity, error)
}

type Writer interface {
    Write(ctx context.Context, entity *Entity) error
}
```

### 3. 에러 무시
```go
// ❌ 절대 하지 말 것
result, _ := someFunction()

// ✅ 항상 에러 처리
result, err := someFunction()
if err != nil {
    return fmt.Errorf("failed to do something: %w", err)
}
```

## 📊 코드 품질 체크리스트

### 구현 전
- [ ] 요구사항이 명확한가?
- [ ] 테스트 케이스를 먼저 작성했는가?
- [ ] 인터페이스를 정의했는가?

### 구현 중
- [ ] 에러 처리를 빠뜨리지 않았는가?
- [ ] 고루틴을 사용한다면 적절히 동기화했는가?
- [ ] 리소스(파일, 연결 등)를 적절히 닫는가?

### 구현 후
- [ ] 모든 테스트가 통과하는가?
- [ ] `go fmt`를 실행했는가?
- [ ] `go vet`에서 경고가 없는가?
- [ ] 테스트 커버리지가 80% 이상인가?
- [ ] 문서를 업데이트했는가?

## 🔧 유용한 Go 명령어

```bash
# 프로젝트 초기화
go mod init github.com/username/projectname

# 의존성 정리
go mod tidy

# 테스트 실행
go test ./...
go test -v ./...                    # verbose
go test -cover ./...                # 커버리지
go test -race ./...                 # race condition 검사

# 벤치마크
go test -bench=. ./...
go test -bench=. -benchmem ./...    # 메모리 할당 포함

# 코드 품질
go fmt ./...                        # 포맷팅
go vet ./...                        # 정적 분석
golangci-lint run                   # 종합 린터 (설치 필요)

# 빌드
go build -o bin/app cmd/app/main.go
go build -ldflags="-s -w" ...       # 바이너리 크기 축소

# 문서 생성
godoc -http=:6060                   # 로컬 문서 서버
```

## 💡 프로젝트별 고려사항

### CLI 애플리케이션
- `cobra` 또는 `urfave/cli` 사용 고려
- 설정은 `viper` 활용
- 진행 표시는 `progressbar` 패키지

### 웹 서버
- 표준 라이브러리 우선 고려
- 복잡한 라우팅이 필요하면 `gorilla/mux` 또는 `chi`
- 미들웨어 체이닝 패턴 활용

### 마이크로서비스
- gRPC 우선 고려
- 서비스 간 통신은 protobuf
- 분산 추적을 위한 OpenTelemetry

## 📝 사용자와의 커뮤니케이션

### 질문할 때
1. 구체적인 선택지 제시
2. 각 선택지의 장단점 설명
3. 추천 사항 명시

### 진행 상황 보고
1. 완료된 작업 요약
2. 현재 진행 중인 작업
3. 다음 단계 예고

### 문제 발생 시
1. 문제 상황 명확히 설명
2. 시도한 해결 방법
3. 대안 제시

## 🎯 성공 기준

프로젝트가 다음을 만족하면 성공적:
- [ ] 모든 요구사항 구현
- [ ] 테스트 커버리지 80% 이상
- [ ] 문서화 완료
- [ ] 코드 리뷰 체크리스트 통과
- [ ] 성능 목표 달성

이 가이드를 참고하여 체계적이고 품질 높은 Go 프로젝트를 진행하세요!