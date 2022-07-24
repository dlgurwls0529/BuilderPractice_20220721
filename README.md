## 빌더 패턴(Builder Pattern)
### 배경
객체를 생성할 때, 필드를 생성자로 설정하는 방식과 setter 방식  
에서 발생하는 문제(후술)를 해결하기 위함이다.  
 
### 의미
객체를 생성하는 과정(생성자)과 객체를 조립하는 과정(파라미터로 필드 주입)  
을 분리하여, 동일한 절차를 따르나(인터페이스에 정의된 공통의 추상 메소드들)  
으로 다양한 결과의 객체를 만들 수 있게 하는 패턴(Concrete Builder 혹은  
오버라이딩 구현에 따라서 구현이 달라진다.)  

객체를 생성하는 과정과 조립 과정을 분리했다는 것이란  

    TourPlan tourPlan = new TourPlan(title, day ..);  
    
위의 생성자는 1)객체를 생성해서 반환하는 과정과, 2)그 필드를 주입하는  
역할을 동시에 하고 있다.  

    TourPlan tourPlan = new TourPlan();
    tourPlan.setTitle("칸쿤 여행");
    tourPlan.setDays(3);
    ...
    
위처럼 생성해서 반환하는 과정(첫 번째 줄)과 필드를 주입하는 과정(두번째 이하 줄)  
을 분리했다는 뜻이다.  

동일한 절차로 다양한 객체를 만들 수 있게 한다는 것이란  

    <TourPlanBuilder.interface>
    public void setTitle(String title);
    public void setDays(int day);
    public void setStartDate(LocalDate startDate);
    public void setWhereToStay(String whereToStay);
    public void setNights(int nights);
    ...
    public TourPlan build();
    
위처럼 동일한 과정들(setTitle, setDays, .. build)를 모두 밟되,  

    <LongTourPlanBuilder.class>
    public void build() {
      if(title == null || days == 0 || startDate == null
      || whereToStay == null || plans.isEmpty() || nights == 0) {
        throw new IllegalStateException();
      }
        
      return new TourPlan(title, days, ...);
    }
    
    <ShortTourPlanBuilder.class>
    public void build() {
      if(title == null || startDate == null) {
        throw new IllegalStateException();
      }
      
      return new TourPlan(title, null, null, ...)
    }
    
같은 build라는 절차이지만, Concrete Builder에 따라서 그 구현이 다르게 된다.  
위와같은 절차는 요구사항에 따라서 더 변화할 수 있다.  
(setTitle에서 제목을 인코딩해서 넣는다든지 하는)  
  
### 구현 방법
1. 공통 프로세스를 인터페이스 추상메소드로 정의한다. (메소드 체이닝)
3. Concrete Builder를 통해 각기 다른 표현을 만드는 방식을 구현한다.  
4. 자신이 원하는 객체의 형태에 따라서(Long Tour인지 Short Tour인지..)
   Concrete Builder를 설정하여 객체를 생성한다.  
   
빌더 패턴에도 필요에 따라 다양한 구현 방법이 있다.  
대표적으로, 필드 주입 순서를 강제하고 싶은 경우에는 다음과 같이 구현한다.  

    <Builder.interface>
    public Builder stepA();
    public Builder stepB();
    public Builder stepC();
    public Product build();
    
구현 후
    
    <StepABuilder.interface>
    public StepBBuilder stepA();
    
    <StepBBuilder.interface>
    public StepCBuilder stepB();
    
    <StepCBuilder.interface>
    public Builder stepC();
    
    <Builder.interface>
    public Product build();
    
    <ConcreteBuilder.class>
    public class ConcreteBuilder implements StepABuilder, StepBBuilder, StepCBuilder, Builder {

        private Object fieldA;
        private Object fieldB;
        private Object fieldC;

        @Override
        public Product build() {
            if(fieldA == null || fieldB == null || fieldC == null) {
                throw new IllegalArgumentException();
            }
            return new Product(fieldA, fieldB, fieldC);
        }

        @Override
        public StepBBuilder stepA(Object fieldA) {
            this.fieldA = fieldA;
            return this;
        }

        @Override
        public StepCBuilder stepB(Object fieldB) {
            this.fieldB = fieldB;
            return this;
        }

        @Override
        public Builder stepC(Object fieldC) {
            this.fieldC = fieldC;
            return this;
        }
    }

    <Client.class>
    Product product = new ConcreteBuilder()
        .stepA("A")
        .stepB("B")
        .stepC("C")
        .build();
        
