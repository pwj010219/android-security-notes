# Week 1-2: Android Hacking 실습 환경 구축

## 1. 기본 용어 및 패키지 설치

### 주요 용어
- **ADB (Android Debug Bridge)**: 안드로이드 기반 기기들의 디버깅에 사용되는 프로그래밍 도구
- **vi upgrade → vim**: 기본 vi 에디터를 더 강력한 vim으로 업그레이드하여 사용

### 필수 패키지 설치
```bash
sudo apt-get update           # 패키지 목록 업데이트
sudo apt-get install vim git  # vim 에디터와 git 설치
```

### Android Studio 실행
```bash
bash studio.sh                # 안드로이드 스튜디오 실행 (bin 폴더 내)
```

---

## 2. ADB (Android Debug Bridge) 설정 및 사용

### 실행 파일 기본 위치
```
/home/사용자명/Android/Sdk/platform-tools/adb
```

### 주요 옵션 (Global Options)
- `-a`: 모든 네트워크 인터페이스에서 수신 (Listen on all network interfaces)
- `-d`: USB 장치 사용 (Use USB device)
- `-e`: 에뮬레이터 사용 (Use emulator)
- `-s`: 시리얼 번호로 특정 장치 지정 사용 (Use device with serial)
- `-H`: ADB 서버 호스트 이름 지정 (Name of adb server host)
- `-P`: 지정된 포트 사용 (Use given port)

### ADB 환경 변수(Path) 설정
매번 전체 경로 입력하지 않기 위함

```bash
# 1. 설정 파일 열기
vi ~/.bashrc

# 2. 파일 맨 아래에 다음 내용 추가 (사용자명 부분 본인 계정에 맞게 수정)
export PATH=$PATH:/home/사용자명/Android/Sdk/platform-tools

# 3. 설정 적용 (재부팅 없이 적용)
source ~/.bashrc

# 4. 확인
adb
studio  # alias 설정 시
```

---

## 3. Android Emulator 및 가상화 설정

### BIOS 가상화(VT-x) 설정 ⭐ 필수 암기
1. PC 재부팅
2. 로고 화면에서 F2 / F10 / Del / ESC (제조사별 상이) 연타하여 BIOS 진입
3. CPU 설정(Configuration) 메뉴로 이동
4. **Virtualization Technology (가상화 기술)** 항목을 **[Enabled]**로 변경
5. F10 눌러서 저장 후 재부팅

### 오류 해결: "Hardware virtualization is not enabled..."
**원인**: BIOS에서 가상화 기술(VT-x)이 꺼져 있거나 윈도우 보안 설정 충돌

**해결 방법**:
- **해결 1**: 위 BIOS 설정 진행
- **해결 2 (Windows 11)**: Windows 보안 > 장치 보안 > 코어 격리 > '메모리 무결성' 끔 > 재부팅

### 가상 디바이스(AVD) 생성 팁
- Virtual Device Manager에서 생성 시 **'x86 Images'**를 선택하면 에뮬레이터 속도가 빠름

### Emulator 단축 실행 설정
```bash
# 1. .bashrc 파일에 에뮬레이터 경로 추가
export PATH=$PATH:/home/사용자명/Android/Sdk/emulator

# 2. 설정 적용
source ~/.bashrc

# 3. 실행 명령어
emulator -list-avds           # 설치된 가상 디바이스 목록 확인
emulator @디바이스명          # 예: emulator @Pixel_6_Pro
```

---

## 4. Android x86 가상머신 구축

### 설치 개요
- **사이트**: https://www.android-x86.org/
- **권장 사양**: Memory 2048MB, Processor 2 Core
- **이미지**: Android-x86-9.0-r2.iso

### 파일 무결성 검사 (Windows cmd)
```cmd
CertUtil -hashfile android-x86-9.0-r2.iso SHA256
```
홈페이지의 서명값과 비교하여 파일 손상 여부 확인

### 문제 해결: CLI 모드에서 멈추거나 화면이 안 나올 때
GRUB 부트로더 설정 수정이 필요함 (nomodeset 설정)

### CLI → UI 모드 변경 및 GRUB 수정 단계
```bash
# 1. 부팅 시 'Debug mode' 선택

# 2. 파일 시스템 마운트 (쓰기 권한)
mount -o remount,rw /mnt

# 3. GRUB 폴더로 이동
cd /mnt/grub

# 4. 설정 파일(menu.lst) 수정
vi menu.lst

# 5. 내용 수정: 첫 번째 title 항목의 kernel 라인 맨 뒤에 다음 옵션 추가
nomodeset xforcevesa

# 6. 재부팅
reboot
```

### IP 주소 확인
```bash
ifconfig
# inet addr 항목 확인
```
