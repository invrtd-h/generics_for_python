## Generics for Python Library.

Note that this library is not yet ready for production use.

### Korean

~~복잡한~~ 간단한 명령어를 통해 Python에서 사용할 수 있는 Abstract Base Class 
코드를 출력하는 라이브러리입니다. 

#### Abstract Base Class

Java에서의 interface와 비슷한 개념입니다. Python에서는 Interface를
만드는 문법을 지원하지 않으나, Abstract Base Class를 사용하면
Interface와 비슷한 역할을 하는 클래스를 만들 수 있습니다.
Abstract Base Class는 그 자체로 인스턴스화가 불가능하고, 클래스
내부의 함수들은 정의되어 있으나 구현되어 있지 않습니다. 구현의
책임은 Abstract Base Class를 상속받은 클래스에게 있습니다.
Abstract Base Class를 사용하는 이유는 비슷한 기능을 하는 여러
가지 클래스에게 동일한 인터페이스를 제공하기 위함이라고 알려져 있습니다.

#### Inspired By

C++20, Concepts

```c++
template<typename T>
concept Hashable = requires(T a)
{
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
```

Rust, Traits

```rust
trait Zero {
    const ZERO: Self;
    fn is_zero(&self) -> bool;
}

impl Zero for i32 {
    const ZERO: Self = 0;

    fn is_zero(&self) -> bool {
        *self == Self::ZERO
    }
}
```

Also learn about **Type Classes** of Haskell.

#### Why we use ABC?

1. IDE의 강력한 기능을 활용할 수 있다.

동적 타이핑 언어에서는 IDE가 변수의 타입을 추론하기 힘들기 때문에 
IDE의 강력한 기능을 활용하기 어렵습니다. 만약 x가 함수의 인자라면
IDE의 입장에서는 x가 어떤 기능을 갖고 있는지 전혀 알 수 없습니다.
Python 3.5의 Type Annotation은 이 문제를 해결하기 위해 도입되었지만,
Generic을 매우 제한적으로만 제공한다는 문제점이 있습니다. 예를 들어
x를 받아 x[0]을 리턴하는 함수를 생각한다면, x의 타입은 list, tuple,
dictionary일 수도 있지만 int일 수는 없을 것입니다. 이 문제점은
Python의 typing 모듈을 통해 일부 해결할 수 있으나 여전히 제한적입니다.
이 프로젝트의 목표는 Python의 Type Annotation을 더 유용하게 사용할
수 있는 Python 코드를 생성하는 것입니다.

2. 더 안전한 코드를 작성할 수 있다.

변수가 필수적으로 구현해야 할 특정한 기능을 명시함으로써 더 안전한
코드를 작성할 수 있습니다. Python의 경우, 만약 어떤 기능이 구현되지 
않아서 Exception이 발생하는데 그 Exception을 무시하는 코드가 내부에 
들어 있다면 디버깅이 힘들어질 수 있습니다. 이런 상황을 막기 위해 변수가
갖고 있는 기능을 미리 명시하는 것이 좋습니다.

#### Main Concepts

```
1. Trait

trait Foo(self):
   // ~~~~
```

trait를 정의합니다.

이름이 trait인 이유는 Python 예약어 중에 trait가 없어서이며, Rust의
trait의 이름을 차용했으나 모든 기능을 구현하지는 않습니다.

Rust의 trait는 trait문 내부에서의 구현을 허용하기 때문에, 사실 
이 기능 자체는 Java의 interface와 더 닮았습니다.

```
2. fn / cls_fn / static_fn / var / cls_var

trait Foo(self):
    fn:
        bar(self, int, int) -> int
    cls_var:
        x: int

```

trait 내부의 함수들을 정의합니다.
() 내부의 원소는 함수의 인자를 의미하며, -> 뒤의 원소는 리턴 타입을 
의미합니다.

```
3. extends

trait Foo(self) extends Bar:
    // ~~~~
```

trait를 상속합니다.

```
4. trait with multiple arguments

trait Addable(self, T):
    fn:
        __add__(self, T) -> T
        
int_addable = Addable[int]
```

trait는 여러 개의 인자를 사용해서 만들 수도 있습니다.
만들어진 trait는 [] 연산자를 이용해서 타입을 바인딩할 수 있습니다.
위의 예제에서 Addable[int]라고 표기하면 T의 위치에 int가 들어가게
되므로, int_addable은 x + (int) 꼴의 표현이 가능한 모든 타입을
지칭합니다.

[] 연산자를 사용하면 1번째 인자인 self에는 어떤 인자도 바인딩되지
않고 2번째 인자부터 바인딩되기 시작합니다. 이는 C++20의 Concept의
관례를 따른 것입니다.

```
5. trait as a predicate

the expression 'int_addable(int)' returns 'True'. 
```

predicate는 type 또는 trait를 인자로 받아서 True/False를 리턴하는
함수입니다. 주로 함수형 프로그래밍에서 case-work를 할 때 사용됩니다.
어떤 predicate에 type을 predicate(type)과 같은 형태로 넣으면 
그 type이 predicate를 만족하는지 여부를 리턴할 수 있습니다. 이
언어에서 모든 type과 trait는 predicate로 사용할 수 있습니다.
이 또한 C++20의 Concept가 predicate로도 사용되는 관례를 따른 
것입니다.

```
6. More about trait declarations

trait Addable(self, T):
    fn:
        __add__(self, T)

trait Foo(self):
    fn:
        bar(self, Addable[int], =float, *, 
            name1:Addable[int], name2:=float) -> int

```

이처럼 함수의 정의부에 trait이 올 수도 있고, python의 default
인자와 keyword 인자를 지원합니다.

더 자세한 예제는 Appendix의 예제를 참고하세요.

#### Policy

전체 문법은 에러 처리 제외 약 150개의 룰로 구성되어 있고, 
ply.yacc()에 따르면 완전한 LALR(1) 문법입니다.

```
"""
program : PROGRAM_BEGIN S_COLON stmts PROGRAM_END S_COLON
        | WS
stmts : stmts stmt
      | stmt
stmt : trait_decl
     | print_stmt
     | generate_stmt
     | assign_stmt
     | S_COLON
trait_decl : TRAIT trait_id LP1 class_args RP1 COLON LP2 trait_decl_stmts RP2 S_COLON
           | TRAIT trait_id LP1 class_args RP1 EXTENDS unary_pred COLON LP2 trait_decl_stmts RP2 S_COLON
trait_id : ID
class_args : main_arg COMMA sub_args
           | main_arg
main_arg : ID
sub_args : necessary_args COMMA optional_args
         | necessary_args
         | optional_args
necessary_args : necessary_args COMMA necessary_arg
               | necessary_arg
necessary_arg : ID
optional_args : optional_args COMMA optional_arg
              | optional_arg
optional_arg : ID ASSIGN unary_pred
trait_decl_stmts : trait_decl_stmts trait_decl_stmt
                 | trait_decl_stmt
                 | PASS S_COLON
trait_decl_stmt : FN COLON LP2 decl_fn_stmts RP2 S_COLON
                | VAR COLON LP2 decl_stmts RP2 S_COLON
                | CLS_FN COLON LP2 decl_cls_fn_stmts RP2 S_COLON
                | CLS_VAR COLON LP2 decl_stmts RP2 S_COLON
                | STATIC_FN COLON LP2 decl_static_fn_stmts RP2 S_COLON
decl_fn_stmts : decl_fn_stmts decl_fn_stmt
              | decl_fn_stmt
decl_fn_stmt : fn_id LP1 pred_args RP1 S_COLON
             | fn_id LP1 pred_args RP1 R_ARROW unary_pred S_COLON
             | fn_id LP3 type_var_args RP3 LP1 pred_args RP1 S_COLON
             | fn_id LP3 type_var_args RP3 LP1 pred_args RP1 R_ARROW unary_pred S_COLON
decl_cls_fn_stmts : decl_cls_fn_stmts decl_cls_fn_stmt
                  | decl_cls_fn_stmt
decl_cls_fn_stmt : fn_id LP1 pred_args RP1 S_COLON
                 | fn_id LP1 pred_args RP1 R_ARROW unary_pred S_COLON
                 | fn_id LP3 type_var_args RP3 LP1 pred_args RP1 S_COLON
                 | fn_id LP3 type_var_args RP3 LP1 pred_args RP1 R_ARROW unary_pred S_COLON
fn_id : ID
pred_args : main_pred COMMA next_preds
          | main_pred
main_pred : ID
next_preds : next_anonymous_necessary_preds COMMA next_anonymous_opt_preds COMMA STAR COMMA next_named_preds
           | next_anonymous_necessary_preds COMMA next_anonymous_opt_preds COMMA STAR
           | next_anonymous_necessary_preds COMMA next_anonymous_opt_preds
           | next_anonymous_necessary_preds COMMA STAR COMMA next_named_preds
           | next_anonymous_necessary_preds COMMA STAR
           | next_anonymous_necessary_preds
           | next_anonymous_opt_preds COMMA STAR COMMA next_named_preds
           | next_anonymous_opt_preds COMMA STAR
           | next_anonymous_opt_preds
           | STAR next_named_preds
           | STAR
next_anonymous_necessary_preds : next_anonymous_necessary_preds COMMA next_anonymous_necessary_pred
                               | next_anonymous_necessary_pred
next_anonymous_necessary_pred : unary_pred
next_anonymous_opt_preds : next_anonymous_opt_preds COMMA next_anonymous_opt_pred
                         | next_anonymous_opt_pred
next_anonymous_opt_pred : ASSIGN unary_pred
next_named_preds : next_named_preds COMMA next_named_pred
                 | next_named_pred
next_named_pred : next_named_necessary_pred
                | next_named_opt_pred
next_named_necessary_pred : arg_name COLON unary_pred
next_named_opt_pred : arg_name COLON ASSIGN unary_pred
arg_name : ID
type_var_args : type_var_args COMMA type_var_arg
              | type_var_arg
type_var_arg : type_var_id
             | type_var_id COLON unary_pred
type_var_id : ID
unary_pred : pred_name
           | unnamed_pred
unnamed_pred : pred_name LP3 args RP3
             | LP3 pred_expr RP3
             | TRAIT_OF LP1 var_expr RP1
             | NONE
pred_name : ID
pred_expr : pred_expr OR pred_expr_a
          | pred_expr_a
pred_expr_a : pred_expr_a AND pred_expr_b
            | pred_expr_b
pred_expr_b : NOT pred_expr_c
            | pred_expr_c
pred_expr_c : unary_pred
            | LP1 pred_expr RP1
var_expr : unary_pred DOT member_var_name
member_var_name : ID
decl_stmts : decl_stmts decl_stmt
           | decl_stmt
decl_stmt : var_id S_COLON
          | var_id COLON unary_pred S_COLON
          | LP3 vars_id RP3 S_COLON
          | LP3 vars_id RP3 COLON unary_pred S_COLON
vars_id : vars_id COMMA var_id
        | var_id
var_id : ID
decl_static_fn_stmts : decl_static_fn_stmts decl_static_fn_stmt
                     | decl_static_fn_stmt
decl_static_fn_stmt : static_fn_id LP1 next_preds RP1 S_COLON
                    | static_fn_id LP1 next_preds RP1 R_ARROW unary_pred S_COLON
                    | static_fn_id LP3 type_var_args RP3 LP1 next_preds RP1 S_COLON
                    | static_fn_id LP3 type_var_args RP3 LP1 next_preds RP1 R_ARROW unary_pred S_COLON
                    | static_fn_id LP1 RP1 S_COLON
                    | static_fn_id LP1 RP1 R_ARROW unary_pred S_COLON
                    | static_fn_id LP3 type_var_args RP3 LP1 RP1 S_COLON
                    | static_fn_id LP3 type_var_args RP3 LP1 RP1 R_ARROW unary_pred S_COLON
static_fn_id : ID
print_stmt : PRINTINFO to_print S_COLON
generate_stmt : GENERATE to_print S_COLON
to_print : ID
         | boolean_expr
         | unnamed_pred
boolean_expr : boolean_expr OR boolean_expr_a
             | boolean_expr_a
boolean_expr_a : boolean_expr_a XOR boolean_expr_b
               | boolean_expr_b
boolean_expr_b : boolean_expr_b AND boolean_expr_c
               | boolean_expr_c
boolean_expr_c : boolean_expr_c EQ boolean_expr_d
               | boolean_expr_c NEQ boolean_expr_d
               | boolean_expr_d
boolean_expr_d : NOT boolean_expr_e
               | boolean_expr_e
boolean_expr_e : atomic_boolean_expr
               | LP1 boolean_expr RP1
atomic_boolean_expr : constants
                    | unary_pred LP1 args RP1
                    | unary_pred IMPLIES unary_pred
                    | LP1 unary_pred EQ unary_pred RP1
                    | LP1 unary_pred NEQ unary_pred RP1
constants : TRUE
          | FALSE
args : args COMMA pred_expr
     | pred_expr
assign_stmt : names ASSIGN assign_expr S_COLON
names : names COMMA name
      | name
name : ID
assign_expr : names ASSIGN assign_expr
            | names
"""
```

