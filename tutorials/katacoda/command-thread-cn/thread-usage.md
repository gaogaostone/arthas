
### 支持一键展示当前最忙的前N个线程并打印堆栈：

`thread -n 3`{{execute T2}}

```bash
$ thread -n 3
"as-command-execute-daemon" Id=29 cpuUsage=75% RUNNABLE
    at sun.management.ThreadImpl.dumpThreads0(Native Method)
    at sun.management.ThreadImpl.getThreadInfo(ThreadImpl.java:440)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand$1.action(ThreadCommand.java:58)
    at com.taobao.arthas.core.command.handler.AbstractCommandHandler.execute(AbstractCommandHandler.java:238)
    at com.taobao.arthas.core.command.handler.DefaultCommandHandler.handleCommand(DefaultCommandHandler.java:67)
    at com.taobao.arthas.core.server.ArthasServer$4.run(ArthasServer.java:276)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1145)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:615)
    at java.lang.Thread.run(Thread.java:745)
 
    Number of locked synchronizers = 1
    - java.util.concurrent.ThreadPoolExecutor$Worker@6cd0b6f8
 
"as-session-expire-daemon" Id=25 cpuUsage=24% TIMED_WAITING
    at java.lang.Thread.sleep(Native Method)
    at com.taobao.arthas.core.server.DefaultSessionManager$2.run(DefaultSessionManager.java:85)
 
"Reference Handler" Id=2 cpuUsage=0% WAITING on java.lang.ref.Reference$Lock@69ba0f27
    at java.lang.Object.wait(Native Method)
    -  waiting on java.lang.ref.Reference$Lock@69ba0f27
    at java.lang.Object.wait(Object.java:503)
    at java.lang.ref.Reference$ReferenceHandler.run(Reference.java:133)
```

### 当没有参数时，显示所有线程的信息。

`thread`{{execute T2}}

```bash
$ thread
Threads Total: 16, NEW: 0, RUNNABLE: 7, BLOCKED: 0, WAITING: 5, TIMED_WAITING: 4, TERMINATED: 0
ID         NAME                             GROUP                 PRIORITY   STATE      %CPU       TIME       INTERRUPTE DAEMON
30         as-command-execute-daemon        system                9          RUNNABLE   72         0:0        false      true
23         as-session-expire-daemon         system                9          TIMED_WAIT 27         0:0        false      true
22         Attach Listener                  system                9          RUNNABLE   0          0:0        false      true
11         pool-2-thread-1                  main                  5          TIMED_WAIT 0          0:0        false      false
12         Thread-2                         main                  5          RUNNABLE   0          0:0        false      true
13         pool-3-thread-1                  main                  5          TIMED_WAIT 0          0:0        false      false
25         as-selector-daemon               system                9          RUNNABLE   0          0:0        false      true
14         Thread-3                         main                  5          TIMED_WAIT 0          0:0        false      false
26         pool-5-thread-1                  system                5          WAITING    0          0:0        false      false
15         Thread-4                         main                  5          RUNNABLE   0          0:0        false      false
1          main                             main                  5          WAITING    0          0:2        false      false
2          Reference Handler                system                10         WAITING    0          0:0        false      true
3          Finalizer                        system                8          WAITING    0          0:0        false      true
4          Signal Dispatcher                system                9          RUNNABLE   0          0:0        false      true
20         NonBlockingInputStreamThread     main                  5          WAITING    0          0:0        false      true
21         Thread-8                         main                  5          RUNNABLE   0          0:0        false      true
```

### thread id， 显示指定线程的运行堆栈

查看线程ID 16的栈：

`thread 16`{{execute T2}}

```bash
$ thread 1
"main" Id=1 WAITING on java.util.concurrent.CountDownLatch$Sync@29fafb28
    at sun.misc.Unsafe.park(Native Method)
    -  waiting on java.util.concurrent.CountDownLatch$Sync@29fafb28
    at java.util.concurrent.locks.LockSupport.park(LockSupport.java:175)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.parkAndCheckInterrupt(AbstractQueuedSynchronizer.java:836)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.doAcquireSharedInterruptibly(AbstractQueuedSynchronizer.java:997)
    at java.util.concurrent.locks.AbstractQueuedSynchronizer.acquireSharedInterruptibly(AbstractQueuedSynchronizer.java:1304)
    at java.util.concurrent.CountDownLatch.await(CountDownLatch.java:231)
```

### thread -b, 找出当前阻塞其他线程的线程

有时候我们发现应用卡住了， 通常是由于某个线程拿住了某个锁， 并且其他线程都在等待这把锁造成的。 为了排查这类问题， arthas提供了`thread -b`， 一键找出那个罪魁祸首。

`thread -b`{{execute T2}}

