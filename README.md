# Reference
이 글은 다음을 참조하여 작성되었습니다. 🙇🏻‍♂️
[TDD - TestDrivenDevelopment](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=37469717)


---

### 1. 객체의 생성에도 유의미한 이름을 사용하라

- 객체의 생성자가 오버로딩 되는 경우 어떠한 값으로 어떻게 생성되는지 정보가 부족할 수 있다. 
- 그러므로 이러한 경우에는 정적 팩토리 메소드를 사용하는 것이 보다 명확한 코드를 작성하게 된다. 
하지만 구현을 드러내는 이름은 피하는 것이 좋다. (p32 참고)

```java
// 두 번째 인자가 무엇인지 파악이 어렵다.
Product product = new Product("사과", 10000);

// 이름을 부여하여 두 번째 인자를 명확하게 파악할 수 있다.
Product product = Product.withPrice("사과", 10000);
```


---


### 2. 함수는 하나의 역할만 해야한다.
- 함수는 지정된 이름 아래에서 한 단계 수준의 추상화 수준을 유지해야 하며, 
그것이 하나의 역할 및 기능만을 하는 것 
- 또는 의미 있는 다른 함수로 추출가능한 부분이 없다면 그것 역시 하나의 역할 및 기능만을 수행하고 있는 것
- 예를 들어 다음과 같은 Switch 문은 정말 흔히 작성하는 코드이지만, 
많은 문제를 내포하고 있다. (p48 참고)

```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
  switch(e.type) {
    case COMMISSIONED:
      return calculateCommisionedPay(e);
    case HOURLY:
      return calculateHourlyPay(e);
    case SALARIED:
      return calculateSalariedPay(e);
    default:
      throw new InvalidEmployeeType(e.type);
  }
  
}
```

하지만 위의 코드는 다음과 같은 문제점 목록을 가지고 있다.


- 함수가 너무 길다. 새로운 직원 타입이 추가되면 더 길어질 것이다.
- 한 가지 작업만을 수행하지 않는다. 
해딩 직원이 어느 타입인지 확인하고 있다.
- SRP를 위반, 새로운 직원 타입이 추가되어도 임금을 계산하는 함수를 변경해야 합니다.
- OCP를 위반, 새로운 직원 타입이 추가되면 새로운 임금 계산 로직을 위하 코드를 변경해야 한다. 
- 유사한 함수가 계속 파생될 수 있다. 이러한 직원의 타입에 따른 코드는 다른 곳에 중첩될 수 있습니다.

이를 해결하기 위해서는 Employee를 추상클래스로 만들고, 직원 유형에 따른 하위 클래스를 선언하도록 해야한다. 물론 하위 객체를 생성하기 위한 switch문은 불가피하지만, 
그래도 유사한 함수마다 분기해주는 것을 처리해줄 수 있으며, 위의 문제점 중 상당수를 해결할 수 있을 것이다.

```java

public abstract class Employee {
    public abstract int calculatePay();
    public abstract void deliverPay();
}

public class EmployeeFactory {

    public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
        switch (r.type) {
            case COMMISSIONED:
                return new CommissionedEmployee(r);
            case HOURLY:
                return HourlyEmployee(r);
            case SALARIED:
                return SalariedEmployee(r);
            default:
                throw new InvalidEmployeeType(r.type);
        }
    }

}
```

---



### 3. 명령과 조회를 분리하라(Command와 Query의 분리) 
앞서 설명한대로 함수는 뭔가를 수행하거나 뭔가를 조회하거나 하나의 역할만을 해야 한다. 두 개의 역할을 동시에 하면 이상한 함수가 탄생하게 된다. 예를 들어 다음과 같이 key값이 존재하는지 확인하고, 존재하지 않으면 데이터를 추가하여 성공하면 true 실패하면 false를 반환하는 함수가 있다고 하자. 

public boolean set(String attribute, String value);

if(set("username", "MangKyu")) {

}
위의 코드를 접한 사람은 이 함수가 key가 존재하는 경우 overwrite하는지 혹은 존재하지 않을 경우에만 업데이트 하는지 등 자세한 내용을 알 수 없을 것이다. 그 이유는 위의 함수가 명령과 조회를 한번에 처리하기 때문이다. 그렇기 때문에 위의 함수를 분리하여 다음와 같이 작성해주는 것이 명확하다. (p56 참고)

```java

public boolean attributeExists(String attribute);
public boolean setAttribute(String attribute, String value);

if(attributeExists("username")) {
    setAttribute("username", "MangKyu");
}
```

---


