# Style
---



## Bracing

**block { } 은 항상 라인의 처음에 위치 한다.**

Bad:
```cs
if (condition){
  Run();
}
```

Good:
```cs
if (condition)
{
  Run();
}
```
<br/>
**if 문 뒤에 한 라인의 문장만 있더라도 Brace안에 넣는다.**

Bad:
```cs
if (condition)
  Run();
```

Good:
```cs
if (condition)
{
  Run();
}
```
<br/>
**get/set 프로퍼티는 예외이다.**
```cs
public string Name
{
     get { return this.name; }
     set { this.name = value; }
}
```
<br/>
## 띄어쓰기
**메서드 파라미터들 사이에 공백을 준다**

Bad:
```cs
String.Compare(strA,indexA,strB,indexB,length);
```
Good:
```cs
String.Compare(strA, indexA, strB, indexB, length);
```

<br/>
**비교연산자 전후에 공백을 준다.**

Bad:
```cs
if(actual>expected)
```
Good:
```cs
if(actual > expected)
```
<br/>

## Name Casing


Casing | 설명 | 예제
------------ | ------------- | ------------
Camel | 첫 문자는 소문자, 계속되는 다음 단어의 첫문자는 대문자  | startIndex
Pascal | 첫 문자와 계속되는 다음 단어의 첫문자는 모두 대문자  | StartIndex

<br/>
**클래스명, 메서드명, 속성명, 이벤트명은 Pascal Case를 사용 한다.**

```cs
class Order // 클래스명
{
  public int OrderId // 속성 명
  { get; set; }

  public void PlaceOrder(int id) // 메서드 명
  {
  }
  public event EventHandler Completed; // 이벤트 명
}
```
<br/>
**필드명은 Camel Case를 사용 한다.**
```cs
class MyClass
{
    int startIndex;   //Camel Case
    string lastName;
}
```
<br/>
**메서드 파라미터는 Camel Case를 사용 한다.**
```cs
CallMethod(startIndex, endIndex);
```
<br/>
**로컬 변수명은 Camel Case를 사용 한다.**
```cs
public void Run()
{
    int startIndex; //로컬변수. Camel Case
}
```
<br/>
**public 상수(public const)와 public 읽기전용 정적 변수(public static readonly)는 Pascal Case를 사용한다.**
```cs
public void Order
{
    public const int MaxOrders = 30;
    public static readonly Tag;
}
```
<br/>
**Enum은 Pascal Case를 사용 한다.**
```cs
public enum OrderStatus
{
    Open,
    InDelivery
    CancelByCustomer
    //...
}
```
<br/>
## 일반적인 명명 규칙
**변수명 앞에 자료형을 나타내는 문자를 첨가하지 않는다. (Hungarian notation)**

Bad:
```
bLive, szName, iVal
```
<br/>
**멤버 필드에 대해 _, m_, s_ 같은 접두어(prefix)를 붙이지 않는다.**

Bad:
```cs
bool _success;
int m_index;
```
<br/>
**클래스 멤버 필드는 명시적으로 표시하기 위하여 this 키워드를 붙인다.**
```cs
this.name
```
<br/>
**인터페이스명은 항상 I로 시작한다.**
```cs
public interface IAlterable
```
<br/>
**Generics 클래스의 Generic 파라미터 타입은 가능하면 T와 같이 한 문자를 선택한다.**
```cs
public class MyList<T>
```
<br/>
**클래스명, enum명, 대리자(delegate)명에는 어떤 Prefix도 붙이지 않는다.**

Bad:
```cs
public enum EOrderType
```
<br/>
**클래스명, 메서드명, 속성명 등의 명칭에 축약된 단어를 사용하지 않는다.**

Bad:
```cs
public Device GetDev()
public Reader GetRdr()
```
Good:
```cs
public Device GetDevice()
public Reader GetReader()
```
<br/>
**꼭 필요하거나 널리 알려진 용어는 축약형을 사용할 수 있다.**
```
UserInterface -> UI , Database -> DB
```
<br/>
**메서드는 행위를 나타내므로 동사(verb)를 사용한다.**

Bad:
```cs
public int Addition(int a, int b)
```
Good:
```cs
public int Add(int a, int b)
```
<br/>
**필드나 속성 혹은 로컬변수는 명사(noun)를 사용한다.**

Bad:
```cs
private bool createTable
```
Good:
```cs
private bool newTable
```
<br/>
**이벤트는 가능하면 동사형으로 표현한다.**

Bad:
```cs
public event EventHandler Completion
```
Good:
```cs
public event EventHandler Completed
```
<br/>
**로컬변수는 가능하면 의미있는 단어를 선택한다.**

Bad:
```cs
int i, j
```
Good:
```cs
int startIndex, endIndex
```
<br/>
## 어셈블리, DLLs 명명
**System.Data와 같은 많은 기능을 제안하는 어셈블리 DLL 이름을 선택하라**

<br/>
**어셈블리 및 DLL 이름은 네임 스페이스 이름과 일치 할 필요는 없지만 어셈블리 이름은 네임 스페이스 이름을 따르는 것이 좋다.**

<br/>
**DLL 이름은 다음 패턴을 따르는것이 좋다.**
```
<Company>.<Component>.dll

ex) SHSystem.Controls.dll
```
<br/>
## 네임 스페이스 명명
