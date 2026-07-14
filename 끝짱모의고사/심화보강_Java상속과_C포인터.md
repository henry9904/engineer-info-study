# 🧩 심화 보강 — Java 상속(parent/child · obj.필드) & C 포인터 완전 정복

> 7·8회를 비롯해 반복 출제되는 **Java 상속/필드 은닉**과 **C 포인터** 문제를 한곳에 모아,
> "왜 이렇게 되는지" 원리부터 라인별 재해설까지 정리한 보강 자료.
> 헷갈리는 `obj.필드`, `super.필드`, `((A)this).필드`, 이중 포인터를 여기서 끝냅니다.

---

# PART A. Java 상속 · 다형성 · 필드 은닉 완전 정복

## 0. ⭐ 딱 3줄 규칙 (이것만 외우면 90% 해결)

```
① 필드(변수)        → "선언 타입"으로 결정 (은닉, 다형성 X)
② 정적 메서드(static) → "선언 타입"으로 결정
③ 인스턴스 메서드     → "실제 객체 타입"으로 결정 (오버라이딩, 다형성 O)
```

- **선언 타입** = 변수를 선언할 때 왼쪽에 쓴 타입. `A obj = new C();` → 선언 타입 A
- **실제 타입** = `new`로 만든 실제 객체 타입. 위 예에서 실제 타입 C

> 💡 한 문장: **"필드는 왼쪽(선언), 메서드는 오른쪽(실제)"**. 정적 메서드만 예외로 왼쪽.

---

## 1. 왜 `obj.x`(필드)는 선언 타입 기준인가?

```java
class A { int x = 10; }
class C extends A { int x = 30; }   // A.x와 C.x는 "다른 변수"(이름만 같음 = 은닉)

A obj = new C();
System.out.println(obj.x);   // → 10  (C.x=30 아님!)
```

**핵심**: 부모와 자식이 같은 이름의 필드를 가지면, 자식 것이 부모 것을 덮는 게 아니라 **둘 다 메모리에 따로 존재**한다(필드 은닉, field hiding). 어느 것을 볼지는 **컴파일 시점의 선언 타입**으로 정해진다.

```
C 객체의 메모리:  [ A.x = 10 ][ C.x = 30 ]   ← 둘 다 존재
obj의 선언 타입이 A → A.x(10)를 봄
```

메서드는 다르다. 메서드는 오버라이딩되면 자식 것이 부모 것을 **진짜로 대체**한다(가상 함수 테이블).

---

## 2. `super.필드` vs `this.필드` vs `((부모)this).필드`

```java
class A { int x = 10; }
class B extends A { int x = 20; }
class C extends B {
    int x = 30;
    void test() {
        System.out.print(x);              // 30  (현재 클래스 C.x)
        System.out.print(this.x);         // 30  (this도 C.x)
        System.out.print(super.x);        // 20  (바로 위 부모 B.x)
        System.out.print(((A)this).x);    // 10  (A로 캐스팅 → A.x)
        System.out.print(((B)this).x);    // 20  (B로 캐스팅 → B.x)
    }
}
```

**정리**:
- `x`, `this.x` → 지금 코드가 있는 클래스의 필드 (C.x = 30)
- `super.x` → 바로 위 부모의 필드 (B.x = 20)
- `((타입)this).x` → **캐스팅한 타입**의 필드 (필드는 캐스팅 타입 기준!)

> ⚠️ 캐스팅으로 필드는 바뀌지만 **메서드는 안 바뀐다**. `((A)this).show()`는 여전히 실제 타입의 show()가 실행된다(메서드는 실제 타입).

### 🔎 23회 Q09 재해설 (이 규칙의 종합 문제)

```java
class A { int x=10; void show(){ print(x); } }              // A.show → A.x
class B extends A { int x=20;
    void show(){ print(x); print(super.x); }                 // B.show → B.x, A.x
    void print2(){ show(); print(x); } }                     // (문제상 메서드명 print)
class C extends B { int x=30;
    void show(){ print(x); print(super.x); print(((A)this).x); } }  // C.x, B.x, A.x

B obj = new C();
```

| 호출 | 무엇이 실행? | 출력 |
| -- | -------- | -- |
| `obj.x` | 필드 → 선언타입 **B** → B.x | **20** |
| `obj.show()` | 인스턴스 메서드 → 실제타입 **C** → C.show() | C.x(30), super.x=B.x(20), ((A)this).x=A.x(10) → **30 20 10** |
| `obj.print()`(B) | show() → 실제타입 C → C.show() → 30 20 10, 그 후 print(x)에서 x는 B.x=20 | **30 20 10 20** |

→ 최종: `20 30 20 10 30 20 10 20`

---