### 4. 오류코드 보다는 예외를 활용하자
오류코드를 반환하면 그에 따른 분기가 일어나게 되고, 또 분기가 필요한 경우 중첩되기 마련이다.
```java
public Status deletePage(Page page) {
    if(deletePage(page) == E_OK) {
        if(registry.deleteReference(page.name) == E_OK) {
            if(configKeys.deleteKey(page.name.makeKey()) == E_OK) {
                log.info("page deleted");
                return E_OK;
            } else {
                log.error("config key not deleted");
            }
        } else {
            log.error("reference not deleted");
        }
    } else {
        log.error("page not deleted");
    }
    return E_ERROR;
}
```

이를 해결하기 위해 각각의 함수에서 예외를 발생시켜 잡는다면 코드를 더욱 간결하게 작성할 수 있다.
```java
public void deletePage(Page page) {
    try {
        deletePage(page);
        registry.deleteReference(page.name);
        configKeys.deleteKey(page.name.makeKey());
    } catch (Exception e) {
        log.error(e.getMessage());
    }
}
```

그리고 이렇게 처리하면 try-catch문이 생기게 되는데, 이 역시 분리하는 것이 코드를 이해하기 쉽게 도와줄 것이다.  (p58 참고)
``` java
public void deletePage(Page page) {
    try {
        deletePageAndAllReferences(page);
    } catch (Exception e) {
        log.error(e.getMessage());
    }
}

public void deletePageAndAllReferences(Page page) throws Exception {
    deletePage(page);
    registry.deleteReference(page.name);
    configKeys.deleteKey(page.name.makeKey());
}
 
```

---

### 5. 여러 예외가 발생하는 경우 Wrapper 클래스로 감싸자 
외부 라이브러리를 이용하면 다양한 예외 클래스를 마주하게 됨.
그리고 이러한 예외들을 처리하려면 다음과 같이 상당히 번거로워진다. (p135 참고)

``` java
ACMEPort port = new ACMEPort(12);
try {
    port.open();
} catch (DeviceResponseException e) {
    log.error(e.getMessage());
} catch (ATM1212UnlockedException e) {
    log.error(e.getMessage());
} catch (GMXError e) {
    log.error(e.getMessage());
} finally {
    ...
}
```
이러한 상황에서 Wrapper 클래스를 이용해 감싸면 효율적으로 예외 처리를 할 수 있습니다. (p136 참고)
``` java
LocalPort port = new LocalPort(12);
try {
    port.open();
} catch (PortDeviceFailure e) {
    log.error(e.getMessage());
} finally {
    ...
}

public class LocalPort {

    private ACMEPort innerPort;

    public LocalPort(int portNumber) {
        this.innerPort = new ACMEPort(portNumber);
    }

    public void open() {
        try {
            innerPort.open();
        } catch (DeviceResponseException e) {
            throw new PortDeviceFailure(e);
        } catch (ATM1212UnlockedException e) {
            throw new PortDeviceFailure(e);
        } catch (GMXError e) {
            throw new PortDeviceFailure(e);
        }
    }

}
```

---


### 6. 테스트 코드의 작성 
>
**TDD(Test-Driven Development)는 실제 코드를 짜기 전에 단위 테스트를 먼저 작성하는 기법**으로, 이를 통해 **유연성, 유지보수성, 재사용성을 제공**받을 수 있습니다. 
TDD의 핵심 규칙 3가지는 다음과 같습니다. (p155 참고)
>
- 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
- 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
- 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.

- 실제 코드를 변경한다는 것은 잠재적인 버그가 발생할 수 있음을 내포.. 
-> 테스트 코드가 있다면 변경된 코드를 검증함으로써 이를 해결할 수 있다. 
- 그리고 실제 코드가 변경되면 테스트 코드 역시 변경해주어야 하는데, 
이러한 이유로 테스트 코드 역시 가독성있게 작성하는 것이 필요하다..

테스트 코드를 작성할 때에는 다음을 준수하는 것이 좋다고 한다.
>
- 1개의 테스트 함수에 대해 assert를 최소화
- 1개의 테스트 함수는 1가지 개념 만을 테스트
또한 깨끗한 테스트 코드는 First라는 5가지 규칙을 따릅니다.
- Fast: 테스트는 빠르게 동작하여 자주 돌릴 수 있어야 합니다.
- Independent: 각각의 테스트는 독립적이며 서로 의존해서는 안됩니다.
- Repeatable: 어느 환경에서도 반복 가능해야 합니다.
- Self-Validating: 테스트는 성공 또는 실패로 bool 값으로 결과를 내어 자체적으로 검증되어야 합니다.
- Timely: 테스트는 적시에 즉, 테스트하려는 실제 코드를 구현하기 직전에 구현해야 합니다.

---

### 7. 클래스의 최소화 
클래스 역시 함수와 마찬가지로 간결하게 작성하는 것이 중요하다고 한다.
함수는 물리적 크기를 측정했다면 클래스는 몇개의 역할 또는 책임을 갖는지를 척도로 활용하며, 단일 책임 원칙에 따라 1가지 책임만을 가져야 한다.
(p173 참고)

