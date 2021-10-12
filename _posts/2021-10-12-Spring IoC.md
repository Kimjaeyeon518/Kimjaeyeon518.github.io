---
layout: post
title: Spring IoC (Inversion of Control)
date: 2021-10-11 23:18 +0800
# last_modified_at: 2021-10-12 23:19 +0800
tags: [Spring, IoC, DI]
toc: true
---

## 1. IoC/DI

**IoC** 는 **DI (Dependency Injection)** 라고 알려져 있으며, 객체에 의존성을 주입하는 것다.

스프링은 모든 객체에 대하여 스프링 컨테이너에 **스프링 빈**으로 등록해놓습니다.

**A Class**와 **B Class**가 있을 때, A Class에서 B Class에 대한 객체를 사용(의존)하려고 한다면 보통 개발자가
직접 **B b = new B()**와 같은 코드를 작성하여 의존성을 주입한 뒤 사용합니다.

하지만 스프링에선 위처럼 개발자가 직접 의존성을 주입하는 것이 아닌 스프링 컨테이너에 등록해놨던 **빈을 해당 객체에 주입**함으로써 객체를 사용합니다.

즉, 메서드나 객체에 대한 제어의 흐름을 사용자가 아닌 스프링이 컨트롤하기 때문에 객체를 제어하는 주체가 역전되었다고 해서 **Inversion of Control** 이라고 불립니다.

이처럼 스프링이 객체에 의존성을 주입하는 방법은 **Setter Injection, Field Injection, Constructor Injection**으로 총 3가지가 있는데, 이에 대한 코드는 다음과 같습니다.

> **스프링 빈**은 스프링 컨테이너에 등록될 때, 기본적으로 **싱글톤**으로 등록됩니다.
> 즉 객체에 대한 **스프링 빈을 유일하게 하나만 등록한 뒤 이를 다른 클래스에서 공유해서 사용하며, 이 때의 빈은 모두 동일한 인스턴스**입니다.

## 1.1. Setter Injection

{% highlight js %}
@Controller
public class HelloController {

    private HelloService helloService;

    @Autowired
    public void setHelloService(HelloService helloService) {
        this.helloService = helloService;
    }

}
{% endhighlight %}

### ※ Setter Injection 사용을 지양하는 이유

**Setter Injection** 으로의 의존성 주입은 스프링 **런타임 중**에 이루어지도록 **낮은 결합도**를 가지게 구현되었습니다.

하지만 이는 만약 **setHelloService()** 메서드로 **Service의 구현체를 주입해주는데 실패**하더라도 일단 **HelloController 객체는 생성이 가능한 상태**입니다.

이는 곧 **HelloController 객체**가 생성되어 내부에 있는 **Service의 method 호출이 가능**하다는 것인데, 위와 같이 Service 구현체가 주입되지 않았다면 이후 해당 객체를 사용하려고 할 때 **NullPointException**이 발생하게 됩니다.

**주입이 필요한 객체가 주입이 되지 않아도 얼마든지 객체를 생성할 수 있다는 것**은 문제가 되기 때문에 Setter Injection의 사용을 지양합니다.

## 1.2. Field Injection

{% highlight js %}
@Controller
public class HelloController {

    @Autowired
    private HelloService helloService;

}
{% endhighlight %}

### ※ Field Injection 사용을 지양하는 이유

**Field Injection**은 변수 선언부에 단순히 @Autowired Annotation만 붙이면 되기 때문에 의존성을 주입하기 매우 쉽다는 장점이 있습니다.

- **하지만 이는 동시에 단일 책임(SRP)의 원칙을 위반하는 단점으로도 여길 수 있습니다.**

의존성을 주입하기 쉬운 만큼 @Autowired 선언 아래 개수 제한 없이 무한정 추가할 수 있기 때문입니다.

만약 **Constructor Injection**을 사용하면 **Constructor의 parameter가 많아짐**과 동시에 하나의 Class가 많은 책임을 떠안는다는 걸 알게되므로 다른 injection 방법에 비해 더 큰 위기감을 느끼게 됩니다.

이러한 징조들은 **Refactoring을 해야한다는 신호**가 될 수 있기 때문에 **Constructor Injection 사용을 지향**합니다.

- **불변성(Immutability)을 보장하지 않습니다.**

**Constructor Injection**과 다르게 **Field Injection**은 객체에 **final을 선언할 수 없기 때문에** 객체가 변할 수 있습니다.

또한 **Setter Injection**과 마찬가지로 **의존성이 있는 객체를 주입하지 않아도 이를 포함하고 있는 객체가 생성 가능(컴파일 시 오류 X)**하기 때문에
이를 인식하지 못하다가 **런타임 시에 오류가 발생**할 수 있으므로 Field Injection 사용을 지양합니다.

## 1.3. Constructor Injection

{% highlight js %}
@Controller
public class HelloController {

    private final HelloService helloService;

    @Autowired
    public HelloController(HelloService helloService) {
        this.helloService = helloService;
    }

}
{% endhighlight %}

### ★ Constructor Injection 을 지향하는 이유

앞서 말한 Injection 타입들의 **단점들을 장점**으로 가져갈 수 있습니다.

- **의존성이 있는 객체를 주입하기 전까진 이를 포함하고 있는 객체를 생성할 수 없습니다. 즉, null 이 주입된 객체를 바로 알아채어 이에 대한 빠른 대처가 가능합니다**

- **final을 사용하여 객체의 불변성을 보장합니다.**

- **순환 의존성(Circular Dependency)**

Constructor Injection에서 순환 의존성을 가질 경우 BeanCurrentlyCreationExeption을 발생시킴으로써 순환 의존성 문제를 방지할 수 있습니다.

> **순환 의존성이란 ?**

> A Class가 B Class를 참조하는데 B Class가 다시 A Class를 참조할 경우,
> A Class가 B Class를 참조하고, B Class가 C Class를 참조하고 C Class가 A Class를 참조하는 경우 이를 **순환 의존성(Circular Dependency)**이라고 부릅니다.

### 결론 : 결국 더 좋은 디자인 패턴과 코드 품질을 위해서는 Constructor Injection을 사용하는 것이 좋습니다.
