# 시니어 개발자 베스트 프랙티스 가이드

## 📚 개요
이 문서는 Extended Copy 프로젝트 개발 시 참고할 시니어 개발자들의 베스트 프랙티스와 원칙들을 정리한 가이드입니다. 
고품질 소프트웨어 개발을 위한 검증된 방법론과 멘탈 모델을 제시합니다.

---

## 🏗 SOLID 원칙

### S - Single Responsibility Principle (단일 책임 원칙)
> 클래스는 단 하나의 책임만 가져야 하며, 변경할 이유도 단 하나여야 한다.

**좋은 예시:**
```swift
// ❌ 나쁜 예 - 여러 책임이 혼재
class ClipboardManager {
    func copyText(_ text: String) { /* 복사 로직 */ }
    func showNotification(_ message: String) { /* UI 로직 */ }
    func saveToFile(_ data: Data) { /* 파일 I/O 로직 */ }
}

// ✅ 좋은 예 - 책임 분리
class ClipboardManager {
    func copyText(_ text: String) { /* 복사 로직만 */ }
}

class NotificationManager {
    func showNotification(_ message: String) { /* 알림 로직만 */ }
}

class FileManager {
    func saveToFile(_ data: Data) { /* 파일 I/O 로직만 */ }
}
```

### O - Open/Closed Principle (개방/폐쇄 원칙)
> 소프트웨어 개체는 확장에는 열려 있어야 하고, 수정에는 닫혀 있어야 한다.

**좋은 예시:**
```swift
// ✅ 확장 가능한 구조
protocol TextProcessor {
    func process(_ text: String) -> String
}

class BasicTextProcessor: TextProcessor {
    func process(_ text: String) -> String {
        return text
    }
}

class UppercaseProcessor: TextProcessor {
    func process(_ text: String) -> String {
        return text.uppercased()
    }
}

class ClipboardManager {
    private let processor: TextProcessor
    
    init(processor: TextProcessor) {
        self.processor = processor
    }
    
    func processAndCopy(_ text: String) {
        let processed = processor.process(text)
        // 복사 로직
    }
}
```

### L - Liskov Substitution Principle (리스코프 치환 원칙)
> 프로그램의 객체는 프로그램의 정확성을 깨뜨리지 않으면서 하위 타입의 인스턴스로 바꿀 수 있어야 한다.

### I - Interface Segregation Principle (인터페이스 분리 원칙)
> 특정 클라이언트를 위한 인터페이스 여러 개가 범용 인터페이스 하나보다 낫다.

### D - Dependency Inversion Principle (의존관계 역전 원칙)
> 프로그래머는 추상화에 의존해야지, 구체화에 의존하면 안 된다.

---

## 🧠 시니어 개발자 멘탈 모델

### 1. Pareto Principle (80/20 법칙)
> 20%의 원인이 80%의 결과를 만든다.

**적용 방법:**
- 20%의 핵심 기능에 80%의 노력 집중
- 80%의 버그는 20%의 코드에서 발생
- 성능 최적화 시 20%의 병목이 80%의 문제 유발

```swift
// ✅ 핵심 기능에 집중
class AccumulativeClipboard {
    // 핵심 20%: 텍스트 누적과 초기화
    func addText(_ text: String) { /* 핵심 로직 */ }
    func reset() { /* 핵심 로직 */ }
    
    // 나머지 80%: 편의 기능들
    func getItemCount() -> Int { /* 편의 기능 */ }
    func getLastItem() -> String? { /* 편의 기능 */ }
}
```

### 2. Occam's Razor (오컴의 면도날)
> 가장 간단한 해결책이 최선이다.

```swift
// ❌ 복잡한 해결책
class ComplexClipboard {
    private var items: [ClipboardItem] = []
    private var indexes: [String: Int] = [:]
    private var metadata: [String: Any] = [:]
    
    func addItem(_ text: String) {
        let item = ClipboardItem(text: text, timestamp: Date(), id: UUID())
        items.append(item)
        indexes[item.id.uuidString] = items.count - 1
        metadata[item.id.uuidString] = ["length": text.count]
    }
}

// ✅ 간단한 해결책
class SimpleClipboard {
    private var items: [String] = []
    
    func addItem(_ text: String) {
        items.append(text)
    }
}
```

