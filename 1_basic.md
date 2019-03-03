## Syntax
syntax는 말그대로 문법입니다. C로 예를들어보죠
    
    printf("abc");
여기에서 보면 printf 함수는 괄호로 인수를 받고 괄호안에는 string이 와야하고 문장의 맨 마지막에는 ; 이 와야함을 알수 있습니다 이렇게 겉으로 보이는 구조적인 측면이 syntax입니다.
## Semantics
Semantics는 언어의 의미론적인 측면입니다. 위 코드에서 무엇을 의미하는가? 하면 "abc"라는 스트링을 stdout으로 출력하라! 하는게 언어의 의미론적인 측면입니다.

## Abstract Syntax, Concrete Syntax
    add(3,4) //Abstract
    {3 + 4}// concrete
문법을 표기하는 방식에는 두가지가 있습니다. 첫째로 Abstract는 실제로 우리가 프로그래밍에서 쓰는 방식입니다. 하지만 이는 추상적인 표현이므로 표현이 내부적으로 어떻게 동작하는지 알기 어렵습니다. 반면 Concrete는 내부적인 구조를 다 나타내기 때문에 동작을 알수 있습니다. PL 수업에서는 Concrete Syntax를 통해 언어의 내부 구조를 배웁니다.

## BNF
    〈AE〉 ::=  {〈AE〉+〈AE〉}//addition
            | {〈AE〉-〈AE〉} //subtraction
            | <num> //number
Backus-Naur Form (BNF)는 Concrete Syntax의 표기 방법입니다.
<>는 집합을 의미합니다
PL 수업에서는 우선 AE(Arithmetic Expression)이라는 매우 심플한 언어형태를 설계합니다. 
## AE
이제 스칼라로 AE를 구현해 볼것입니다.
위의 Concrete Syntax를 scala 문법으로 표기하면 아래와 같습니다.

    trait AE
    case class Num(n: Int) extends AE
    case class Add(l: AE, r: AE) extends AE
    case class Sub(l: AE, r: AE) extends AE
 BNF를 스칼라로 구현하기 위해 AE라는 trait(상위클래스라고 생각하면 됩니다.)을 만든뒤 이들의 SubClass들로 Num, Add, Sub를 만들었습니다. 하지만 아직 이들이 어떠한 동작을 하는지는 지정해주지 않았습니다. 이를 위해 스칼라에서는 패턴매칭을 사용합니다.
## 파싱
스칼라에서 "{+ 3 4}" 스트링을 ADD(3 ,4) 클래스로
"2"를 스트링을 Num(2) 클래스로 변환하려면 어떻게 해야할까요? 
이러한 과정을 파싱(Parsing)이라고 하며 이를 수행하는 주체를 파서(Parser)이라고 부릅니다. 파서는 스트링을 받아 적절하게 해석하여 적절한 객체로 변환해줍니다

## 패턴매칭
    def interp(ae: AE): Int = ae match {
        case Num(n) => n
        case Add(l, r) => interp(l) + interp(r)
        case Sub(l, r) => interp(l) - interp(r)
    }
이제 파싱된 객체들의 연산을 수행하고 결과를 알려주도록 interp함수를 구현해야 합니다.
interp 함수는 AE를 받아서 이 AE가 무엇이냐에 따라 행동을 지정해주고 있습니다. 이 AE가 무엇인지를 구분하는 역할을 스칼라의 패턴 매칭 기능을 이용해 구현하고 있습니다.
패턴매칭은 쉽게말하면 대상의 케이스들을 구분해주는 조건문 역할입니다.
패턴매칭에 대한 자세한 사항은 [Link](http://www.bench87.com/content/30)를 참고하세요.

## interp 함수 사용
스칼라에서 구현한 interp 함수는  AE를 해석하여 결과를 리턴합니다.
예제를 보면 빠르게 이해가 됩니다
    
    interp(AE("{3}"))
    => 3
    
    interp(AE("{+ 1 4}"))
    => 5

## AE의 수학적 표현
위에서 배운 AE는 아래와 같이 수학적 기호로도 나타낼수 있습니다.

    (n∈Z)/(n∈A)     //Num
    (e1∈A  e2∈A)/({+e1e2}∈A)    //Add
    (e1∈A e2∈A)/({-e1e2}∈A)     //Sub

    의미 : if(분자 is true) than 분모 
분자는 조건역할을 분모는 액션 역할을 합니다. 분자에 조건이 여러개인경우 띄어쓰기로 구분하며 조건이 여러개일경우 모두 만족해야 true로 간주합니다.








