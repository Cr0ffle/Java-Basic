# #8 예외처리(Exception Handling)

## 예외처리(Exception Handling)

### 프로그램 오류

- 프로그램이 실행 중 어떤 원인에 의해서 오작동을 하거나 비정상적으로 종료되는 경우가 있음
- 이를 발생 시점에 따라 컴파일에러와 런타임에러로 나눌 수 있음 + 논리적에러 (물건의 재고가 음수)
  - 컴파일 에러 : 컴파일 시에 발생하는 에러
  - 런타임 에러 : 실행 시에 발생하는 에러
  - 논리적 에러 : 실행은 되지만, 의도와 다르게 동작하는 것
- 자바에서는 런타임에서 발생할 수 있는 프로그램 오류를 Error(에러)와 Exception(예외) 두가지로 구분했음
- Error는 OOM이나 StackOverFlowError와 같이 일단 발생하면 복구할 수 없는 심각한 오류
- Exception은 발생하더라도 수습될 수 있는 비교적 덜 심각한 것
- Error가 발생하면 프로그램의 비정상적인 종료를 막을 길이 없지만 예외는 발생하더라도 프로그래머가 이에 대한 적절한 코드를 미리 작성해 놓음으로써 프로그램의 비정상적인 종료를 막을 수 있음
  - Error: 프로그램 코드에 의해서 수습될 수 없는 심각한 오류
  - Exception: 프로그램 코드에 의해서 수습될 수 있는 다소 미약한 오류

### 예외 클래스의 계층구조

