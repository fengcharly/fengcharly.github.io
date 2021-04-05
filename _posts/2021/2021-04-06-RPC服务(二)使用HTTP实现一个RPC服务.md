---
title: RPC服务(二)使用HTTP实现一个RPC服务
categories:
 - RPC
tags:
 - 后端开发
description: RPC本质上就是“像调用本地方法一样调用远程方法”，主要涉及到客户端和服务端的数据的传输...
---

#### 1.RPC服务框架的基本结构

RPC本质上就是“像调用本地方法一样调用远程方法”，主要涉及到客户端和服务端的数据的传输，整体的RPC的框架服务就如下所示：

![6zcIYD.png](https://z3.ax1x.com/2021/03/27/6zcIYD.png)

#### 2.使用HTTP实现服务的组成部分

1. ##### 注册中心：zookeeper

2. ##### 序列化方式：json

3. ##### 网络通信：http协议

4. ##### 代理方式：jdk动态代理

#### 3.项目结构

```java
├─rpc-http-api
│  ├─src
│  │  └─main
│  │      ├─java
│  │      │  └─com
│  │      │      └─home
│  │      │          └─api
│  │      └─resources
│  └─target
│      ├─classes
│      │  └─com
│      │      └─home
│      │          └─api
│      └─generated-sources
│          └─annotations
├─rpc-http-consumer
│  ├─src
│  │  └─main
│  │      ├─java
│  │      │  └─com
│  │      │      └─home
│  │      │          └─consumer
│  │      └─resources
│  └─target
│      ├─classes
│      │  └─com
│      │      └─home
│      │          └─consumer
│      └─generated-sources
│          └─annotations
├─rpc-http-core
│  ├─src
│  │  └─main
│  │      ├─java
│  │      │  ├─client
│  │      │  └─com
│  │      │      └─home
│  │      │          └─core
│  │      │              ├─entity
│  │      │              └─server
│  │      └─resources
│  └─target
│      ├─classes
│      │  ├─client
│      │  └─com
│      │      └─home
│      │          └─core
│      │              ├─entity
│      │              └─server
│      └─generated-sources
│          └─annotations
└─rpc-http-provider
    ├─src
    │  └─main
    │      ├─java
    │      │  └─com
    │      │      └─home
    │      │          └─provider
    │      │              ├─config
    │      │              └─controller
    │      └─resources
    └─target
        ├─classes
        │  └─com
        │      └─home
        │          └─provider
        │              ├─config
        │              └─controller
        └─generated-sources
            └─annotations
```

##### ① RPC服务的提供者:

服务的提供者在启动的时候回进行初始化，在zookeeper上面创建的节点名称如类的权限定名+IP+下划线+端口号，节点的值为："provider"。

```java
@Slf4j
@SpringBootApplication
public class RpcHttpProviderApplication {


    public static void main(String[] args) throws Exception {
        // 启动前进行初始化
        init();
        SpringApplication.run(RpcHttpProviderApplication.class, args);
    }

    private static void init() throws Exception {
        // zookeeper 新建节点
        RetryPolicy retryPolicy = new ExponentialBackoffRetry(1000, 3);
        CuratorFramework client = CuratorFrameworkFactory.builder().
                connectString("localhost:2181").
                namespace("rpc-http").
                retryPolicy(retryPolicy)
                .build();
        client.start();

        // 建立用户服务的节点 全限定名的方式 com.home.api.UserService
        String userServiceName = "com.home.api.UserService";
        registerService(client, userServiceName);

    }

    /**
     * 向zookeeper注册相应的服务节点
     *
     * @param client
     * @param serviceName
     */
    private static void registerService(CuratorFramework client, String serviceName) throws Exception {

        // 用户提供者相关定义
        ServiceProviderDesc userServiceProviderDesc = ServiceProviderDesc.builder()
                .host(InetAddress.getLocalHost().getHostAddress())
                .port(8081)
                .serviceClass(serviceName)
                .build();

        // 创建节点
        try {
            if (null == client.checkExists().forPath("/" + serviceName)) {
                client.create().withMode(CreateMode.PERSISTENT).forPath("/" + serviceName, "service".getBytes());
            }
        } catch (Exception e) {
            log.error("创建提供者节点异常:{}", e.getMessage());
        }


        // 创建服务的临时节点
        client.create().withMode(CreateMode.EPHEMERAL).
                forPath( "/" + serviceName + "/" + userServiceProviderDesc.getHost() + "_" + userServiceProviderDesc.getPort(), "provider".getBytes());

    }
}

```

##### ② RPC服务的消费者:

服务的消费者通过JDK代理的方式使用http请求提供者的接口，获取JSON数据并解析：

```java
/**
 * 服务的消费者
 */
@Slf4j
@SpringBootApplication
public class RpcHttpConsumerApplication {

    /**
     * 消费端进行消费
     */
    public static void main(String[] args) {

        UserService userService = RpcClient.create(UserService.class, "http://localhost:8081/");
        User user = userService.findById(1);
        log.info("find user id=1 from server: {}", user.getName());
        
    }
}
```

主要的RpcClient代理类：

```java
/**
 * RPC客户端
 */
@Slf4j
public final class RpcClient {

    static {
        ParserConfig.getGlobalInstance().addAccept("com.home");
    }

    public static <T, filters> T createFromRegistry(final Class<T> serviceClass, final String zkUrl, Router router, LoadBalancer loadBalance, Filter filter) {

        // curator Provider list from zk
        List<String> invokers = new ArrayList<>();
        // 1. 简单：从zk拿到服务提供的列表
        // 2. 挑战：监听zk的临时节点，根据事件更新这个list（注意，需要做个全局map保持每个服务的提供者List）

        // router, loadbalance
        List<String> urls = router.route(invokers);
        String url = loadBalance.select(urls);

        return (T) create(serviceClass, url, filter);

    }

    public static <T> T create(final Class<T> serviceClass, final String url, Filter... filters) {

        // JDK动态代理
        return (T) Proxy.newProxyInstance(
                RpcClient.class.getClassLoader(),
                new Class[]{serviceClass},
                new RpcInvocationHandler(serviceClass, url, filters)
        );

    }


    public static class RpcInvocationHandler implements InvocationHandler {

        public static final MediaType JSONTYPE = MediaType.get("application/json; charset=utf-8");

        private final Class<?> serviceClass;
        private final String url;
        private final Filter[] filters;

        public <T> RpcInvocationHandler(Class<T> serviceClass, String url, Filter... filters) {
            this.serviceClass = serviceClass;
            this.url = url;
            this.filters = filters;
        }

        // 可以尝试，自己去写对象序列化，二进制还是文本的，，，rpc是xml自定义序列化、反序列化
        // int byte char float double long bool
        // [], data class
        @Override
        public Object invoke(Object proxy, Method method, Object[] params) throws Throwable {

            RpcRequest request = new RpcRequest();
            request.setServiceClass(this.serviceClass.getName());
            request.setMethod(method.getName());
            request.setParams(params);

            if (null != filters) {
                for (Filter filter : filters) {
                    if (!filter.filter(request)) {
                        return null;
                    }
                }
            }

            RpcResponse response = post(request, url);
            return JSON.parse(response.getResult().toString());
        }

        /**
         * 请求提供者的接口获取相关数据信息
         *
         * @param req
         * @param url
         * @return
         * @throws IOException
         */
        private RpcResponse post(RpcRequest req, String url) throws IOException {
            String reqJson = JSON.toJSONString(req);

            log.info("req json: {}",reqJson);

            // 1.可以复用client
            // 2.尝试使用httpclient或者netty client
            OkHttpClient client = new OkHttpClient();
            final Request request = new Request.Builder()
                    .url(url)
                    .post(RequestBody.create(JSONTYPE, reqJson))
                    .build();
            String respJson = client.newCall(request).execute().body().string();

            log.info("resp json: {}",respJson);
            return JSON.parseObject(respJson, RpcResponse.class);
        }
    }
}
```

请求响应的结果如下：

```java
 [main] INFO client.RpcClient - req json: {"method":"findById","params":[1],"serviceClass":"com.home.api.UserService"}
 [main] INFO client.RpcClient - resp json: {"result":"{\"@type\":\"com.home.api.User\",\"id\":1,\"name\":\"rpc time: 2021-03-06T12:12:16.697\"}","status":true,"exception":null}
 [main] INFO com.home.consumer.RpcHttpConsumerApplication - find user id=1 from server: rpc time: 2021-03-06T12:12:16.697
```

#### 4. 项目地址:

```java
https://github.com/fengcharly/rpc-demo.git
```













