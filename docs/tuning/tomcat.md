```yaml
server:
  tomcat:
    accept-count: 1000
    max-connections: 10000 #最大可被连接数，默认为10000
    max-threads: 800 #最大工作线程数
    min-spare-threads: 100 #最小工作线程数
```

4核8G内存单进程调度线程数800-1000，超过这个并发数之后，将会花费巨大的时间在cpu调度上。

修改keepalivetimeout和maxKeepAliveRequests开启长连接

```java
@SpringBootConfiguration
public class TomcatConfig implements WebServerFactoryCustomizer<ConfigurableWebServerFactory> {
    @Override
    public void customize(ConfigurableWebServerFactory factory) {
        ((TomcatServletWebServerFactory) factory).addConnectorCustomizers(connector -> {
            Http11NioProtocol protocol = (Http11NioProtocol) connector.getProtocolHandler();
            // 定制化KeepAliveTimeout,设置30秒内没有请求则服务端自动断开keepalive链接
            protocol.setKeepAliveTimeout(30000);
            // 当客户端发送超过10000个请求则自动断开keepalive链接
            protocol.setMaxKeepAliveRequests(10000);
        });
    }
}
```