### 3. YAGNI (You Aren't Gonna Need It)
> 필요할 때까지 구현하지 마라.

### 4. DRY (Don't Repeat Yourself)
> 반복을 피하라.

```swift
// ❌ 코드 중복
func saveToUserDefaults(_ text: String, key: String) {
    UserDefaults.standard.set(text, forKey: key)
    UserDefaults.standard.synchronize()
}

func saveToKeychain(_ text: String, key: String) {
    // keychain 저장 로직
    UserDefaults.standard.synchronize() // 중복!
}

// ✅ 공통 로직 추출
protocol Storage {
    func save(_ text: String, key: String)
}

class UserDefaultsStorage: Storage {
    func save(_ text: String, key: String) {
        UserDefaults.standard.set(text, forKey: key)
        synchronize()
    }
    
    private func synchronize() {
        UserDefaults.standard.synchronize()
    }
}
```

---

## 🧹 클린 코드 원칙

### 의미 있는 이름 사용
```swift
// ❌ 나쁜 이름
var d: Int // 무엇을 의미하는지 불분명
var data: [String] // 너무 일반적
func calc() -> Int // 무엇을 계산하는지 불분명

// ✅ 좋은 이름
var daysSinceCreation: Int
var clipboardItems: [String]
func calculateTotalCharacterCount() -> Int
```

### 함수는 작고 한 가지 일만
```swift
// ❌ 너무 많은 일을 하는 함수
func processClipboard(_ text: String) {
    // 유효성 검사
    guard !text.isEmpty else { return }
    
    // 텍스트 변환
    let processed = text.trimmingCharacters(in: .whitespacesAndNewlines)
    
    // 클립보드에 추가
    clipboardItems.append(processed)
    
    // UI 업데이트
    updateStatusBar()
    
    // 알림 표시
    showNotification("Added to clipboard")
    
    // 로그 기록
    logger.log("Clipboard updated")
}

// ✅ 책임이 분리된 함수들
func validateText(_ text: String) -> Bool {
    return !text.isEmpty
}

func processText(_ text: String) -> String {
    return text.trimmingCharacters(in: .whitespacesAndNewlines)
}

func addToClipboard(_ text: String) {
    clipboardItems.append(text)
}

func updateUI() {
    updateStatusBar()
    showNotification("Added to clipboard")
}
```

### 주석보다는 코드로 설명
```swift
// ❌ 주석에 의존
func process(_ items: [String]) -> [String] {
    var result: [String] = []
    
    // 빈 문자열 제거
    for item in items {
        if !item.isEmpty {
            result.append(item)
        }
    }
    
    // 중복 제거
    var unique: [String] = []
    for item in result {
        if !unique.contains(item) {
            unique.append(item)
        }
    }
    
    return unique
}

// ✅ 자체 설명하는 코드
func process(_ items: [String]) -> [String] {
    return items
        .filter(isNotEmpty)
        .removingDuplicates()
}

private func isNotEmpty(_ text: String) -> Bool {
    return !text.isEmpty
}

extension Array where Element: Equatable {
    func removingDuplicates() -> [Element] {
        var unique: [Element] = []
        for item in self {
            if !unique.contains(item) {
                unique.append(item)
            }
        }
        return unique
    }
}
```

---

## 🛡 Defensive Programming

### 입력 유효성 검사
```swift
func addToClipboard(_ text: String?) {
    // ✅ 방어적 프로그래밍
    guard let text = text, !text.isEmpty else {
        logger.warning("Attempted to add empty text to clipboard")
        return
    }
    
    guard text.count <= maximumTextLength else {
        logger.error("Text too long: \(text.count) characters")
        showError("Text is too long to copy")
        return
    }
    
    // 실제 로직 수행
    clipboardItems.append(text)
}
```

### 예상치 못한 상황 처리
```swift
func getCurrentClipboardText() -> String? {
    do {
        let pasteboard = NSPasteboard.general
        
        // ✅ 안전한 타입 검사
        guard let availableTypes = pasteboard.types,
              availableTypes.contains(.string) else {
            logger.info("No text content in clipboard")
            return nil
        }
        
        return pasteboard.string(forType: .string)
        
    } catch {
        // ✅ 예외 상황 처리
        logger.error("Failed to access clipboard: \(error)")
        showError("Unable to access clipboard")
        return nil
    }
}
```