#### How to test the module

AST를 출력하기 위해 main.py를 실행합니다.

Parse Tree를 출력하기 위해 syntax/parser.py를 실행합니다.

이 프로그램은 현재 Parse Tree와 AST를 출력하는 기능까지 구현되어
있습니다.

#### preprocessor

python 문법을 c-like 문법으로 전처리하는 프로그램입니다. 이는 python
문법이 CFG로 표현하기 어렵기 때문입니다. 프로그램은 stack을 이용하여
세미콜론과 괄호를 자동으로 생성하며, 시간복잡도는 O(n)입니다.

python 문법에 따라 언제나 각 줄의 마지막에 세미콜론이 찍히는 것은
아님에 유의하세요. 예를 들어 다음과 같은 경우가 있습니다.

```python
x, y, z, w = [
    1, 2, 3, 4
] # -> 세미콜론은 맨 마지막 줄에만 찍혀야 함
```

#### lexer

lexical analysis를 실행합니다. 

부가적인 기능으로, 줄바꿈을 인식해 토큰에 넣어주는 기능과
공백을 인식한 뒤 버리는 기능이 있습니다. 줄바꿈을 인식하는
기능은 에러 메시지를 출력할 때 유용하게 사용됩니다.

#### parser

ply.yacc을 이용해 Parse Tree를 생성합니다.
Parse Tree는 수십 개의 클래스들로 이루어진 노드들로 구성되어 있고,
모든 노드는 ptnodes.py에 정의되어 있습니다.

#### semantic analysis

Parse Tree를 해석해 Abstract Syntax Tree를 생성합니다.
Parse Tree를 해석하는 방문자(visitor) 객체가 visitor.py에 정의되어
있습니다. Visitor는 Parse Tree에서 DFS를 수행하며 자기 자신의 상태를
바꾸어 나가면서 트리를 해석할 수 있습니다.

생성된 AST는 기존의 Parse Tree보다 해석하기 쉽습니다. 이는 토큰을
버리는 등의 여러 가지 해석 작업을 수행해 주기 때문입니다. 주요한
해석 작업으로는 tree를 flattening하는 것들이 있습니다. 예를 들어

```
stmts
== stmts
==== stmts
====== stmts
======== stmt
======== ;
====== ;
==== ;
== ;
```

위와 같은 Parse Tree는

```
stmts
== stmt
== ;
== stmt
== ;
== stmt
== ;
== stmt
== ;
```

와 같은 AST로 변환됩니다.

### English

A library that prints the abstract base class code of a python class.

#### Abstract Base Class

It's a concept similar to interface in Java. Python does not 
support the syntax for creating Interface, but Abstract base 
class allows you to create classes that act like Interface. 
Abstract base class cannot be instantiated by itself, and functions 
within the class are defined but not implemented. The 
responsibility for implementation lies with the class that 
inherited the Abstract Base Class. The reason for using 
abstract base class is known to provide the same interface 
to multiple classes with similar functions.

#### Inspired By

C++20, Concepts

```c++
template<typename T>
concept Hashable = requires(T a)
{
    { std::hash<T>{}(a) } -> std::convertible_to<std::size_t>;
};
```

Rust, Traits

```rust
trait Zero {
    const ZERO: Self;
    fn is_zero(&self) -> bool;
}

impl Zero for i32 {
    const ZERO: Self = 0;

    fn is_zero(&self) -> bool {
        *self == Self::ZERO
    }
}
```

Also learn about **Type Classes** of Haskell.

#### Why we use ABC?

1. You can take advantage of the powerful capabilities of IDE.

In dynamic typing languages, it is difficult to take advantage 
of IDE's powerful capabilities because IDE is difficult to get 
information about the type of variable. If x is an argument 
in a function, the IDE has no idea what function x has. 
Python 3.5's Type Annotation was introduced to solve this issue, 
but the problem is that it provides only a very limited amount 
of generic. For example, if you think of a function that takes 
x and returns x[0], the type of x might be list, tuple, dictionary, 
but not int. We need a name that can represent all of these types.
This problem can be solved in part through Python's typing module, 
but it is still limited. The goal of this project is to generate 
Python code that makes Python's Type Annotation more useful.

2. You can write more secure code.

You can write more secure code by specifying specific features 
that variables must implement. For Python, if an exception occurs 
because a function is not implemented, and there is a code inside 
that ignores the exception, debugging can be difficult. To prevent 
this situation, it is recommended to specify the function of the 
variable in advance.

#### Reserved Keywords

```
1. Trait

trait Foo(self):
   // ~~~~
```

Define the trait.

The reason why the name is trait is that there is no 'trait' in 
Python reserved words, and we borrowed the name of Rust's trait, 
but it does not implement all the features of it.

Because Rust's trace allows implementation within the trait 
statement, in fact, this feature itself is more like Java's interface.

```
2. fn / cls_fn / static_fn / var / cls_var

trait Foo(self):
    fn:
        bar(self, int, int) -> int
    cls_var:
        x: int

```

Defines functions within the trait.
The element inside () means the type of arguments of the function, and 
the element after -> means the return type.

```
3. extends

trait Foo(self) extends Bar:
    // ~~~~
```

Inherits the trait.

```
4. trait with multiple arguments

trait Addable(self, T):
    fn:
        __add__(self, T) -> T
        
int_addable = Addable[int]
```

You can also use multiple arguments to create a trait. 
The created trait can bind the type using the [] operator. 
In the example above, the expression Addable[int] puts int in 
the position of T, so int_addable refers to any type that can 
be expressed in x + (int).

The [] operator does not bind any factors to the first factor, self, 
but starts binding to the second factor. This follows the 
convention of the concept of C++20.

```
5. trait as a predicate

the expression 'int_addable(int)' returns 'True'. 
```

A predicte is a function that takes types or traits as arguments 
and returns True/False. It is mainly used for case-work in 
functional programming. If you put a type in a predicate in the 
form as a predicate(type), it returns whether the type satisfies 
the predicate. In this language, all type names and traits are 
available as a predicate. This is also in accordance with the 
convention that the concept of C++20 is also used as a predicte.

```
6. More about trait declarations

trait Addable(self, T):
    fn:
        __add__(self, T)

trait Foo(self):
    fn:
        bar(self, Addable[int], =float, *, 
            name1:Addable[int], name2:=float) -> int

```

Like this example, a trait can come to the definition of the 
function, and it supports Python's default and keyword arguments.

See Appendix for more examples.

#### Policy

The entire grammar consists of approximately 150 rules, excluding 
error handling, and according to ply.yacc(), it is a complete 
LALR(1) grammar.

```
"""
program : PROGRAM_BEGIN S_COLON stmts PROGRAM_END S_COLON
        | WS
stmts : stmts stmt
      | stmt
stmt : trait_decl
     | print_stmt
     | generate_stmt
     | assign_stmt
     | S_COLON
trait_decl : TRAIT trait_id LP1 class_args RP1 COLON LP2 trait_decl_stmts RP2 S_COLON
           | TRAIT trait_id LP1 class_args RP1 EXTENDS unary_pred COLON LP2 trait_decl_stmts RP2 S_COLON
trait_id : ID
class_args : main_arg COMMA sub_args
           | main_arg
main_arg : ID
sub_args : necessary_args COMMA optional_args
         | necessary_args
         | optional_args
necessary_args : necessary_args COMMA necessary_arg
               | necessary_arg
necessary_arg : ID
optional_args : optional_args COMMA optional_arg
              | optional_arg
optional_arg : ID ASSIGN unary_pred
trait_decl_stmts : trait_decl_stmts trait_decl_stmt
                 | trait_decl_stmt
                 | PASS S_COLON
trait_decl_stmt : FN COLON LP2 decl_fn_stmts RP2 S_COLON
                | VAR COLON LP2 decl_stmts RP2 S_COLON
                | CLS_FN COLON LP2 decl_cls_fn_stmts RP2 S_COLON
                | CLS_VAR COLON LP2 decl_stmts RP2 S_COLON
                | STATIC_FN COLON LP2 decl_static_fn_stmts RP2 S_COLON
decl_fn_stmts : decl_fn_stmts decl_fn_stmt
              | decl_fn_stmt
decl_fn_stmt : fn_id LP1 pred_args RP1 S_COLON
             | fn_id LP1 pred_args RP1 R_ARROW unary_pred S_COLON
             | fn_id LP3 type_var_args RP3 LP1 pred_args RP1 S_COLON
             | fn_id LP3 type_var_args RP3 LP1 pred_args RP1 R_ARROW unary_pred S_COLON
decl_cls_fn_stmts : decl_cls_fn_stmts decl_cls_fn_stmt
                  | decl_cls_fn_stmt
decl_cls_fn_stmt : fn_id LP1 pred_args RP1 S_COLON
                 | fn_id LP1 pred_args RP1 R_ARROW unary_pred S_COLON
                 | fn_id LP3 type_var_args RP3 LP1 pred_args RP1 S_COLON
                 | fn_id LP3 type_var_args RP3 LP1 pred_args RP1 R_ARROW unary_pred S_COLON
fn_id : ID
pred_args : main_pred COMMA next_preds
          | main_pred
main_pred : ID
next_preds : next_anonymous_necessary_preds COMMA next_anonymous_opt_preds COMMA STAR COMMA next_named_preds
           | next_anonymous_necessary_preds COMMA next_anonymous_opt_preds COMMA STAR
           | next_anonymous_necessary_preds COMMA next_anonymous_opt_preds
           | next_anonymous_necessary_preds COMMA STAR COMMA next_named_preds
           | next_anonymous_necessary_preds COMMA STAR
           | next_anonymous_necessary_preds
           | next_anonymous_opt_preds COMMA STAR COMMA next_named_preds
           | next_anonymous_opt_preds COMMA STAR
           | next_anonymous_opt_preds
           | STAR next_named_preds
           | STAR
next_anonymous_necessary_preds : next_anonymous_necessary_preds COMMA next_anonymous_necessary_pred
                               | next_anonymous_necessary_pred
next_anonymous_necessary_pred : unary_pred
next_anonymous_opt_preds : next_anonymous_opt_preds COMMA next_anonymous_opt_pred
                         | next_anonymous_opt_pred
next_anonymous_opt_pred : ASSIGN unary_pred
next_named_preds : next_named_preds COMMA next_named_pred
                 | next_named_pred
next_named_pred : next_named_necessary_pred
                | next_named_opt_pred
next_named_necessary_pred : arg_name COLON unary_pred
next_named_opt_pred : arg_name COLON ASSIGN unary_pred
arg_name : ID
type_var_args : type_var_args COMMA type_var_arg
              | type_var_arg
type_var_arg : type_var_id
             | type_var_id COLON unary_pred
type_var_id : ID
unary_pred : pred_name
           | unnamed_pred
unnamed_pred : pred_name LP3 args RP3
             | LP3 pred_expr RP3
             | TRAIT_OF LP1 var_expr RP1
             | NONE
pred_name : ID
pred_expr : pred_expr OR pred_expr_a
          | pred_expr_a
pred_expr_a : pred_expr_a AND pred_expr_b
            | pred_expr_b
pred_expr_b : NOT pred_expr_c
            | pred_expr_c
pred_expr_c : unary_pred
            | LP1 pred_expr RP1
var_expr : unary_pred DOT member_var_name
member_var_name : ID
decl_stmts : decl_stmts decl_stmt
           | decl_stmt
decl_stmt : var_id S_COLON
          | var_id COLON unary_pred S_COLON
          | LP3 vars_id RP3 S_COLON
          | LP3 vars_id RP3 COLON unary_pred S_COLON
vars_id : vars_id COMMA var_id
        | var_id
var_id : ID
decl_static_fn_stmts : decl_static_fn_stmts decl_static_fn_stmt
                     | decl_static_fn_stmt
decl_static_fn_stmt : static_fn_id LP1 next_preds RP1 S_COLON
                    | static_fn_id LP1 next_preds RP1 R_ARROW unary_pred S_COLON
                    | static_fn_id LP3 type_var_args RP3 LP1 next_preds RP1 S_COLON
                    | static_fn_id LP3 type_var_args RP3 LP1 next_preds RP1 R_ARROW unary_pred S_COLON
                    | static_fn_id LP1 RP1 S_COLON
                    | static_fn_id LP1 RP1 R_ARROW unary_pred S_COLON
                    | static_fn_id LP3 type_var_args RP3 LP1 RP1 S_COLON
                    | static_fn_id LP3 type_var_args RP3 LP1 RP1 R_ARROW unary_pred S_COLON
static_fn_id : ID
print_stmt : PRINTINFO to_print S_COLON
generate_stmt : GENERATE to_print S_COLON
to_print : ID
         | boolean_expr
         | unnamed_pred
boolean_expr : boolean_expr OR boolean_expr_a
             | boolean_expr_a
boolean_expr_a : boolean_expr_a XOR boolean_expr_b
               | boolean_expr_b
boolean_expr_b : boolean_expr_b AND boolean_expr_c
               | boolean_expr_c
boolean_expr_c : boolean_expr_c EQ boolean_expr_d
               | boolean_expr_c NEQ boolean_expr_d
               | boolean_expr_d
boolean_expr_d : NOT boolean_expr_e
               | boolean_expr_e
boolean_expr_e : atomic_boolean_expr
               | LP1 boolean_expr RP1
atomic_boolean_expr : constants
                    | unary_pred LP1 args RP1
                    | unary_pred IMPLIES unary_pred
                    | LP1 unary_pred EQ unary_pred RP1
                    | LP1 unary_pred NEQ unary_pred RP1
constants : TRUE
          | FALSE
args : args COMMA pred_expr
     | pred_expr
assign_stmt : names ASSIGN assign_expr S_COLON
names : names COMMA name
      | name
name : ID
assign_expr : names ASSIGN assign_expr
            | names
"""
```

