# Week 10: Content Providers & SQL Database Attack

## PART 1: 핵심 이론 - Service & Content Provider

### 1. Service (서비스)

#### 정의
- 화면(UI) 없이 백그라운드에서 오래 실행되는 작업
- 예: 음악 재생, 파일 다운로드

#### 종류 3가지
1. **Foreground**: 사용자에게 "나 실행 중이야"라고 알림(Notification)을 꼭 띄워야 함
2. **Background**: 사용자 모르게 뒤에서 조용히 작업함
3. **Bound**: 다른 앱과 연결(Bind)되어 서버처럼 동작함

---

### 2. Content Provider (CP) - "데이터 창고지기"

#### 접근방법: URI

#### 정의
- 보안 때문에 막혀있는 앱 간의 데이터를 공유하기 위한 통로

#### URI 구조
```
content://[Authority]/[Path]/[ID]
```

- **Authority**: 앱의 고유 패키지 이름 (누구네 집인지)
- **Path**: 테이블 이름 (어느 방인지)
- **ID**: 특정 데이터 번호 (몇 번째 서랍인지)

---

### 3. 권한(Permission) 완벽 분석 ⭐ 시험 포인트

시스템 권한과 커스텀 권한의 차이 이해하기

#### (1) 안드로이드 기본 시스템 권한
```xml
<uses-permission android:name="android.permission.INTERNET" />
```
- **설명**: 인터넷, 저장소 읽기/쓰기 등 안드로이드가 원래 만들어둔 권한들

#### (2) 개발자가 만든 커스텀 권한 (Sieve 앱 전용)
```xml
<permission android:name="com.mwr...READ_KEYS" ... />
```
- **설명**: 'Keys'라는 비밀 테이블을 보호하기 위해 개발자가 직접 만든 자물쇠
- **상황**: 우리는 해커(ADB Shell)라서 이 '열쇠(uses-permission)'가 없음
- **결과**: 정상적으로 접근하면 "Permission Denial(권한 거부)" 에러 뜸

#### (3) Path Permission (구멍 난 보안)
```xml
<path-permission android:path="/Keys" ... />
```
- **의미**: "전체 집은 개방하지만(/), '/Keys'라는 안방에 들어올 때만 권한을 검사하겠다."
- **취약점**: "/Keys"라는 글자만 검사하므로, "/Keys///" 처럼 주소를 살짝 바꾸면 검사기가 속아서 문을 열어줌

---

## PART 2: 분석 도구 준비 & 소스코드 해부

### 1. JADX 설치 및 실행 (소스코드가 없을 때 사용)

```bash
# 터미널 명령어 순서 (복사해서 쓰세요)
wget https://github.com/skylot/jadx/releases/download/v1.5.0/jadx-1.5.0.zip
unzip jadx-1.5.0.zip -d jadx
./jadx/bin/jadx-gui
```

**역할**: APK 파일을 뜯어서 Java 소스코드로 보여줌 (Decompiler)

---

### 2. 실제 DB 파일 위치 찾는 공식

**질문**: "이 앱의 데이터베이스 파일은 폰 내부 어디에 저장되는가?"

**찾는 법**:
1. JADX에서 DBHelper 파일을 찾아 `DATABASE_NAME` 변수 확인 ("database.db")
2. 안드로이드 표준 경로 규칙에 대입

**정답 경로**:
```
/data/data/com.mwr.example.sieve/databases/database.db
```

---

### 3. 소스코드 분석 결과 (Sieve 앱)

#### (1) UriMatcher의 역할 (교통정리)
- 요청 주소(URI)를 보고 번호표(ID)를 발급함
- **100번표** → Passwords 테이블 (공개, 보안 X)
- **200번표** → Key 테이블 (비밀, 보안 O)

#### (2) 실제 테이블 구조 (PWTable.java)
- **[비밀] 테이블명**: "Key" (컬럼: Password, pin)
- **[공개] 테이블명**: "Passwords" (컬럼: service, username, email...)

---

## PART 3: 해킹 공격 3대장 ⭐ 시험 100% 출제

### [공격 0] 취약점 확인 (간보기)

