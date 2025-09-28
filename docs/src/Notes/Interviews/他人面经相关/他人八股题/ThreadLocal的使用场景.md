### 网上找到的来自字节的面试

#### ThreadLocal有哪些使用场景？

场景一、使用ThreadLocal进行线程隔离

数据库连接独享Hibernate：判断当前线程中有没有放进去session，如果还没有，那么通过sessionFactory().openSession()来创建一个Session ，再将Session 设置到ThreadLocal变量中，这个Session 相当于线程的私有变量

```java
private static final ThreadLocal threadSession = new ThreadLocal();  

public static Session getSession() throws InfrastructureException {  
    Session s = (Session) threadSession.get();  
    try {  
        if (s == null) {  
            s = getSessionFactory().openSession();  
            threadSession.set(s);  
        }  
    } catch (HibernateException ex) {  
        throw new InfrastructureException(ex);  
    }  
    return s;  
}  
```

场景二、使用ThreadLocal进行跨函数数据传递

通过ThreadLocal在函数之间传递用户信息，会话信息等。

```java
package com.crazymaker.springcloud.common.context;
...省略import
public class SessionHolder
{
    // session id  线程本地变量
    private static final ThreadLocal<String> sidLocal = new ThreadLocal<>("sidLocal");

    // 用户信息  线程本地变量
    private static final ThreadLocal<UserDTO> sessionUserLocal = new ThreadLocal<>("sessionUserLocal");

    // session  线程本地变量
    private static final ThreadLocal<HttpSession> sessionLocal = new ThreadLocal<>("sessionLocal");
...省略其他  

    /**
     *保存session在线程本地变量中
      */
    public static void setSession(HttpSession session)
    {
        sessionLocal.set(session);
    }

    /**
     * 取得绑定在线程本地变量中的session 
      */
    public static HttpSession getSession()
    {
        HttpSession session = sessionLocal.get();
        Assert.notNull(session, "session 未设置");
        return session;
    }
    ...省略其他  
}
```

场景三、ThreadLocal在Java框架中的应用

Spring：
- Spring框架中，存储数据库连接等线程特定的资源。由于数据库连接是线程不安全的，因此每个线程都需要有自己的连接副本。Spring通过ThreadLocal将数据库连接与当前线程关联起来，从而避免了多线程环境下的数据竞争和不一致问题。
- Spring事务管理中，确保了每个线程都有自己的事务上下文，包括事务状态、回滚点等信息，从而实现了事务的隔离性。

MyBatis：
- 存储SqlSession对象。由于SqlSession不是线程安全的，因此每个线程都应该拥有自己独立的SqlSession实例。通过ThreadLocal，MyBatis可以方便地实现SqlSession的线程局部存储，确保每个线程都能正确地执行SQL操作。

分布式系统：
- 传递全局ID和分支ID等关键信息。这些ID对于分布式事务的追踪和诊断至关重要。通过将这些ID存储在ThreadLocal中，可以确保它们在整个请求处理过程中都能被正确传递和使用。

日志框架：
- 存储与当前线程相关的日志上下文信息，如用户ID、操作类型等。这样，在记录日志时，可以方便地获取这些信息，并将其添加到日志条目中，从而方便后续的日志分析和排查问题。

RPC：
- 存储和传递与当前调用相关的上下文信息。这些上下文信息可能包括调用者的身份、调用的参数、超时设置等。通过将这些信息存储在ThreadLocal中，可以确保它们在RPC调用过程中能够被正确地传递和使用。

Hibernate：
- SessionContext: 用于存储当前线程的Hibernate会话相关数据，如当前会话、持久化上下文等。
- TransactionManager: 管理事务状态，每个线程可以有独立的事务状态，如当前是否在事务中。

Tomcat:
- 跟踪每个请求的会话信息、用户认证数据等，确保这些数据不会在请求之间共享。

Kafka:
- Producer and Consumer Threads: 在消息生产和消费过程中，使用ThreadLocal来存储线程特定的配置和状态信息。

https://www.cnblogs.com/crazymakercircle/p/18130926#autoid-h3-3-4-0