#### How to test the module

Run main.py to print the abstract syntax tree.

Run syntax/parser.py to print the parse tree.

The program now has the ability to output Parse Tree and AST.

#### preprocessor

It is a program that preprocesss Python grammar into c-like grammar. 
This is because Python grammar is hard to express in CFG.
The program automatically generates semicolons and brackets 
using stacks, and the time complexity is O(n).

Note that according to python grammar, a semicolon is not always 
taken at the end of each line. For example:

```python
x, y, z, w = [
    1, 2, 3, 4
] # -> The semicolon should be only here.
```

#### lexer

Do the lexical analysis.

As additional functions, there is a function of recognizing a line 
change and putting it into a token, and a function of recognizing 
and discarding whitespaces. The ability to recognize line breaks 
is useful when outputting error messages.

#### parser

Create a Parse Tree using ply.yacc.
Parse Tree consists of nodes of dozens of classes, all of
which are defined at ptnodes.py.

#### semantic analysis

Analyze the parse tree to create an abstract ayntax tree.
A visitor object interpreting the parse tree is defined at 
visitor.py. The Visitor performs DFS in the Parse Tree and 
can interpret the tree by changing its own state.

The generated AST is easier to interpret than the existing 
Parse Tree. This is because it performs various interpretations,
such as discarding tokens. The main interpretation tasks are 
flattening the tree. For example:

```
stmts
== stmts
==== stmts
====== stmts
======== stmt
======== ;
====== ;
==== ;
== ;
```

The parse tree above is converted to the following AST.

```
stmts
== stmt
== ;
== stmt
== ;
== stmt
== ;
== stmt
== ;
```

### Appendix

#### INPUT

```

# 1. trait definition
# the following trait definition says that
# typename 'self' should support .append(x) and .pop() operation
# and __len__ method

trait any(self):
    pass

trait StackLike(self):
    fn:
        append(self, any) -> [self or None]
        pop(self) -> [self or None]
        popop(self) -> [self or (None and None)]

    fn:
        __len__(self) -> int


# the following trait definition says that
# the instance of the class 'self' should have 'x', 'y' member variables

trait PointLike(self):
    var:
        x: int
        y: int


trait AnyRandom(self):
    var:
        x: int
        y: int
        z
    fn:
        __init__(self, =int, =int, =any) -> None

    cls_fn:
        foo(cls, int) -> str
        foo(cls, float)

    cls_var:
        xxx
        yyy: int

    static_fn:
        bar() -> None

printinfo trait_of(AnyRandom.x)


t_random = any_random


trait decl_var_with_comma(self):
    var:
        [x, y, z]: int


# 2. traits with 2 or more parameters.

trait Addable(self, T):
    fn:
        __add__(self, T) -> self

trait Multipliable(self, T):
    fn:
        __mul__(self, T) -> self


# 3. trait(type) expression is evaluated as a boolean.

printinfo StackLike(int) # -> False
printinfo StackLike(list) # -> True

printinfo Addable(str, int) # -> False
printinfo Multipliable(str, int) # -> True, since "str" * 3 is a valid expression


# 4. If you want to bind some arguments to a trait, use trait[]() expression.

printinfo Addable[int](str) # -> False, exactly same meaning with Addable(str, int)


# 6. trait extensions.

trait AddableSubable(self, T) extends Addable[T]:
    fn:
        __sub__(self, T)


# 7. printinfo operator.

trait Fooable(self):
    fn:
        foo(self, any)

printinfo Fooable

# possible output:
# """
# trait Fooable supports:
# member function .foo(any)
# """

# 8. trait and/or/not operator.

trait FooBarable(self):
    fn:
        foo(self, any)
        bar(self, any)

trait BarBazable(self):
    fn:
        bar(self, any)
        baz(self, any)

printinfo [FooBarable and BarBazable]

# possible output:
# """
# trait (unnamed @ line 88) supports:
# member function .foo(any)
# member function .bar(any)
# member function .baz(any)
# """

printinfo [FooBarable or BarBazable]

# possible output:
# """
# trait (unnamed @ line 98) supports:
# member function .bar(any)
# """


# 11. generate operator.

generate Fooo
generate FooBarable


# 12. implies operator.

printinfo [FooBarable and BarBazable] implies [FooBarable or BarBazable]

# 13. assignments.

a, b = b, a

# 14. default arguments.

trait DefaultArgsExample(self, arg_a, arg_b, arg_c=[Addable and Subable], arg_d=float):
    pass

trait TraitWithAllOptionalArgs(self, arg_a=int, arg_b=str):
    fn:
        whatever(self, int, int, =int, =int, *, a:=int, b:int)


# 15. traits with generic type variable.
trait GenericTrait(self):
    fn:
        foo[T](self, T) -> T


trait vector7D(self) extends vector[vector[vector[vector[vector[vector[vector[int]]]]]]]:
    fn:
        begin(self) -> iterator[vector[any] or vector[vector[any] or any] or vector[vector[vector[any]]]]
        end(self) -> iterator[int or [trait_of(vector.a)] and trait_of([None or vector5D[v[u], v[u]]].y)]
        front(self, =[int or (str and not float or vector[int])])

printinfo (trait_of(x.z) implies trait_of(y.z)) and (True == False or False != int(int))


```

#### OUTPUT (Parse Tree)

