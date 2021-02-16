---
layout: post
title: 어노테이션 프로세싱(Annotation Processing) - 1
date: 2021-02-16 21:33:17 +0900
category: Java
---
# 들어가기 전에
이 글은 [출처](http://hannesdorfmann.com/annotation-processing/annotationprocessing101/) 에서 읽은 글을 손번역하며 주관적인 생각도 덧붙여서 쓴 글이다.

# 어노테이션 프로세싱이란?
- 어노테이션을 이용하여 컴파일 단계에서 class 파일이 생성될 때, 소스를 자동으로 생성해주는 작업이다.
- boiler plate code 를 줄이기 위한 효과적인 방법이며 트렌디한 기술로 각광받고 있다. (lombok, jpql 등)
- javac에 의해 compile 중 동작한다.

## AbstractProcessor
Processor API 는 다음과 같으며, 모든 Processer는 AbstractProcessor를 상속받는다.

```java
package com.example;

public class MyProcessor extends AbstractProcessor {

	@Override
	public synchronized void init(ProcessingEnvironment env){ }

	@Override
	public boolean process(Set<? extends TypeElement> annoations, RoundEnvironment env) { }

	@Override
	public Set<String> getSupportedAnnotationTypes() { }

	@Override
	public SourceVersion getSupportedSourceVersion() { }

}
```
- init(ProcessingEnvironment env)   
	annotation processor의 생성자는 반드시 비어있어야 한다. 대신 init 메서드를 사용하여 ProcessingEnviroment 객체를 부여받는데, 이 객체에는 Elements, Types, Filer가 포함되어있다.
- process(Set<? extends TypeElement> annoations, RoundEnvironment env)   
	processor의 main()과 같은 역할이다. 코드를 탐색, 평가, 처리를 통해 java 파일을 생성할 수 있다. RoundEnvironment를 통해 특정 어노테이션 속성을 조건으로 조회한다.
- getSupportedAnnotationTypes()   
	annotation processor에 적합한 annotation을 말한다.
- getSupportedSourceVersion()   
	java 버전을 의미하며, 보통은 SourceVersion.latestSupported()를 return 한다.

**자바 7에서는 annotation으로 다음과 같이 표현할 수도 있으나, 많은 복잡성 이슈로(특히 안드로이드) @Override를 통해 작성하는 것을 권장한다.**
```java
@SupportedSourceVersion(SourceVersion.latestSupported())
@SupportedAnnotationTypes({
   // Set of full qullified annotation type names
 })
public class MyProcessor extends AbstractProcessor {

	@Override
	public synchronized void init(ProcessingEnvironment env){ }

	@Override
	public boolean process(Set<? extends TypeElement> annoations, RoundEnvironment env) { }
}
```

**다음으로, annotation processor가 자신의 jvm에서 실행된다는 것을 알아야 한다.**
javac가 java를 컴파일하면서 vm에서 annotation processor를 수행한다. 결국, java application 라면 모두 가능하다.
guava라이브러리를 사용할 것을 권장한다.
**단지, 아무리 작은 processor를 쓰더라도 효과적인 알고리즘과 디자인패턴을 사용해야 함을 명심해라.**

## Processor 등록하기

어떻게 MyProcessor를 javac에 등록하느냐면 .jar파일로 제공해야 한다. 추가적으로 javax.annotation.processing.Processor에 위치한 META-INF/services 또한 포함시켜 패키징해라. 그러면 다음과 같은 모양이 된다.

- MyProcessor.jar
	- com
		- example
			- MyProcessor.class

	- META-INF
		- services
			- javax.annotation.processing.Processor

MyProcessor.jar에 패키징된 javax.annotation.processing.Processor는 유효한 클래스 명의 processor들을 명시한다.

```java
com.example.MyProcessor
com.foo.OtherProcessor
net.blabla.SpecialProcessor
```

javac가 MyProcessor.jar의 buildpath를 자동으로 탐색하고 읽어서 MyProcessor를 annotation processor로 등록할 것이다.

다음 글에서는 예제를 통해 실습하겠다.
