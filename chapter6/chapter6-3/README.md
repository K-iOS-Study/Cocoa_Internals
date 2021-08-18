# 6.3 순서가 없는 집합 컬렉션 8/4

집합(set) 컬렉션은 참조하는 객체들을 순서없이 담고 있는 컬렉션이다. 집합은 특정 객체가 내부에 담고 있는 객체 중에 있는 지를 확인해서 동일한 객체를 **중복 참조하지 않도록 한다.**

대부분 컬렉션 내부 참조 객체의 존재 여부를 확인하는 경우는 배열형보다 집합형 Set을 사용하는 것이 성능에 더 좋다.

![Untitled.png](./assets/Untitled1.png)

집합 컬렉션의 종류

1. NSSet - 한번 만들고 나면 변경할 수 없는 불변 집합
2. NSMutableSet - 추가나 삭제가 가능한 가변 집합
3. NSCountedSet - 집합에 동일하게 객체가 중복해서 들어갈 수 있는 집합

1. NSSet

NSSet은 불변 집합 객체로 초기화 이후 참조할 객체를 추가 및 삭제가 불가능
하지만, 참조하는 객체가 가변 객체라면 그 객체는 수정이 가능함

The constant declaration creates a **read-only reference** to a value

```swift
NSSet *set = [NSSet setWithObjects:@"Hello",@"World",nil];
NSLog(@"%@",set);

// (Hello, World)
```

```swift
NSMutableString *str1 = [[NSMutableString alloc] initWithCapacity:20];  
NSMutableString *str2 = [[NSMutableString alloc] initWithCapacity:20];
[str1 setString:@"Hello World1"];
[str2 setString:@"Hello World2"];
NSSet *set = [NSSet setWithObjects:str1, str2, nil];
[str1 appendString: @" Wrong String"];
NSLog(@"%@",set);

// ("Hello World1 Wrong String", "Hello World2")
```

1. NSMutableSet

가변 집합 객체로 내부에 있는 객체 중에 -hash와 -isEqual: 메소드로 비교해서 동일한 객체가 아니면 객체를 추가하거나 삭제가 가능

```swift
NSMutableSet *set = [[NSMutableSet alloc] init];
[set addObject: @"Hello World"];
[set addObject: @"Hello World"];
[set addObject: @"Hello World"];

NSLog(@"%@",set);
// ("Hello World")
```

```swift
@implementation NSArray (Approximate)
- (BOOL)isEqualToArray:(NSArray *)array {
  if (!array || [self count] != [array count]) {
    return NO;
  }

  int idx;
  for (idx = 0; idx < [array count]; idx++) {
    if (![[self objectAtIndex: idx] isEqual: [array objectAtIndex: idx]]) {
          return NO;
      }
  }
 
  return YES;
}
 
- (BOOL)isEqual:(id)object {
  if (self == object) {
    return YES;
  }
 
  if (![object isKindOfClass:[NSArray class]]) {
    return NO;
  }
 
  return [self isEqualToArray:(NSArray *)object];
}

@end

NSArray *array1 =[NSArray arrayWithObjects:@"Hello",@"World", nil];        
NSArray *array2 = [NSArray arrayWithObjects: @"Hello", @"World", nil];
[set addObject: array1];
[set addObject: array2];

NSLog(@"%@", set); // ((Hello, World))
```

참조할 객체가 너무 많아서 컬렉션 자체가 너무 큰 경우에 사용하는 것이 메모리 관레 유리

3. NSCountedSet 

집합 내부에 동일한 객체를 여러 번 참조하기 위해서는 NSCountedSet

중복은 Count가 추가되어 나타남

```swift
NSCountedSet *countedSet = [[NSCountedSet alloc] initWithCapacity: 10];
NSSet *otherSet = [NSSet setWithArray: @[@"3.14", @"2.71", @"1.414"]];

[countedSet addObject:@"3.14"];
[countedSet addObject:@"0.123"];

[countedSet unionSet:otherSet];

NSLog(@"%@", countedSet);
// (0.123 [1], 3.14 [2], 2.71 [1], 1.414 [1])
```

6.3.2 

포인터 집합

NSHashTable

NSHashTable은 NSPointerFunctions.Option 설정값으로 각각 다른 옵션을 설정할 수 있다.

```swift
NSHashTable *hashTable = [NSHashTable hashTableWithOptions: NSPointerFunctionsCopyIn];
[hashTable addObject: @"foo"];
[hashTable addObject: @"bar"];
[hashTable addObject: @"foo"];
[hashTable addObject: @"42"];
NSLog (@ "Members:% @", [hashTable allObjects]);
```

NSPointerFunctionsCopyIn 옵션을 통해 object를 복사해서 set에 add하도록 함

추가적인 옵션들

[Apple Developer Documentation](https://developer.apple.com/documentation/foundation/nspointerfunctions/options)

6.3.3 Performance

Search: NSMutableArray VS NSMutableSet

N O(N^2)

```swift
NSMutableArray *array = [[NSMutableArray alloc] initWithCapacity: 100];
long i = 0;
for (i = 0; i < 100; i++) {
    NSString *string = [NSString stringWithFormat: @"unique-%10ld", i];
    [array addObject: string];
}

int count = 0;
for (i = 0; i < 100; i++) {
    if ([array containsObject: [NSString stringWithFormat: @"unique-%10ld", i]]) {
        count++;
    }
}
```

O(N)

```swift
NSMutableSet *set = [[NSMutableSet alloc] initWithCapacity: 100];
long i = 0;
for (i = 0; i < 100; i++) {
    NSString *string = [NSString stringWithFormat: @"unique-%10ld", i];
    [set addObject: string];
}

int count = 0;
for (i = 0; i < 100; i++) {
    if ([set containsObject: [NSString stringWithFormat: @"unique-%10ld", i]]) {
        count++;
    }
}
```

![Untitled2.png](./assets/Untitled2.png)
