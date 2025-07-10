# Extended Copy 개발 가이드라인

## 📋 개요
이 문서는 Extended Copy 프로젝트의 실제 개발 시 따라야 할 구체적인 가이드라인과 규칙을 정의합니다.
코드 일관성, 품질 유지, 팀 협업을 위한 실용적인 규칙들을 제시합니다.

---

## 🏗 프로젝트 구조 규칙

### 디렉토리 구조
```
ExtendedCopy/
├── ExtendedCopy/
│   ├── App/                    // 앱 진입점 및 설정
│   │   ├── AppDelegate.swift
│   │   ├── main.swift
│   │   └── Info.plist
│   │
│   ├── Core/                   // 비즈니스 로직 (가장 중요)
│   │   ├── Models/
│   │   │   ├── ClipboardItem.swift
│   │   │   └── ClipboardState.swift
│   │   ├── Services/
│   │   │   ├── AccumulativeClipboard.swift
│   │   │   └── TextProcessor.swift
│   │   └── Configuration/
│   │       └── AppConfiguration.swift
│   │
│   ├── System/                 // 시스템 레벨 연동
│   │   ├── KeyboardEventMonitor.swift
│   │   ├── ClipboardManager.swift
│   │   └── PermissionManager.swift
│   │
│   ├── UI/                     // 사용자 인터페이스
│   │   ├── StatusBar/
│   │   │   ├── StatusBarController.swift
│   │   │   └── StatusBarViewModel.swift
│   │   ├── Notifications/
│   │   │   └── NotificationManager.swift
│   │   └── Settings/
│   │       └── SettingsView.swift
│   │
│   ├── Extensions/             // Swift 확장
│   │   ├── String+Extensions.swift
│   │   └── NSPasteboard+Extensions.swift
│   │
│   ├── Utils/                  // 유틸리티
│   │   ├── Logger.swift
│   │   └── Constants.swift
│   │
│   └── Resources/              // 리소스 파일
│       ├── Assets.xcassets
│       └── Localizable.strings
│
├── ExtendedCopyTests/          // 테스트
└── Documentation/              // 문서
```

### 파일 명명 규칙
- **클래스**: PascalCase (`AccumulativeClipboard.swift`)
- **프로토콜**: PascalCase + Protocol 접미사 (`ClipboardManagerProtocol.swift`)
- **확장**: 원본타입 + Extensions (`String+Extensions.swift`)
- **뷰모델**: 기능명 + ViewModel (`StatusBarViewModel.swift`)
- **테스트**: 원본클래스 + Tests (`AccumulativeClipboardTests.swift`)

---

## 📝 Swift 코딩 컨벤션

### 네이밍 컨벤션
```swift
// ✅ 올바른 네이밍
class AccumulativeClipboard {
    private var clipboardItems: [String] = []
    private let maximumItemCount = 100
    
    func addItem(_ text: String) { }
    func getCurrentItemCount() -> Int { }
    func resetAllItems() { }
}

// ❌ 잘못된 네이밍
class clipboard {  // 클래스명은 PascalCase
    private var items: [String] = []  // 구체적이지 않음
    private let MAX_COUNT = 100  // Swift는 lowerCamelCase
    
    func add(_ text: String) { }  // 무엇을 추가하는지 불명확
    func get() -> Int { }  // 무엇을 가져오는지 불명확
    func clear() { }  // reset이 더 명확
}
```

### 타입 명시와 타입 추론
```swift
// ✅ 타입 추론 활용
let statusBarController = StatusBarController()
let items = ["Hello", "World"]
let count = items.count

// ✅ 명시적 타입 선언이 필요한 경우
let delegate: ClipboardManagerDelegate? = nil
let timeout: TimeInterval = 5.0
let configuration: AppConfiguration = AppConfiguration.shared

// ❌ 불필요한 타입 명시
let statusBarController: StatusBarController = StatusBarController()
let items: [String] = ["Hello", "World"]
let count: Int = items.count
```