## 3. ⭐ 생성자 안에서 오버라이딩 메서드 호출 → 자식 필드는 아직 null/0

이게 7·8회를 포함해 **가장 많이 틀리는 함정**이다.

### 객체 생성 순서 (반드시 암기)

```
1) 부모 생성자 실행 (super())
2) 자식 필드 초기화 (int x = 30; 같은 것)
3) 자식 생성자 본문 실행
```

즉 **부모 생성자가 돌 때는 자식 필드가 아직 초기화되지 않은 상태(참조형 null, 정수 0)**.

```java
class Parent {
    Parent() { print(); }              // ← 자식 생성 중 여기서 print() 호출
    void print() { System.out.println("Parent"); }
}
class Child extends Parent {
    String msg = "Child";
    void print() { System.out.println(msg); }   // 오버라이딩
}
new Child();
```

**실행**:
```
Child() → super() Parent() → Parent() 안에서 print() 호출
   print()는 오버라이딩 → 실제 타입 Child → Child.print() 실행
   그런데 이 시점 Child.msg는 아직 초기화 전 → null
   → "null" 출력  (Child 아님!)
그 후 msg = "Child" 초기화됨
```

### 🔎 8회 Q15 완전 재해설 (super 체인 + 필드 은닉의 끝판왕)

```java
class A { int num=100; A(){ num+=50; } void show(){ print(num+","); } }   // A.num
class B extends A { int num=200; int val;
    B(){ val=num+30; }                                                     // B.num, B.val
    void show(){ print(val+","); } void superShow(){ super.show(); } }
class C extends B { int num=300; int val=400;
    C(){ super(); num+=50; val+=100; }
    void show(){ print(val+","); }
    void display(){ show(); super.show(); super.superShow(); print(num+","); println(super.num); } }
C c = new C();  c.display();
```

**1단계 — 각 필드가 어느 클래스 것이고 최종값은?**

| 필드 | 소속 | 초기화 과정 | 최종값 |
| -- | -- | -------- | -- |
| A.num | A | 100 → A() `num+=50` → 150 | **150** |
| B.num | B | 200 (그대로) | **200** |
| B.val | B | `val=num+30`에서 num은 **B.num=200** → 230 | **230** |
| C.num | C | 300 → C() `num+=50` → 350 | **350** |
| C.val | C | 400 → C() `val+=100` → 500 | **500** |

> ⚠️ B()의 `val=num+30`에서 num은 B 코드 안이므로 **B.num(200)**. 결과 val=230.

**2단계 — display() 라인별** (`c`는 C 객체)

| 코드 | 실행 위치 판단 | 값 |
| -- | -------- | -- |
| `show()` | 인스턴스 메서드, 실제타입 C → C.show() → C.val | **500,** |
| `super.show()` | B.show() → B.val | **230,** |
| `super.superShow()` | B.superShow() → super.show() = **A.show()** → A.num | **150,** |
| `print(num+",")` | display는 C에 있음 → num = C.num | **350,** |
| `println(super.num)` | super = B → B.num | **200** |

→ 최종: `500,230,150,350,200`

> 🎯 포인트: `super.superShow()`가 A.show()까지 두 단계 올라가 **A.num(150)**을 출력하는 게 함정. 필드는 코드가 있는 클래스 기준.

---

## 4. this() / super() 생성자 체인

- `this(...)` = **같은 클래스**의 다른 생성자 호출
- `super(...)` = **부모** 생성자 호출
- 둘 다 **생성자의 첫 줄**에만 올 수 있음

### 🔎 11회 Q15 / 19회 Q14 유형

```java
class A { A(){ this(10); print("A"); } A(int n){ print("B"); } }
// new A() → this(10) → A(int){"B"} 먼저 → 돌아와서 "A"
// 출력: BA
```

핵심: `this(10)`을 만나면 **그 생성자를 먼저 끝까지 실행**하고, 돌아와서 나머지를 실행.

---

## 5. 정적 필드 은닉 (Parent.total ≠ Child.total)

```java
class Parent { static int total = 0; }
class Child extends Parent { static int total = 10; }   // 이름만 같은 별개 변수!
```

`Parent.total`과 `Child.total`은 **완전히 다른 저장 공간**. `new Child()`가 `super()`로 Parent 생성자를 타면 Parent.total도 바뀌고, Child 본문에서 total++하면 Child.total이 바뀐다. (13회 Q19 참고)

---

## 6. 정적 메서드는 오버라이딩이 아니라 "은닉"

```java
class A { static void sm(){ print("SA"); } }
class C extends A { static void sm(){ print("SC"); } }
A obj = new C();
obj.sm();   // → "SA"  (정적 메서드는 선언 타입 A 기준!)
```