```

 program
-- PROGRAM_BEGIN
-- ;
== stmts
==== stmts
====== stmts
======== stmts
========== stmts
============ stmts
============== stmts
================ stmts
================== stmts
==================== stmts
====================== stmts
======================== stmts
========================== stmts
============================ stmts
============================== stmts
================================ stmts
================================== stmts
==================================== stmts
====================================== stmts
======================================== stmts
========================================== stmts
============================================ stmts
============================================== stmts
================================================ stmts
================================================== stmts
==================================================== stmts
====================================================== stmts
======================================================== stmts
========================================================== stmts
============================================================ stmts
============================================================== stmt
================================================================ trait_decl
------------------------------------------------------------------ trait_id
================================================================== trait_id
-------------------------------------------------------------------- any
------------------------------------------------------------------ (
================================================================== class_args
==================================================================== main_arg
---------------------------------------------------------------------- self
------------------------------------------------------------------ )
------------------------------------------------------------------ :
------------------------------------------------------------------ {
================================================================== trait_decl_stmts
-------------------------------------------------------------------- pass
-------------------------------------------------------------------- ;
------------------------------------------------------------------ }
------------------------------------------------------------------ ;
============================================================ stmt
============================================================== trait_decl
---------------------------------------------------------------- trait_id
================================================================ trait_id
------------------------------------------------------------------ StackLike
---------------------------------------------------------------- (
================================================================ class_args
================================================================== main_arg
-------------------------------------------------------------------- self
---------------------------------------------------------------- )
---------------------------------------------------------------- :
---------------------------------------------------------------- {
================================================================ trait_decl_stmts
================================================================== trait_decl_stmts
==================================================================== trait_decl_stmt
---------------------------------------------------------------------- fn
---------------------------------------------------------------------- :
---------------------------------------------------------------------- {
====================================================================== decl_fn_stmts
======================================================================== decl_fn_stmts
========================================================================== decl_fn_stmts
============================================================================ decl_fn_stmt
============================================================================== fn_id
-------------------------------------------------------------------------------- append
------------------------------------------------------------------------------ (
============================================================================== pred_args
================================================================================ main_pred
---------------------------------------------------------------------------------- self
-------------------------------------------------------------------------------- ,
================================================================================ next_preds
================================================================================== next_anonymous_necessary_preds
==================================================================================== next_anonymous_necessary_pred
====================================================================================== unary_pred
======================================================================================== pred_name
------------------------------------------------------------------------------------------ any
------------------------------------------------------------------------------ )
------------------------------------------------------------------------------ ->
============================================================================== unary_pred
================================================================================ unnamed_pred
---------------------------------------------------------------------------------- [
================================================================================== pred_expr
==================================================================================== pred_expr
====================================================================================== pred_expr_a
======================================================================================== pred_expr_b
========================================================================================== pred_expr_c
============================================================================================ unary_pred
============================================================================================== pred_name
------------------------------------------------------------------------------------------------ self
------------------------------------------------------------------------------------ or
==================================================================================== pred_expr_a
====================================================================================== pred_expr_b
======================================================================================== pred_expr_c
========================================================================================== unary_pred
============================================================================================ unnamed_pred
---------------------------------------------------------------------------------------------- None
---------------------------------------------------------------------------------- ]
------------------------------------------------------------------------------ ;
========================================================================== decl_fn_stmt
============================================================================ fn_id
------------------------------------------------------------------------------ pop
---------------------------------------------------------------------------- (
============================================================================ pred_args
============================================================================== main_pred
-------------------------------------------------------------------------------- self
---------------------------------------------------------------------------- )
---------------------------------------------------------------------------- ->
============================================================================ unary_pred
============================================================================== unnamed_pred
-------------------------------------------------------------------------------- [
================================================================================ pred_expr
================================================================================== pred_expr
==================================================================================== pred_expr_a
====================================================================================== pred_expr_b
======================================================================================== pred_expr_c
========================================================================================== unary_pred
============================================================================================ pred_name
---------------------------------------------------------------------------------------------- self
---------------------------------------------------------------------------------- or
================================================================================== pred_expr_a
==================================================================================== pred_expr_b
====================================================================================== pred_expr_c
======================================================================================== unary_pred
========================================================================================== unnamed_pred
-------------------------------------------------------------------------------------------- None
-------------------------------------------------------------------------------- ]
---------------------------------------------------------------------------- ;
======================================================================== decl_fn_stmt
========================================================================== fn_id
---------------------------------------------------------------------------- popop
-------------------------------------------------------------------------- (
========================================================================== pred_args
============================================================================ main_pred
------------------------------------------------------------------------------ self
-------------------------------------------------------------------------- )
-------------------------------------------------------------------------- ->
========================================================================== unary_pred
============================================================================ unnamed_pred
------------------------------------------------------------------------------ [
============================================================================== pred_expr
================================================================================ pred_expr
================================================================================== pred_expr_a
==================================================================================== pred_expr_b
====================================================================================== pred_expr_c
======================================================================================== unary_pred
========================================================================================== pred_name
-------------------------------------------------------------------------------------------- self
-------------------------------------------------------------------------------- or
================================================================================ pred_expr_a
================================================================================== pred_expr_b
==================================================================================== pred_expr_c
-------------------------------------------------------------------------------------- (
====================================================================================== pred_expr
======================================================================================== pred_expr_a
========================================================================================== pred_expr_a
============================================================================================ pred_expr_b
============================================================================================== pred_expr_c
================================================================================================ unary_pred
================================================================================================== unnamed_pred
---------------------------------------------------------------------------------------------------- None
------------------------------------------------------------------------------------------ and
========================================================================================== pred_expr_b
============================================================================================ pred_expr_c
============================================================================================== unary_pred
================================================================================================ unnamed_pred
-------------------------------------------------------------------------------------------------- None
-------------------------------------------------------------------------------------- )
------------------------------------------------------------------------------ ]
-------------------------------------------------------------------------- ;
---------------------------------------------------------------------- }
---------------------------------------------------------------------- ;
================================================================== trait_decl_stmt
-------------------------------------------------------------------- fn
-------------------------------------------------------------------- :
-------------------------------------------------------------------- {
==================================================================== decl_fn_stmts
====================================================================== decl_fn_stmt
======================================================================== fn_id
-------------------------------------------------------------------------- __len__
------------------------------------------------------------------------ (
======================================================================== pred_args
========================================================================== main_pred
---------------------------------------------------------------------------- self
------------------------------------------------------------------------ )
------------------------------------------------------------------------ ->
======================================================================== unary_pred
========================================================================== pred_name
---------------------------------------------------------------------------- int
------------------------------------------------------------------------ ;
-------------------------------------------------------------------- }
-------------------------------------------------------------------- ;
---------------------------------------------------------------- }
---------------------------------------------------------------- ;
========================================================== stmt
============================================================ trait_decl
-------------------------------------------------------------- trait_id
============================================================== trait_id
---------------------------------------------------------------- PointLike
-------------------------------------------------------------- (
============================================================== class_args
================================================================ main_arg
------------------------------------------------------------------ self
-------------------------------------------------------------- )
-------------------------------------------------------------- :
-------------------------------------------------------------- {
============================================================== trait_decl_stmts
================================================================ trait_decl_stmt
------------------------------------------------------------------ var
------------------------------------------------------------------ :
------------------------------------------------------------------ {
================================================================== decl_stmts
==================================================================== decl_stmts
====================================================================== decl_stmt
======================================================================== var_id
-------------------------------------------------------------------------- x
------------------------------------------------------------------------ :
======================================================================== unary_pred
========================================================================== pred_name
---------------------------------------------------------------------------- int
------------------------------------------------------------------------ ;
==================================================================== decl_stmt
====================================================================== var_id
------------------------------------------------------------------------ y
---------------------------------------------------------------------- :
====================================================================== unary_pred
======================================================================== pred_name
-------------------------------------------------------------------------- int
---------------------------------------------------------------------- ;
------------------------------------------------------------------ }
------------------------------------------------------------------ ;
-------------------------------------------------------------- }
-------------------------------------------------------------- ;
======================================================== stmt
========================================================== trait_decl
------------------------------------------------------------ trait_id
============================================================ trait_id
-------------------------------------------------------------- AnyRandom
------------------------------------------------------------ (
============================================================ class_args
============================================================== main_arg
---------------------------------------------------------------- self
------------------------------------------------------------ )
------------------------------------------------------------ :
------------------------------------------------------------ {
============================================================ trait_decl_stmts
============================================================== trait_decl_stmts
================================================================ trait_decl_stmts
================================================================== trait_decl_stmts
==================================================================== trait_decl_stmts
====================================================================== trait_decl_stmt
------------------------------------------------------------------------ var
------------------------------------------------------------------------ :
------------------------------------------------------------------------ {
======================================================================== decl_stmts
========================================================================== decl_stmts
============================================================================ decl_stmts
============================================================================== decl_stmt
================================================================================ var_id
---------------------------------------------------------------------------------- x
-------------------------------------------------------------------------------- :
================================================================================ unary_pred
================================================================================== pred_name
------------------------------------------------------------------------------------ int
-------------------------------------------------------------------------------- ;
============================================================================ decl_stmt
============================================================================== var_id
-------------------------------------------------------------------------------- y
------------------------------------------------------------------------------ :
============================================================================== unary_pred
================================================================================ pred_name
---------------------------------------------------------------------------------- int
------------------------------------------------------------------------------ ;
========================================================================== decl_stmt
============================================================================ var_id
------------------------------------------------------------------------------ z
---------------------------------------------------------------------------- ;
------------------------------------------------------------------------ }
------------------------------------------------------------------------ ;
==================================================================== trait_decl_stmt
---------------------------------------------------------------------- fn
---------------------------------------------------------------------- :
---------------------------------------------------------------------- {
====================================================================== decl_fn_stmts
======================================================================== decl_fn_stmt
========================================================================== fn_id
---------------------------------------------------------------------------- __init__
-------------------------------------------------------------------------- (
========================================================================== pred_args
============================================================================ main_pred
------------------------------------------------------------------------------ self
---------------------------------------------------------------------------- ,
============================================================================ next_preds
------------------------------------------------------------------------------ ,
============================================================================== next_anonymous_optional_preds
================================================================================ next_anonymous_optional_preds
================================================================================== next_anonymous_optional_preds
==================================================================================== next_anonymous_optional_pred
-------------------------------------------------------------------------------------- =
====================================================================================== unary_pred
======================================================================================== pred_name
------------------------------------------------------------------------------------------ int
---------------------------------------------------------------------------------- ,
================================================================================== next_anonymous_optional_pred
------------------------------------------------------------------------------------ =
==================================================================================== unary_pred
====================================================================================== pred_name
---------------------------------------------------------------------------------------- int
-------------------------------------------------------------------------------- ,
================================================================================ next_anonymous_optional_pred
---------------------------------------------------------------------------------- =
================================================================================== unary_pred
==================================================================================== pred_name
-------------------------------------------------------------------------------------- any
-------------------------------------------------------------------------- )
-------------------------------------------------------------------------- ->
========================================================================== unary_pred
============================================================================ unnamed_pred
------------------------------------------------------------------------------ None
-------------------------------------------------------------------------- ;
---------------------------------------------------------------------- }
---------------------------------------------------------------------- ;
================================================================== trait_decl_stmt
-------------------------------------------------------------------- cls_fn
-------------------------------------------------------------------- :
-------------------------------------------------------------------- {
==================================================================== decl_cls_fn_stmts
====================================================================== decl_cls_fn_stmts
======================================================================== decl_cls_fn_stmt
========================================================================== fn_id
---------------------------------------------------------------------------- foo
-------------------------------------------------------------------------- (
========================================================================== pred_args
============================================================================ main_pred
------------------------------------------------------------------------------ cls
---------------------------------------------------------------------------- ,
============================================================================ next_preds
============================================================================== next_anonymous_necessary_preds
================================================================================ next_anonymous_necessary_pred
================================================================================== unary_pred
==================================================================================== pred_name
-------------------------------------------------------------------------------------- int
-------------------------------------------------------------------------- )
-------------------------------------------------------------------------- ->
========================================================================== unary_pred
============================================================================ pred_name
------------------------------------------------------------------------------ str
-------------------------------------------------------------------------- ;
====================================================================== decl_cls_fn_stmt
======================================================================== fn_id
-------------------------------------------------------------------------- foo
------------------------------------------------------------------------ (
======================================================================== pred_args
========================================================================== main_pred
---------------------------------------------------------------------------- cls
-------------------------------------------------------------------------- ,
========================================================================== next_preds
============================================================================ next_anonymous_necessary_preds
============================================================================== next_anonymous_necessary_pred
================================================================================ unary_pred
================================================================================== pred_name
------------------------------------------------------------------------------------ float
------------------------------------------------------------------------ )
------------------------------------------------------------------------ ;
-------------------------------------------------------------------- }
-------------------------------------------------------------------- ;
================================================================ trait_decl_stmt
------------------------------------------------------------------ cls_var
------------------------------------------------------------------ :
------------------------------------------------------------------ {
================================================================== decl_stmts
==================================================================== decl_stmts
====================================================================== decl_stmt
======================================================================== var_id
-------------------------------------------------------------------------- xxx
------------------------------------------------------------------------ ;
==================================================================== decl_stmt
====================================================================== var_id
------------------------------------------------------------------------ yyy
---------------------------------------------------------------------- :
====================================================================== unary_pred
======================================================================== pred_name
-------------------------------------------------------------------------- int
---------------------------------------------------------------------- ;
------------------------------------------------------------------ }
------------------------------------------------------------------ ;
============================================================== trait_decl_stmt
---------------------------------------------------------------- static_fn
---------------------------------------------------------------- :
---------------------------------------------------------------- {
================================================================ decl_static_fn_stmts
================================================================== decl_static_fn_stmt
==================================================================== static_fn_id
---------------------------------------------------------------------- bar
-------------------------------------------------------------------- (
-------------------------------------------------------------------- )
-------------------------------------------------------------------- ->
==================================================================== unary_pred
====================================================================== unnamed_pred
------------------------------------------------------------------------ None
-------------------------------------------------------------------- ;
---------------------------------------------------------------- }
---------------------------------------------------------------- ;
------------------------------------------------------------ }
------------------------------------------------------------ ;
====================================================== stmt
======================================================== print_stmt
---------------------------------------------------------- printinfo
========================================================== to_print
============================================================ unnamed_pred
-------------------------------------------------------------- trait_of
-------------------------------------------------------------- (
============================================================== var_expr
================================================================ unary_pred
================================================================== pred_name
-------------------------------------------------------------------- AnyRandom
---------------------------------------------------------------- .
================================================================ member_var_name
------------------------------------------------------------------ x
-------------------------------------------------------------- )
---------------------------------------------------------- ;
==================================================== stmt
====================================================== assign_stmt
======================================================== names
========================================================== name
------------------------------------------------------------ t_random
-------------------------------------------------------- =
======================================================== assign_expr
========================================================== names
============================================================ name
-------------------------------------------------------------- any_random
-------------------------------------------------------- ;
================================================== stmt
==================================================== trait_decl
------------------------------------------------------ trait_id
====================================================== trait_id
-------------------------------------------------------- decl_var_with_comma
------------------------------------------------------ (
====================================================== class_args
======================================================== main_arg
---------------------------------------------------------- self
------------------------------------------------------ )
------------------------------------------------------ :
------------------------------------------------------ {
====================================================== trait_decl_stmts
======================================================== trait_decl_stmt
---------------------------------------------------------- var
---------------------------------------------------------- :
---------------------------------------------------------- {
========================================================== decl_stmts
============================================================ decl_stmt
-------------------------------------------------------------- [
============================================================== vars_id
================================================================ vars_id
================================================================== vars_id
==================================================================== var_id
---------------------------------------------------------------------- x
------------------------------------------------------------------ ,
================================================================== var_id
-------------------------------------------------------------------- y
---------------------------------------------------------------- ,
================================================================ var_id
------------------------------------------------------------------ z
-------------------------------------------------------------- ]
-------------------------------------------------------------- :
============================================================== unary_pred
================================================================ pred_name
------------------------------------------------------------------ int
-------------------------------------------------------------- ;
---------------------------------------------------------- }
---------------------------------------------------------- ;
------------------------------------------------------ }
------------------------------------------------------ ;
================================================ stmt
================================================== trait_decl
---------------------------------------------------- trait_id
==================================================== trait_id
------------------------------------------------------ Addable
---------------------------------------------------- (
==================================================== class_args
====================================================== main_arg
-------------------------------------------------------- self
------------------------------------------------------ ,
====================================================== sub_args
======================================================== necessary_args
========================================================== necessary_arg
------------------------------------------------------------ T
---------------------------------------------------- )
---------------------------------------------------- :
---------------------------------------------------- {
==================================================== trait_decl_stmts
====================================================== trait_decl_stmt
-------------------------------------------------------- fn
-------------------------------------------------------- :
-------------------------------------------------------- {
======================================================== decl_fn_stmts
========================================================== decl_fn_stmt
============================================================ fn_id
-------------------------------------------------------------- __add__
------------------------------------------------------------ (
============================================================ pred_args
============================================================== main_pred
---------------------------------------------------------------- self
-------------------------------------------------------------- ,
============================================================== next_preds
================================================================ next_anonymous_necessary_preds
================================================================== next_anonymous_necessary_pred
==================================================================== unary_pred
====================================================================== pred_name
------------------------------------------------------------------------ T
------------------------------------------------------------ )
------------------------------------------------------------ ->
============================================================ unary_pred
============================================================== pred_name
---------------------------------------------------------------- self
------------------------------------------------------------ ;
-------------------------------------------------------- }
-------------------------------------------------------- ;
---------------------------------------------------- }
---------------------------------------------------- ;
============================================== stmt
================================================ trait_decl
-------------------------------------------------- trait_id
================================================== trait_id
---------------------------------------------------- Multipliable
-------------------------------------------------- (
================================================== class_args
==================================================== main_arg
------------------------------------------------------ self
---------------------------------------------------- ,
==================================================== sub_args
====================================================== necessary_args
======================================================== necessary_arg
---------------------------------------------------------- T
-------------------------------------------------- )
-------------------------------------------------- :
-------------------------------------------------- {
================================================== trait_decl_stmts
==================================================== trait_decl_stmt
------------------------------------------------------ fn
------------------------------------------------------ :
------------------------------------------------------ {
====================================================== decl_fn_stmts
======================================================== decl_fn_stmt
========================================================== fn_id
------------------------------------------------------------ __mul__
---------------------------------------------------------- (
========================================================== pred_args
============================================================ main_pred
-------------------------------------------------------------- self
------------------------------------------------------------ ,
============================================================ next_preds
============================================================== next_anonymous_necessary_preds
================================================================ next_anonymous_necessary_pred
================================================================== unary_pred
==================================================================== pred_name
---------------------------------------------------------------------- T
---------------------------------------------------------- )
---------------------------------------------------------- ->
========================================================== unary_pred
============================================================ pred_name
-------------------------------------------------------------- self
---------------------------------------------------------- ;
------------------------------------------------------ }
------------------------------------------------------ ;
-------------------------------------------------- }
-------------------------------------------------- ;
============================================ stmt
============================================== print_stmt
------------------------------------------------ printinfo
================================================ to_print
================================================== boolean_expr
==================================================== boolean_expr_a
====================================================== boolean_expr_b
======================================================== boolean_expr_c
========================================================== boolean_expr_d
============================================================ boolean_expr_e
============================================================== atomic_boolean_expr
================================================================ unary_pred
================================================================== pred_name
-------------------------------------------------------------------- StackLike
---------------------------------------------------------------- (
================================================================ args
================================================================== pred_expr
==================================================================== pred_expr_a
====================================================================== pred_expr_b
======================================================================== pred_expr_c
========================================================================== unary_pred
============================================================================ pred_name
------------------------------------------------------------------------------ int
---------------------------------------------------------------- )
------------------------------------------------ ;
========================================== stmt
============================================ print_stmt
---------------------------------------------- printinfo
============================================== to_print
================================================ boolean_expr
================================================== boolean_expr_a
==================================================== boolean_expr_b
====================================================== boolean_expr_c
======================================================== boolean_expr_d
========================================================== boolean_expr_e
============================================================ atomic_boolean_expr
============================================================== unary_pred
================================================================ pred_name
------------------------------------------------------------------ StackLike
-------------------------------------------------------------- (
============================================================== args
================================================================ pred_expr
================================================================== pred_expr_a
==================================================================== pred_expr_b
====================================================================== pred_expr_c
======================================================================== unary_pred
========================================================================== pred_name
---------------------------------------------------------------------------- list
-------------------------------------------------------------- )
---------------------------------------------- ;
======================================== stmt
========================================== print_stmt
-------------------------------------------- printinfo
============================================ to_print
============================================== boolean_expr
================================================ boolean_expr_a
================================================== boolean_expr_b
==================================================== boolean_expr_c
====================================================== boolean_expr_d
======================================================== boolean_expr_e
========================================================== atomic_boolean_expr
============================================================ unary_pred
============================================================== pred_name
---------------------------------------------------------------- Addable
------------------------------------------------------------ (
============================================================ args
============================================================== args
================================================================ pred_expr
================================================================== pred_expr_a
==================================================================== pred_expr_b
====================================================================== pred_expr_c
======================================================================== unary_pred
========================================================================== pred_name
---------------------------------------------------------------------------- str
-------------------------------------------------------------- ,
============================================================== pred_expr
================================================================ pred_expr_a
================================================================== pred_expr_b
==================================================================== pred_expr_c
====================================================================== unary_pred
======================================================================== pred_name
-------------------------------------------------------------------------- int
------------------------------------------------------------ )
-------------------------------------------- ;
====================================== stmt
======================================== print_stmt
------------------------------------------ printinfo
========================================== to_print
============================================ boolean_expr
============================================== boolean_expr_a
================================================ boolean_expr_b
================================================== boolean_expr_c
==================================================== boolean_expr_d
====================================================== boolean_expr_e
======================================================== atomic_boolean_expr
========================================================== unary_pred
============================================================ pred_name
-------------------------------------------------------------- Multipliable
---------------------------------------------------------- (
========================================================== args
============================================================ args
============================================================== pred_expr
================================================================ pred_expr_a
================================================================== pred_expr_b
==================================================================== pred_expr_c
====================================================================== unary_pred
======================================================================== pred_name
-------------------------------------------------------------------------- str
------------------------------------------------------------ ,
============================================================ pred_expr
============================================================== pred_expr_a
================================================================ pred_expr_b
================================================================== pred_expr_c
==================================================================== unary_pred
====================================================================== pred_name
------------------------------------------------------------------------ int
---------------------------------------------------------- )
------------------------------------------ ;
==================================== stmt
====================================== print_stmt
---------------------------------------- printinfo
======================================== to_print
========================================== boolean_expr
============================================ boolean_expr_a
============================================== boolean_expr_b
================================================ boolean_expr_c
================================================== boolean_expr_d
==================================================== boolean_expr_e
====================================================== atomic_boolean_expr
======================================================== unary_pred
========================================================== unnamed_pred
============================================================ pred_name
-------------------------------------------------------------- Addable
------------------------------------------------------------ [
============================================================ args
============================================================== pred_expr
================================================================ pred_expr_a
================================================================== pred_expr_b
==================================================================== pred_expr_c
====================================================================== unary_pred
======================================================================== pred_name
-------------------------------------------------------------------------- int
------------------------------------------------------------ ]
-------------------------------------------------------- (
======================================================== args
========================================================== pred_expr
============================================================ pred_expr_a
============================================================== pred_expr_b
================================================================ pred_expr_c
================================================================== unary_pred
==================================================================== pred_name
---------------------------------------------------------------------- str
-------------------------------------------------------- )
---------------------------------------- ;
================================== stmt
==================================== trait_decl
-------------------------------------- trait_id
====================================== trait_id
---------------------------------------- AddableSubable
-------------------------------------- (
====================================== class_args
======================================== main_arg
------------------------------------------ self
---------------------------------------- ,
======================================== sub_args
========================================== necessary_args
============================================ necessary_arg
---------------------------------------------- T
-------------------------------------- )
-------------------------------------- extends
====================================== unary_pred
======================================== unnamed_pred
========================================== pred_name
-------------------------------------------- Addable
------------------------------------------ [
========================================== args
============================================ pred_expr
============================================== pred_expr_a
================================================ pred_expr_b
================================================== pred_expr_c
==================================================== unary_pred
====================================================== pred_name
-------------------------------------------------------- T
------------------------------------------ ]
-------------------------------------- :
-------------------------------------- {
====================================== trait_decl_stmts
======================================== trait_decl_stmt
------------------------------------------ fn
------------------------------------------ :
------------------------------------------ {
========================================== decl_fn_stmts
============================================ decl_fn_stmt
============================================== fn_id
------------------------------------------------ __sub__
---------------------------------------------- (
============================================== pred_args
================================================ main_pred
-------------------------------------------------- self
------------------------------------------------ ,
================================================ next_preds
================================================== next_anonymous_necessary_preds
==================================================== next_anonymous_necessary_pred
====================================================== unary_pred
======================================================== pred_name
---------------------------------------------------------- T
---------------------------------------------- )
---------------------------------------------- ;
------------------------------------------ }
------------------------------------------ ;
-------------------------------------- }
-------------------------------------- ;
================================ stmt
================================== trait_decl
------------------------------------ trait_id
==================================== trait_id
-------------------------------------- Fooable
------------------------------------ (
==================================== class_args
====================================== main_arg
---------------------------------------- self
------------------------------------ )
------------------------------------ :
------------------------------------ {
==================================== trait_decl_stmts
====================================== trait_decl_stmt
---------------------------------------- fn
---------------------------------------- :
---------------------------------------- {
======================================== decl_fn_stmts
========================================== decl_fn_stmt
============================================ fn_id
---------------------------------------------- foo
-------------------------------------------- (
============================================ pred_args
============================================== main_pred
------------------------------------------------ self
---------------------------------------------- ,
============================================== next_preds
================================================ next_anonymous_necessary_preds
================================================== next_anonymous_necessary_pred
==================================================== unary_pred
====================================================== pred_name
-------------------------------------------------------- any
-------------------------------------------- )
-------------------------------------------- ;
---------------------------------------- }
---------------------------------------- ;
------------------------------------ }
------------------------------------ ;
============================== stmt
================================ print_stmt
---------------------------------- printinfo
================================== to_print
------------------------------------ Fooable
---------------------------------- ;
============================ stmt
============================== trait_decl
-------------------------------- trait_id
================================ trait_id
---------------------------------- FooBarable
-------------------------------- (
================================ class_args
================================== main_arg
------------------------------------ self
-------------------------------- )
-------------------------------- :
-------------------------------- {
================================ trait_decl_stmts
================================== trait_decl_stmt
------------------------------------ fn
------------------------------------ :
------------------------------------ {
==================================== decl_fn_stmts
====================================== decl_fn_stmts
======================================== decl_fn_stmt
========================================== fn_id
-------------------------------------------- foo
------------------------------------------ (
========================================== pred_args
============================================ main_pred
---------------------------------------------- self
-------------------------------------------- ,
============================================ next_preds
============================================== next_anonymous_necessary_preds
================================================ next_anonymous_necessary_pred
================================================== unary_pred
==================================================== pred_name
------------------------------------------------------ any
------------------------------------------ )
------------------------------------------ ;
====================================== decl_fn_stmt
======================================== fn_id
------------------------------------------ bar
---------------------------------------- (
======================================== pred_args
========================================== main_pred
-------------------------------------------- self
------------------------------------------ ,
========================================== next_preds
============================================ next_anonymous_necessary_preds
============================================== next_anonymous_necessary_pred
================================================ unary_pred
================================================== pred_name
---------------------------------------------------- any
---------------------------------------- )
---------------------------------------- ;
------------------------------------ }
------------------------------------ ;
-------------------------------- }
-------------------------------- ;
========================== stmt
============================ trait_decl
------------------------------ trait_id
============================== trait_id
-------------------------------- BarBazable
------------------------------ (
============================== class_args
================================ main_arg
---------------------------------- self
------------------------------ )
------------------------------ :
------------------------------ {
============================== trait_decl_stmts
================================ trait_decl_stmt
---------------------------------- fn
---------------------------------- :
---------------------------------- {
================================== decl_fn_stmts
==================================== decl_fn_stmts
====================================== decl_fn_stmt
======================================== fn_id
------------------------------------------ bar
---------------------------------------- (
======================================== pred_args
========================================== main_pred
-------------------------------------------- self
------------------------------------------ ,
========================================== next_preds
============================================ next_anonymous_necessary_preds
============================================== next_anonymous_necessary_pred
================================================ unary_pred
================================================== pred_name
---------------------------------------------------- any
---------------------------------------- )
---------------------------------------- ;
==================================== decl_fn_stmt
====================================== fn_id
---------------------------------------- baz
-------------------------------------- (
====================================== pred_args
======================================== main_pred
------------------------------------------ self
---------------------------------------- ,
======================================== next_preds
========================================== next_anonymous_necessary_preds
============================================ next_anonymous_necessary_pred
============================================== unary_pred
================================================ pred_name
-------------------------------------------------- any
-------------------------------------- )
-------------------------------------- ;
---------------------------------- }
---------------------------------- ;
------------------------------ }
------------------------------ ;
======================== stmt
========================== print_stmt
---------------------------- printinfo
============================ to_print
============================== unnamed_pred
-------------------------------- [
================================ pred_expr
================================== pred_expr_a
==================================== pred_expr_a
====================================== pred_expr_b
======================================== pred_expr_c
========================================== unary_pred
============================================ pred_name
---------------------------------------------- FooBarable
------------------------------------ and
==================================== pred_expr_b
====================================== pred_expr_c
======================================== unary_pred
========================================== pred_name
-------------------------------------------- BarBazable
-------------------------------- ]
---------------------------- ;
====================== stmt
======================== print_stmt
-------------------------- printinfo
========================== to_print
============================ unnamed_pred
------------------------------ [
============================== pred_expr
================================ pred_expr
================================== pred_expr_a
==================================== pred_expr_b
====================================== pred_expr_c
======================================== unary_pred
========================================== pred_name
-------------------------------------------- FooBarable
-------------------------------- or
================================ pred_expr_a
================================== pred_expr_b
==================================== pred_expr_c
====================================== unary_pred
======================================== pred_name
------------------------------------------ BarBazable
------------------------------ ]
-------------------------- ;
==================== stmt
====================== generate_stmt
------------------------ generate
======================== to_print
-------------------------- Fooo
------------------------ ;
================== stmt
==================== generate_stmt
---------------------- generate
====================== to_print
------------------------ FooBarable
---------------------- ;
================ stmt
================== print_stmt
-------------------- printinfo
==================== to_print
====================== boolean_expr
======================== boolean_expr_a
========================== boolean_expr_b
============================ boolean_expr_c
============================== boolean_expr_d
================================ boolean_expr_e
================================== atomic_boolean_expr
==================================== unary_pred
====================================== unnamed_pred
---------------------------------------- [
======================================== pred_expr
========================================== pred_expr_a
============================================ pred_expr_a
============================================== pred_expr_b
================================================ pred_expr_c
================================================== unary_pred
==================================================== pred_name
------------------------------------------------------ FooBarable
-------------------------------------------- and
============================================ pred_expr_b
============================================== pred_expr_c
================================================ unary_pred
================================================== pred_name
---------------------------------------------------- BarBazable
---------------------------------------- ]
------------------------------------ implies
==================================== unary_pred
====================================== unnamed_pred
---------------------------------------- [
======================================== pred_expr
========================================== pred_expr
============================================ pred_expr_a
============================================== pred_expr_b
================================================ pred_expr_c
================================================== unary_pred
==================================================== pred_name
------------------------------------------------------ FooBarable
------------------------------------------ or
========================================== pred_expr_a
============================================ pred_expr_b
============================================== pred_expr_c
================================================ unary_pred
================================================== pred_name
---------------------------------------------------- BarBazable
---------------------------------------- ]
-------------------- ;
============== stmt
================ assign_stmt
================== names
==================== names
====================== name
------------------------ a
-------------------- ,
==================== name
---------------------- b
------------------ =
================== assign_expr
==================== names
====================== names
======================== name
-------------------------- b
---------------------- ,
====================== name
------------------------ a
------------------ ;
============ stmt
============== trait_decl
---------------- trait_id
================ trait_id
------------------ DefaultArgsExample
---------------- (
================ class_args
================== main_arg
-------------------- self
------------------ ,
================== sub_args
==================== necessary_args
====================== necessary_args
======================== necessary_arg
-------------------------- arg_a
---------------------- ,
====================== necessary_arg
------------------------ arg_b
-------------------- ,
==================== optional_args
====================== optional_args
======================== optional_arg
-------------------------- arg_c
-------------------------- =
========================== unary_pred
============================ unnamed_pred
------------------------------ [
============================== pred_expr
================================ pred_expr_a
================================== pred_expr_a
==================================== pred_expr_b
====================================== pred_expr_c
======================================== unary_pred
========================================== pred_name
-------------------------------------------- Addable
---------------------------------- and
================================== pred_expr_b
==================================== pred_expr_c
====================================== unary_pred
======================================== pred_name
------------------------------------------ Subable
------------------------------ ]
---------------------- ,
====================== optional_arg
------------------------ arg_d
------------------------ =
======================== unary_pred
========================== pred_name
---------------------------- float
---------------- )
---------------- :
---------------- {
================ trait_decl_stmts
------------------ pass
------------------ ;
---------------- }
---------------- ;
========== stmt
============ trait_decl
-------------- trait_id
============== trait_id
---------------- TraitWithAllOptionalArgs
-------------- (
============== class_args
================ main_arg
------------------ self
---------------- ,
================ sub_args
================== optional_args
==================== optional_args
====================== optional_arg
------------------------ arg_a
------------------------ =
======================== unary_pred
========================== pred_name
---------------------------- int
-------------------- ,
==================== optional_arg
---------------------- arg_b
---------------------- =
====================== unary_pred
======================== pred_name
-------------------------- str
-------------- )
-------------- :
-------------- {
============== trait_decl_stmts
================ trait_decl_stmt
------------------ fn
------------------ :
------------------ {
================== decl_fn_stmts
==================== decl_fn_stmt
====================== fn_id
------------------------ whatever
---------------------- (
====================== pred_args
======================== main_pred
-------------------------- self
------------------------ ,
======================== next_preds
========================== next_anonymous_necessary_preds
============================ next_anonymous_necessary_preds
============================== next_anonymous_necessary_pred
================================ unary_pred
================================== pred_name
------------------------------------ int
---------------------------- ,
============================ next_anonymous_necessary_pred
============================== unary_pred
================================ pred_name
---------------------------------- int
-------------------------- ,
========================== next_anonymous_optional_preds
============================ next_anonymous_optional_preds
============================== next_anonymous_optional_pred
-------------------------------- =
================================ unary_pred
================================== pred_name
------------------------------------ int
---------------------------- ,
============================ next_anonymous_optional_pred
------------------------------ =
============================== unary_pred
================================ pred_name
---------------------------------- int
-------------------------- ,
-------------------------- *
-------------------------- ,
========================== next_named_preds
============================ next_named_preds
============================== next_named_pred
================================ next_named_optional_pred
================================== arg_name
------------------------------------ a
---------------------------------- :
---------------------------------- =
================================== unary_pred
==================================== pred_name
-------------------------------------- int
---------------------------- ,
============================ next_named_pred
============================== next_named_necessary_pred
================================ arg_name
---------------------------------- b
-------------------------------- :
================================ unary_pred
================================== pred_name
------------------------------------ int
---------------------- )
---------------------- ;
------------------ }
------------------ ;
-------------- }
-------------- ;
======== stmt
========== trait_decl
------------ trait_id
============ trait_id
-------------- GenericTrait
------------ (
============ class_args
============== main_arg
---------------- self
------------ )
------------ :
------------ {
============ trait_decl_stmts
============== trait_decl_stmt
---------------- fn
---------------- :
---------------- {
================ decl_fn_stmts
================== decl_fn_stmt
==================== fn_id
---------------------- foo
-------------------- [
==================== type_var_args
====================== type_var_arg
======================== type_var_id
-------------------------- T
-------------------- ]
-------------------- (
==================== pred_args
====================== main_pred
------------------------ self
---------------------- ,
====================== next_preds
======================== next_anonymous_necessary_preds
========================== next_anonymous_necessary_pred
============================ unary_pred
============================== pred_name
-------------------------------- T
-------------------- )
-------------------- ->
==================== unary_pred
====================== pred_name
------------------------ T
-------------------- ;
---------------- }
---------------- ;
------------ }
------------ ;
====== stmt
======== trait_decl
---------- trait_id
========== trait_id
------------ vector7D
---------- (
========== class_args
============ main_arg
-------------- self
---------- )
---------- extends
========== unary_pred
============ unnamed_pred
============== pred_name
---------------- vector
-------------- [
============== args
================ pred_expr
================== pred_expr_a
==================== pred_expr_b
====================== pred_expr_c
======================== unary_pred
========================== unnamed_pred
============================ pred_name
------------------------------ vector
---------------------------- [
============================ args
============================== pred_expr
================================ pred_expr_a
================================== pred_expr_b
==================================== pred_expr_c
====================================== unary_pred
======================================== unnamed_pred
========================================== pred_name
-------------------------------------------- vector
------------------------------------------ [
========================================== args
============================================ pred_expr
============================================== pred_expr_a
================================================ pred_expr_b
================================================== pred_expr_c
==================================================== unary_pred
====================================================== unnamed_pred
======================================================== pred_name
---------------------------------------------------------- vector
-------------------------------------------------------- [
======================================================== args
========================================================== pred_expr
============================================================ pred_expr_a
============================================================== pred_expr_b
================================================================ pred_expr_c
================================================================== unary_pred
==================================================================== unnamed_pred
====================================================================== pred_name
------------------------------------------------------------------------ vector
---------------------------------------------------------------------- [
====================================================================== args
======================================================================== pred_expr
========================================================================== pred_expr_a
============================================================================ pred_expr_b
============================================================================== pred_expr_c
================================================================================ unary_pred
================================================================================== unnamed_pred
==================================================================================== pred_name
-------------------------------------------------------------------------------------- vector
------------------------------------------------------------------------------------ [
==================================================================================== args
====================================================================================== pred_expr
======================================================================================== pred_expr_a
========================================================================================== pred_expr_b
============================================================================================ pred_expr_c
============================================================================================== unary_pred
================================================================================================ unnamed_pred
================================================================================================== pred_name
---------------------------------------------------------------------------------------------------- vector
-------------------------------------------------------------------------------------------------- [
================================================================================================== args
==================================================================================================== pred_expr
====================================================================================================== pred_expr_a
======================================================================================================== pred_expr_b
========================================================================================================== pred_expr_c
============================================================================================================ unary_pred
============================================================================================================== pred_name
---------------------------------------------------------------------------------------------------------------- int
-------------------------------------------------------------------------------------------------- ]
------------------------------------------------------------------------------------ ]
---------------------------------------------------------------------- ]
-------------------------------------------------------- ]
------------------------------------------ ]
---------------------------- ]
-------------- ]
---------- :
---------- {
========== trait_decl_stmts
============ trait_decl_stmt
-------------- fn
-------------- :
-------------- {
============== decl_fn_stmts
================ decl_fn_stmts
================== decl_fn_stmts
==================== decl_fn_stmt
====================== fn_id
------------------------ begin
---------------------- (
====================== pred_args
======================== main_pred
-------------------------- self
---------------------- )
---------------------- ->
====================== unary_pred
======================== unnamed_pred
========================== pred_name
---------------------------- iterator
-------------------------- [
========================== args
============================ pred_expr
============================== pred_expr
================================ pred_expr
================================== pred_expr_a
==================================== pred_expr_b
====================================== pred_expr_c
======================================== unary_pred
========================================== unnamed_pred
============================================ pred_name
---------------------------------------------- vector
-------------------------------------------- [
============================================ args
============================================== pred_expr
================================================ pred_expr_a
================================================== pred_expr_b
==================================================== pred_expr_c
====================================================== unary_pred
======================================================== pred_name
---------------------------------------------------------- any
-------------------------------------------- ]
-------------------------------- or
================================ pred_expr_a
================================== pred_expr_b
==================================== pred_expr_c
====================================== unary_pred
======================================== unnamed_pred
========================================== pred_name
-------------------------------------------- vector
------------------------------------------ [
========================================== args
============================================ pred_expr
============================================== pred_expr
================================================ pred_expr_a
================================================== pred_expr_b
==================================================== pred_expr_c
====================================================== unary_pred
======================================================== unnamed_pred
========================================================== pred_name
------------------------------------------------------------ vector
---------------------------------------------------------- [
========================================================== args
============================================================ pred_expr
============================================================== pred_expr_a
================================================================ pred_expr_b
================================================================== pred_expr_c
==================================================================== unary_pred
====================================================================== pred_name
------------------------------------------------------------------------ any
---------------------------------------------------------- ]
---------------------------------------------- or
============================================== pred_expr_a
================================================ pred_expr_b
================================================== pred_expr_c
==================================================== unary_pred
====================================================== pred_name
-------------------------------------------------------- any
------------------------------------------ ]
------------------------------ or
============================== pred_expr_a
================================ pred_expr_b
================================== pred_expr_c
==================================== unary_pred
====================================== unnamed_pred
======================================== pred_name
------------------------------------------ vector
---------------------------------------- [
======================================== args
========================================== pred_expr
============================================ pred_expr_a
============================================== pred_expr_b
================================================ pred_expr_c
================================================== unary_pred
==================================================== unnamed_pred
====================================================== pred_name
-------------------------------------------------------- vector
------------------------------------------------------ [
====================================================== args
======================================================== pred_expr
========================================================== pred_expr_a
============================================================ pred_expr_b
============================================================== pred_expr_c
================================================================ unary_pred
================================================================== unnamed_pred
==================================================================== pred_name
---------------------------------------------------------------------- vector
-------------------------------------------------------------------- [
==================================================================== args
====================================================================== pred_expr
======================================================================== pred_expr_a
========================================================================== pred_expr_b
============================================================================ pred_expr_c
============================================================================== unary_pred
================================================================================ pred_name
---------------------------------------------------------------------------------- any
-------------------------------------------------------------------- ]
------------------------------------------------------ ]
---------------------------------------- ]
-------------------------- ]
---------------------- ;
================== decl_fn_stmt
==================== fn_id
---------------------- end
-------------------- (
==================== pred_args
====================== main_pred
------------------------ self
-------------------- )
-------------------- ->
==================== unary_pred
====================== unnamed_pred
======================== pred_name
-------------------------- iterator
------------------------ [
======================== args
========================== pred_expr
============================ pred_expr
============================== pred_expr_a
================================ pred_expr_b
================================== pred_expr_c
==================================== unary_pred
====================================== pred_name
---------------------------------------- int
---------------------------- or
============================ pred_expr_a
============================== pred_expr_a
================================ pred_expr_b
================================== pred_expr_c
==================================== unary_pred
====================================== unnamed_pred
---------------------------------------- [
======================================== pred_expr
========================================== pred_expr_a
============================================ pred_expr_b
============================================== pred_expr_c
================================================ unary_pred
================================================== unnamed_pred
---------------------------------------------------- trait_of
---------------------------------------------------- (
==================================================== var_expr
====================================================== unary_pred
======================================================== pred_name
---------------------------------------------------------- vector
------------------------------------------------------ .
====================================================== member_var_name
-------------------------------------------------------- a
---------------------------------------------------- )
---------------------------------------- ]
------------------------------ and
============================== pred_expr_b
================================ pred_expr_c
================================== unary_pred
==================================== unnamed_pred
-------------------------------------- trait_of
-------------------------------------- (
====================================== var_expr
======================================== unary_pred
========================================== unnamed_pred
-------------------------------------------- [
============================================ pred_expr
============================================== pred_expr
================================================ pred_expr_a
================================================== pred_expr_b
==================================================== pred_expr_c
====================================================== unary_pred
======================================================== unnamed_pred
---------------------------------------------------------- None
---------------------------------------------- or
============================================== pred_expr_a
================================================ pred_expr_b
================================================== pred_expr_c
==================================================== unary_pred
====================================================== unnamed_pred
======================================================== pred_name
---------------------------------------------------------- vector5D
-------------------------------------------------------- [
======================================================== args
========================================================== args
============================================================ pred_expr
============================================================== pred_expr_a
================================================================ pred_expr_b
================================================================== pred_expr_c
==================================================================== unary_pred
====================================================================== unnamed_pred
======================================================================== pred_name
-------------------------------------------------------------------------- v
------------------------------------------------------------------------ [
======================================================================== args
========================================================================== pred_expr
============================================================================ pred_expr_a
============================================================================== pred_expr_b
================================================================================ pred_expr_c
================================================================================== unary_pred
==================================================================================== pred_name
-------------------------------------------------------------------------------------- u
------------------------------------------------------------------------ ]
---------------------------------------------------------- ,
========================================================== pred_expr
============================================================ pred_expr_a
============================================================== pred_expr_b
================================================================ pred_expr_c
================================================================== unary_pred
==================================================================== unnamed_pred
====================================================================== pred_name
------------------------------------------------------------------------ v
---------------------------------------------------------------------- [
====================================================================== args
======================================================================== pred_expr
========================================================================== pred_expr_a
============================================================================ pred_expr_b
============================================================================== pred_expr_c
================================================================================ unary_pred
================================================================================== pred_name
------------------------------------------------------------------------------------ u
---------------------------------------------------------------------- ]
-------------------------------------------------------- ]
-------------------------------------------- ]
---------------------------------------- .
======================================== member_var_name
------------------------------------------ y
-------------------------------------- )
------------------------ ]
-------------------- ;
================ decl_fn_stmt
================== fn_id
-------------------- front
------------------ (
================== pred_args
==================== main_pred
---------------------- self
-------------------- ,
==================== next_preds
---------------------- ,
====================== next_anonymous_optional_preds
======================== next_anonymous_optional_pred
-------------------------- =
========================== unary_pred
============================ unnamed_pred
------------------------------ [
============================== pred_expr
================================ pred_expr
================================== pred_expr_a
==================================== pred_expr_b
====================================== pred_expr_c
======================================== unary_pred
========================================== pred_name
-------------------------------------------- int
-------------------------------- or
================================ pred_expr_a
================================== pred_expr_b
==================================== pred_expr_c
-------------------------------------- (
====================================== pred_expr
======================================== pred_expr
========================================== pred_expr_a
============================================ pred_expr_a
============================================== pred_expr_b
================================================ pred_expr_c
================================================== unary_pred
==================================================== pred_name
------------------------------------------------------ str
-------------------------------------------- and
============================================ pred_expr_b
---------------------------------------------- not
============================================== pred_expr_c
================================================ unary_pred
================================================== pred_name
---------------------------------------------------- float
---------------------------------------- or
======================================== pred_expr_a
========================================== pred_expr_b
============================================ pred_expr_c
============================================== unary_pred
================================================ unnamed_pred
================================================== pred_name
---------------------------------------------------- vector
-------------------------------------------------- [
================================================== args
==================================================== pred_expr
====================================================== pred_expr_a
======================================================== pred_expr_b
========================================================== pred_expr_c
============================================================ unary_pred
============================================================== pred_name
---------------------------------------------------------------- int
-------------------------------------------------- ]
-------------------------------------- )
------------------------------ ]
------------------ )
------------------ ;
-------------- }
-------------- ;
---------- }
---------- ;
==== stmt
====== print_stmt
-------- printinfo
======== to_print
========== boolean_expr
============ boolean_expr_a
============== boolean_expr_b
================ boolean_expr_b
================== boolean_expr_c
==================== boolean_expr_d
====================== boolean_expr_e
------------------------ (
======================== boolean_expr
========================== boolean_expr_a
============================ boolean_expr_b
============================== boolean_expr_c
================================ boolean_expr_d
================================== boolean_expr_e
==================================== atomic_boolean_expr
====================================== unary_pred
======================================== unnamed_pred
------------------------------------------ trait_of
------------------------------------------ (
========================================== var_expr
============================================ unary_pred
============================================== pred_name
------------------------------------------------ x
-------------------------------------------- .
============================================ member_var_name
---------------------------------------------- z
------------------------------------------ )
-------------------------------------- implies
====================================== unary_pred
======================================== unnamed_pred
------------------------------------------ trait_of
------------------------------------------ (
========================================== var_expr
============================================ unary_pred
============================================== pred_name
------------------------------------------------ y
-------------------------------------------- .
============================================ member_var_name
---------------------------------------------- z
------------------------------------------ )
------------------------ )
---------------- and
================ boolean_expr_c
================== boolean_expr_d
==================== boolean_expr_e
---------------------- (
====================== boolean_expr
======================== boolean_expr
========================== boolean_expr_a
============================ boolean_expr_b
============================== boolean_expr_c
================================ boolean_expr_c
================================== boolean_expr_d
==================================== boolean_expr_e
====================================== atomic_boolean_expr
======================================== constants
------------------------------------------ True
-------------------------------- ==
================================ boolean_expr_d
================================== boolean_expr_e
==================================== atomic_boolean_expr
====================================== constants
---------------------------------------- False
------------------------ or
======================== boolean_expr_a
========================== boolean_expr_b
============================ boolean_expr_c
============================== boolean_expr_c
================================ boolean_expr_d
================================== boolean_expr_e
==================================== atomic_boolean_expr
====================================== constants
---------------------------------------- False
------------------------------ !=
============================== boolean_expr_d
================================ boolean_expr_e
================================== atomic_boolean_expr
==================================== unary_pred
====================================== pred_name
---------------------------------------- int
------------------------------------ (
==================================== args
====================================== pred_expr
======================================== pred_expr_a
========================================== pred_expr_b
============================================ pred_expr_c
============================================== unary_pred
================================================ pred_name
-------------------------------------------------- int
------------------------------------ )
---------------------- )
-------- ;
-- PROGRAM_END
-- ;

```

