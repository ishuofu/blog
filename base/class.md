在开发过程中，经常遇到某些JAR满足部分需求而不满足场景，需要对JAR进行拓展
1. 对于提供接口，可实现其接口
2. 对于未提供接口，通过织入代码的方式
* source and compile
> 下载源码然后添加代码再编译，但是不同项目，代码编译工具不同，如maven，gradle，sbt 等等
* aspectj方式
> 在运行时织入代码
```
@Aspect
@Slf4j
public class RateAspect {
    @Around("execution(* io.vertx.core.net.impl.VertxHandler.channelRead(..)) || execution(* kafka.consumer.SimpleConsumer.fetch(..)) ||execution(* io.vertx.ext.sql.SQLConnection.query*(..))")
    public Object memoryRateRate(ProceedingJoinPoint joinPoint) throws Throwable {
        MemoryRate.acquire();
        return joinPoint.proceed();
    }
}
```
* maven-shade-plugin
> 在编译，打包时排除与重命名特定类
```
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <configuration>
        <keepDependenciesWithProvidedScope>false</keepDependenciesWithProvidedScope>
        <createDependencyReducedPom>false</createDependencyReducedPom>
        <relocations>
            <relocation>
                <pattern>org.elasticsearch.spark.rdd.shade.EsRDDWriter</pattern>
                <shadedPattern>org.elasticsearch.spark.rdd.EsRDDWriter</shadedPattern>
            </relocation>
        </relocations>
        <filters>
            <filter>
                <artifact>*:*</artifact>
                <excludes>
                    <exclude>org/elasticsearch/spark/rdd/EsRDDWriter*.class</exclude>
                </excludes>
            </filter>
        </filters>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