정적 메서드는 객체가 아니라 클래스에 속하므로 **선언 타입**으로 결정된다(17회 Q5).

---

## 7. ✅ Java 상속 문제 자가진단 체크리스트

문제를 만나면 순서대로 물어보세요:

```
Q1. 접근하는 게 필드인가 메서드인가?
    필드 → 선언 타입(변수 왼쪽) 기준. 끝.
    메서드 → Q2로.
Q2. 정적(static) 메서드인가?
    Yes → 선언 타입 기준.
    No(인스턴스) → 실제 객체 타입 기준(오버라이딩).
Q3. 생성자 안에서 오버라이딩 메서드를 호출하나?
    Yes → 그 시점 자식 필드는 미초기화(null/0)일 수 있음! 순서 확인.
Q4. super.x / ((타입)this).x 가 있나?
    super.x = 바로 위 부모 필드.
    ((타입)this).x = 캐스팅한 그 타입의 필드.
```

---

# PART B. C 포인터 완전 정복

## 0. 포인터 기본 기호

```c
int a = 10;
int *p = &a;    // p는 a의 주소를 저장
*p;             // p가 가리키는 값 = 10 (역참조)
&a;             // a의 주소
p;              // 주소 그 자체
```

- `&` = 주소 구하기, `*` = 그 주소의 값 꺼내기(역참조)
- `*p = 20;` → p가 가리키는 곳(a)에 20 저장 → a도 20이 됨

---

## 1. 배열과 포인터의 관계 (핵심 등식)

```c
int arr[5] = {10,20,30,40,50};
arr        ==  &arr[0]        // 배열 이름 = 첫 원소 주소
arr[i]     ==  *(arr + i)     // ⭐ 이 등식이 전부
*(arr + 2) ==  arr[2]  ==  30
```

포인터 산술 `arr + i`는 "바이트 + i"가 아니라 **"i번째 원소로 이동"**(자료형 크기만큼 자동 계산).

```c
int *p = arr;
*(p + 2)   // arr[2] = 30
p[2]       // 동일하게 arr[2] = 30  (포인터도 [] 사용 가능)
```

---

## 2. ⭐ 포인터 배열 vs 배열 포인터 (가장 헷갈림)

```c
int *p[3];     // 포인터 "배열": int* 3개가 든 배열. p[0], p[1], p[2]가 각각 int 주소
int (*p)[3];   // 배열 "포인터": int[3] 배열을 가리키는 포인터 1개
```

읽는 법: **`[]`가 `*`보다 우선** → `*p[3]`은 `*(p[3])` = "p는 배열, 원소는 포인터".
`(*p)[3]`은 괄호로 `*p` 먼저 = "p는 포인터, 가리키는 건 배열".

### 예: 포인터 배열 (7회 Q11, 13회 Q15 유형)

```c
char *arr[] = {"pointer", "array", "language"};  // 문자열 3개의 시작 주소 배열
char **p = arr;                                   // 이중 포인터로 가리킴
*(p+1)      // arr[1] = "array" (문자열)
*(*(p+2)+1) // arr[2]="language"의 [1] = 'a'
```

---

## 3. 이중 포인터 `**` (한 단계씩 벗기기)

```c
int a = 10;
int *p = &a;     // p → a
int **pp = &p;   // pp → p → a

*pp     // p (주소)
**pp    // *p = a = 10
```

**요령**: `**pp`는 "별을 하나씩 벗긴다"고 생각. `*pp` = 한 겹 벗김(p), `**pp` = 두 겹 벗김(a).

### 예: 2차원 배열을 포인터 배열로 (11회 Q2, 21회 Q13)

```c
int data[4][4] = {...};
int *matrix[4];
for (int i=0;i<4;i++) matrix[i] = data[i];   // 각 행의 시작 주소
int **m = matrix;

*(*(m + v) + i)   // m[v][i] = data[v][i]
```

`*(m+v)` = matrix[v] = v행 주소, `+i` 후 `*` = 그 행의 i열 값.

---

## 4. 포인터 산술 — 문자열 / 구조체

### 문자열 포인터 (3회 Q15, 5회 Q16)

```c
const char *g = "DC";
g;        // 'D'의 주소, 즉 "DC"
g + 1;    // 'C'의 주소, 즉 "C"
printf("%s", g+1);   // "C" 출력 (g+1부터 끝까지)
*(g+1);   // 'C' 하나 (문자)
```

> ⚠️ `%s`는 "그 주소부터 널까지" 전체 문자열, `%c`는 "그 위치 한 글자".

### 이중 포인터로 문자열 배열 (5회 Q16)