```bash
$ thread -b
"http-bio-8080-exec-4" Id=27 TIMED_WAITING
    at java.lang.Thread.sleep(Native Method)
    at test.arthas.TestThreadBlocking.doGet(TestThreadBlocking.java:22)
    -  locked java.lang.Object@725be470 <---- but blocks 4 other threads!
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:624)
    at javax.servlet.http.HttpServlet.service(HttpServlet.java:731)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:303)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.apache.tomcat.websocket.server.WsFilter.doFilter(WsFilter.java:52)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at test.filter.TestDurexFilter.doFilter(TestDurexFilter.java:46)
    at org.apache.catalina.core.ApplicationFilterChain.internalDoFilter(ApplicationFilterChain.java:241)
    at org.apache.catalina.core.ApplicationFilterChain.doFilter(ApplicationFilterChain.java:208)
    at org.apache.catalina.core.StandardWrapperValve.invoke(StandardWrapperValve.java:220)
    at org.apache.catalina.core.StandardContextValve.invoke(StandardContextValve.java:122)
    at org.apache.catalina.authenticator.AuthenticatorBase.invoke(AuthenticatorBase.java:505)
    at com.taobao.tomcat.valves.ContextLoadFilterValve$FilterChainAdapter.doFilter(ContextLoadFilterValve.java:191)
    at com.taobao.eagleeye.EagleEyeFilter.doFilter(EagleEyeFilter.java:81)
    at com.taobao.tomcat.valves.ContextLoadFilterValve.invoke(ContextLoadFilterValve.java:150)
    at org.apache.catalina.core.StandardHostValve.invoke(StandardHostValve.java:170)
    at org.apache.catalina.valves.ErrorReportValve.invoke(ErrorReportValve.java:103)
    at org.apache.catalina.core.StandardEngineValve.invoke(StandardEngineValve.java:116)
    at org.apache.catalina.connector.CoyoteAdapter.service(CoyoteAdapter.java:429)
    at org.apache.coyote.http11.AbstractHttp11Processor.process(AbstractHttp11Processor.java:1085)
    at org.apache.coyote.AbstractProtocol$AbstractConnectionHandler.process(AbstractProtocol.java:625)
    at org.apache.tomcat.util.net.JIoEndpoint$SocketProcessor.run(JIoEndpoint.java:318)
    -  locked org.apache.tomcat.util.net.SocketWrapper@7127ee12
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at org.apache.tomcat.util.threads.TaskThread$WrappingRunnable.run(TaskThread.java:61)
    at java.lang.Thread.run(Thread.java:745)
 
    Number of locked synchronizers = 1
    - java.util.concurrent.ThreadPoolExecutor$Worker@31a6493e
```

**注意**， 目前只支持找出synchronized关键字阻塞住的线程， 如果是`java.util.concurrent.Lock`， 目前还不支持。

### thread -i, 指定采样时间间隔

`thread -n 3 -i 5000`{{execute T2}}

```bash
$ thread -n 3 -i 1000
"as-command-execute-daemon" Id=4759 cpuUsage=23% RUNNABLE
    at sun.management.ThreadImpl.dumpThreads0(Native Method)
    at sun.management.ThreadImpl.getThreadInfo(ThreadImpl.java:440)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand.processTopBusyThreads(ThreadCommand.java:133)
    at com.taobao.arthas.core.command.monitor200.ThreadCommand.process(ThreadCommand.java:79)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl.process(AnnotatedCommandImpl.java:96)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl.access$100(AnnotatedCommandImpl.java:27)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl$ProcessHandler.handle(AnnotatedCommandImpl.java:125)
    at com.taobao.arthas.core.shell.command.impl.AnnotatedCommandImpl$ProcessHandler.handle(AnnotatedCommandImpl.java:122)
    at com.taobao.arthas.core.shell.system.impl.ProcessImpl$CommandProcessTask.run(ProcessImpl.java:332)
    at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1142)
    at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:617)
    at java.lang.Thread.run(Thread.java:756)
 
    Number of locked synchronizers = 1
    - java.util.concurrent.ThreadPoolExecutor$Worker@546aeec1
...
```

### thread –state ，查看指定状态的线程

`thread --state WAITING`{{execute T2}}

```bash
[arthas@28114]$ thread --state WAITING
Threads Total: 15, NEW: 0, RUNNABLE: 7, BLOCKED: 0, WAITING: 5, TIMED_WAITING: 3, TERMINATED: 0
ID       NAME                      GROUP            PRIORITY STATE   %CPU     TIME     INTERRU DAEMON
198      AsyncAppender-Worker-arth system           9        WAITING 0        0:0      false   true
3        Finalizer                 system           8        WAITING 0        0:0      false   true
14       RMI Scheduler(0)          system           9        WAITING 0        0:0      false   true
2        Reference Handler         system           10       WAITING 0        0:0      false   true
204      pool-8-thread-1           system           5        WAITING 0        0:0      false   false
```