#### OUTPUT (AST)

```

 init
== Program
==== statement
====== trait declaration
-------- any
======== class args
---------- self
==== statement
====== trait declaration
-------- StackLike
======== class args
---------- self
======== declaring statements
========== member functions
============ fn
-------------- append
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- any
============== trait AND/OR trait expression
================ predicate expression, LHS
================== named trait
-------------------- self
---------------- or
================ predicate expression, RHS
================== None trait
-------------------- None
============ fn
-------------- pop
============== (args)
---------------- self
============== trait AND/OR trait expression
================ predicate expression, LHS
================== named trait
-------------------- self
---------------- or
================ predicate expression, RHS
================== None trait
-------------------- None
============ fn
-------------- popop
============== (args)
---------------- self
============== trait AND/OR trait expression
================ predicate expression, LHS
================== named trait
-------------------- self
---------------- or
================ predicate expression, RHS
================== predicate expression, LHS
==================== None trait
---------------------- None
------------------ and
================== predicate expression, RHS
==================== None trait
---------------------- None
========== member functions
============ fn
-------------- __len__
============== (args)
---------------- self
============== named trait
---------------- int
==== statement
====== trait declaration
-------- PointLike
======== class args
---------- self
======== declaring statements
========== member variables
============ var
-------------- x
============== named trait
---------------- int
============ var
-------------- y
============== named trait
---------------- int
==== statement
====== trait declaration
-------- AnyRandom
======== class args
---------- self
======== declaring statements
========== member variables
============ var
-------------- x
============== named trait
---------------- int
============ var
-------------- y
============== named trait
---------------- int
============ var
-------------- z
========== member functions
============ fn
-------------- __init__
============== (args)
---------------- self
================ anonymous optional
================== named trait
-------------------- int
================== named trait
-------------------- int
================== named trait
-------------------- any
============== None trait
---------------- None
========== class functions
============ fn
-------------- foo
============== (args)
---------------- cls
================ anonymous necessary
================== named trait
-------------------- int
============== named trait
---------------- str
============ fn
-------------- foo
============== (args)
---------------- cls
================ anonymous necessary
================== named trait
-------------------- float
========== class variables
============ var
-------------- xxx
============ var
-------------- yyy
============== named trait
---------------- int
========== static functions
============ fn
-------------- bar
============== None trait
---------------- None
==== print:
====== trait of a member variable
======== named trait
---------- AnyRandom
-------- .
-------- x
==== statement
====== names
-------- t_random
====== names
-------- any_random
------ assign
==== statement
====== trait declaration
-------- decl_var_with_comma
======== class args
---------- self
======== declaring statements
========== member variables
============ var
-------------- x
-------------- y
-------------- z
============== named trait
---------------- int
==== statement
====== trait declaration
-------- Addable
======== class args
---------- self
========== sub args
------------ T
======== declaring statements
========== member functions
============ fn
-------------- __add__
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- T
============== named trait
---------------- self
==== statement
====== trait declaration
-------- Multipliable
======== class args
---------- self
========== sub args
------------ T
======== declaring statements
========== member functions
============ fn
-------------- __mul__
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- T
============== named trait
---------------- self
==== print:
====== predicate call, boolean
======== named trait
---------- StackLike
-------- fn_call
======== args
========== predicate expression
============ named trait
-------------- int
==== print:
====== predicate call, boolean
======== named trait
---------- StackLike
-------- fn_call
======== args
========== predicate expression
============ named trait
-------------- list
==== print:
====== predicate call, boolean
======== named trait
---------- Addable
-------- fn_call
======== args
========== predicate expression
============ named trait
-------------- str
========== predicate expression
============ named trait
-------------- int
==== print:
====== predicate call, boolean
======== named trait
---------- Multipliable
-------- fn_call
======== args
========== predicate expression
============ named trait
-------------- str
========== predicate expression
============ named trait
-------------- int
==== print:
====== predicate call, boolean
======== unnamed_pred_rule_2
---------- Addable
========== args
============ predicate expression
============== named trait
---------------- int
-------- fn_call
======== args
========== predicate expression
============ named trait
-------------- str
==== statement
====== trait declaration
-------- AddableSubable
======== class args
---------- self
========== sub args
------------ T
======== unnamed_pred_rule_2
---------- Addable
========== args
============ predicate expression
============== named trait
---------------- T
======== declaring statements
========== member functions
============ fn
-------------- __sub__
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- T
==== statement
====== trait declaration
-------- Fooable
======== class args
---------- self
======== declaring statements
========== member functions
============ fn
-------------- foo
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- any
==== print:
------ Fooable
==== statement
====== trait declaration
-------- FooBarable
======== class args
---------- self
======== declaring statements
========== member functions
============ fn
-------------- foo
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- any
============ fn
-------------- bar
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- any
==== statement
====== trait declaration
-------- BarBazable
======== class args
---------- self
======== declaring statements
========== member functions
============ fn
-------------- bar
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- any
============ fn
-------------- baz
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- any
==== print:
====== trait AND/OR trait expression
======== predicate expression, LHS
========== named trait
------------ FooBarable
-------- and
======== predicate expression, RHS
========== named trait
------------ BarBazable
==== print:
====== trait AND/OR trait expression
======== predicate expression, LHS
========== named trait
------------ FooBarable
-------- or
======== predicate expression, RHS
========== named trait
------------ BarBazable
==== generate:
------ Fooo
==== generate:
------ FooBarable
==== print:
====== implies expression, boolean
======== trait AND/OR trait expression
========== predicate expression, LHS
============ named trait
-------------- FooBarable
---------- and
========== predicate expression, RHS
============ named trait
-------------- BarBazable
-------- implies
======== trait AND/OR trait expression
========== predicate expression, LHS
============ named trait
-------------- FooBarable
---------- or
========== predicate expression, RHS
============ named trait
-------------- BarBazable
==== statement
====== names
-------- a
-------- b
====== names
-------- b
-------- a
------ assign
==== statement
====== trait declaration
-------- DefaultArgsExample
======== class args
---------- self
========== sub args
------------ arg_a
------------ arg_b
============ optional
-------------- arg_c
============== trait AND/OR trait expression
================ predicate expression, LHS
================== named trait
-------------------- Addable
---------------- and
================ predicate expression, RHS
================== named trait
-------------------- Subable
============ optional
-------------- arg_d
============== named trait
---------------- float
==== statement
====== trait declaration
-------- TraitWithAllOptionalArgs
======== class args
---------- self
========== sub args
============ optional
-------------- arg_a
============== named trait
---------------- int
============ optional
-------------- arg_b
============== named trait
---------------- str
======== declaring statements
========== member functions
============ fn
-------------- whatever
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- int
================== named trait
-------------------- int
================ anonymous optional
================== named trait
-------------------- int
================== named trait
-------------------- int
================ named argument
------------------ a
------------------ rule=optional
================== named trait
-------------------- int
================ named argument
------------------ b
------------------ rule=necessary
================== named trait
-------------------- int
==== statement
====== trait declaration
-------- GenericTrait
======== class args
---------- self
======== declaring statements
========== member functions
============ fn
-------------- foo
============== [args]
---------------- T
============== (args)
---------------- self
================ anonymous necessary
================== named trait
-------------------- T
============== named trait
---------------- T
==== statement
====== trait declaration
-------- vector7D
======== class args
---------- self
======== unnamed_pred_rule_2
---------- vector
========== args
============ predicate expression
============== unnamed_pred_rule_2
---------------- vector
================ args
================== predicate expression
==================== unnamed_pred_rule_2
---------------------- vector
====================== args
======================== predicate expression
========================== unnamed_pred_rule_2
---------------------------- vector
============================ args
============================== predicate expression
================================ unnamed_pred_rule_2
---------------------------------- vector
================================== args
==================================== predicate expression
====================================== unnamed_pred_rule_2
---------------------------------------- vector
======================================== args
========================================== predicate expression
============================================ unnamed_pred_rule_2
---------------------------------------------- vector
============================================== args
================================================ predicate expression
================================================== named trait
---------------------------------------------------- int
======== declaring statements
========== member functions
============ fn
-------------- begin
============== (args)
---------------- self
============== unnamed_pred_rule_2
---------------- iterator
================ args
================== predicate expression
==================== predicate expression, LHS
====================== predicate expression, LHS
======================== unnamed_pred_rule_2
-------------------------- vector
========================== args
============================ predicate expression
============================== named trait
-------------------------------- any
---------------------- or
====================== predicate expression, RHS
======================== unnamed_pred_rule_2
-------------------------- vector
========================== args
============================ predicate expression
============================== predicate expression, LHS
================================ unnamed_pred_rule_2
---------------------------------- vector
================================== args
==================================== predicate expression
====================================== named trait
---------------------------------------- any
------------------------------ or
============================== predicate expression, RHS
================================ named trait
---------------------------------- any
-------------------- or
==================== predicate expression, RHS
====================== unnamed_pred_rule_2
------------------------ vector
======================== args
========================== predicate expression
============================ unnamed_pred_rule_2
------------------------------ vector
============================== args
================================ predicate expression
================================== unnamed_pred_rule_2
------------------------------------ vector
==================================== args
====================================== predicate expression
======================================== named trait
------------------------------------------ any
============ fn
-------------- end
============== (args)
---------------- self
============== unnamed_pred_rule_2
---------------- iterator
================ args
================== predicate expression
==================== predicate expression, LHS
====================== named trait
------------------------ int
-------------------- or
==================== predicate expression, RHS
====================== predicate expression, LHS
======================== trait AND/OR trait expression
========================== trait of a member variable
============================ named trait
------------------------------ vector
---------------------------- .
---------------------------- a
---------------------- and
====================== predicate expression, RHS
======================== trait of a member variable
========================== trait AND/OR trait expression
============================ predicate expression, LHS
============================== None trait
-------------------------------- None
---------------------------- or
============================ predicate expression, RHS
============================== unnamed_pred_rule_2
-------------------------------- vector5D
================================ args
================================== predicate expression
==================================== unnamed_pred_rule_2
-------------------------------------- v
====================================== args
======================================== predicate expression
========================================== named trait
-------------------------------------------- u
================================== predicate expression
==================================== unnamed_pred_rule_2
-------------------------------------- v
====================================== args
======================================== predicate expression
========================================== named trait
-------------------------------------------- u
-------------------------- .
-------------------------- y
============ fn
-------------- front
============== (args)
---------------- self
================ anonymous optional
================== trait AND/OR trait expression
==================== predicate expression, LHS
====================== named trait
------------------------ int
-------------------- or
==================== predicate expression, RHS
====================== predicate expression, LHS
======================== predicate expression, LHS
========================== named trait
---------------------------- str
------------------------ and
======================== predicate expression, RHS
-------------------------- not
========================== named trait
---------------------------- float
---------------------- or
====================== predicate expression, RHS
======================== unnamed_pred_rule_2
-------------------------- vector
========================== args
============================ predicate expression
============================== named trait
-------------------------------- int
==== print:
====== boolean expression
======== boolean expression, LHS
========== implies expression, boolean
============ trait of a member variable
============== named trait
---------------- x
-------------- .
-------------- z
------------ implies
============ trait of a member variable
============== named trait
---------------- y
-------------- .
-------------- z
-------- and
======== boolean expression, RHS
========== boolean expression
============ boolean expression, LHS
============== constant, boolean
---------------- False
-------------- eq
============== constant, boolean
---------------- False
------------ or
============ boolean expression, RHS
============== constant, boolean
---------------- False
-------------- neq
============== predicate call, boolean
================ named trait
------------------ int
---------------- fn_call
================ args
================== predicate expression
==================== named trait
---------------------- int


```