```c
char *pArr[] = {"SYSTEM","MEMORY","MODULE"};
char **dp = pArr;
*(dp+1)       // pArr[1] = "MEMORY" (문자열)
*(*(dp+1)+2)  // "MEMORY"[2] = 'M' (문자)
*(dp+2)+3     // "MODULE"+3 = "ULE" (문자열)
**dp + 1      // 'S' + 1 = 'T'  ← 문자 아스키 연산! (문자열 아님)
```

> 🎯 함정: `**dp + 1`은 `(**dp) + 1` = 문자 'S'(83) + 1 = 84 = 'T'. `%c`로 출력.

---

## 5. 함수 인자 — 값 전달 vs 포인터(참조) 전달

```c
void byValue(int x)  { x = 100; }        // 복사본만 바뀜 (원본 무변)
void byPointer(int *x){ *x = 100; }      // 원본이 바뀜

int a = 5;
byValue(a);    // a는 여전히 5
byPointer(&a); // a = 100
```

### 이중 포인터를 인자로 (연결 리스트 head 변경)

```c
void push(Node **head, int v){   // head 자체를 바꿔야 하니 **
    Node *n = malloc(...);
    n->next = *head;
    *head = n;                    // 호출자의 head가 실제로 바뀜
}
push(&head, 10);   // &head로 넘김
```

> 💡 "함수 안에서 포인터 변수 자체(head)를 바꾸려면 그 포인터의 주소(이중 포인터)를 넘긴다." 연결 리스트 삽입/삭제에서 필수 패턴.

### static 지역변수 (12회 Q4, 23회 Q11)

```c
int calc(){ static int s = 1; s += 2; return s; }
// 호출할 때마다 s가 "유지"됨: 첫 호출 3, 둘째 5, 셋째 7...
```

static 지역변수는 함수가 끝나도 사라지지 않고 **하나만 유지**된다. 여러 포인터가 그 주소를 받으면 전부 같은 변수를 가리킨다.

---

## 6. ⭐ 포인터 증감 연산 우선순위 (18회 Q10 같은 극악 문제)

```
후위 ++/-- > 역참조 * > 전위 ++/--
```

```c
*p++      // *(p++) : 현재 *p 사용 후 p 이동
(*p)++    // p가 가리키는 값을 증가
*++p      // p 먼저 이동 후 역참조
++*p      // *p 값을 먼저 증가
```

> ⚠️ 이런 문제는 시험장에서 **한 연산자씩 우선순위대로 천천히** 풀어야 함. 헷갈리면 괄호로 묶어 해석.

---

## 7. ✅ C 포인터 문제 자가진단 체크리스트

```
1. arr[i]는 무조건 *(arr+i)로 바꿔 생각.
2. 선언을 [] 우선으로 읽기: int *p[] = 포인터 배열, int (*p)[] = 배열 포인터.
3. **pp는 별을 하나씩 벗긴다(한 겹=주소, 두 겹=값).
4. %s는 "주소부터 널까지", %c는 "한 글자", 문자+1은 아스키 연산.
5. 함수에서 원본 바꾸려면 주소(&) 전달, 포인터 변수 자체 바꾸려면 이중 포인터.
6. static 지역변수는 호출 간 유지 → 여러 포인터가 같은 것 가리킴.
7. 증감 우선순위: 후위++ > * > 전위++. 천천히 한 개씩.
```

---

# 📌 관련 회차 문제 빠른 링크

### Java 상속·필드 은닉
- [3회 Q4](./3회.md) 생성자+필드 초기화 · [5회 Q3](./5회.md) static 바인딩 · [6회 Q12](./6회.md) 생성자 가상 호출
- [8회 Q15](./8회.md) ⭐super 체인 종합 · [9회 Q17](./9회.md) 3단계 생성자 · [11회 Q19](./11회.md) 필드 은닉 종합
- [13회 Q19](./13회.md) 정적 필드 은닉 · [17회 Q5](./17회.md) 오버라이딩 vs 정적 · [19회 Q4](./19회.md) 3단계
- [22회 Q14](./22회.md) 생성자 가상 호출 · [23회 Q9](./23회.md) ⭐캐스팅 필드 접근

### C 포인터
- [7회 Q11](./7회.md) 이중 포인터 문자열 · [10회 Q2](./10회.md) 이중 포인터 산술 · [11회 Q2](./11회.md) 인접행렬
- [13회 Q15](./13회.md) 문자열 배열 · [18회 Q10](./18회.md) ⭐증감 연쇄 · [21회 Q13](./21회.md) 이중 포인터+단락
- [22회 Q1](./22회.md) 포인터 배열 이동 · [23회 Q11](./23회.md) static+포인터 · [5회 Q16](./5회.md) 문자 아스키 연산

> 원리(PART A/B) → 위 문제들로 반복 연습하면 상속·포인터 문제는 완전히 잡힙니다.