### 상태 불변성 보장
```swift
class ClipboardState {
    private var _items: [String] = []
    
    // ✅ 읽기 전용 접근
    var items: [String] {
        return _items
    }
    
    // ✅ 제어된 수정
    func addItem(_ text: String) {
        guard validateText(text) else { return }
        _items.append(text)
    }
    
    private func validateText(_ text: String) -> Bool {
        return !text.isEmpty && text.count <= 10000
    }
}
```

---

## 🚨 에러 핸들링 전략

### 구체적인 에러 타입 정의
```swift
enum ClipboardError: Error, LocalizedError {
    case accessDenied
    case textTooLarge(size: Int, limit: Int)
    case invalidFormat
    case systemUnavailable
    
    var errorDescription: String? {
        switch self {
        case .accessDenied:
            return "Clipboard access denied. Please check permissions."
        case .textTooLarge(let size, let limit):
            return "Text too large (\(size) chars). Limit is \(limit) chars."
        case .invalidFormat:
            return "Invalid text format for clipboard operation."
        case .systemUnavailable:
            return "Clipboard system is currently unavailable."
        }
    }
}
```

### Result 타입 활용
```swift
func addTextToClipboard(_ text: String) -> Result<Void, ClipboardError> {
    guard !text.isEmpty else {
        return .failure(.invalidFormat)
    }
    
    guard text.count <= 10000 else {
        return .failure(.textTooLarge(size: text.count, limit: 10000))
    }
    
    do {
        try performClipboardOperation(text)
        return .success(())
    } catch {
        return .failure(.systemUnavailable)
    }
}

// 사용법
let result = addTextToClipboard(userInput)
switch result {
case .success:
    showSuccessMessage()
case .failure(let error):
    showError(error.localizedDescription)
}
```

---

## 🧪 테스트 주도 개발 (TDD)

### TDD 사이클
1. **Red**: 실패하는 테스트 작성
2. **Green**: 테스트를 통과하는 최소 코드 작성
3. **Refactor**: 코드 개선

### 좋은 테스트 작성법
```swift
class AccumulativeClipboardTests: XCTestCase {
    
    var clipboard: AccumulativeClipboard!
    
    override func setUp() {
        super.setUp()
        clipboard = AccumulativeClipboard()
    }
    
    override func tearDown() {
        clipboard = nil
        super.tearDown()
    }
    
    // ✅ 테스트 이름이 의도를 명확히 표현
    func testAddItem_WhenTextIsValid_ShouldIncreaseItemCount() {
        // Given
        let initialCount = clipboard.itemCount
        let testText = "Test text"
        
        // When
        clipboard.addItem(testText)
        
        // Then
        XCTAssertEqual(clipboard.itemCount, initialCount + 1)
        XCTAssertEqual(clipboard.getAllContent(), testText)
    }
    
    func testAddItem_WhenTextIsEmpty_ShouldNotAddItem() {
        // Given
        let initialCount = clipboard.itemCount
        
        // When
        clipboard.addItem("")
        
        // Then
        XCTAssertEqual(clipboard.itemCount, initialCount)
    }
    
    func testReset_WhenCalled_ShouldClearAllItems() {
        // Given
        clipboard.addItem("Item 1")
        clipboard.addItem("Item 2")
        XCTAssertGreaterThan(clipboard.itemCount, 0)
        
        // When
        clipboard.reset()
        
        // Then
        XCTAssertEqual(clipboard.itemCount, 0)
        XCTAssertTrue(clipboard.getAllContent().isEmpty)
    }
}
```

