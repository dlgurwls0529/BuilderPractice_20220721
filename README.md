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

<br>

## 메소드 체이닝