**방법**: 쿼리문에 홑따옴표(')나 쌍따옴표(")를 일부러 넣어봄

**에러 메시지**:
1. "syntax error" (문법 에러)
2. "unrecognized token" (알 수 없는 기호)

**의미**: "아싸! 내가 입력한 특수문자가 쿼리문에 영향을 주는구나! (SQL Injection 가능)"

---

### [공격 1] SQL Injection (주석 -- 사용)

**목표**: 권한이 있는 방(Passwords)에 들어가서, 몰래 안방(Key) 물건 가져오기

**원리**: SQL 주석('--')을 사용하여 뒤따라오는 원래 쿼리(FROM Passwords)를 지워버림

**명령어**:
```bash
adb shell content query --uri content://com.mwr.example.sieve.DBContentProvider/Passwords --projection "* FROM Key--"
```

**해석**: 시스템은 "SELECT * FROM Key" 까지만 실행하고 뒤는 무시함

---

### [공격 2] Path Traversal (슬래시 /// 우회)

**목표**: 권한 검사기를 바보로 만들어서 Key 테이블 접근하기

**원리**: 안드로이드 권한 검사는 문자열("/Keys")만 확인하므로, "/Keys///"는 다른 경로로 인식하여 통과됨

**명령어**:
```bash
adb shell content query --uri content://com.mwr.example.sieve.DBContentProvider/Keys///
```

#### ⭐ 방어 기법 - Secure Coding (중요)

**해결책**: 개발자는 `getCanonicalPath()` 함수를 사용하여 경로를 정규화해야 함

**원리**: "../"나 "///" 같은 속임수를 모두 제거하고 '진짜 절대 경로'를 구한 뒤에 권한을 검사함

> 💡 참고: 이 `getCanonicalPath()` 방어 기법은 11주차 파일 해킹 실습에서도 핵심 내용으로 다시 나옵니다.

---

### [공격 3] Insert Injection (로그인 우회)

**목표**: 비밀 테이블(Key)에 내 마음대로 비밀번호(12345)를 심어서 로그인 뚫기

**명령어**:
```bash
adb shell content insert --uri content://com.mwr.example.sieve.DBContentProvider/Keys/// --bind Password:s:12345 --bind pin:i:1234
```

---

## PART 4: 실습 과제용 필수 명령어 (CRUD Cheat Sheet)

### 기본 URI
```
content://com.mwr.example.sieve.DBContentProvider/Passwords
```

### 1. 조회 (Query)
```bash
adb shell content query --uri [URI]
```

### 2. 추가 (Insert)
**문법**: `--bind [컬럼명]:[타입s/i]:[값]` (s=문자열, i=정수)

```bash
adb shell content insert --uri [URI] --bind service:s:InsertEx --bind username:s:MyName --bind email:s:test@email.com
```

### 3. 수정 (Update)
**문법**: `--where "[컬럼]='[값]'"`

```bash
adb shell content update --uri [URI] --bind service:s:UpdateEx --where "service='InsertEx'"
```

### 4. 삭제 (Delete)
```bash
adb shell content delete --uri [URI] --where "service='UpdateEx'"
```

---

## PART 5: 암기 꿀팁 - Java Code vs SQL 순서

**질문**: "Java 코드의 query() 파라미터가 SQL로 바뀔 때 순서가 헷갈려요."

**해결**: "크로스(X) 법칙" 기억하기

```
[Java 코드]    query( Uri,  Projection,  Selection )
                      X (교차!)           | (일치)
[SQL 문장]    SELECT Projection  FROM Uri  WHERE Selection
```

💡 **팁**: "앞에 두 놈(주소랑 메뉴)만 자리를 바꾼다!"

---

## PART 6: 교수님 강조 질문 (서술형 완벽 대비)

**핵심 질문**: "DB가 어떻게 생겼는가? 무엇을 보고 찾아갔는가? 왜 막혔고 어떻게 뚫었는가?"

### 1. DB가 어떻게 생겼는가? (Structure & Appearance)

**파일명**: "database.db" (PWDBHelper.java에서 확인)

**내부 구조 (2개의 테이블로 구성됨)**:

#### (1) 비밀 테이블 (보안 O)
- **테이블 이름**: "Key" (대문자 K)
- **컬럼(속성)**: "Password", "pin"
- **특징**: 우리가 탈취하려는 중요 정보가 들어있음

#### (2) 일반 테이블 (보안 X)
- **테이블 이름**: "Passwords"
- **컬럼(속성)**: "_id", "service", "username", "email", "password"(BLOB)
- **특징**: 여기 있는 password 컬럼은 BLOB(이진 데이터)이라서 읽을 수가 없음

---

### 2. 이 구성을 무엇을 보고 알아냈는가? (Discovery Process)

**도구**: APK 역분석 도구인 **'JADX'** 사용

**추적 경로**:
1. **DBContentProvider.java**: URI 주소와 테이블 ID(100, 200) 연결 확인
2. **PWDBHelper.java**: DB 파일명("database.db") 확인
3. **PWTable.java** (결정적 단서): 이 파일 안의 `CREATE TABLE...` SQL문을 보고 위 1번의 테이블 이름과 컬럼명을 정확히 알아냄

---

### 3. 왜 정보를 바로 보지 못했는가? (Obstacle)

**현상**: ADB Shell에서 `.../Keys` URI로 조회 시 **"Permission Denial"** 에러 발생

**원인**: 
- **AndroidManifest.xml** 파일 분석 결과, `<path-permission>` 태그가 "/Keys" 경로를 막고 있었음
- 접근하려면 **"READ_KEYS"** 권한이 필요한데, 해커(ADB)에겐 그 권한이 없기 때문

---

### 4. 어떻게 우회(Bypass) 했는가? (Solution)

#### (1) SQL Injection (주석 -- 사용)
- **원리**: 검사가 없는 `/Passwords` 경로로 들어가서 SQL 문법 조작
- **방법**: Projection에 **"* FROM Key--"** 삽입
- **결과**: '--' 뒤의 원래 쿼리(FROM Passwords)가 무시되고, Key 테이블 내용을 가져옴

#### (2) Path Traversal (슬래시 /// 사용)
- **원리**: 권한 검사기가 문자열("/Keys")만 단순 비교한다는 점 악용
- **방법**: URI 뒤에 **`/Keys///`** 입력
- **결과**: 권한 검사기는 다른 경로로 착각해 문을 열어주고, DB는 정상 경로("Key")로 인식해 데이터를 줌
