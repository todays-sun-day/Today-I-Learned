화면 회전에 따른 생명 주기
- [x] 앱을 처음 실행했을 때
```
onCreate -> onStart -> onResume
```

- [x] 홈 버튼을 눌렀을 때
```
onPause -> onStop
```

- [x] 다시 앱으로 돌아왔을 때
```
onRestart -> onStart -> onResume
```

- [x] 화면을 회전시켰을 때
```
빈화면이라서 변화 없음
```

- [x] 뒤로가기 버튼을 눌렀을 때
```
onPause -> onStop
```

- [x] 다른 앱이 위에 뜰 때 (전화 수신 등)
```
이미 뒤로 간 후에는 변경 X
```
