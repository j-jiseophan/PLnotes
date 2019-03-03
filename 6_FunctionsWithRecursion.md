## 외전 : Lambda Calculus Grammar
이전까지 우리는 기초적인 AE를 확장하여 상당히 복잡해진 문법을 구현했습니다.

    <FWAE> ::= <num>
        | {+ <FWAE> <FWAE>}
        | {- <FWAE> <FWAE>}
        | {with {<id> <FWAE>} <FWAE>}
        | <id>
        | {<id> <FWAE>}                
        | {fun {<id>} <FWAE>}

이에 [람다대수](https://ko.wikipedia.org/wiki/%EB%9E%8C%EB%8B%A4_%EB%8C%80%EC%88%98)의 방법을 적용하면 상당히 간결하게 줄일 수 있습니다. 이 방법은 튜링머신의 작동 원리를 따릅니다. 우선 아래 BNF 표현부터 봅시다

    <LC> ::= <id>
            | {<LC> <LC>}
            | {fun {<id>} <LC>}

그럼 이런 표현들만으로 어떻게 모든 명령들을 표현할 수 있을까요? 튜링 머신에서 모든 명령들은 0과 1로 표현이 가능합니다 .. tobewritten

## 재귀함수의 구현
이번 포스트의 메인 주제는 지난 시간에 배운 FAWE를 이용해 재귀 함수를 구현하는 것입니다. 그러려면 우선 FAWE에 한줄을 추가해야 합니다.

    <FWAE> ::= <num>
        | {+ <FWAE> <FWAE>}
        | {- <FWAE> <FWAE>}
        | {with {<id> <FWAE>} <FWAE>}
        | <id>
        | {<id> <FWAE>}                
        | {fun {<id>} <FWAE>}
        | {if0 <RCFAE> <RCFAE> <RCFAE>}
if0 절은 RCFAE1이 0일경우 RCFAE2를 그렇지 않을경우는 RCFAE3를 실행합니다.
재귀함수를 이용한 가장 대표적인 문제인 팩토리얼을 구현해 봅시다.

    {with {fac {fun {n}
                {if0 n
                     1
                     {* n {fac {- n 1}}}}}}
          {fac 10}}
    
이 코드는 정상적으로 작동하지 않습니다. 왜냐하면 3번째 줄에 있는 fac은 Static Scoping에의해 Binding 되지 않기때문입니다. 따라서 Free Identifier 에러가 납니다. 따라서 이를 해결하기 위해서는 함수자체를 인자로 넣어 주어야 합니다.

        {with {facX {fun {facY n}
                        {if0 n
                        1
                        {* n {facY facY {- n 1}}}}}}
            {facX facX 10}}

코드를 해석하기가 상당히 난해합니다. 함수의 정의부를 보면 두가지 인수를 받도록 되어있습니다. 이를 이용해 마지막줄에서 인자로 함수의 인자로 함수자신과 10을 넣습니다.

하지만 이는 사용이 너무 복잡합니다. 따라서 아래와 같이 몇줄을 더 추가에서 간단하게 사용하도록 해줍니다.

    {with {fac
            {fun {n}
                {with {facX 
                        {fun {facY n}
                             {if0 n
                                  1
                                  {* n {facY facY{- n 1}}}}}}
                      {facX facX n}}}}
          {fac 10}

하지만 이러한 방식으로는 불가능합니다 왜냐하면 FWAE의 함수정의는 인자를 한가지만 받을 수 있도록 설계 되어있기 때문입니다. 따라서 코드를 아래와 같이 개선합니다.

    {with {fac
            {fun {n}
             {with {facX
                    {fun {facY}
                      {fun {n}
                           {if0 n
                                1
                                {* n {{facY facY} {- n 1}}}}}}}
                  {{facX facX} n}}}}
          {fac 10}}
//todo: code explanation
Lambda Calculus에서의 η(Eta) reduction을 이용하면 이를 간단하게 줄일 수 있습니다. η reduction 을 이용하면 {fun {n} {with {f ...} {{f f} n}}}을 {with {f ...} {f f}} 로 변환 할 수 있습니다. 이에따라 코드는 아래와 같이 변합니다.

    {with {fac
            {with {facX
                    {fun {facY}
                        {fun {n}
                            {if0 n
                                 1
                                 {* n {{facY facY} {- n 1}}}}}}}   
                  {facX facX}}}
          {fac 10}}
