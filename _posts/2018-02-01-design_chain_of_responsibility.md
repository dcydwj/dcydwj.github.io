---
layout: post
title: Chain of Responsibility Pattern
categories: [Design-Pattern]
description: 扯淡
keywords: design,架构
---
## 定义

是为了避免请求的发送者和接收者之间的耦合关系，使多个接收对象都有机会处理请求。将这些对象连城一条链，并沿着这条连
传递该请求，直到有一个对象处理它为止。这种类型的设计模式属于行为型模式。

## 介绍

意图：避免请求发送者与接收者耦合在一起，让多个对象都有可能接收请求，将这些对象连接成一条链，并且沿着这条链传递请求，直到有对象处理它为止。

主要解决：职责链上的处理者负责处理请求，客户只需要将请求发送到职责链上即可，无须关心请求的处理细节和请求的传递，所以职责链将请求的发送者和请求的处理者解耦了。

何时使用：在处理消息的时候以过滤很多道。

如何解决：拦截的类都实现统一接口。

关键代码：Handler 里面聚合它自己，在 HanleRequest 里判断是否合适，如果没达到条件则向下传递，向谁传递之前 set 进去。

应用实例： 1、红楼梦中的"击鼓传花"。 2、JS 中的事件冒泡。 3、JAVA WEB 中 Apache Tomcat 对 Encoding 的处理，Struts2 的拦截器，jsp servlet 的 Filter。

优点： 1、降低耦合度。它将请求的发送者和接收者解耦。 2、简化了对象。使得对象不需要知道链的结构。 3、增强给对象指派职责的灵活性。通过改变链内的成员或者调动它们的次序，允许动态地新增或者删除责任。 4、增加新的请求处理类很方便。

缺点： 1、不能保证请求一定被接收。 2、系统性能将受到一定影响，而且在进行代码调试时不太方便，可能会造成循环调用。 3、可能不容易观察运行时的特征，有碍于除错。

使用场景： 1、有多个对象可以处理同一个请求，具体哪个对象处理该请求由运行时刻自动确定。 2、在不明确指定接收者的情况下，向多个对象中的一个提交一个请求。 3、可动态指定一组对象处理请求。

注意事项：在 JAVA WEB 中遇到很多应用。

## 实现

话不多说，直接上代码,整体结构如下图：
![design_chain_of_responsibility.png](/imgs/design_chain_of_responsibility.png)

### Handler 代码

```java
public abstract class AbstractHandler {

    private AbstractHandler nextHandler;

    public final void handleRequest(AbstractRequest request) {
        if (this.getHandlerLevel() == request.getRequestLevel()) {
            this.handle(request);
        } else {
            if (this.nextHandler != null) {
                System.out.println("当前处理者 0" + this.getHandlerLevel() + "不足以处理 0" + request.getRequestLevel() + "请求");
                this.nextHandler.handleRequest(request);
            } else {
                System.out.println("都不能处理");
            }
        }

    }

    public AbstractHandler getNextHandler() {
        return nextHandler;
    }

    public void setNextHandler(AbstractHandler nextHandler) {
        this.nextHandler = nextHandler;
    }

    protected abstract int getHandlerLevel();

    protected abstract void handle(AbstractRequest request);

}


public class Handler01 extends AbstractHandler {
    @Override
    protected int getHandlerLevel() {
        return Levels.LEVEL_01;
    }

    @Override
    protected void handle(AbstractRequest request) {
        System.out.println("处理者01 处理" + request.getContent() + "\n");
    }
}

public class Handler02 extends AbstractHandler {
    @Override
    protected int getHandlerLevel() {
        return Levels.LEVEL_02;
    }

    @Override
    protected void handle(AbstractRequest request) {
        System.out.println("处理者02 处理" + request.getContent() + "\n");
    }
}

public class Handler03 extends AbstractHandler {
    @Override
    protected int getHandlerLevel() {
        return Levels.LEVEL_03;
    }

    @Override
    protected void handle(AbstractRequest request) {
        System.out.println("处理者03 处理" + request.getContent() + "\n");
    }
}
```

### request 代码
```java
public abstract class AbstractRequest {
    private String content;

    public AbstractRequest(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }

    public abstract int getRequestLevel();
}

public class Request01 extends AbstractRequest {

    public Request01(String content) {
        super(content);
    }

    @Override
    public int getRequestLevel() {
        return Levels.LEVEL_01;
    }
}

public class Request02 extends AbstractRequest {

    public Request02(String content) {
        super(content);
    }

    @Override
    public int getRequestLevel() {
        return Levels.LEVEL_02;
    }
}

public class Request03 extends AbstractRequest {

    public Request03(String content) {
        super(content);
    }

    @Override
    public int getRequestLevel() {
        return Levels.LEVEL_03;
    }
}

```

### client 代码

```java

public interface Levels {
    int LEVEL_01 = 1;
    int LEVEL_02 = 2;
    int LEVEL_03 = 3;
}


public class Client {
    public static void main(String[] args) {
        //创建职责连所有的节点
        AbstractHandler handler01 = new Handler01();
        AbstractHandler handler02 = new Handler02();
        AbstractHandler handler03 = new Handler03();

        handler01.setNextHandler(handler02);
        handler02.setNextHandler(handler03);

        AbstractRequest request01 = new Request01("请求01");
        AbstractRequest request02 = new Request02("请求02");
        AbstractRequest request03 = new Request03("请求03");

        handler01.handleRequest(request01);
        System.out.println("-----------------");
        handler01.handleRequest(request02);
        System.out.println("-----------------");
        handler01.handleRequest(request03);

    }
}
```
