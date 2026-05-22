---
title: Flutter 개발 환경 초기 세팅 (macOS)
date: 2026-05-22 00:00:00 +0900
categories: [개발, Flutter]
tags: [flutter, dart, ios, android, setup]
---

Flutter 개발 환경을 처음 세팅하는 과정을 정리했다. macOS 기준이다.

---

## Flutter SDK 설치

### Homebrew로 설치

```bash
brew install --cask flutter
```

### 직접 다운로드

[flutter.dev](https://docs.flutter.dev/install/manual)에서 최신 stable 버전을 내려받는다. Mac의 CPU 아키텍처에 따라 다른 파일을 받아야 한다.

- Apple Silicon (M1 이상): ARM64용 번들
- Intel Mac: x86_64용 번들

SDK를 저장할 폴더를 만들고 압축을 푼다.

```bash
mkdir -p ~/develop
unzip ~/Downloads/flutter_macos_*-stable.zip -d ~/develop/
```

PATH에 추가한다. 공식 문서 기준으로 `~/.zprofile`에 추가하는 걸 권장한다. (`~/.zshrc`에 추가해도 동작하지만, login shell 기준으로는 `.zprofile`이 더 적합하다.)

```bash
export PATH="$HOME/develop/flutter/bin:$PATH"
```

변경사항을 적용한다.

```bash
source ~/.zprofile
```

---

## Flutter Doctor

설치 후 가장 먼저 실행해야 할 명령어다.

```bash
flutter doctor
```

개발 환경에 필요한 구성 요소들이 제대로 설치됐는지 체크해서 결과를 보여준다.

```
Doctor summary (to see all details, run flutter doctor -v):
[✓] Flutter (Channel stable, 3.x.x)
[✓] Android toolchain
[✓] Xcode
[✓] Chrome
[✓] Android Studio
[✓] VS Code
[✓] Connected device
```

항목마다 `[✓]`, `[!]`, `[✗]`로 상태를 표시한다. `[!]`는 경고, `[✗]`는 필수 항목 누락이다. 모두 `[✓]`가 될 때까지 아래 단계를 진행한다.

---

## Flutter 채널 설정

설치 후 stable 채널로 변경하고 최신 버전으로 업그레이드한다.

```bash
flutter channel stable
flutter upgrade
```

---

## Android 환경 설정

### JDK 17 설치

Android 빌드에 JDK 17이 필요하다. Android Studio를 설치하면 번들로 포함되어 있어서 별도 설치 없이 아래처럼 JAVA_HOME을 설정할 수 있다.

`~/.zshrc`에 추가한다.

```bash
export JAVA_HOME="/Applications/Android Studio.app/Contents/jbr/Contents/Home"
export PATH="$JAVA_HOME/bin:$PATH"
```

적용한다.

```bash
source ~/.zshrc
```

확인한다.

```bash
java --version
```

### Android Studio 설치

[developer.android.com](https://developer.android.com/studio)에서 내려받아 설치한다.

### SDK 및 도구 설치

Android Studio 실행 후 **SDK Manager**를 연다.

- Welcome 화면: **More Actions → SDK Manager**
- 프로젝트가 열려 있을 때: **Tools → SDK Manager**

**SDK Platforms** 탭에서 최신 API Level이 설치되어 있는지 확인한다.

**SDK Tools** 탭에서 아래 항목들을 설치한다.

- Android SDK Build-Tools
- Android SDK Command-line Tools
- Android Emulator
- Android SDK Platform-Tools
- CMake
- NDK (Side by side)

### Android 라이선스 동의

```bash
flutter doctor --android-licenses
```

모든 항목에 `y`를 입력한다.

### 에뮬레이터 생성

Android Studio → **Device Manager** → **+** (Create Virtual Device)에서 에뮬레이터를 생성한다.

- Form Factor: Phone 또는 Tablet
- 디바이스: Pixel 계열 권장
- 시스템 이미지: 개발 Mac의 아키텍처에 맞는 이미지 선택 (Apple Silicon이면 ARM Images)
- Additional Settings → Emulated Performance → Graphics: **Hardware** 포함 옵션 선택

---

## iOS 환경 설정 (macOS 전용)

### Xcode 설치

[developer.apple.com](https://developer.apple.com/xcode/)에서 내려받거나 App Store에서 설치한다. 용량이 크기 때문에 시간이 걸린다.

### Xcode 커맨드라인 도구 설정

```bash
sudo sh -c 'xcode-select -s /Applications/Xcode.app/Contents/Developer && xcodebuild -runFirstLaunch'
```

라이선스 동의도 필요하다.

```bash
sudo xcodebuild -license
```

### iOS 플랫폼 및 시뮬레이터 다운로드

```bash
xcodebuild -downloadPlatform iOS
```

시뮬레이터 이미지까지 포함해서 내려받는다. 시간이 상당히 걸린다.

### CocoaPods 설치

iOS 의존성 관리 도구다. 네이티브 iOS 코드를 사용하는 Flutter 플러그인을 지원하기 위해 필요하다.

```bash
sudo gem install cocoapods
```

Ruby 버전 이슈가 생기면 rbenv로 관리하는 걸 권장한다.

```bash
brew install rbenv
rbenv install 3.2.0
rbenv global 3.2.0
gem install cocoapods
```

### Swift Package Manager 비활성화

Flutter 최신 버전은 Swift Package Manager(SPM) 통합을 실험적으로 추가하는데, CocoaPods를 사용하는 프로젝트에서 Firebase 등 일부 패키지와 충돌이 발생할 수 있다. CocoaPods 기반 프로젝트라면 SPM을 꺼두는 게 안전하다.

```bash
flutter config --no-enable-swift-package-manager
```

### iOS 시뮬레이터 실행

```bash
open -a Simulator
```

또는 Xcode → **Window** → **Devices and Simulators**에서 실행할 수 있다.

---

## VS Code 설정

Flutter 개발에 가장 많이 쓰는 에디터다.

**필수 익스텐션 설치**

- Flutter (Dart Code 팀 공식)
- Dart

익스텐션 설치 후 VS Code를 재시작하면 Flutter 프로젝트에서 자동완성, 디버깅, 핫 리로드 등을 지원한다.

---

## 첫 프로젝트 생성

```bash
flutter create my_app
cd my_app
flutter run
```

`flutter run`을 실행하면 연결된 디바이스 또는 에뮬레이터 목록이 표시된다. 번호를 선택하면 앱이 실행된다.

여러 디바이스가 연결된 경우 직접 지정할 수 있다.

```bash
flutter run -d ios        # iOS 시뮬레이터
flutter run -d android    # Android 에뮬레이터
flutter run -d chrome     # 웹 브라우저
```

---

## 자주 쓰는 명령어

| 명령어 | 설명 |
|--------|------|
| `flutter run` | 앱 실행 |
| `flutter build apk` | Android APK 빌드 |
| `flutter build ios` | iOS 빌드 |
| `flutter pub get` | 패키지 설치 |
| `flutter pub upgrade` | 패키지 업데이트 |
| `flutter clean` | 빌드 캐시 초기화 |
| `flutter doctor` | 환경 점검 |
| `flutter devices` | 연결된 디바이스 목록 |
| `flutter emulators` | 사용 가능한 에뮬레이터 목록 |

---

## flutter doctor 자주 보이는 오류

**`Android licenses not accepted`**

```bash
flutter doctor --android-licenses
```

**`CocoaPods not installed`**

```bash
sudo gem install cocoapods
```

**`Xcode not installed or outdated`**

App Store 또는 Apple Developer 사이트에서 Xcode를 최신 버전으로 업데이트한다. 구버전 macOS에서는 지원하는 Xcode 버전이 제한될 수 있다.

**`No devices available`**

에뮬레이터가 실행 중인지 확인하거나, 실제 디바이스를 연결한 뒤 다시 실행한다.

```bash
flutter devices    # 연결된 디바이스 목록 확인
```

**`Unable to find bundled Java version`**

Android Studio 업데이트 후 간혹 발생한다. Android Studio를 최신 버전으로 재설치하거나, `flutter config --android-studio-dir` 명령으로 경로를 재설정한다.
