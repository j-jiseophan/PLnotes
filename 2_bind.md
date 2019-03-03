## Identifier
Identifier(ID)란 쉽게말해 변수입니다. 이 변수는 숫자를 담을수도 있고 클래스를 담을수도 있고 또는 함수를 담을수도 있습니다.
우리는 이제 이 ID를 Binding, Bound, Free 세가지 종류로 구분해 볼 것입니다.

## Binding Occurence
변수가 선언부분에 있으면 이는 Binding Occurence 입니다.

    def f(x, y) = x + y + z
    def g() = {
        val a = 3
        val c = 4
        a + b + c
    }
위 예제에서는 f(x, y)안에있는 x,y 그리고 a=3 c=4 부분의 a,c가 각각 ID의 Binding Occurence라고 할 수 있습니다.

## Bound Occurence
변수가 선언부에서 **Binding 된후 사용이 되면** 이는 ID의 Bound Occurence라고 합니다. 따라서 위 예제에서는 x+y+z의 x,y가 a+b+c의 a,c가 Bound Occurence 입니다.

## Free ID
만약 변수가 Binding 되지 않았는데 사용된다면 어떻게 될까요? 이러한경우를 Free ID라고 하며 이는 에러(Free Identifier Error)를 유발합니다. 위 예제에서는 z, b가 선언되지 않았는데 사용되었으므로 Free ID라 할 수 있습니다.

## WAE
이전 포스트에서는 Num, Add, Sub가 구현된 AE를 배웠습니다. 그럼 AE에 Bind 기능(변수사용 기능)을 추가하기 위해 새로운 표현 with를 추가한 WAE를 알아봅시다
    
    <WAE> ::= <num>
            | {+ <WAE> <WAE>}
            | {- <WAE> <WAE>}
            | {with {<id> <WAE>} <WAE>}
            | <id>
두줄이 추가되었습니다 with 와 id 입니다. id는 변수이름을 나타내는 표현이고 with는 id와 value를 연결해주는 (일종의 변수 초기화) 표현입니다.
{with {a b} c} 표현은 변수 a를 b로 초기화하고 c에 행동을 수행하라 라는 의미입니다 
예제를 보면 이해가 빠르게 됩니다

    {with {x {+ 1 2}}{+ x x}}
먼저 x를 {+ 1 2} 의 값으로 초기화 해줘야합니다.{+ 1 2}는 3과 같으므로 위 식은 아래와 같이 정리됩니다

    {+ x x} ..[x=3]
따라서 최종적인 결과는 6 이됩니다.

## 변수 Shadowing
만약 아래와 같은 식이면 어떻게 해야할까요?

    {with {x {+ 1 2}}
        {with {x {- 4 3}}
            {+ x x}}}
보면 알수있듯이 With 안에 With가 있습니다. 문제는 x가 두번 Binding 된다는것인데 그렇다면 각각의 연산에서는 어떤 x값을 사용해야 할까요? 이러한 문제를 해결하기 위해 Variable Shadowing이라는 기법을 사용합니다. 이는 변수선언이 유효한 스코프(범위)를 제한 하는 방식입니다.

    1   {with {x {- 4 3}}{+ x x}}} .. [x=3]
    2   {+ x x} .. [x=1]
    3   {2}
먼저 가장 바깥 with에서 x=3으로 선언 됩니다. 그 후 안쪽 스코프로 들어가면 또다른 with가 x=1로 선언됩니다. 이 스코프 내에서는 x=1이 바깥쪽의 선언을 덮어쓰기 때문에 {+ x x}는 2로 해석됩니다.







