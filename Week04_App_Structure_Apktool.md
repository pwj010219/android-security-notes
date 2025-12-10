# Week 4: Android App Structure & Apktool

## 1. 정적 분석 및 파일 구조 (Static Analysis & Structure)

### 정적 분석 (Static Analysis)
- 앱을 실행하지 않고 분석하는 방법 (App not running)

### 압축 해제 도구 비교

#### zip (unzip)
- `classes.dex`, `AndroidManifest.xml` 파일이 이진(Binary) 형태라 내용을 볼 수 없음 (no view xml file)

#### apktool
- 바이너리 XML 파일을 디코딩하여 사람이 읽을 수 있는 형태로 변환
- 예: AndroidManifest.xml = 내용 확인 가능 (yes view)
- xml = 앱 데이터 (App data)

```bash
# 명령어 예시
unzip spacepeng.apk
```

---

## 2. 패키지 매니저 및 APK 추출 (Package Manager & Pull)

### pm = 패키지 매니저 (Package Manager)
- `pm help`: 도움말 확인

### ADB 쉘 명령어 흐름
```bash
# 쉘 접속
adb shell

# 기기에 설치된 모든 패키지 리스트 확인
pm list packages

# 패키지 검색 (grep -i: 대소문자 무시하고 검색)
pm list packages | grep -i <app-name>

# APK의 정확한 경로(Path) 확인
pm path <package-name>

# 쉘 종료
exit

# APK 파일을 PC로 가져오기 (Pull)
adb pull <정확한 APK 경로> .
```

---

## 3. Apktool 실행 및 설정 (Run Apktool)

### 설치 및 실행
1. `apktool_2.9.3.jar` 다운로드
2. 실행: `java -jar apktool_2.9.3.jar` (jar는 자바 파일임)

### 단축키(Alias) 설정
```bash
vi ~/.bashrc

# 파일 끝에 아래 내용 추가
# Alias to launch APK-Tool
alias apktool="java -jar /home/wisdom/tools/apktool/apktool_2.9.3.jar"

# 설정 적용
source ~/.bashrc
```

---

## 4. Apktool 명령어 및 옵션 (Commands & Options)

```bash
# Apktool 실행
apktool
```

### 주요 옵션
- **`apktool --advanced`**: 확장 정보 출력 (print Extended information)
- **`-d` (debug)**: AndroidManifest.xml의 `android:debuggable` 속성을 "true"로 설정
- **`-b` (build)**: 디컴파일된 폴더를 다시 APK로 빌드 (Packaging)
- **`--force-manifest`**: 리소스 디코딩이 'false'로 설정되어 있어도 매니페스트 디코딩을 강제로 수행

### 실습 명령어
```bash
# APK 디컴파일 (Decompile)
apktool d base.apk
```

---

## 5. 권한 분석 (Permissions)

- **`<uses-permission>`**: 권한 요구 (Request permission)
  - 예: 진동, 인터넷, 파일 접근 권한 등
