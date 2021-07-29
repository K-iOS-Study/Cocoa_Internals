**5.2 가변 객체**

**5.2.1 가변 객체의 특징**

`가변 객체의 특징`은 불변 객체와 정확히 반대 요소를 갖고 있다.

- 초기화 이후에도 객체 내부 값이나 상태를 추가, 삭제, 변경할 수 있다.

- 여러 객체나 여러 스레드에서 참조하기 위해서는 동시 접근에 대한 예외 처리가 필요하다.

- 성능 특성을 고려해야 한다. 불변 객체보다 설계가 복잡하고 구현하기 어렵다.

- 어느 시점이든 값이 변경되서 부작용(side effects)이 생길 수 있다.

<br>

**5.2.2 가변 객체 클래스**

코코아 프레임워크에서 자주 사용하는 가변 객체 클래스는 다음과 같다.

- 다른 객체를 참조하는 클래스: NSMutableArray, NSMutableDictionary, NSMutableSet

- 특정 타입을 집합으로 다루는 클래스: NSMutableIndexSet, NSMutableCharacterSet

- 문자열을 다루는 클래스: NSMutableString, NSMutableAttributedString

- 특정 데이터 구조를 클래스: NSMutableData, NSMutableURLRequest

 공통적으로 접두어 `Mutable` 를 사용해서 가변 객체임을 나타내고 있다. 이런 class들은 기존의 불변 객체를 상속해서 추가로 변경을 위한 메서드를 제공한다.

<br>

> NSMutableArray는 NSArray를 상속받아 insertObject나 removeObject를 제공한다

<br>

**5.2.3 가변 객체 참조 사례1 : 가변 모델 객체와 뷰 객체**

가변 객체를 참조하는 경우는 참조하는 객체 내용이 변경되면 추가 작업이 필요하다.

```objectivec
@property (nonatomic) NSArray* tableItems;

- (void)viewDidLoad {
    [super viewDidLoad];
    PenHolder* penHolder = [[PenHolder alloc] init];
    // PenHolder.pens 에는 20개의 Pen이 포함되어 있다고 가정 //1
    self.tableItems = penHolder.pens; //2
}

#pragma mark - TableView DataSource
-(NSInteger)tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)
section {
return self.tableItems.count; //3
}
```

1. Pen 객체가 가변 배열(NSMutableArray)에 들어있다.

2. 내부 인스턴스 변수 tableItems는 penHolder.pens를 참조하고있다.

3. tableItems.count 로 cell 갯수를 계산한다.

 만약 penHolder.pens 갯수가 달라지면 어떻게될까, tableItems.count도 같이 바뀌지만 테이블뷰는 바뀐 사실을 모른다.

KVO를 하거나 NSNotificationCenter 같은 옵저버 패턴으로 변경을 감지해서 테이블뷰를 다시 그려야 한다.

`결국 가변 데이터의 흐름을 따라서 컨트롤러와 뷰까지 영향을 주는 코드가 이어지게 된다.`

PenHolder.pens를 불변 객체로 만들더라도 기술적으로 완벽한 불변 객체는 아니다. KVC를 이용해서 우회적으로 프로퍼티를 변경할 수 있기 때문이다. 그래서 읽기전용 프로퍼티를 노출하기보다는 아예 감추는것이 낫다.

**객체를 감추는 방법은 구현부에서 클래스 확장 카테고리로 확장하거나, 내부를 변경하는 인터페이스를 제공하고 인터페이스에서 데이터 흐름에 따라 다른 코드로 이어지도록 만들기를 권장한다.**

<br>

**5.2.4 가변 객체 참조 사례2 : NSMutableSet와 가변 객체**

```objectivec
NSMutableSet* variableSet = [NSMutableSet set];
[variableSet addObject:@"unique-key"]; //1
NSLog(@"variableSet = %@", variableSet);
// 결과: variableSet = {("unique-key")}

[variableSet addObject:@"unique-key"]; //2
NSLog(@"variableSet = %@", variableSet);
// 결과: variableSet = {("unique-key")}

NSMutableString* varibleString = [NSMutableString stringWithFormat:@"unique"];
[variableSet addObject:varibleString]; //3
NSLog(@"variableSet = %@", variableSet);
// 결과: variableSet = {("unique", "unique-key")}

[varibleString appendString:@"-key"]; //4
NSLog(@"variableSet = %@", variableSet);
// 결과: variableSet = {("unique-key","unique-key")}

NSSet *copySet = [variableSet copy]; //5
NSLog(@"copySet = %@", copySet);
// 결과: variableSet = {("unique-key")}
```

1. variableSet 가변 집합을 만들어서 "unique-key" 문자열을 추가한다.

2. 동일 객체는 Set에 추가되지 않는다.

3. 다른 객체는 Set에 추가된다.

4. 3에서 추가한 가변 문자열 객체에 "-key"를 덧붙이면 실제로는 동일한 내용 의 문자열이 집합 내부에 존재하게 된다.

5. 이런 상태에서 Set 인스턴스를 복사하면 집합 내부를 다시 비교하기때문에 내부의 중복 객체가 사라진다.

>  4번 상황까지는 일시적으로 정상이라면, 복사한 5번 객체가 부작용의 결과일 수 있다. 혹은 반대로 5번처럼 늘 중복되지 않아야 하는 상황에서는 4번 상황이 의도하지 않은 버그일 수도 있다. 이처럼 가변 객체를 참조하는 경우에는 의도하지 않은 예외 상황에 대해 대비해야만 한다.

<br>

> 객체 중복성 검사
> 
> NSSet이나 NSDictionary처럼 키 값을 사용하는 컬렉션은 내부적으로 객체 중복성을 검사할 때, 객체의 –hash와 –isEqual: 메서드가 중요한 역할을 담당한다. 반면에 정렬한 배열처럼 순서가 중요한 컬렉션은, 순서를 정하기 위한 비교 메서드가 중요한 역할을 담당한다.

<br>

**5.2.5 요약**

 가변 객체를 사용하는 경우에는 가변 객체의 내부 값이 바뀌기 때문에 생기는 부작용에 대비해야 한다. 그러기 위해서 가변 객체를 변경하는 메서드가 배타적으로 동작해야 한다.

 데이터 내용이 바뀌는 시점에 따라 데이터 흐름을 처리하는 코드가 있다면, 변화를 감지하기 위한 디자인 패턴을 적용하는 게 좋다. 가변 객체를 참조하는 경우는 의도하지 않은 변화에 대비해서 동작 방식을 정확하게 이해하고 있어야 한다.