#### INPUT (ERROR SYNTAX)

```

a, b =

a, b = c, d = e, f =

printinfo True or (True or False) xor (False and True) and (xor False)
printinfo True or (and True or False) xor (False and and True) and (xor False)
printinfo foo trait bar

printinfo [Addable and Subable](int,,, int)
printinfo [Addable and Subable](,,,int)

```

#### OUTPUT (ERROR SYNTAX)

```
==================================================
Syntax error at line 2, column 6
LALR Parser state : 45


PROGRAM_BEGIN;

a, b =;
      ^
no RHS expression after the assign operator
==================================================
Syntax error at line 4, column 20
LALR Parser state : 127


a, b =;

a, b = c, d = e, f =;
                    ^
no RHS expression after the assign operator
==================================================
Syntax error at line 6, column 60
LALR Parser state : 32


a, b = c, d = e, f =;

printinfo True or (True or False) xor (False and True) and (xor False);
                                                            ^
expected a valid boolean expression here
==================================================
Syntax error at line 7, column 19
LALR Parser state : 32

a, b = c, d = e, f =;

printinfo True or (True or False) xor (False and True) and (xor False);
printinfo True or (and True or False) xor (False and and True) and (xor False);
                   ^
expected a valid boolean expression here
==================================================
Syntax error at line 7, column 53
LALR Parser state : 74

a, b = c, d = e, f =;

printinfo True or (True or False) xor (False and True) and (xor False);
printinfo True or (and True or False) xor (False and and True) and (xor False);
                                                     ^
expected a valid boolean expression here
==================================================
Syntax error at line 7, column 68
LALR Parser state : 32

a, b = c, d = e, f =;

printinfo True or (True or False) xor (False and True) and (xor False);
printinfo True or (and True or False) xor (False and and True) and (xor False);
                                                                    ^
expected a valid boolean expression here
==================================================
Syntax error at line 8, column 14
LALR Parser state : 24


printinfo True or (True or False) xor (False and True) and (xor False);
printinfo True or (and True or False) xor (False and and True) and (xor False);
printinfo foo trait bar;
              ^
Unexpected token, expected "implies" here
==================================================
Syntax error at line 10, column 36
LALR Parser state : 137

printinfo True or (and True or False) xor (False and and True) and (xor False);
printinfo foo trait bar;

printinfo [Addable and Subable](int,,, int);
                                    ^
expected a valid predicate expression here
==================================================
Compiler shut down due to syntax error
```