써보면 알겠지만, stepA다음에는 stepB밖에 호출하지 못한다.  

혹은 빌더의 생성자로 필수 파라미터를 받고, 나머지를 
빌더 메소드로 받아도 된다.  

    <ConcreteBuilder.class>
    public ConcreteBuilder(String id) { ... };
    public ConcreteBuilder setField1(String field1) { ... };
    ...
    public Product build() { 
        if(id == null) throw new IllegalArgumentException()
        else return new Product(....)
    }
    
    <Client.class>
    Product product = new ConcreteBuilder(124)
            .setField1(...)
            . ...
            .build();

그냥 빌더 메소드로 돌려도 되지만, 저렇게 하는게 필수 파라미터  
임을 더 잘 나타내는 것 같다.  
    
### 장점
후술 

### 단점
후술

### lombok annotation
빌더 패턴을 직접 구현할 필요 없이, 어노테이션으로 대체할 수도 있다.  
클래스 위에다가 아래처럼 어노테이션 붙이면 된다.  

    @AllArgsConstructor(access = AccessLevel.PRIVATE) 
    // 전체 인자의 생성자를 private으로 만든다.
    @Builder(builderMethodName = "빌더 메소드 이름, builderMethod")
    // Builder패턴을 자동으로 생성해준다.  
    // 생성되는 빌더 메소드 이름을 적을 수도 있다.  
    
    <TravelCheckList.class>
    field = {id, passport, flightTicket, creditCard}
    
    public static TravelCheckListBuilder builder(Long id) {
        if(id == null) {
            throw new IllegalArgumentException("필수 파라미터 누락");
        }
        else {  
            return builderMethod().id(id);  
            // builderMethod() <= 이게 생성된 빌더 메소드(어노테이션에 명시)  
            // id는 필수 필드?니까, id를 먼저 설정한 빌더를 반환한다.  
        }
    }
    
    <Client.class>
    TravelCheckList travelCheckList = TravelCheckList.builder(145L)
            .passport("M12345")
            .flightTicket("Paris flight ticket")
            . ...
            .build();
            
    // 필수 필드 지정 안할꺼면 어노테이션에서 명시한
    // 빌더 메소드를 활용해서 아래처럼 해도 된다.  
    
    TravelCheckList travelCheckList = TravelCheckList.builderMethod()
            .id(145L)
            .passport("M12345")
            . ...
            .build()
    

### Builder 필드 없애는 법
빌더 메소드를 구현할 때, 해당 클래스에다가 Product의 필드를 전부 선언  
해야 하는 번거로움이 있다. 이때, 그냥 Product 인스턴스 자체를 필드로  
선언하고, 해당 인스턴스로 필드를 접근해도 된다. 가령,

    <ConcreteBuilder.class>
    private Product product;
    
    public ConcreteBuilder() {
        this.product = new Product();
    }
    
    ...
    
    public ConcreteBuilder setField1(String s) {
        this.product.setField1(s);
        return this
    }
    
    ...
    
근데 이렇게 하면, Product 클래스에서 해당 필드의 setter 메소드를  
구현해야 한다. (은닉이 안됨)  
그리고 어찌되었든 Product 클래스에 의존하는 부분이 하나 더 생기니까  
적절히 사용하도록 하자.  

### Director
디렉터는 꼭 필요한 것은 아니지만, 빌더를 써서 자주 만드는 객체가  
있다면, 동일한 코드를 만들지 않아도 된다. 그리고 복잡한 메소드  
체이닝을 숨길 수 있다. (그냥 로직 따로 뺀거다.)

    <Director.class>
    private TourPlanBuilder tourPlanBuilder;
    
    public Director(TourPlanBuilder tourPlanBuilder) {
        this.tourPlanBuilder = tourPlanBuilder;
    }
    
    public TourPlan createOregonTourPlan() {
        return tourPlanBuilder.setTitle("오레곤 롱비치 여행")
                 .setDays(3)
                 .setNights(2)
                 .build();
    }
    
    public TourPlan createCancunTourPlan() {
        return tourPlanBuilder.setTitle("칸쿤 여행")
                .setDays(3)
                .setNights(2)
                .setWhereToStay("신라호텔")
                .setCost(1535100000)
                .addPlan(0, "체크인하고 짐풀기")
                . ...
                .build();
    }
    
    <Client.class>
    TourPlan cancunTourPlan = new Director(new LongTourPlanBuilder())
            .createCancunTourPlan();
    TourPlan oregonTourPlan = new Director(new ShortTourPlanBuilder())
            .createOregonTourPlan();