### 옵셔널 처리
```swift
// ✅ Guard 문 활용
func processClipboardText(_ text: String?) {
    guard let text = text, !text.isEmpty else {
        logger.warning("Invalid text provided")
        return
    }
    
    // 메인 로직
    addToClipboard(text)
}

// ✅ Nil coalescing 연산자
let displayText = clipboardText ?? "No content"
let itemCount = clipboardItems?.count ?? 0

// ❌ 강제 언래핑 (피할 것)
let text = clipboardText!  // 크래시 위험
```

### 클로저와 고차함수
```swift
// ✅ 간결한 클로저 문법
let validItems = items
    .filter { !$0.isEmpty }
    .map { $0.trimmingCharacters(in: .whitespacesAndNewlines) }
    .compactMap { $0.isEmpty ? nil : $0 }

// ✅ Trailing closure 문법
UIView.animate(withDuration: 0.3) {
    self.statusView.alpha = 1.0
}

// ❌ 불필요하게 복잡한 클로저
let validItems = items.filter { item in
    return !item.isEmpty
}.map { item in
    return item.trimmingCharacters(in: .whitespacesAndNewlines)
}
```

---

## 🏛 아키텍처 가이드라인

### MVVM 패턴 적용
```swift
// ✅ ViewModel (UI 로직)
class StatusBarViewModel: ObservableObject {
    @Published var statusIcon: String = "📋"
    @Published var isAccumulating: Bool = false
    
    private let clipboardService: ClipboardServiceProtocol
    private var cancellables = Set<AnyCancellable>()
    
    init(clipboardService: ClipboardServiceProtocol) {
        self.clipboardService = clipboardService
        setupBindings()
    }
    
    private func setupBindings() {
        clipboardService.statePublisher
            .map { state in
                switch state {
                case .normal: return "📋"
                case .accumulating(let count): return "📋\(count)"
                }
            }
            .assign(to: \.statusIcon, on: self)
            .store(in: &cancellables)
    }
}

// ✅ View (UI 표시)
class StatusBarController {
    private let viewModel: StatusBarViewModel
    private let statusItem: NSStatusItem
    
    init(viewModel: StatusBarViewModel) {
        self.viewModel = viewModel
        self.statusItem = NSStatusBar.system.statusItem(withLength: NSStatusItem.variableLength)
        setupUI()
    }
    
    private func setupUI() {
        statusItem.button?.title = viewModel.statusIcon
        
        viewModel.$statusIcon
            .receive(on: DispatchQueue.main)
            .sink { [weak self] icon in
                self?.statusItem.button?.title = icon
            }
            .store(in: &cancellables)
    }
}
```

### 의존성 주입
```swift
// ✅ 프로토콜 기반 의존성
protocol ClipboardServiceProtocol {
    var statePublisher: AnyPublisher<ClipboardState, Never> { get }
    func addItem(_ text: String)
    func reset()
}

// ✅ 생성자 주입
class ClipboardCoordinator {
    private let clipboardService: ClipboardServiceProtocol
    private let notificationService: NotificationServiceProtocol
    
    init(
        clipboardService: ClipboardServiceProtocol,
        notificationService: NotificationServiceProtocol
    ) {
        self.clipboardService = clipboardService
        self.notificationService = notificationService
    }
}

// ✅ 팩토리 패턴으로 객체 생성
class AppFactory {
    static func createClipboardCoordinator() -> ClipboardCoordinator {
        let clipboardService = ClipboardService()
        let notificationService = NotificationService()
        return ClipboardCoordinator(
            clipboardService: clipboardService,
            notificationService: notificationService
        )
    }
}
```