#### INPUT (ERROR SYNTAX)

```
trait ERR(self):
    fn:
        yaya()

trait ERR2(self):
    static_fn:
        []() -> int
        okay(int, bool)
        static_yaya[]() -> int

trait ERR3(self):
    fn:

trait ERR4(self):
    fn:
        [][][][]


trait ERR5(self):
    fn:
        somewhat_err_fn(self, any, int, =int, int)
    static_fn:
        somewhat_stt_err_fn(any, int, *, int, :int, a:=int)
    fn:
        err_fn2(self, any, int, *, a=int, b:=int, c:int, d=:int)
        err_fn3(self, int str float bool)
    cls_fn:
        err_fn4(self) -> True
        err_fn5(True, self)
        err_fn6(self, True, int) -> int

```

#### OUTPUT (ERROR SYNTAX)

```
==================================================
Syntax error at line 4, column 14
LALR Parser state : 286


trait ERR(self):{
    fn:{
        yaya();
              ^
You need at least one argument here (conventionally "self")
==================================================
Syntax error at line 8, column 8
LALR Parser state : 211


};};trait ERR2(self):{
    static_fn:{
        []() -> int;
        ^
You need to give a name to the function or the variable
==================================================
Syntax error at line 10, column 21
LALR Parser state : 331

    static_fn:{
        []() -> int;
        okay(int, bool);
        static_yaya[]() -> int;
                     ^
You need to put at least one argument in the [] parenthesis
==================================================
Syntax error at line 15, column 1
LALR Parser state : 219

};};trait ERR3(self):{
    fn:{

};};trait ERR4(self):{
 ^
Empty declaration statement
==================================================
Syntax error at line 17, column 8
LALR Parser state : 207


};};trait ERR4(self):{
    fn:{
        [][][][];
        ^
You need to give a name to the function or the variable
==================================================
Syntax error at line 22, column 46
LALR Parser state : 445


};};trait ERR5(self):{
    fn:{
        somewhat_err_fn(self, any, int, =int, int);
                                              ^
Invalid form of arguments
==================================================
Syntax error at line 24, column 44
LALR Parser state : 420

    fn:{
        somewhat_err_fn(self, any, int, =int, int);
    };static_fn:{
        somewhat_stt_err_fn(any, int, *, int, :int, a:=int);
                                            ^
Invalid form of arguments
==================================================
Syntax error at line 26, column 37
LALR Parser state : 452

    };static_fn:{
        somewhat_stt_err_fn(any, int, *, int, :int, a:=int);
    };fn:{
        err_fn2(self, any, int, *, a=int, b:=int, c:int, d=:int);
                                     ^
Invalid declaration of arguments : Did you mean ":" instead of "=" ?
==================================================
Syntax error at line 26, column 60
LALR Parser state : 484

    };static_fn:{
        somewhat_stt_err_fn(any, int, *, int, :int, a:=int);
    };fn:{
        err_fn2(self, any, int, *, a=int, b:=int, c:int, d=:int);
                                                            ^
Invalid declaration of arguments : Did you mean ":=" instead of "=:" ?
==================================================
Syntax error at line 27, column 26
LALR Parser state : 58

        somewhat_stt_err_fn(any, int, *, int, :int, a:=int);
    };fn:{
        err_fn2(self, any, int, *, a=int, b:=int, c:int, d=:int);
        err_fn3(self, int str float bool);
                          ^
Invalid form of arguments
==================================================
Syntax error at line 31, column 22
LALR Parser state : 341

    };cls_fn:{
        err_fn4(self) -> True;
        err_fn5(True, self);
        err_fn6(self, True, int) -> int;
                      ^
Invalid form of arguments
==================================================
Parsing completed successfully
```