<br>

## 객체 생성 패턴
객체를 생성하는 방법은 1. 생성자의 파라미터를 통해 인스턴스 필드를 주입  
2. 그냥 생성한 뒤, setter메소드로 필드를 주입이 있다. 3. 빌더 패턴은  
그 단점을 개선한 방법이라고 할 수 있다.  

### 생성자를 통해 필드를 주입하여 생성
생성자에다가 파라미터를 추가하고 그것으로 this.field = field처럼  
초기화를 하는 것이다.  

장점으로는 
- 변하지 않는 객체를 만들 수 있다. (setter 안만들어도 됨)
- 생성자 단 하나만으로 모든 생성(및 필드 주입) 과정을 끝낸다.  

단점으로는  
- 필드가 많으면 그에 따른 생성자를 많이 만들어야 한다.
- 필드가 많으면 몇번째 인자가 뭔지 헷갈린다. (비직관적)

### setter 메소드를 통해 필드를 주입하여 생성
생성자를 통해 그냥 객체를 만들고 setter 메소드로  
필드를 주입하여 객체를 만든다.  

장점으로는  
- getter, setter만 만들면 된다. 경우의 수 같은거 안신경써도 된다.  
- setter 함수의 이름을 통해서 어떤 필드를 주입하는지 쉽게 파악할 수 있다. (직관적)

단점으로는
- 변하지 않는 객체를 만들 수 없다. (setter를 통한 의도치 않은 접근 가능성)  
- 생성하고나서도 setter 함수를 여러개 작성해야 한다. (복잡함)

### Builder Pattern을 활용해 필드를 주입하여 생성
메소드 체이닝을 통해 객체를 생성 및 필드를 주입하는 Builder interface  
및 Concrete Builder를 추가하고, 거기에서 Product를 받는다.  

장점으로는  
- 변하지 않는 객체를 만들 수 있다. (Product 클래스에 setter 없어도 됨)  
- 단 하나의 작업? 한 번에 객체 생성을 마친다.  
- 어떤 변수를 주입하는지 쉽게 파악할 수 있다.  
- 필드의 경우의 수에 따라 생성자같은거 다 안만들어도 된다.  
- 유연하다. 객체마다 들어가야 할 인자가 다를 때 좋다. 가령,  

        Pen p = new Pen(1235, "red", "RedPen", "white", "star")
        // 제품번호, 색, 이름, 지우개 색, 무늬 종류

        // 지우게랑 무늬가 없는 펜을 만든다면 
        p = new Pen(1235, "red", "RedPen", null, null)
        
null 값처럼 어떤 값이든 꼭 넣어줘야 할 뿐더러 순서도 지켜야 한다.  
빌더는 안그래도 된다.  

        p = new penBuilder()
            .setNum(1235).setColor("red").setName("RedPen")
            .build();

        p = new penBuilder()
            .setColor("red").setName("RedPen").setNum(1235)
            .build();

생성자와 setter의 장점을 모두 취한 형태로 보면 될 것 같다.  

단점으로는
- 구조가 복잡하다. 

인자가 많으면 많을수록 Builder를 통해 얻는 효과가 커진다. 이런 상황에서라면  
웬만하면 쓰는 게 좋을 것 같다.  
그리고 위에서 설명했듯이 순서 강제나 빌더 생성자로 필수 파라미터를 받는 등  
정말 다양하게 패턴을 구현할 수 있다.  

<br>

## 메소드 체이닝
여러 메소드를 이어서 호출하는 방식이다. 메소드가 객체(보통 this)를 반환하여  
여러 메소드를 순차로 호출할 수 있게 한다. Builder Pattern에 적용된 것이 이거다.  

    const dbc = new DBConnector()
        .setHost('127.0.0.1')
        .setPort('8080')
        .setUser('yudakyum')
        .setPasswd('1234')
        .connect();

장점으로는 문장을 마치치 않고 호출이 가능하고, 가독성이 좋다.  
반면 에러가 발생한 경우에 정확한 지점을 찾기가 어려워진다.  
