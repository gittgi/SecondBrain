---
tags:
  - CS_지식의_정석
---
# static

```table-of-contents
```

##  static이란?

- 스태틱 키워드는 클래스의 [인스턴스가 아닌 클래스](클래스,%20객체%20그리고%20인스턴스.md)에 속함
- 클래스의 변수, 메서드 등을 공유하는데 사용
	- 각 인스턴스마다 중복적으로 할당될 변수, 메서드 들을 클래스 차원에서 정의하여 한번만 메모리에 할당
- 따라서 해당 클래스로 만들어지는 객체 사이에서 중복되는 메서드, 속성을 효율적으로 정의할 때 쓰임
	- 단순히 전역변수인 것이 아니라 클래스 내의 static 키워드로 선언하여 이 클래의 객체들끼리 사용되는 메서드 또는 속성 -> 이러한 점을 **명시적으로 나타내주는 장점**도 있음


## 코드
```java
public class Person { 
	// 멤버변수(속성)
	String name;  
	int IQ;  
	int str;
	private static final String together = "happy";

	// constructor  
	public Person(String name, int IQ, int str){
		this.name = name; 
		this.IQ = IQ; 
		this.str = str;
	}  
	public Person(){
		this.name = "markus"; this.IQ = 100;  
		this.str = 100;
	}

	// 메서드  
	public void levelup(){
	
		this.IQ = this.IQ + 1;  
		this.str = this.str + 1;  
		System.out.println(this.name + "의 지능과 힘이 증가했습니다" + this.IQ + " / " + this.str); 
	}
	private static void talk(Person a, Person ) {
		System.out.println(a.name + " & " + b.name + "이 대화를 시작");
	}

	public static void main(String[] args) { 
		Person a = new Person(); // 객체 >> 인스턴스 a.levelup();
	
		Person b; // 객체  
		b = new Person("gittgi", 1, 1000); // 인스턴스 b.levelup();
		Person.talk(a, b)
		System.out.println(Person.together);
	} 
}
```


## static 단점

- 스태틱 키워드로 선언된 변수, 블록, 메서드 등은 선언과 동시에 (heap 영역이 아닌) 미리 Method area 메모리 영역에 할당
- 프로그램이 종료될 때까지 GC에 의해 메모리가 회수되지 않기 때문에, 만약 클래스가 객체로 쓰이지 않는다면 메모리 낭비
