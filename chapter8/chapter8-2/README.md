# 8-2 스위프트 클로저

클로저는 접근 가능한 특정 범위(scope) 내에서 사용하는 값을 (함수 내부에) 갖고있는 함수를 의미한다. 

스위프트에서 함수는 모두 클로저이다.
스위프트에서 클로저는 함수이거나 이름 없는 그냥 클로저 중 하나이다.

```swift
({
    print("hi")
}())
```

```swift
func squared(n: Int) -> Int { return n * n }

let numberArray = [2, 7, 1, 2, 3]
let result = numberArray.map(squared) // 4 49 1 4 9
```

참조 범위가 전체인 글로벌 함수(Global Function)은 '이름은 있지만 캡처하는 변수가 없는 클로저'

```swift
class Number: ExpressibleByIntegerLiteral {
    typealias IntegerLiteralType = Int
    required init(integerLiteral value: Int) {
        self.value = value
    }
    static func + (left: Number, right: Number) -> Number {
        return Number(integerLiteral: left.value + right.value)
    }

    var value: Int = 0
}

var test: Number = 10

let closure = { (variable: Number) -> Number in
    return test + variable
}

test = 20
print(closure(20).value) // ?
```

```swift
var test = 10

let closure = { variable -> Int in
    return test + variable
}

test = 20
print(closure(20)) // ?
```

## 값 캡쳐 (Capturing Values)

클로저는 특정 문맥의 상수나 변수의 값을 캡쳐할 수 있다. 
다시말해 원본 값이 사라져도 클로져의 body안에서 그 값을 활용할 수 있다.

```swift
func makeIncrementer(forIncrement amount: Int) -> () -> Int {
    var runningTotal = 0
    func incrementer() -> Int {
        runningTotal += amount
        return runningTotal
    }
    return incrementer
}

let incrementByTen = makeIncrementer(forIncrement: 10)

incrementByTen() // ?
incrementByTen() // ?
incrementByTen() // ?
```

```swift
let newIncrementByTen = makeIncrementer(forIncrement: 10)

newIncrementByTen() // ?
incrementByTen() // ?
newIncrementByTen() // ?
```

## 클로저는 참조 타입 (Closures Are Reference Types)

```swift
let newnewIncrementByTen = incrementByTen

newnewIncrementByTen() // ?
```

## 이스케이핑 클로저 (Escaping Closures)

함수 밖(함수가 끝나고)에서 실행 가능한 클로저

```swift
var completionHandlers: [() -> Void] = []

func someFunctionWithEscapingClosure(completionHandler: @escaping () -> Void) {
    completionHandlers.append(completionHandler)
}

func someFunctionWithNonescapingClosure(completionHandler: () -> Void) {
    completionHandler() 
}

class SomeClass {
    var x = 10
    func doSomething() {
        someFunctionWithEscapingClosure { self.x = 100 }
        someFunctionWithNonescapingClosure { x = 200 }
    }
}

let instance = SomeClass()
instance.doSomething()
print(instance.x) // ?

completionHandlers.first?()
print(instance.x) // ?
```

## 자동클로저 (Autoclosures)

인자 값이 없으며 특정 표현을 감싸서 다른 함수에 전달 인자로 사용할 수 있는 클로저

```swift

var customersInLine = ["Ewa", "Barry", "Daniella"]
func serve(customer customerProvider: @autoclosure () -> String) {
    print("Now serving \(customerProvider())!")
}
serve(customer: customersInLine.remove(at: 0))

```

```swift
var customerProviders: [() -> String] = []
func collectCustomerProviders(_ customerProvider: @autoclosure @escaping () -> String) {
    customerProviders.append(customerProvider)
}
collectCustomerProviders(customersInLine.remove(at: 0)) 
collectCustomerProviders(customersInLine.remove(at: 0))

print("Collected \(customerProviders.count) closures.") // ?
for customerProvider in customerProviders {
    print("Now serving \(customerProvider())!")
}
```

## 즉시 실행 함수(IIFE)

```swift
({
    let test = 10
    print("hi", test)
}())

print(test)
```

```swift
let value = ({ variable -> Int in
    let test = 10
    return test + variable
}(10))

print(value)
```

## Lexical Scope

함수를 어디서 호출하는지가 아니라 어디에 선언하였는지에 따라 결정되는 것을 말한다.

```swift
var name = "zero"
func log() {
  print(name)
}

func wrapper() {
  name = "nero"
  log()
}
wrapper() // nero
```

```swift
var name = "zero"
func log() {
  print(name)
}

func wrapper() {
  var name = "nero"
  log()
}
wrapper() // zero
```

함수를 처음 선언하는 순간, 함수 내부의 변수는 자기 스코프로부터 가장 가까운 곳(상위 범위에서)에 있는 변수를 계속 참조하게 된다.

```swift
var name = "zero"
func wow(_ word: String) {
    print(word + " " + name)
}
func say () {
  var name = "nero"
  print(name, terminator: " ")
  wow("hello")
}
say() // ? nero hellow zero
```

## 함수 표현 유형 (Representation Type)

스위프트 중간 언어(SIL)에 선언된 함수 표현 유형에 따르면

스위프트 함수는 CFunctionPointer, ObjcMethod, Block과 연결 가능하다.

스위프트 컴파일러가 연결 가능한 외부 언어(Foreign Lanaguage)는 스위프트 2.2 기준으로 C언어, ObjectiveC 뿐

```swift
main.swift

import Foundation

var output: CInt = 0
getInput(&output)

println(output)
```

```swift
#include <stdio.h>

void getInput(int *output) {
    scanf("%i", output);
}
```

```swift
cliinput-Bridging-Header.h

void getInput(int *output);
```

## 요약

스위프트의 함수, 고차함수를 파라미터로 받아 처리하는 동작 범위가 넓은 함수부터 동작을 한정적으로 구현하는 방식처럼 클로저가 가진 역할과 동작을 잘 이해해서 함수의 역할에 따라 적합한 방식을 적용할 수 있어야한다.

## REFERENCE

[클로저 (Closures)](https://jusung.gitbook.io/the-swift-language-guide/language-guide/07-closures)
