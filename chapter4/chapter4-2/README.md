
# 7/21 얕은 복사 vs 깊은 복사

NSArray처럼 내부에 다른 객체를 포함하는 경우에는 객체를 복사할 때 주의해야할 사항!

(객체가 객체를 포함하고 있다는 것도 메모리 참조 관점에서 보면 참조하는 대상 객체의 메모리 주소를 포인터 변수로 접근하는 것)

```swift
@interface PenHolder : NSObject <NSCopying> {
  NSMutableArray *_pens;
}
-(void) addPen:(Pen*)pen;
-(void) removePen:(Pen*)pen;

@end

@implementation PenHolder

- (id)copyWithZone:(NSZone *)zone {
  PenHolder *copiedHolder = [[[self class] alloc] init];
  copiedHolder->_pens = [_pens mutableCopy];
  return copiedHolder;
}

@end
```

`PenHolder` 객체를 복사하는 `-copyWithZone:` 메서드는 새로운 `PenHolder` 객체 인스턴스 `copiedHolder`를 만들고, 자신의 _pens 배열을 "복사"해서 `copiedHolder` 객체 _pens 변수에 설정한다

### **문제점**

![./CopyImage.png](/CopyImage.png)

`copiedHolder->_pens = [_pens mutableCopy];`  이 라인을 통해 `NSMutableArray`가 복사된다.

하지만 Foundation Framework에 있는 모든 클래스는 얕은 복사(shallow copy) 형태로 구현되어 있어 `NSMutableArray` 객체는 복사되지만 배열의 요소는 복사되지 않아 같은 요소의 주소를 가리키게 된다.

이 상태에서 `copiedHolder` 인스턴스 내 `_pens` 집합에 있는 `Pen` 객체 내부의 값을 변경하면, 기존 `holder` 인스턴스 내 `_pens` 집합에 있는 동일한 `Pen` 객체 내용도 바뀌게 된다.

엄밀히 말하여 이것은 복사가 이루어진 것이 아니다!

### 해결책

**Deep Copy**

```swift
NSArray *_newArray = [[NSArray alloc] initWithArray:_oldArray copyItems:true];
```

NSArray 계열 컬렉션 클래스의 경우 -initWithArray: copyItems: 초기화 메서드를 이용해서 깊은 복사를 할 수 있다. 

먄약? 복사되는 아이템들도 내부에 다른 객체를 참조하고 있다면?
→ 트리형태로 DFS를 사용해서 Deep-Copy를 구현해야한다. 하지만, 순환 참조 문제가 있을 수 있기 때문에 약한 참조를 지정해야하는데 깊은 복사가 어떤 형태의 객체 그래프를 그리는지 확인 한 뒤에 참조 관계에 따라 복사 방식, 참조 방식을 결정해야한다.

### 요약

Cocoa Framework의 클래스는 객체를 복사할 경우 얕은 복사 형태로 참조 관계를 유지한다. 특히 NSArray, NSSet 같은 컬렉션 객체나 다른 객체를 참조하는 객체를 복사해야하는 경우 깊은 복사 방식을 고민해야한다.

```swift
UIView* newView = [NSKeyedUnarchiver unarchiveObjectWithData:[NSKeyedArchiver archivedDataWithRootObject:oldView]];
```
