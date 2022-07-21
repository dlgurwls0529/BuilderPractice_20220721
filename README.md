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
### 장점
### 단점
### lombok annotation
### Builder 필드 없애는 법
### Director

<br>

## 객체 생성 패턴

<br>

## 메소드 체이닝
