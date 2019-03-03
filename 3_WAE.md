## With의 구현과 Env
이제 저번 포스트에서 설명한 With를 스칼라 코드로 구현해 볼 것입니다. 이를 구현하기 위해 잠깐 지난 시간의 예제를 다시 보겠습니다.

    {with {x {+ 1 2}}
        {with {x {- 4 3}}
            {+ x x}}}
이 코드는 아래순서로 해석 됩니다.

    1   {with {x {- 4 3}}{+ x x}}} .. [x=3]
    2   {+ x x} .. [x=1]
    3   {2}

위 예제를 보면 ID를 이용하여 연산을 하기 위해 ID의 값을 저장해놓는 곳이 있습니다. 이를 Env(Environment)라고 합니다. Env는 변수와 그에 대응되는 값을 맵(Key : Value) 형태로 저장합니다. 이전에는 interp함수의 패래미터로 AE하나만 있었지만 이제는 여기에 Env를 추가해줍니다. 예제를 통해 알아보겠습니다.

    1   interp(WAE("{with {X 1}
                        {with {y 2}
                            {+ x y}}}), Map()) //empty env
    2   interp(WAE("{with {y 2}
                        {+ x y}}), Map("x" -> 1)) //env에 x=1 쌍이 추가됨
    3   interp(WAE("{+ x y}"), Map("x" -> 1, "y" -> 2)) //env에 x=2 쌍이 추가됨

    4   3

위와 같이 interp 함수는 재귀적으로 Env에 키:밸류 쌍을 추가하면서 해석하게 됩니다. 함수는 ID를 마주치면 이에 해당되는 밸류를 env에서 찾습니다. 이를 lookUp이라고 합니다. 만약 해당되는 밸류가 있으면 이를 이용하여 연산을 수행하고 그렇지 않으면 free identifier error를 출력하는 것입니다.

이제 interp 함수는 아래와 같이 확장되었습니다.

    def interp(wae: WAE, env: Env): Int = wae match {
        case Num(n) => n
        case Add(l, r) => interp(l, env) + interp(r, env)
        case Sub(l, r) => interp(l, env) - interp(r, env)
        case With(x, i, b) => interp(b, env + (x -> interp(i, env))
        case Id(x) => lookup(x, env)
    }




    

