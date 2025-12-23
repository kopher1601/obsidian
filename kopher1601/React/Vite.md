

- `tsconfig.app.json`
	- react 앱에서 앱을 구동하는 코드들을 위한 타입스크립트 설정 파일, 브라우저에서 구동되는 파일 (컴포넌트들)
- `tsconfig.node.json`
	- nodejs 환경에서 실행되고 브라우저에서는 실행되지 않는 파일들을 위한 설정 파일(테스트 파일등)
- `tsconfig.json`
	- 위 두 파일을 Composite 하는 파일
```json
// tsconfig.json
{  
  "files": [],  
  "references": [  
    { "path": "./tsconfig.app.json" },  
    { "path": "./tsconfig.node.json" }  
  ]  
}
```