### 테스트 가능한 코드 설계
```swift
// ❌ 테스트하기 어려운 코드
class ClipboardManager {
    func addItem(_ text: String) {
        let pasteboard = NSPasteboard.general // 하드코딩된 의존성
        pasteboard.setString(text, forType: .string)
        
        let notification = NSUserNotification() // 테스트 시 실제 알림 발생
        notification.title = "Added to clipboard"
        NSUserNotificationCenter.default.deliver(notification)
    }
}

// ✅ 테스트 가능한 코드
protocol PasteboardService {
    func setString(_ string: String)
}

protocol NotificationService {
    func showNotification(_ message: String)
}

class ClipboardManager {
    private let pasteboardService: PasteboardService
    private let notificationService: NotificationService
    
    init(pasteboardService: PasteboardService, 
         notificationService: NotificationService) {
        self.pasteboardService = pasteboardService
        self.notificationService = notificationService
    }
    
    func addItem(_ text: String) {
        pasteboardService.setString(text)
        notificationService.showNotification("Added to clipboard")
    }
}

// 테스트에서 Mock 사용 가능
class MockPasteboardService: PasteboardService {
    var setStringCalled = false
    var lastSetString = ""
    
    func setString(_ string: String) {
        setStringCalled = true
        lastSetString = string
    }
}
```

---

## 📊 코드 리뷰 가이드라인

### 체크리스트
- [ ] **기능성**: 코드가 의도한 대로 동작하는가?
- [ ] **가독성**: 다른 개발자가 쉽게 이해할 수 있는가?
- [ ] **성능**: 불필요한 성능 저하가 없는가?
- [ ] **보안**: 보안 취약점이 없는가?
- [ ] **테스트**: 적절한 테스트가 포함되어 있는가?
- [ ] **일관성**: 기존 코드 스타일과 일관성이 있는가?

### 좋은 코드 리뷰 코멘트
```swift
// ✅ 건설적인 피드백
// "이 부분에서 optionals을 unwrapping할 때 guard let을 사용하면 
// early return으로 더 읽기 쉬운 코드가 될 것 같습니다."

// "메모리 누수 방지를 위해 weak reference를 고려해보시면 어떨까요?"

// "이 로직을 별도 함수로 분리하면 재사용성이 높아질 것 같습니다."

// ❌ 도움이 되지 않는 코멘트
// "이 코드는 잘못되었습니다."
// "왜 이렇게 했나요?"
```

---

## 🏛 아키텍처 패턴

### MVC vs MVVM
```swift
// ✅ MVVM 패턴 (Extended Copy에 적합)
class StatusBarViewModel: ObservableObject {
    @Published var icon: String = "📋"
    @Published var itemCount: Int = 0
    
    private let clipboardManager: ClipboardManager
    
    init(clipboardManager: ClipboardManager) {
        self.clipboardManager = clipboardManager
        setupBindings()
    }
    
    func setupBindings() {
        clipboardManager.statePublisher
            .map { state in
                switch state {
                case .normal: return "📋"
                case .accumulating(let count): 
                    return count == 1 ? "📋+" : "📋\(count)"
                }
            }
            .assign(to: &$icon)
    }
}

class StatusBarController {
    private let viewModel: StatusBarViewModel
    private var statusItem: NSStatusItem
    
    init(viewModel: StatusBarViewModel) {
        self.viewModel = viewModel
        self.statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
        setupBindings()
    }
    
    private func setupBindings() {
        viewModel.$icon
            .sink { [weak self] icon in
                self?.statusItem.button?.title = icon
            }
            .store(in: &cancellables)
    }
}
```

---

## 📈 성능 최적화 원칙

### Premature Optimization 피하기
> "Premature optimization is the root of all evil" - Donald Knuth

1. **먼저 동작하게 만들어라**
2. **측정하라**
3. **병목을 찾아라**
4. **최적화하라**
5. **다시 측정하라**

### 메모리 관리
```swift
// ✅ 메모리 효율적인 코드
class ClipboardManager {
    private var items: [String] = []
    private let maxItems = 100 // 메모리 사용량 제한
    
    func addItem(_ text: String) {
        items.append(text)
        
        // 오래된 항목 제거
        if items.count > maxItems {
            items.removeFirst(items.count - maxItems)
        }
    }
}

// ✅ 약한 참조로 메모리 누수 방지
class NotificationManager {
    weak var delegate: NotificationDelegate?
    
    func showNotification(_ message: String) {
        delegate?.didReceiveNotification(message)
    }
}
```

---

## 🔍 디버깅 전략

