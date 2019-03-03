## 일급함수
이번장에서는 지난 F1WAE의 함수를 약간 수정하여 일급함수를 구현할 것입니다. 일급함수란 지난 장에서 다룬 일차함수와는 다르게 함수의 리턴값이 아닌 함수 그 자체를 Value로 취급할수 있는 함수입니다.

    {with {f {fun {x} {+ 1 x}}}
                            {f 10}}
위는 일급함수를 설명하는 아주 간단한 예시입니다. with를 이용해 f에 어떤 값이 아닌 함수의 선언 그 자체를 담고 있고 이를 이용해 f 10을 수행하고 있습니다. 결과는 11.
이러한 일급함수는 함수형 프로그래밍의 기초적인 개념입니다.

## FWAE : Concrete Syntax
    <FWAE> ::= <num>
        | {+ <FWAE> <FWAE>}
        | {- <FWAE> <FWAE>}
        | {with {<id> <FWAE>} <FWAE>}
        | <id>
        | {<id> <FWAE>}                
        | {fun {<id>} <FWAE>}
이전의 일차함수와는 달리 함수의 정의부또한 FAWE에 통합되었습니다. 이를통해 함수 자체를 값으로 취급하여 변수에 넣거나 인자로 사용할수 있게됩니다.
id FAWE 부분은 함수의 사용부입니다 id는 함수가 할당(binding)된 변수또는 함수선언 그자체이며, FAWE는 패러미터입니다. 
fun id FAWE는 함수의 정의부입니다. 여기서 id는 **함수이름이 아닌 함수의 인자입니다** FAWE는 함수의 바디(구현부)입니다.
아래 예시를 보면 쉽게 이해가 됩니다.

    Usage1:    {with {f {fun {x} {+ x x}}}
        {- 20 {f 10}}}
    
    Usage2:   {- 20 {{fun {x} {+ x x}} 17}}}

첫번째 예시는 함수를 with를 통해 f에 바인딩하여 사용하고 있고 두번째 예시는 함수의 정의 자체를 이용하여 함수를 사용하고 있습니다.

## FWAE : Abstract Syntax
위에서 다룬 Concrete Syntax를 스칼라로 추상화하면 아래와 같습니다.

    trait FWAE
    case class Num(n: Int) extends FWAE
    case class Add(l: FWAE, r: FWAE) extends FWAE
    case class Sub(l: FWAE, r: FWAE) extends FWAE
    case class With(x: String, i: FWAE, b: FWAE)extends FWAE
    case class Id(x: String) extends FWAE
    case class Fun(x: String, b: FWAE) extends FWAE
    case class App(f: FWAE, a: FWAE) extends FWAE

## FAWE : Semantics
이제 interp 함수를 알아볼 차례입니다.

    def interp(e: FWAE, env: Env): FWAE = e match {
        case Num(n) => n
        case Add(l, r) => numAdd(interp(l, env), interp(r, env))
        case Sub(l, r) => numSub(interp(l, env), interp(r, env))
        case With(x, i, b) =>
                    interp(b, env + (x -> interp(i, env)))
        case Id(x) => lookup(x, env)
        case Fun(x, b) => ...
        case App(f, a) => ...
    }
Fun, App 케이스의 구현은 뒤에 알아보기로 하고 우선 가장 눈에띄는 변화는 Env와 Fd가 통합되었다는 것입니다. 함수가 객체로 간주될 수 있게 되었기 때문에 이는 당연한 변화입니다.

## FAWE의 Values
함수가 객체취급되면서 문제가 발견했습니다. 이전의 AE에서는 모든 연산의 리턴값이 int형태였는데 이제는 함수 또한 리턴될수 있으므로 다른 타입간에 연산시 타입에러가 유발될 수 있습니다.
{+ fun {x} {+ x x}} 17}} 예를 들면 이런 코드가 에러를 유발합니다.

이러한 문제를 해결하기 위해 FAWEValue 속성을 정의하고 이들의 SubClass로 리턴 타입들을 정의해줄 필요가 있습니다.

    trait FWAEValue
    case class NumV(n: Int) extends FWAEValue
    case class CloV(param: String,body: FWAE,env: Env) extends FWAEValue
    type Env = Map[String, FWAEValue]

먼저 Env의 키:밸류 쌍을 스트링:FAWEValue로 정의해주었고 FAWEValue의 하위클래스로 숫자형태의 자료형인 NumV와 함수형태의 자료형인 Clov를 정의해 주었습니다.

따라서 이제 interp 함수를 실행하면 아래와 같이 동작해야 합니다.

    scala > test(interp(FWAE("{with {y 10} {fun {x} {+ y x}}}"), Map())
       CloV("x", Add(Id("y"), Id("x")), ("y" -> NumV(10))))First-Class Functions26/40

## Interpreter 개선
그럼 이제 interp 함수가 FAWEValue를 리턴하도록 수정해 봅시다.

    def interp(e: FWAE, env: Env): FWAEValue = e match {
        case Num(n) => NumV(n)
        case Add(l, r) => numVAdd(interp(l, env), interp(r, env))
        case Sub(l, r) => numVSub(interp(l, env), interp(r, env))
        case With(x, i, b) =>interp(b, env + (x -> interp(i, env)))
        case Id(x) => lookup(x, env)
        case Fun(x, b) => CloV(x, b, env)
        case App(f, a) => interp(f, env) match {
            case ColV(x, b, fenv) =>
                interp(b, fenv + (x -> interp(a, env)))
            case v =>
                error(s"not a closure: $v")
        }
    }
먼저 interp 함수는 이제 FAWE를 반환하는게 아닌 FAWEValue를 반환 함ㅇ르 알수 있습니다. 따라서 case Num(n)도 이제는 n을 반환하는게 아닌 NumV형을 반환합니다. 또한 Fun 케이스는 함수 자료형인 Clov형을 반환합니다. 마지막으로 함수를 사용하는 부분인 App(f, a)는 이제 패턴매칭을 이용하여 함수가 정의되었는지 아닌지 확인한후 정의되었으면 그 결과를, 그렇지 않으면 에러를 출력합니다.

## 필요없는 with 표현
이제 위에 코드를 다시 자세히보면 중복되는 부분이 보입니다. With case랑 App case의 동작이 똑같습니다.
따라서 이제 아래 두 코드는 동치입니다.
    
    {with {x 10} x}

    {{fun {x} x} 10}

이를 일반화 하면 아래와 같습니다.

    {with {<id> <FWAE1>} <FWAE2>}
    {{fun {<id>} <FWAE2>} <FWAE1>}

이로써 앞으로는 with를 사용하지 않을 것입니다.