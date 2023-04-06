---
title: Firebase를 이용한 구글 Sign In
categories:
- flutter
tags:
- ["firebase","codelab"]
---

## 기본 준비 
- [firebase console](https://console.firebase.google.com) 에서 Authentication 시작하기 
- Sign-in method 에서 구글 선택 후 시키는대로 입력 
- SHA1 등록하고 google-services.json 업데이트 
  - 프로젝트 개용에서 안드로이드 앱선택하고 설정 화면 진입 
  - 디지털 지문 추가 
  - debug key 지문 확인 방법
```bash
keytool -list -v -keystore ~/.android/debug.keystore -alias androiddebugkey -storepass android -keypass android
```

## firebase 초기화 
- `flutter pub add firebase_core` 로 firebase 플러그인 추가 
- `flutter pub add google_sign_in` 구글인증 플러그인을 추가합니다. 
- FirebaseApp 초기화
```dart
void main() async {  // <-- 2
    WidgetsFlutterBinding.ensureInitialized(); // <-- 3
    await Firebase.initializeApp(); // <-- 1
    runApp(const App());
}
```
  1. Firebase.initializeApp 이 async 함수 이고, 초기화를 보장하려면 await 을 해줘야 함
  2. 1.에서 await 을 하려면 main() 은 async 함수여야 함
  3. WidgetsFlutterBinding.ensureInitialized 함수는  Flutter 엔진의 초기화를 보장하는 함수로 Firebase.initializeApp 이전에 호출해야 함



## firebase 로 sign in 하기 
```dart
class App extends StatelessWidget {
  const App({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData(primarySwatch: Colors.pink),
      home: const MainPage(),
    );
  }
}

class MainPage extends StatelessWidget {
  const MainPage({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            children: [
              ElevatedButton(onPressed: () {}, child: const Text("구글 로그인"))
            ],
          ),
        ),
      ),
    );
  }
}
```


### sign in / sign out 함수 구성 
```dart

class MainPage extends StatefulWidget {
  const MainPage({Key? key}) : super(key: key);

  @override
  State<MainPage> createState() => _MainPageState();
}

class _MainPageState extends State<MainPage> {
  final signIn = GoogleSignIn();
  GoogleSignInAccount? account;

  @override
  void initState() {
    _checkSignIn();
    super.initState();
  }

  Future<void> _checkSignIn() async {
    if (await signIn.isSignedIn()) {
      var result = await signIn.signInSilently();
      setState(() {
        account = result;
      });
    }
  }

  Future<void> _signIn() async {
    var result = await signIn.signIn();
    setState(() {
      account = result;
    });
  }

  Future<void> _signOut() async {
    var result = await signIn.signOut();
    setState(() {
      account = result;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: Column(
            children: [
              ElevatedButton(onPressed: () {}, child: const Text("구글 로그인"))
            ],
          ),
        ),
      ),
    );
  }
}
```
- statefulWidget 으로 변경 후 
- googleSignIn 에 관련된 함수 추가 


### 구글 계정 정보를 보여주는 Widget 구성
```dart
class AccountWidget extends StatelessWidget {
  const AccountWidget({Key? key, required this.account}) : super(key: key);
  final GoogleSignInAccount account;

  @override
  Widget build(BuildContext context) {
    return Column(
      mainAxisSize: MainAxisSize.min,
      children: [
        Image.network(account.photoUrl ?? ""),
        const SizedBox(height: 8),
        Text(account.displayName ?? "NONAME"),
      ],
    );
  }
}

```

### sign in UI 구성
```dart
class _MainPageState extends State<MainPage> {
  
  ....

  @override
  Widget build(BuildContext context) {
    Widget? bodyWidget;
    if (account == null) {
      bodyWidget = ElevatedButton(
        onPressed: () => _signIn(),
        child: const Text("구글 로그인"),
      );
    } else {
      bodyWidget = Column(
        mainAxisSize: MainAxisSize.min,
        children: [
          AccountWidget(account: account!),
          ElevatedButton(
            onPressed: () => _signOut(),
            child: const Text("구글 로그아웃"),
          )
        ],
      );
    }
    return Scaffold(
      body: Center(
        child: Padding(
          padding: const EdgeInsets.all(24.0),
          child: bodyWidget,
        ),
      ),
    );
  }
}

```