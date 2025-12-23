[[@ExtendWith]] 와 같이 사용해야 한다
```kotlin
@ExtendWith(SpringExtension::class)  
@ContextConfiguration(classes = [TestObjectFactory::class])
```
