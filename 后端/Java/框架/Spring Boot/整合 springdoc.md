---
order: 99

author: 钟舒艺
---
# 整合 SpringDoc 文档

## 信息来源

[官方文档](https://springdoc.org/#Introduction)
[GitHub](https://github.com/springdoc/springdoc-openapi)

## maven 集成

我们使用 Javadoc 直接生成接口文档，并且不需要使用 `swagger-ui`

父模块或公共模块`pom.xml` 文件

```xml
<dependencies>
  <!-- 去除 swagger-ui 的springdoc-->
 <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-webmvc-core</artifactId>
      <version>1.6.11</version>

  </dependency>
 <!-- javadoc 插件-->
  <dependency>
      <groupId>org.springdoc</groupId>
      <artifactId>springdoc-openapi-javadoc</artifactId>
      <version>1.6.11</version>
  </dependency>
</dependencies>

 <build>
        <plugins>
            <!--maven的默认编译使用的jdk版本貌似很低，使用maven-compiler-plugin插件可以指定项目源码的jdk版本，编译后的jdk版本，以及编码-->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.8.1</version>
                <configuration>
                    <source>11</source>
                    <target>11</target>
                    <encoding>UTF-8</encoding>
                    <annotationProcessorPaths>
                        <!--需要读取javadoc-->
                        <path>
                            <groupId>com.github.therapi</groupId>
                            <artifactId>therapi-runtime-javadoc-scribe</artifactId>
                            <version>0.14.0</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
        </plugins>
    </build>

```

添加后就会通过 `JavaDoc` 生成 `OpenAPI` 的 `JSON` 格式的文档了
访问： `http://localhost:8080/v3/api-docs` 就可以看到了

## 与 swagger 对比

| **swagger** | **springdoc** | **javadoc** |
| --- | --- | --- |
| @Api(name = "xxx") | @Tag(name = "xxx") | java 类注释第一行 |
| @Api(description= "xxx") | @Tag(description= "xxx") | java 类注释 |
| @ApiOperation | @Operation | java 方法注释 |
| @ApiIgnore | @Hidden | 无 |
| @ApiParam | @Parameter | java 方法@param 参数注释 |
| @ApiImplicitParam | @Parameter | java 方法@param 参数注释 |
| @ApiImplicitParams | @Parameters | 多个@param 参数注释 |
| @ApiModel | @Schema | java 实体类注释 |
| @ApiModelProperty | @Schema | java 属性注释 |
| @ApiModelProperty(hidden = true) | @Schema(accessMode = READ_ONLY) | 无 |
| @ApiResponse | @ApiResponse | java 方法@return 返回值注释 |

## 常用配置

```yaml
springdoc:
  swagger-ui:
    # 修改 Swagger UI 路径
    path: /swagger-ui.html
    # 开启 Swagger UI 界面
    enabled: true
  api-docs:
    # 修改 api-docs 路径
    path: /v3/api-docs
    # 开启 api-docs
    enabled: true
  # 配置需要生成接口文档的扫描包
  packages-to-scan: xxx.xxx.xxx
  # 配置需要生成接口文档的接口路径
  paths-to-match: /xxx/**,/xxx/**

```