### 에러 처리 패턴
```swift
// ✅ Result 타입 활용
enum ClipboardError: Error, LocalizedError {
    case permissionDenied
    case textTooLarge(Int)
    case systemUnavailable
    
    var errorDescription: String? {
        switch self {
        case .permissionDenied:
            return "Clipboard access permission denied"
        case .textTooLarge(let size):
            return "Text too large: \(size) characters"
        case .systemUnavailable:
            return "Clipboard system unavailable"
        }
    }
}

func addToClipboard(_ text: String) -> Result<Void, ClipboardError> {
    // 유효성 검사
    guard hasPermission() else {
        return .failure(.permissionDenied)
    }
    
    guard text.count <= Constants.maxTextLength else {
        return .failure(.textTooLarge(text.count))
    }
    
    // 실제 작업 수행
    do {
        try performClipboardOperation(text)
        return .success(())
    } catch {
        return .failure(.systemUnavailable)
    }
}
```

---

## 🧪 테스트 가이드라인

### 테스트 구조
```swift
class AccumulativeClipboardTests: XCTestCase {
    
    // MARK: - Properties
    var sut: AccumulativeClipboard!
    var mockNotificationService: MockNotificationService!
    
    // MARK: - Setup & Teardown
    override func setUp() {
        super.setUp()
        mockNotificationService = MockNotificationService()
        sut = AccumulativeClipboard(notificationService: mockNotificationService)
    }
    
    override func tearDown() {
        sut = nil
        mockNotificationService = nil
        super.tearDown()
    }
    
    // MARK: - Tests
    func test_addItem_whenValidText_shouldIncreaseCount() {
        // Given
        let initialCount = sut.itemCount
        let testText = "Test text"
        
        // When
        let result = sut.addItem(testText)
        
        // Then
        XCTAssertEqual(result, .success(()))
        XCTAssertEqual(sut.itemCount, initialCount + 1)
        XCTAssertTrue(mockNotificationService.showNotificationCalled)
    }
    
    func test_addItem_whenEmptyText_shouldReturnError() {
        // Given
        let emptyText = ""
        
        // When
        let result = sut.addItem(emptyText)
        
        // Then
        XCTAssertEqual(result, .failure(.invalidText))
        XCTAssertFalse(mockNotificationService.showNotificationCalled)
    }
}
```

### Mock 객체 작성
```swift
class MockNotificationService: NotificationServiceProtocol {
    var showNotificationCalled = false
    var lastNotificationMessage: String?
    
    func showNotification(_ message: String) {
        showNotificationCalled = true
        lastNotificationMessage = message
    }
}

class MockClipboardService: ClipboardServiceProtocol {
    var items: [String] = []
    var stateSubject = CurrentValueSubject<ClipboardState, Never>(.normal)
    
    var statePublisher: AnyPublisher<ClipboardState, Never> {
        stateSubject.eraseToAnyPublisher()
    }
    
    func addItem(_ text: String) {
        items.append(text)
        stateSubject.send(.accumulating(count: items.count))
    }
    
    func reset() {
        items.removeAll()
        stateSubject.send(.normal)
    }
}
```

### 테스트 네이밍
```swift
// ✅ 명확한 테스트 이름 패턴
// test_[메서드명]_when[조건]_should[예상결과]

func test_addItem_whenValidText_shouldIncreaseCount()
func test_addItem_whenEmptyText_shouldReturnError()
func test_addItem_whenTextTooLarge_shouldReturnError()
func test_reset_whenCalled_shouldClearAllItems()
func test_getAllContent_whenItemsExist_shouldReturnJoinedText()
```

---

## 🔐 보안 가이드라인

### 권한 처리
```swift
class PermissionManager {
    
    enum PermissionStatus {
        case granted
        case denied
        case notDetermined
    }
    
    func checkAccessibilityPermission() -> PermissionStatus {
        let trusted = AXIsProcessTrusted()
        return trusted ? .granted : .denied
    }
    
    func requestAccessibilityPermission() {
        let options = [kAXTrustedCheckOptionPrompt.takeUnretainedValue(): true]
        AXIsProcessTrustedWithOptions(options as CFDictionary)
    }
    
    // ✅ 권한 상태 모니터링
    func startMonitoringPermissionStatus() {
        Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            let currentStatus = self?.checkAccessibilityPermission()
            // 권한 상태 변경 시 적절한 처리
        }
    }
}
```

