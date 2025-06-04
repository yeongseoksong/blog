---
{"dg-publish":true,"permalink":"///visitor-pattern/"}
---

#### 장점
- 작업 대상(방문 공간) 은 단지 데이터를 담고있는 자료구조로 만들고,
- 작업 주체(방문자) 는 `visit()` 안에 이 작업 대상을 입력받아 작업 항목을 처리하면 된다.
- 즉, 데이터와 알고리즘이 분리되어, 데이터의 독립성을 높여준다.
- **OCP 를** 지킴
#### 단점
- 방문자와 방문공간 간의 결합도가 높아진다.


```java

public interface Visitor {  
    void visitCircle(Circle circle);  
    void visitRectangle(Rectangle rectangle);  
	/* 더블 디스패치
		visit(Circle circle)
		visit(Rectangle rectangle)
	*/
}

public class VisitorImpl implements Visitor{
    public void export(Shape ... args){  
        for(Shape shape:args){  
             shape.accept(this);  
        }    
    }    
        
        
	@Override  
    public void visitCircle(Circle circle) {  
        System.out.println("Circle Func Run . . .");  
        circle.print();  
  
    }  
    @Override  
    public void visitRectangle(Rectangle rectangle) {  
        System.out.println("Rec Func Run . . .");  
        rectangle.print();  
    }
}

```


```java
 interface Shape {  
	 //필수는 아님
    void print();
    // application 혹은 visitor 에서 호출  
    void accept(Visitor visitor); 
}

class Circle implements Shape{  
    @Override  
    public void print() {  
        System.out.println("Circle Hello");  
    }  
    @Override  
    public void accept(Visitor visitor) {  
         visitor.visitCircle(this);
         //더블 디스패치
         //visitor.visit(this);    
    }
}

class Rectangle implements Shape{  
    @Override  
    public void print() {  
        System.out.println("Rectangle Hello");  
    }  
    @Override  
    public void accept(Visitor visitor) {  
        visitor.visitRectangle(this); 
	   //더블 디스패치
	 //visitor.visit(this);  
    }
}


```



```java
  
public class Main {  
  
    public static void main(String[] args) {  
        Circle circle = new Circle();  
        Rectangle rectangle = new Rectangle();  
        VisitorImpl visitor = new VisitorImpl();  
        visitor.export(circle,rectangle);  

		/*for(Shape shape:args){  
             shape.accept(visior);  
        }*/

	// VisitorImpl2 가 추가 되면 VisitorImpl2 코드 작성만으로 기능 확장이 가능해지고 Shape 에는 변경사항이 없다.
		
  
    }}
```
