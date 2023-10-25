# Computing metrics

## Dependency

```xml
<dependency>
    <groupId>io.quarkus</groupId>
    <artifactId>quarkus-smallrye-metrics</artifactId>
</dependency>
```

With this dependency added, the application will create an application metrics page at

http://localhost:8080/q/metrics

```bash
# curl -i localhost:8080/q/metrics | grep <metric_name>
curl -i localhost:8080/q/metrics | grep base_classloader_loadedClasses_count
```


