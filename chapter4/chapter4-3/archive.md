## 4.3 아카이브

- 코코아 프레임워크에서 깊은 복사를 구현하기 위해 코어데이터를 활용하는 방식과 아카이브를 활용하는 2가지 방식이 있음

### 객체 직렬화와 아카이브의 차이

- 직렬화는 사전, 배열, 문자열 및 이진 데이터와 같은 값 개체의 간단한 계층 구조를 저장
- 직렬화는 계층 구조에서 개체의 값과 위치만 보존하기 때문에 동일한 값 개체에 대한 여러 참조는 역직렬화될 때 여러 개체가 될 수 있음
- NSPropertyListSerialization 클래시는 직렬화와 관련있는 클래스로, NSDictionary, NSArray, NSString, NSDate, NSData, NSNumber 타입으로 저장되어 있는 데이터 구조를 plist파일로 직렬화해서 저장하게 해준다.
- NSUserDefaults내부적으로 NSPropertyListSerialization 사용
- 객체 관계가 복잡하거나 일정 규 모 이상으로 커지면 plist 방식을 사용하기에 부적합
- plist는 클 래스 타입을 모두 지원하지 않으며, 가변 객체나 다중 참조 관계도 원래대로 복 원하지 못함

### NSCoding 프로토콜

```objc
@protocol NSCoding
- (void)encodeWithCoder:(NSCoder *)coder;
- (nullable instancetype)initWithCoder:(NSCoder *)coder; // NS_DESIGNATED_INITIALIZER
@end
```

NSCoder 클래스는 메모리상에 있는 객체 인스턴스 변수 를 다른 형태로 변환하기 위한 인터페이스를 선언한 추상화 클래스로 실제로는 NSKeyedArchiver, NSKeyedUnarchiver, NSPortCoder 같은 하위 클래스 구현체를 사용한다.

#### [codable vs NSCoding](https://codesquad-yoda.medium.com/codable-vs-nscoding-차이점-4b47e240c0b8)

- 객체를 인코딩, 디코딩하기위한 프로토콜
- codable은 json코더에서 사용가능
- 상속관계와 다형성 관계에서 codable로 표현 불가능한 부분이 NSCoding으로 표현가능 

#### 객체 그래프와 아카이브

![image](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Archiving/Art/cocoaobjects_2x.png)

- NSCoder에서는 뿌리객체와 조건부 객체라는 두가지 개념 사용
- 뿌리 객체는 객체 그래프에 대한 탐색의 시작점으로 무조건 루트일 필요는 없고, 아카이브 시작하는 시점일 뿐이다.
- 뿌리 객체부터 탐색을 시작해서 기존에 인코딩했던 객체를 다시 참조하는 경우에는 새로운 객체로 인코딩하지 않고, 기존 객체를 참조한다. (encodeRootObject: 사용)
- 조건부 객체는 객체 그래프에서 반드시 아카이브하지 않아도 되는 참조객체를 의미한다. 어느 시점에 반드시 인코딩되는 객체를 참조하기 때문에 다시 인코딩이 필요없다. (encodeConditionalObject:forkey: 사용)
-

#### NSSecureCoding

- NSCoding 프로토콜에 보안성을 강화한 확장해 supportsSecureCoding 프로퍼티 구현 해야함
- decode ObjectForKey: 메서드 대신 –decodeObjectOfClass:forKey: 메서드를 사용해서 아카이브한 파일 내부 클래스를 대체하는 공격 방식에 대비할 수 있다. 
- 아카이 브한 바이너리 데이터를 네트워크상으로 주고 받는 경우에 해킹 위험이 있어서 보안성을 강화한 버전이다.
- 특히 XPC(eXtra inter-Process Communication) 방 식으로 객체를 아카이브해서 주고 받을 때는 NSSecureCoding 프로토콜을 반 드시 구현해야만 함

### 이름 있는 아카이브

- NSKeyedArchiver와 NSKyedUnarchiver를 이용해 NSCoding을 준수하는 객체를 인코딩, 디코디딩
- delegate도 있어 각 객체를 인코딩, 디코딩 하는 시점에 알림 받을 수 있음
- NSObject와 NSCoding 혹은 NSSecureCoding 필수

####  pen 객체

- 뿌리 객체를 사용한 아카이브시 iOS12부터 NSSecureCoding 필수 
- 직접 아카이브하면 NSCoding으로 가능

``` objc

@interface Pen: NSObject <NSSecureCoding>

@property (nonatomic, strong, readonly) NSString *name;

- (instancetype)initWithName:(NSString *)aName;

@end

@implementation Pen

+ (BOOL)supportsSecureCoding {
    return YES;
}

- (instancetype)initWithName:(NSString *)aName {
    self = [super init];
    
    if (self) {
        _name = aName;
    }
    
    return self;
}

- (void)encodeWithCoder:(NSCoder *)coder {
    [coder encodeObject:_name forKey:@"name"];
}

- (nullable instancetype)initWithCoder:(NSCoder *)coder {
    NSString *sName = (NSString *)[coder decodeObjectForKey:@"name"];
    return [self initWithName:sName];
}

@end
```

#### 뿌리 객체를 사용한 아카이브

``` objc
    NSError *archiveError = nil;
    NSError *unarchiveError = nil;

    Pen *pen1 = [[Pen alloc] initWithName:@"pen1"];

    NSData *archivedData = [NSKeyedArchiver archivedDataWithRootObject:pen1 requiringSecureCoding:false error:&archiveError];

    Pen *pen2 = (Pen *)[NSKeyedUnarchiver unarchivedObjectOfClass:[Pen class] fromData:archivedData error:&unarchiveError];

    NSLog(@"%@", pen2.name);
```

#### 직접 아카이브

``` objc
    Pen *pen1 = [[Pen alloc] initWithName:@"name1"];
    NSKeyedArchiver *archiver = [[NSKeyedArchiver alloc] initRequiringSecureCoding:YES];
    [archiver encodeObject:pen1 forKey:@"key"];
    NSData *data = [archiver encodedData];
    [archiver finishEncoding];
    
    NSError *unarchiveError = nil;
    NSKeyedUnarchiver *unarchiver = [[NSKeyedUnarchiver alloc] initForReadingFromData:data error:&unarchiveError];
    unarchiver.requiresSecureCoding = NO;
    Pen *pen2 = [unarchiver decodeObjectForKey:@"key"];
    NSLog(@"%@", pen2.name);
```