### 데이터 보안
```swift
class SecureClipboardManager {
    
    // ✅ 메모리에만 저장, 디스크 저장 금지
    private var secureItems: [String] = []
    
    func addSecureItem(_ text: String) {
        // 민감한 데이터 감지
        if containsSensitiveData(text) {
            logger.warning("Sensitive data detected in clipboard")
            // 민감한 데이터는 제한된 시간만 보관
            DispatchQueue.main.asyncAfter(deadline: .now() + 30) {
                self.clearSensitiveData()
            }
        }
        
        secureItems.append(text)
    }
    
    private func containsSensitiveData(_ text: String) -> Bool {
        // 신용카드 번호, 패스워드 패턴 등 감지
        let patterns = [
            "\\d{4}[\\s-]?\\d{4}[\\s-]?\\d{4}[\\s-]?\\d{4}", // 신용카드
            "password[\\s:=]+\\w+", // 패스워드
        ]
        
        return patterns.contains { pattern in
            text.range(of: pattern, options: .regularExpression) != nil
        }
    }
    
    private func clearSensitiveData() {
        secureItems.removeAll()
        logger.info("Sensitive data cleared from memory")
    }
}
```

---

## ⚡ 성능 가이드라인

### 메모리 관리
```swift
class MemoryEfficientClipboard {
    private var items: [String] = []
    private let maxMemoryUsage: Int = 50 * 1024 * 1024 // 50MB
    private let maxItemCount: Int = 1000
    
    func addItem(_ text: String) {
        items.append(text)
        
        // ✅ 메모리 사용량 모니터링
        let currentMemoryUsage = calculateMemoryUsage()
        if currentMemoryUsage > maxMemoryUsage || items.count > maxItemCount {
            trimOldestItems()
        }
    }
    
    private func calculateMemoryUsage() -> Int {
        return items.reduce(0) { total, item in
            total + item.utf8.count
        }
    }
    
    private func trimOldestItems() {
        while items.count > maxItemCount / 2 {
            items.removeFirst()
        }
        logger.info("Trimmed old clipboard items to manage memory")
    }
}
```

### 비동기 처리
```swift
class AsyncClipboardManager {
    private let backgroundQueue = DispatchQueue(label: "clipboard.background", qos: .utility)
    private let mainQueue = DispatchQueue.main
    
    func addItemAsync(_ text: String, completion: @escaping (Result<Void, Error>) -> Void) {
        backgroundQueue.async { [weak self] in
            do {
                // 무거운 처리 작업
                let processedText = try self?.processLargeText(text)
                
                // UI 업데이트는 메인 큐에서
                self?.mainQueue.async {
                    completion(.success(()))
                }
            } catch {
                self?.mainQueue.async {
                    completion(.failure(error))
                }
            }
        }
    }
    
    private func processLargeText(_ text: String) throws -> String {
        // CPU 집약적인 텍스트 처리
        return text.trimmingCharacters(in: .whitespacesAndNewlines)
    }
}
```

---

## 📱 macOS 특화 가이드라인

### Accessibility 지원
```swift
extension StatusBarController {
    
    func setupAccessibility() {
        guard let button = statusItem.button else { return }
        
        // ✅ VoiceOver 지원
        button.accessibilityLabel = "Extended Copy Status"
        button.accessibilityHelp = "Shows the current state of clipboard accumulation"
        button.accessibilityRole = .button
        
        // ✅ 동적 접근성 업데이트
        updateAccessibilityInfo()
    }
    
    private func updateAccessibilityInfo() {
        let statusText = isAccumulating ? 
            "Accumulating \(itemCount) items" : 
            "Normal clipboard mode"
        
        statusItem.button?.accessibilityValue = statusText
    }
}
```

