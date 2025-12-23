
## 문자형 숫자를 누적하는 방법
'0', '1', '2', '5'가 주어지면 그대로 누적시키는 방법
```java
int answer = 0;
for (char x : s.toCharArray()) {
	if (x >= 48 && x <= 57) {
		answer = answer * 10 + (x - 48)		
	}
}
System.out.println(answer); // 125
```