![](https://user-images.githubusercontent.com/30790184/109766400-74d49100-7c39-11eb-93b8-3cd169c44c2f.png)

- RuntimeException하위 클래스들은 프로그래머의 실수에 의해서 발생될 수 있는 예외들로 자바의 프로그래밍 요소들과 관계가 깊음 ex: ArrayIndexOutOfBoundsException, NPE, etc...

### 예외 처리하기 try-catch문

- 예외처리란 프로그램 실행 시 발생할 수 있는 예기치 못한 예외의 발생에 대비한 코드를 작성하는 것임
- 예외처리의 목적은 예외의 발생으로 인한 실행 중인 갑작스런 비정상 종료를 막고, 정상적인 실행 상태를 유지할 수 있도록 하는 것
- 발생한 예외를 처리하지 못하면 프로그램은 비정상적으로 종료되고 처리되지 못한 예외는 jvm의 예외처리기(UncaughtExceptionHandler)가 받아서 예외의 원인을 화면에 출력함

> UncaughtExceptionHandler는 기본적으로 스택 트레이스를 프린트함 기본 구현체는 ThreadGroup 클래스
>
> ```java
> /**
>  * Called by the Java Virtual Machine when a thread in this
>  * thread group stops because of an uncaught exception, and the thread
>  * does not have a specific {@link Thread.UncaughtExceptionHandler}
>  * installed.
>  */
>     public void uncaughtException(Thread t, Throwable e) {
>         if (parent != null) {
>             parent.uncaughtException(t, e);
>         } else {
>             Thread.UncaughtExceptionHandler ueh =
>                 Thread.getDefaultUncaughtExceptionHandler();
>             if (ueh != null) {
>                 ueh.uncaughtException(t, e);
>             } else if (!(e instanceof ThreadDeath)) {
>                 System.err.print("Exception in thread \""
>                                  + t.getName() + "\" ");
>                 e.printStackTrace(System.err);
>             }
>         }
>     }
> ```
>
> setDefaultUncaughtExceptionHandler()로 커스텀 UncaughtExceptionHandler을 thread에 셋팅할 수 있음
>
> Thread.setDefaultUncaughtExceptionHandler()
>
> ```java
> public static void setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler eh) {
>     SecurityManager sm = System.getSecurityManager();
>     if (sm != null) {
>         sm.checkPermission(
>             new RuntimePermission("setDefaultUncaughtExceptionHandler")
>                 );
>     }
> 
>      defaultUncaughtExceptionHandler = eh;
>  }
> ```

- 예외를 처리하기 위해서는 try-catch문을 사용함

```java
try {
	// 예외가 발생할 가능성이 있는 문장들을 넣음
} catch (Exception1 e1) {
	// Exception1이 발생했을 경우 이를 처리하기 위한 문장을 넣음
} catch (Exception2 e2) {
  // Exception2이 발생했을 경우 이를 처리하기 위한 문장을 넣음
}
```

- 하나의 try 블럭과 n개의 catch 블럭이 올 수 있음
- 이 중 발생한 예외의 종류와 일치하는 단 한개의 catch 블럭만 수행됨
- 발생한 예외의 종류와 일치하는 catch블럭이 없으면 예외는 처리되지 않음
- 여러개의 catch문에 참조변수명은 겹치면 안됨

### try-catch문에서의 흐름

- try 블럭 내에서 예외가 발생한 경우.
  - 발생한 예외와 일치하는 catch블럭이 있는지 확인함
  - 일치하는 catch 블럭을 찾게 되면, 그 catch 블럭 내의 문장들을 수행하고 전체 try-catch문을 빠져나가서 그 다음 문장을 계속해서 수행함. 만일 일치하는 catch 블럭을 찾지 못하면 예외는 처리되지 못함
- try 블럭 내에서 예외가 발생하지 않은 경우.
  - catch블럭을 거치지 않고 try-catch문을 빠져나가서 수행을 계속함
- try 블럭에 포함시킬 코드의 범위를 잘 선택해야함

### 예외의 발생과 catch 블럭

- 예외가 발생하면 발생한 예외에 해당하는 클래스의 인스턴스가 만들어짐
- 첫번째 catch 블록부터 차례대로 내려가면서 catch에 선언된 예외와 생성된 예외 클래스의 인스턴스에 instanceof연산자를 이용해서 검사 결과가 true인 catch 블록을 만날때까지 하향식으로 검사함
- 검사 결과가 전부 false라면 예외는 처리되지 않음
- 모든 예외 클래스는 Exception 클래스의 자손이므로 catch 블럭의 괄호에 Exception 클래스 타입의 참조 변수를 선언해 놓으면 어떤 종류의 예외가 발생하더라도 catch 가능
- 예외가 발생했을 때 생성되는 예외 클래스의 인스턴스에는 발생한 예외에 대한 정보가 담겨 있음 
  - getMessage() : 발생한 예외클래스의 인스턴스에 저장된 메시지를 얻을 수 있음
  - printStackTrace() : 예외발생 당시 호출스택에 있었던 메서드의 정보와 예외 메시지를 화면에 출력

- jdk 1.7부터 다음과 같이 멀티 catch블럭을 사용할 수 있음

```java
try {

} catch (ExceptionA | ExceptionB e) {
	e.printStackTrace();
}
```

- | 기호로 연결된 예외 클래스가 조상-자손관계라면 컴파일에러 발생
- 멀티 catch에서의 특정 ExceptionA.methodA는 호출 불가( instaceof로 한번 묶으면 사용 가능 ), e 변수는 final

### 예외 발생시키기

- 키워드 throw를 사용해서 프로그래머가 고의로 예외를 발생시킬 수 있음

```java
Exception e = new Exception();
throw e;
```

- 컴파일러가 예외처리를 확인하지 않은 RuntimeException 클래스들은 Unchecked 예외라하고 컴파일러가 예외처리를 확인한다면Checked Exception 예외라고함

### 메서드에 예외 선언하기

- 예외를 처리하는 또다른 방법으로 throws를 사용할 수 있음

```java
void method() thorws Exception1, Exception2, ... ExceptionN {

}
```

- 발생할 수 있는 예외를 적어두면 메서드를 사용하려는 사람이 메서드의 선언부를 보았을 때, 어떤 예외들이 처리되어야하는지 쉽게 알 수 있음
- 일반적으로 RuntimeException 클래스들은 적지 않음
- throws는 자신을 호출한 메서드에게 예외를 전달하는 것 어디선가는 try-catch로 예외를 처리해야함
- 맨 마지막에 있는 main메서드에서도 예외가 처리되지 않으면 프로그램이 종료됨

### finally블럭

- finally 블럭은 예외의 발생 여부에 상관 없이 실행되어야 할 코드를 포함시킬 목적으로 사용됨(optional)

```java
try {
	// 예외가 발생할 가능성이 있는 문장들을 넣음
} catch (Exception e){
	// 예외 처리를 위한 문장을 적음
} finally {
	// 예외의 발생여부에 관계없이 항상 수행되어야하는 문장들을 넣음
}
```



> finally를 사용하는 Main클래스와 bytecode
>
> ```java
> public class Main {
> 
>     public static void main(String[] args) {
> 
>         try {
>             throw new RuntimeException();
>         } catch (Exception e) {
>             e.printStackTrace();
>         } finally {
>             System.out.println("finally");
>         }
>     }
> }
> ```
>
> ```
>  0 new #7 <java/lang/RuntimeException>
>  3 dup
>  4 invokespecial #9 <java/lang/RuntimeException.<init> : ()V>
>  7 athrow
>  8 astore_1
>  9 aload_1
> 10 invokevirtual #12 <java/lang/Exception.printStackTrace : ()V>
> 13 getstatic #15 <java/lang/System.out : Ljava/io/PrintStream;>
> 16 ldc #21 <finally>
> 18 invokevirtual #23 <java/io/PrintStream.println : (Ljava/lang/String;)V>
> 21 goto 35 (+14)
> 24 astore_2
> 25 getstatic #15 <java/lang/System.out : Ljava/io/PrintStream;>
> 28 ldc #21 <finally>
> 30 invokevirtual #23 <java/io/PrintStream.println : (Ljava/lang/String;)V>
> 33 aload_2
> 34 athrow
> 35 return
> 
> ```



### 자동 자원 반환 -try-with-resources문

- 주로 입출력에 사용되는 클래스 중에서는 사용한 후에 꼭 닫아줘야하는것들이 있는데 이때 try-with-resources를 사용하면 됨

```java
//일반 try-catch-finally

try {
  FileInputStream fis = new FileInputStream("data.dat")
  DataInputStream dis = new DataInputStream(fis)
} catch (Exception e) {
	
} finally {
  try {
    if(dis != null) {
      dis.close();
    }
  } catch (IOException ie) {
    
  }
}


//try-with-resources
try(FileInputStream fis = new FileInputStream("data.dat");
    DataInputStream dis = new DataInputStream("data.dat")) {

} catch (Exception e) {

}
```

- try-with-resources를 사용하면 자동으로 close()가 호출됨
- try-with-resources에의해 자동으로 객체의 close()가 호출될 수 있으려면 클래스가 AutoCloseable이라는 인터페이스를 구현한 것이어야만 함
- close() 내부에서 예외가 발생했다면 supressed(억제된) 이라는 의미의 머리말과 함께 출력됨



### 사용자정의 예외 만들기

- 프로그래머가 새로운 예외 클래스를 정의하여 사용할 수 있음

```java
class MyException extends Exception {
	MyException(String msg)() {
		super(msg);
	}
}
```

- unchecked예외 또는 checked예외를 상속받아 예외처리 여부를 선택하게 할 수 있음

### 예외 되던지기(exception re-throwing)

- 예외를 처리하고 인위적으로 다시 발생시킬 수 있음
- 이 방법은 하나의 예외에 대해서 예외가 발생한 메서드와 이를 호출한 메서드 양쪽 모두에서 처리해줘야 할 작업이 있을 때 사용됨

```java
class ExceptionEx {
	public static void main(String[] args) {
		try {
			method1();
		} catch (Exception e) {
		}
	}
	
	static void method1() throws Exception {
		try {
			throw new Exception();
		} else {
			throw e; //<= 예외 재발생
		}
	}
}
```

- 메서드 return 시그니처가 void가 아니라면 catch에서도 리턴을 해야하지만 예외 되던지기의 경우 return문을 대신할 수 있음.

  

### 연결된 예외(chained exception)

- 예외를 연결시켜서 한 예외가 다른 예외를 발생시킬수도 있음

```java
try {
	startInstall();
	copyFiles();
} catch (SpaceException e) {
	InstallException ie = new InstallException("설치중 예외 발생");
	ie.initCause(e);
	throw ie;
}
```

- 예외를 연결시키는 이유는 여러가지 예외를 하나의 큰 분류의 예외로 묶어서 다루기 위해서이고 checked예외를 한번 감싸서 unchecked예외로 던져서 예외처리를 선택적으로 하도록 할 수 있음


