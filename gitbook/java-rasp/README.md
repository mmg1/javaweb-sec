# RASP技术

## 什么是RASP技术?

> 最近三五年RASP技术已日渐被人们所接收，国内的各大安全厂商也在不断的完善自身产品。RASP技术触及到了应用程序的底层实现，如何实现一款高稳定性和高可用的RASP产品也就变得比较困难了。

在2013年的时候乌云研发了应该是国内第一款商业`RASP产品`：`防护云`。借助于PHP的`auto_prepend_file`和Java的`Filter`特性我们实现了这两种语言层的攻击防御。在2014年`RASP`（`Runtime application self-protection`运行时应用自我保护）技术提出之前“防护云”早就已经实现了基于客户端拦截和云端可视化分析为一体化的安全防御系统研发。

但是这种基于请求过滤(`Filter机制`)实现的防御相较于传统的WAF已经有了较大的能力提升，但也因为其建立在请求过滤的基础上的实现导致了防御的深度和准确性还是不够，而且基于`Filter`实现的`RASP`本身也会有非常多的坑。为了解决这个问题2016年的时候我开始了基于`Java Agent`机制的`灵蜥RASP`技术研究，希望通过增强Java语言底层的API来实现更加深层次的防御。

## RASP简介

`Runtime application self-protection`一词，简称为RASP。它是一种新型应用安全保护技术，它将保护程序像疫苗一样注入到应用程序中，应用程序融为一体，能实时检测和阻断安全攻击，使应用程序具备自我保护能力。当应用程序遭受到实际攻击伤害，就可以自动对其进行防御，而不需要进行人工干预。

RASP技术可以快速的将安全防御功能整合到正在运行的应用程序中，它拦截从应用程序到系统的所有调用，确保它们是安全的，并直接在应用程序内验证数据请求。Web和非Web应用程序都可以通过RASP进行保护。该技术不会影响应用程序的设计，因为RASP的检测和保护功能是在应用程序运行的系统上运行的。

## 灵蜥Agent与传统WAF的区别

传统的WAF(网站防火墙)一般都是基于流量解析的(CDN、硬件盒子)，他们的面临如下几个比较严重的问题:

1. 无法精准的解析Http协议，可能会直接导致漏洞或误报。
2. CDN类型的防御找到用户真实IP就可以绕过防御。
3. 无法结合应用程序运行时环境分析黑客攻击。
4. 没有后门扫描功能，无法监控用户文件系统。
5. 硬件盒子类无法及时更新规则，针对新型漏洞防御能力较低。
6. 不支持请求参数值清理，只支持请求拦截，对用户业务系统影响很大。
7. 溯源难、安全响应速度较慢。
8. WAF大部分都是基于正则来实现的，导致绕过率较高。
9. 通常WAF规则只能应对已经公开的通用漏洞。
10. 无法精准的获取数据库最终执行的SQL语句、Java表达式(`Ognl`、`SpEL`、`MVEL2`)，所有的检测只能针对请求参数。
11. 不支持或者难支持一些比较复杂的业务系统和框架，如`Spring MVC`、`Java反序列化`等。

## 灵蜥Agent独特防御功能

1. 灵蜥云端提供了攻击可视化功能，可以近实时对应用进行监控、分析，同时也提供了云端规则实时同步、动态补丁下发等特色功能。
2. 灵蜥客户端嵌入了用户运行时环境，可以非常准确的对黑客攻击参数进行解析，当黑客访问后门文件时可以实时防御，同时还可以对用户请求对参数进行净化或拦截处理，一定程度上避免了由于程序误报导致对用户应用系统不可用等问题。
3. 新版灵蜥实现了对程序语言底层API的防御，提供了更加强大的基于行为事件的拦截，可以轻易的拦截到黑客最终执行的恶意程序、命令、SQL语句等功能，而这些防御能力是传统的WAF难以实现的。
4. 灵蜥独特的请求参数净化功能，可以把疑似攻击的请求参数值经过特殊的转义后转换成对服务端无攻击性的参数。以净化代替请求阻断可以有效的避免误报导致的业务无法正常使用的问题。
5. 灵蜥插件机制允许用户实现定制化防御，可针对用户自研发或已爆发漏洞但无法对源码进行完善的系统进行定制化防御。
6. 客户端支持多种模式：监控(只记录不防御)模式、防御模式(支持对某些恶意类型的参数值清洗)。
7. 灵蜥`Java Agent`针对Java安全做了深入的研究，对`NIO`、`Spring MVC`框架、Java其他特性都做了深入支持。
8. 对于未公开的通用漏洞或者企业内部自主开发的系统所产生的攻击可实现自动拦截。
9. 快速定位漏洞点，可直观的看到黑客攻击调用链。
10. 支持`Attach`(附加灵蜥Agent到Java进程的方式)模式运行，已实现无需重启容器或服务即可实现防御。
11. 自适应能力强大！灵蜥基于`Java Servlet API`和`JDBC API`实现了对接口实现类的方法级别Hook。理论上灵蜥已经实现了`支持一切标准的Web容器和一切基于JDBC的数据库驱动`。
12. 支持对`JNI`方法调用Hook的支持。

## Agent 缺陷

因为Agent的实现是完全性的嵌入到语言代码层的，我们通常把防御和正常的业务功能看作一个整体，所以Agent的稳定性直接影响着用户的业务服务，研究和实践中遇到的主要问题如下：

1. Agent与用户业务高度的紧密性，一旦Agent出现任何异常可能是致命的(如错误的使用了ASM的操作指令去修改类、类验证错误等)，所以对Agent自身的稳定性、兼容性要求非常高，其次是一旦嵌入到用户环境就非常容易遭到质疑，即便是用户自己的bug、性能消耗可能也会归咎于是Agent造成的。
2. 客户端防御消耗着用户服务器CPU、内存资源，对Agent的效率和性能消耗要求较高。
3. 不同的开发语言需要去独立适配，如：PHP和Java的Agent实现方式都重度依赖语言的实现方式，两则之间截然不同。
4. 部署、防御方式较为麻烦，实现容器的自动安装需要做适配,Attach机制虽然可以一定程度的解决这个问题。

个人认为以上缺陷最为致命的就是稳定性，尤其是涉及到字节码操作、类加载、异常处理等问题一旦处理不好就会全功尽弃！通过Agent机制获取的东西多了，但是与之而来的稳定性要求也就越高了，所以我们在追求可用性的同时稳定性应该永远的放在第一位。