### 다크 모드 지원
```swift
class ThemeManager {
    
    func setupAppearance() {
        if #available(macOS 10.14, *) {
            // ✅ 시스템 테마 변경 감지
            DistributedNotificationCenter.default.addObserver(
                forName: Notification.Name("AppleInterfaceThemeChangedNotification"),
                object: nil,
                queue: .main
            ) { [weak self] _ in
                self?.updateAppearance()
            }
        }
    }
    
    private func updateAppearance() {
        let isDarkMode = NSApp.effectiveAppearance.bestMatch(from: [.darkAqua, .aqua]) == .darkAqua
        
        // 테마에 따른 UI 업데이트
        updateStatusBarIcon(isDarkMode: isDarkMode)
    }
}
```

### 시스템 이벤트 처리
```swift
class SystemEventManager {
    
    func setupSystemEventHandlers() {
        // ✅ 앱 활성화/비활성화 감지
        NotificationCenter.default.addObserver(
            forName: NSApplication.didBecomeActiveNotification,
            object: nil,
            queue: .main
        ) { [weak self] _ in
            self?.handleAppActivation()
        }
        
        // ✅ 시스템 종료 감지
        NotificationCenter.default.addObserver(
            forName: NSApplication.willTerminateNotification,
            object: nil,
            queue: .main
        ) { [weak self] _ in
            self?.handleAppTermination()
        }
        
        // ✅ 화면 잠금 감지
        DistributedNotificationCenter.default.addObserver(
            forName: Notification.Name("com.apple.screenIsLocked"),
            object: nil,
            queue: .main
        ) { [weak self] _ in
            self?.handleScreenLock()
        }
    }
    
    private func handleAppActivation() {
        // 앱 활성화 시 필요한 처리
        logger.info("App activated")
    }
    
    private func handleScreenLock() {
        // 보안상 클립보드 초기화
        clipboardManager.reset()
        logger.info("Clipboard cleared due to screen lock")
    }
}
```

---

## 📊 로깅 및 모니터링

### 구조화된 로깅
```swift
import os

class Logger {
    static let shared = Logger()
    
    private let subsystem = "com.extended-copy.app"
    private let generalLog = OSLog(subsystem: "com.extended-copy.app", category: "general")
    private let performanceLog = OSLog(subsystem: "com.extended-copy.app", category: "performance")
    private let securityLog = OSLog(subsystem: "com.extended-copy.app", category: "security")
    
    // ✅ 로그 레벨별 메서드
    func debug(_ message: String, category: LogCategory = .general) {
        os_log(.debug, log: logForCategory(category), "%@", message)
    }
    
    func info(_ message: String, category: LogCategory = .general) {
        os_log(.info, log: logForCategory(category), "%@", message)
    }
    
    func warning(_ message: String, category: LogCategory = .general) {
        os_log(.default, log: logForCategory(category), "⚠️ %@", message)
    }
    
    func error(_ message: String, category: LogCategory = .general) {
        os_log(.error, log: logForCategory(category), "❌ %@", message)
    }
    
    // ✅ 성능 측정
    func measurePerformance<T>(_ operation: String, block: () throws -> T) rethrows -> T {
        let startTime = CFAbsoluteTimeGetCurrent()
        let result = try block()
        let timeElapsed = CFAbsoluteTimeGetCurrent() - startTime
        
        os_log(.info, log: performanceLog, "%@ completed in %.3f seconds", operation, timeElapsed)
        return result
    }
    
    private func logForCategory(_ category: LogCategory) -> OSLog {
        switch category {
        case .general: return generalLog
        case .performance: return performanceLog
        case .security: return securityLog
        }
    }
}

enum LogCategory {
    case general
    case performance
    case security
}
```

### 사용 예시
```swift
// ✅ 로깅 사용법
class ClipboardService {
    
    func addItem(_ text: String) {
        Logger.shared.info("Adding item to clipboard", category: .general)
        
        // 성능 측정
        let result = Logger.shared.measurePerformance("Clipboard operation") {
            performClipboardOperation(text)
        }
        
        if containsSensitiveData(text) {
            Logger.shared.warning("Sensitive data detected", category: .security)
        }
        
        Logger.shared.debug("Clipboard now has \(items.count) items")
    }
}
```

