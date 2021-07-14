#3.2 ARC 구현 방식

ARC: 컴파일러가 객체 생명주기를 판단해서 메모리 관리 코드를 자동으로 추가해주는 방식

컴파일러가 추가하는 코드는 objc 런타임 함수로 구성되고 새 os 버전이 나올 때마다 성능이 개선되고 있다.

##3.2.1 강한 참조

NSString __strong *aString = [[NSString alloc] init]; 

위처럼 strong 변수를 선언했을 때 컴파일러가 변환한 코드

id tmp = objc_msgSend(NSString, @selector(alloc)); objc_msgSend(tmp, @selector(init)); NSString* aString; objc_storeStrong(&aString, tmp); 

objc_storeStong() 함수의 구현은 다음과 같다.

void objc_storeStrong(id *location, id obj) { 
	id prev = *location; 
	if (obj == prev) { 
		return; 
	} 
	objc_retain(obj); 
	*location = obj; 
	objc_release(prev); 
} 

새로 저장하는 obj는 retain으로 소유권을 갖고 기존에 location이 참조하던 객체는 release로 소유권을 반환한다.


##3.2.2 자동 반환용 리턴 값

objc에서는 '두 단계 초기화 패턴'으로 객체 인스턴스를 만든다
객체 인스턴스를 힙 공간에 생성 -> 할당한 메모리 공간을 초기 값으로 채워넣음.

이 두 단계 초기화 패턴을 한꺼번에 처리해주는 초기화 메서드인 
Convinience Method로 객체를 만드는 경우 객체는 자동 해제 대상이 된다.
```objectivec
NSDictionary __strong *dictionary = [NSDictionary dictionary]; 
```
위처럼 convinience method로 객체 생성할 경우 컴파일러가 변환한 코드

```objectivec
id tmp = objc_msgSend(NSDictionary, @selector(dictionary)); 
objc_retainAutoreleasedReturnValue(tmp); 
NSDictionary *dictionary; 
objc_storeStrong(&dictionary, tmp); 
```

objc_retainAutoreleasedReturnValue() 함수를 사용해서 객체를 AutoreleasePool에 등록하고, 등록된 객체를 반환받아 그 객체에 대해 소유권을 갖는 것이다(retain) 
근데 항상 retain 하는 것은 아니고 스레드 TLS 영역에 정보를 저장하는 최적화 루틴을 포함한다. 

```objectivec
+ (instancetype) dictionary { 
	instancetype tmp = objc_msgSend(NSDictionary, @selector(alloc)); 
	objc_msgSend(tmp, @selector(init));	
  return objc_autoreleaseReturnValue(tmp); 
} 
```

##3.2.3 약한 참조

```objectivec
NSString __weak *aString = [[NSString alloc] init]; 
```

위처럼 weak 변수를 선언했을 때 컴파일러가 변환한 코드

```objectivec
d tmp = objc_msgSend(NSString, @selector(alloc)); 
objc_msgSend(tmp, @selector(init));
NSString* aString;
objc_initWeak(&aString, tmp); 
objc_release(tmp);
objc_destroyWeak(&aString); 
```

objc_initWeak은 다음과 같이 구현되어 있다.

```objectivec
id objc_initWeak(id *addr, id val) { 
	*addr = 0;	
  if (!val) return nil;	
  return objc_storeWeak(addr, val); 
}
```

objc_storeWeak(addr, val)은 addr 포인터에 있던 이전 객체에 대한 약한 참조는 해지하고 val 객체에 대한 약한 참조를 등록한다.

objc_destroyWeak은 다음과 같이 구현되어있다. 

```objectivec
void objc_destroyWeak(id *addr) { 
	if (!*addr) return; 
	return objc_destroyWeak_slow(addr); 
}
```
 
###약한 참조 불가능한 객체

allowsWeakReference 메서드(objc_storeWeak() 함수 내부에서 사용) 의 리턴 값이 NO이면 메모리가 중복 해제됐다고 가정하여 에러를 표시한다. 
retainWeakReference 메서드(objc_loadWeak() 함수 내부에서 사용) 가 구현되어 있지않거나 NO를 반환할 경우 마찬가지로 약한 참조가 불가능한 객체이다.

##3.2.4 자동 반환 방식 

##3.2.5 요약
ARC 구현 방식은 새로운 운영체제 버전이 나올 때마다 개선된다. OS X 10.9 까지는 약한 참조로 객체를 참조할 때마다 자동 반환 목록에 등록하던 방식으로 동작했었지만, OS X 10.10부터는 더 이상 자동 반환 목록에 등록하지 않는다. ARC 환경에서도 객체 인스턴스에 대한 메모리 관리는 신경 써야만 한다. 




