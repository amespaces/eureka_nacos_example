- LoadBalancerInterceptor

```java
import com.sun.deploy.net.HttpRequest;

import java.io.IOException;

public class LoadBalancerInterceptor implements ClientHttpRequestInterceptor {
    @Override
    public ClinetHttpResponse intercept(final HttpRequest request, final byte[] body, final ClientHttpRequestExecution execution) throws IOException {
        //获取uri，本例为http://userservice/user/101
        final URI originalUri = request.getURI();
        String serviceName = originalUri.getHost();//获取主机名userserive
        Assert.state(serviceName != null,"request uri does not contain a vaild host");
        return this.loadBalancer.execute(serviceName,requestFactory.createRequest(request,body,execution));
    }
}
//继续跟入execute方法
    //根据服务id获取LoadBalancer，而ILoadBalance会拿着服务id去eureka中获取服务列表并保存起来
ILoadBalancer loadBalancer = getLoadBalancer(serviveId);
server server = getServer(loadBalancer);
//利用内置的负载均衡算法从服务列表中选择一个

//跟入getServer
protected Server(ILoadBalancer loadBalancer){
    if(loadBalancer == null){
        return null;
    }
    return loadBalancer.chooseServer("default");
}
//继续跟入chooseServer方法
public Server chooseServer(Object key){
    //其中有一个rule.choose(key)
}
//跟入rule
private final static IRule DEFAULT_RULE = new RoundRobinRule();
protected IRule rule = DEFAULT_RULE;
```
- 总结Ribbon流程如下
    - SpringCloudRibbon的底层采用拦截器，拦截了RestTemplate发出的请求，对地址进行了修改
    - 拦截RestTemplate请求http://userservice/user/101