---

## 📦 배포 및 빌드 가이드라인

### 빌드 설정
```swift
// Build Settings
// MARK: - Release Configuration
// - Enable optimizations
// - Strip debug symbols
// - Enable bitcode (if applicable)
// - Set deployment target to macOS 12.0

// Info.plist 설정
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>CFBundleDisplayName</key>
    <string>Extended Copy</string>
    <key>CFBundleIdentifier</key>
    <string>com.extended-copy.app</string>
    <key>CFBundleVersion</key>
    <string>1.0.0</string>
    <key>LSUIElement</key>
    <true/> <!-- 메뉴바 앱으로 설정 -->
    <key>NSHumanReadableCopyright</key>
    <string>© 2024 Extended Copy. All rights reserved.</string>
</dict>
</plist>
```

### 코드 사이닝
```bash
# ✅ 개발용 사이닝
codesign --force --verify --verbose --sign "Developer ID Application: Your Name" ExtendedCopy.app

# ✅ 배포용 공증
xcrun notarytool submit ExtendedCopy.app.zip --keychain-profile "AC_PASSWORD" --wait
```

---

## 🔍 코드 리뷰 체크리스트

### 기능성 체크
- [ ] 코드가 요구사항을 충족하는가?
- [ ] 엣지 케이스가 고려되었는가?
- [ ] 에러 처리가 적절한가?

### 성능 체크
- [ ] 불필요한 메모리 할당이 없는가?
- [ ] 비동기 처리가 적절히 사용되었는가?
- [ ] 메모리 누수 가능성이 없는가?

### 보안 체크
- [ ] 입력 유효성 검사가 충분한가?
- [ ] 민감한 데이터가 안전하게 처리되는가?
- [ ] 권한 체크가 적절한가?

### 코드 품질 체크
- [ ] 네이밍이 명확하고 일관성 있는가?
- [ ] 함수가 단일 책임을 가지는가?
- [ ] 주석이 필요한 부분에 적절히 작성되었는가?

### 테스트 체크
- [ ] 단위 테스트가 작성되었는가?
- [ ] 테스트 커버리지가 충분한가?
- [ ] Mock 객체가 적절히 사용되었는가?

---

## 📚 참고 자료 및 도구

### 개발 도구
- **Xcode**: 최신 버전 사용 권장
- **SwiftLint**: 코드 스타일 체크
- **SwiftFormat**: 자동 코드 포맷팅
- **Instruments**: 성능 프로파일링

### 유용한 명령어
```bash
# SwiftLint 실행
swiftlint lint

# SwiftFormat 실행
swiftformat .

# 테스트 실행 (커버리지 포함)
xcodebuild test -scheme ExtendedCopy -enableCodeCoverage YES

# 메모리 누수 감지
leaks -atExit -- /path/to/ExtendedCopy.app/Contents/MacOS/ExtendedCopy
```

### 권장 라이브러리 (최소한으로 사용)
- **Combine**: 비동기 처리 (iOS 13+)
- **SwiftUI**: 설정 화면 (macOS 11+)
- **OSLog**: 구조화된 로깅

---

## 🎯 마지막으로

이 가이드라인은 Extended Copy 프로젝트의 코드 품질과 일관성을 유지하기 위한 실용적인 규칙들입니다. 
모든 규칙을 엄격히 따르되, 상황에 따라 합리적인 예외는 인정하되 그 이유를 명확히 문서화하시기 바랍니다.

**핵심 원칙:**
1. **일관성** - 팀의 규칙을 일관되게 적용
2. **가독성** - 다른 개발자가 쉽게 이해할 수 있는 코드
3. **안정성** - 철저한 테스트와 에러 처리
4. **성능** - 효율적인 메모리 사용과 응답성
5. **보안** - 사용자 데이터와 시스템 보안 우선

지속적으로 이 가이드라인을 개선하고, 프로젝트 진행 중 발견되는 베스트 프랙티스를 추가해 나가시기 바랍니다.