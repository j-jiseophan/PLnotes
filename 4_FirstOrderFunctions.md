## 일차함수
이번챕터에서는 이전의 WAE에 함수기능을 추가하는 과정을 다룹니다. 특히 이번 장에서 다루는 '함수'란 일차함수 (First Order Function)을 말합니다. 이 일차함수란 y=x+1 과 같은 수학적 의미가 아닌 함수안에 함수를 인자(Parameter)로 넣을 수 없는 함수를 말합니다. 

    someFunction(someFunction(1))

그렇다면 위함수는 일차함수일까요 아닐까요? 함수안에 함수가 들어갔으니 일차함수가 아닐거라 생각할 수 있지만 이는 실제로는 함수안에 **함수의 리턴값이**들어간 것이므로 일차함수라고 할 수 있습니다.

## F1WAE : Concrete Syntax
    <FunDef> ::= {deffun {<id> <id>} <F1WAE>}
    <F1WAE> ::= <num>
        | {+ <F1WAE> <F1WAE>}
        | {- <F1WAE> <F1WAE>}
        | {with {<id> <F1WAE>} <F1WAE>}
        | <id>
        | {<id> <F1WAE>}
일차함수가 추가된 F1WAE는 위와 같습니다.
먼저 FunDef 부분은 이름에서 알수 있듯이 함수를 정의하는 부분입니다. 첫번째 id는 함수이름 두번째 id는 패래미터 그리고 F1WAE 부분은 함수의 바디(함수의 구현 내용)부분입니다. 둘째로 F1WAE의 하위 속성으로 추가된 맨 마지막줄은 정의된 함수를 사용하는 부분입니다 id는 함수 이름을 F1WAE 부분은 패래미터 입니다. 

그럼 굳이 이렇게 함수의 정의부를 F1WAE가 아닌 FunDef로 따로 구분한 이유는 무엇일까요? 만약 함수의 정의 표현이 F1WAE의 하위 표현이라면 정의표현을 ADD나 SUB에도 넣을수 있게 됩니다. 하지만 일차함수는 함수자체를 값으로 취급할 수 없으므로 이는 오류를 유발합니다. 이러한 문제를 막기 위해 FunDef를 분리한 것입니다.

사용예시는 아래와 같습니다.

    {deffun {twice x} {+ x x}}
    {- 20 {twice {+ 3 3}}}

## F1WAE : Abstract Syntax
Concrete Syntax로 설계를 했으니 이제 Scala를 이용해 abstract Syntax로 표현할 차례입니다. 이는 아래와 같습니다.

    case class FunDef(f: String, x: String, b: F1WAE)
    
    trait F1WAE
    case class Num(n: Int) extends F1WAE
    case class Add(l: F1WAE, r: F1WAE) extends F1WAE
    case class Sub(l: F1WAE, r: F1WAE) extends F1WAEcase class With(x: String, i: F1WAE, b: F1WAE)extends F1WAE
    case class Id(x: String) extends F1WAE
    case class App(f: String, a: F1WAE) extends F1WAE

함수의 사용을 App 클래스를 통해서 구현했습니다.

## F1WAE : Semantics
이제 Syntax구현을 완료했으니 Semantics 부분을 구현할 차례입니다. 이전에도 그랬듯이 Semantics 부분은 interpreter 역할을 하는 interp 함수로 구현합니다 이는 아래와 같습니다.

    type FDs = List[FunDef]
    def interp(e: F1WAE, env: Env, fs: FDs): Int = e match {
        case Num(n) => n
        case Add(l, r) => interp(l, env, fs) + interp(r, env, fs)
        case Sub(l, r) => interp(l, env, fs) - interp(r, env, fs)
        case With(x, i, b) =>interp(b, env + (x -> interp(i, env, fs)), fs)
        case Id(x) => lookup(x, env)
        case App(f, a) => lookupFD(f, fs) match {
            case FunDef(fname, pname, body) =>
            val aval = interp(a, env, fs)
            interp(body, Map() + (pname -> aval), fs)
    }


이전의 interp함수와 달라진점을 먼저 찾아보자면 interp의 인자로 FDs형식의 fs가 추가되었습니다. env가 변수(id)의 값을 저장하는 map 자료구조라면 FD는 정의된 함수들을 저장하는 리스트입니다. 마찬가지로 lookup이 id가 env에 포함되어있는지 확인하는 역할을 한다면 App의 lookupFD은 f가 fs에 포함되어있는지 확인해줍니다.

## Scoping
함수의 변수허용 범위를 정하는 방식을 스코핑이라 합니다. 스코핑에는 정적스코핑과 동적스코핑이 있습니다.
to be written..