---


### 8. 클래스의 응집도 
객체 지향 관련 공부를 하면 높은 응집도와 낮은 결합도라는 얘기를 듣기 마련이다.
여기서 응집도란 클래스의 메소드와 변수가 얼마나 의존하여 사용되는지를 의미한다. (p177 참고)

예를 들어 다음과 같은 Stack 클래스는 size()를 제외한 모든 함수에서 두 인스턴스 변수를 사용하므로 응집도가 아주 높아지게 된다.
```java
public class Stack {
    
    private int topOfStack=0;
    private List<Integer> elements = new LinkedList<>();
    
    public int size() {
        return topOfStack;
    }
    
    public void push(int element) {
        topOfStack++;
        elements.add(element);
    }
    
    public int pop() throws PoppedWhenEmpty{
        if(topOfStack == 0) {
            throw new PoppedWhenEmpty();
        }
        int element = elements.get(--topOfStack);
        elements.remove(topOfStack);
        return element;
    }
    
}
```

---

 

### 9. 변경하기 쉬운 클래스 
요구사항은 수시로 변하기 때문에, 변경하기 쉬운 클래스를 만드는 것이 중요하다.
변경하기 쉬운 클래스는 기본적으로 단일 책임 원칙을 지켜야 한다. 
또한 구현체 보다는 추상체에 의존하여야 합니다. 결국 핵심은 다형성이다.

이와 관련된 자세한 예제를 참고하고 싶다면 p186과 p189에서 참고한다.

``` java
abstract public class SQL {
    public SQL(String table, Column[] columns)
    abstract public String generate();
}

public class CreateSQL extends SQL {
    public CreateSQL(String table, Column[] columns)
    @Override public String generate()
}

public class SelectSQL extends SQL {
    public SelectSQL(String table, Column[] columns)
    @Override public String generate()
}
```



---


### 10. 설계 품질을 높여주는 4가지 규칙 

>
1. 모든 테스트를 실행하라: 테스트가 쉬운 코드를 작성하다 보면 SRP를 준수하고, 더 낮은 결합도를 갖는 설계를 얻을 수 있습니다.
2. 중복을 제거하라: 깔끔한 시스템을 만들기 위해 단 몇 줄이라도 중복을 제거해야 합니다.
3. 프로그래머의 의도를 표현하라: 좋은 이름, 작은 클래스와 메소드의 크기, 표준 명칭, 단위 테스트 작성 등을 통해 이를 달성할 수 있습니다.
4. 클래스와 메소드의 수를 최소로 줄여라: 클래스와 메소드를 작게 유지함으로써 시스템 크기 역시 작게 유지할 수 있습니다.

2~4는 리팩토링 과정에 해당한다. 2~4의 작업은 모든 테스트케이스를 작성한 후에 코드와 클래스를 정리하기 때문에 안전하다고 한다. (p216 참고)

 ---


### 11. 변경하기 쉬운 클래스 
예시로 작성된 클린코드(p247) 역시 바로 나온 것이 아니고, 
반복적인 정리 끝에 나온 코드라고 한다. 
결국 좋은 코드를 만들기 위해서는 많은 시간과 노력이 필요한 것..
앞으로 좋은 코드를 작성하기 위해 시간과 노력을 투자하는 것을 아끼지 말도록 하라고 한다.

 
---
 

### 12. 디미터 법칙 
디미터의 법칙 :  어떤 모듈이 호출하는 객체의 속사정을 몰라야 한다
그렇기에 객체는 자료를 숨기고 함수를 공개해야 한다.
만약 자료를 그대로 노출하면 내부 구조가 드러나 결합도가 높아지게 됩니다.

- 지나치게 객체의 속사정에 깊이 관여하는 코드 예시
```java
final String outputDir = FileManager.getInstance().getOptions().getModule().getAbsolutePath();
```

위와 같은 코드는 다음과 같이 나누는 것이 좋다.
```java
Options options = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```
그리고 만약 위의 클래스들인 자료 구조라면 괜찮다. 
하지만 객체라면 내부 구조를 숨겨야 하므로 한번 더 수정을 해주어야 한다. 

내부 코드를 자세히 살펴보니 위의 코드로 절대 경로를 얻는 이유는 임시 파일을 생성하기 위해서이다. 그렇기에 우리는 위의 코드를 다음과 같이 임시 파일을 생성하라는 메세지를 보내는 것이 좋다.
```java
BufferedOutputStream bos = ctxt.createScartchFileStream(classFileName);
```

즉, 데이터가 아닌 객체를 참고할 때 여러 번의 .을 사용하는 경우, 
객체에게 메세지를 보내도록 변경한다.