### 로깅 시스템
```swift
import os

class Logger {
    static let shared = Logger()
    private let osLog = OSLog(subsystem: "com.example.extendedcopy", category: "general")
    
    func debug(_ message: String, file: String = #file, function: String = #function, line: Int = #line) {
        os_log(.debug, log: osLog, "%@:%@ - %@", 
               URL(fileURLWithPath: file).lastPathComponent, 
               function, 
               message)
    }
    
    func error(_ message: String, file: String = #file, function: String = #function, line: Int = #line) {
        os_log(.error, log: osLog, "%@:%@ - %@", 
               URL(fileURLWithPath: file).lastPathComponent, 
               function, 
               message)
    }
}

// 사용법
Logger.shared.debug("Adding item to clipboard: \(text)")
Logger.shared.error("Failed to access pasteboard: \(error)")
```

### 어설션 활용
```swift
func addItem(_ text: String) {
    // ✅ 개발 중 조건 검증
    assert(!text.isEmpty, "Text should not be empty")
    assert(text.count <= 10000, "Text too long: \(text.count)")
    
    // 실제 로직
    items.append(text)
}
```

---

## 📱 플랫폼별 고려사항 (macOS)

### Accessibility 준수
```swift
// ✅ VoiceOver 지원
statusItem.button?.accessibilityLabel = "Extended Copy Status"
statusItem.button?.accessibilityHelp = "Shows current clipboard accumulation status"

// ✅ 키보드 네비게이션 지원
menu.addItem(withTitle: "Settings", action: #selector(showSettings), keyEquivalent: ",")
```

### 시스템 통합
```swift
// ✅ 다크 모드 지원
class StatusBarController {
    private func updateAppearance() {
        if #available(macOS 10.14, *) {
            let appearance = NSApp.effectiveAppearance
            if appearance.bestMatch(from: [.darkAqua, .aqua]) == .darkAqua {
                // 다크 모드 UI
            } else {
                // 라이트 모드 UI
            }
        }
    }
}

// ✅ 시스템 이벤트 감지
NotificationCenter.default.addObserver(
    forName: NSApplication.didChangeOcclusionStateNotification,
    object: nil,
    queue: .main
) { _ in
    // 앱 가시성 변경 처리
}
```

---

## 🚀 배포 및 유지보수

### 버전 관리
```swift
// ✅ 시멘틱 버저닝
// MAJOR.MINOR.PATCH
// 1.0.0 -> 1.0.1 (버그 수정)
// 1.0.1 -> 1.1.0 (새 기능)
// 1.1.0 -> 2.0.0 (호환성 깨는 변경)
```

### 설정 관리
```swift
class Configuration {
    static let shared = Configuration()
    
    @UserDefault("separator", defaultValue: "\n")
    var separator: String
    
    @UserDefault("maxItems", defaultValue: 100)
    var maxItems: Int
    
    @UserDefault("autoReset", defaultValue: true)
    var autoReset: Bool
}

@propertyWrapper
struct UserDefault<T> {
    let key: String
    let defaultValue: T
    
    var wrappedValue: T {
        get {
            UserDefaults.standard.object(forKey: key) as? T ?? defaultValue
        }
        set {
            UserDefaults.standard.set(newValue, forKey: key)
        }
    }
}
```

---

## 💡 핵심 원칙 요약

1. **단순함을 추구하라** - 복잡한 것보다 단순한 해결책
2. **일찍, 자주 테스트하라** - TDD로 안정성 확보
3. **가독성을 우선하라** - 코드는 사람이 읽는 것
4. **책임을 분리하라** - 각 클래스는 하나의 책임만
5. **방어적으로 프로그래밍하라** - 예상치 못한 상황 대비
6. **성능은 측정 후 최적화하라** - 추측보다는 데이터
7. **일관성을 유지하라** - 팀의 코딩 스타일 준수
8. **실패에서 빠르게 학습하라** - 피드백 루프 단축

---

이 가이드는 Extended Copy 프로젝트뿐만 아니라 모든 소프트웨어 개발 프로젝트에서 활용할 수 있는 검증된 베스트 프랙티스들을 담고 있습니다. 지속적으로 참고하여 고품질 코드를 작성하시기 바랍니다.