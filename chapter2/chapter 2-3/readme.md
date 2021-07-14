## 객체 초기화

객체 인스턴스를 메모리에 할당한 직후, 객체 내부 변수를 초기 값으로 지정하기 위해 init 메서드를 사용한다.  
NSObject에 구현된 **\-init 기본 초기화 메서드는 상속받아 만들어진 모든 객체에서 기본 초기화 메서드**로 사용한다.

특정 객체가 포함하는 하위 객체는 상위 객체 인스턴스가 만들어지면서 동시에 만들어진다.  
이런 소유권을 갖는 하위 객체는 대부분 초기화 메서드에서 만들어진다.  
객체 내부 자원에 해당하는 인스턴스 변수를 준비한다는 측면에서 초기화 메서드는 중요하다.  
If 상속시) Objective-C 에서 -init 초기화 메서드를 명시적으로 구현하지 않으면, 상속받은 상위 객체에 구현된 초기화 메서드를 호출한다.

### 여러 초기화 메서드

NSObject 클래스에 선언되어 있는 기본 초기화 메서드는 -(id)init 형태이다.  
상속받아 만드는 클래스는 init 메서드를 다음과 같은 형태로 오버라이드해서 구현한다.

```
-(id)init{
    self = [super init];
    if(self != nil){
        //인스턴스 변수 초기화
    }
    return self;
}
```

기본 동작이 상속받은 부모 객체의 인스턴스 내부 변수와 리소스를 초기화하고, 자신의 내부 리소스를 초기화하는 순서를 권장하기 때문이다.  
초기화 메서드가 기본값이 아닌 외부 데이터에 의존해서 초기값을 설저앻야 한다면, 추가적으로 초기화 메서드를 추가해야 한다.  
메서드 명은 -init으로 시작하면 된다.

> 인스턴스타입(instanceType)  
> id 타입은 Objective-C에서 모든 객체를 표현할 수 있는 **다이내믹 타입**이다 -> AnyObject와 비슷 
> id 타입으로 리턴받은 객체는 타입 정보가 부족해서 특정 메세지를 보낼 수 있는지 없는지 컴파일러가 판단하기 어려움  
> 실제로 메세지를 보내면 리턴받은 객체가 해당 메세지를 받아서 처리할 수 없어 앱이 죽기도 한다  
> 최신 런타임 구조에서는 이런 생성/초기화 관련 메서드 리턴 타입을 id -> instanceType으로 변경했다.  


#### Example
```
@interface AObject: NSObject
+ (instancetype)factoryMethodA;
+ (id)factoryMethodB;
@end

@implementation AObject
+ (instancetype)factoryMethodA {return [[[self class] alloc] init];}
+ (id)factoryMethodB {return [[[self class] alloc] init];}
@end

void aa(){
	NSUInteger x, y;
    
    //AObject instancetype
    x = [[AObject factoryMethodA] count]; // Warning -> AObject may not respond to 'count'
    // id
	y = [[AObject factoryMethodB] count]; // Not Warning
}

```




### 초기화 메서드 구현하기

코코아 프레임워크에서 -init 계열 메서드를 구현하기 위해 **권장하는 가이드**이다.

1.  상속받은 Super Class의 초기화 메서드를 먼저 호출한다.
2.  Super Class 초기화 메서드 리턴 값을 확인해서 nil이면, 내부 리소스 초기화를 하지않고 그대로 nil을 리턴한다.
3.  내부 리소스를 초기화하면서 객체느 copy나 retain 메서드를 호출해서 소유권을 갖는다.
4.  인스턴스 변수들을 적정한 값으로 초기화하고 나면 self를 리턴한다.
5.  인스턴스 변수들 초기화 과정에서 오류가 발생한 경우에는 self를 해제하고 nil 리턴
6.  self가 아닌 객체 인스턴스를 리턴하는 경우라 하더라고 self를 해제해야 한다.

<img src="https://user-images.githubusercontent.com/11826495/125586322-2f72d20e-dffa-4246-b178-786b962bf233.jpeg" width="50%" height="400">


#### Failable init in Swift

```
class A{
	var a: Int
    //Swift의 실패가능한 init은 실패 시에는 nil을 반환하지만, 성공시에는 값을 반환하지 않는다.
    //사용하는 이유 : 실수로 만들어지는 객체들로 인한 메모리 낭비를 방지, 예상하지 못한 상황 방지
    init?(value: Int){
    	if value < 0{
        	return nil
        }
        self.a = value
    }
}
```

### 객체 초기화 관련한 문제

한번 -init 계열 메서드로 객체 인스턴스를 초기화한 후, 또다시 -init 계열 메서드를 호출하면 안된다. -> exception 발생

초기화 하는 객체가 +alloc 메서드를 통해 정삭적으로 메모리에 생선한 객체가 아닌 경우도 조심해야 한다.  
**해당 객체 인스턴스가 하나만 존재해야 하는 싱글톤 인스턴스**인 경우도 있고, **내부 인스턴스 변수 객체 중 싱글톤** 형태로 존재하는 경우도 있다.

> 싱글톤 프로퍼티를 갖고 있거나, 클래스 자체가 싱글톤으로 사용된다면 주의

마지막으로 초기화 메서드가 실패한 경우를 대비해야만 한다.  
\-initWithString: 메서드에 인자 값 문자열이 nil인 경우가 있을 수 있고,  
\-initWithArray: 메서드에 인자값이 NSArray 대신 NSDictionary일 수도 있다.  
이렇게 인자 값이 nil이거나 원하지 않는 객체가 들어오면, 새로 할당한 객체는 반환하고 nil을 리턴해야 한다.
