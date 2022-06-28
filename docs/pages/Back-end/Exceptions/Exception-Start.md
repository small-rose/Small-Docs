

# 常见异常

## SpringBoot 相关
----------------

**现象**: springboot 启动不了

**错误信息**: 

```text
Error running' xxxxxx': Command line is too long. Shorten command line for xxxxxxxxx
```
**思路**：

    在 run -> edit configuration 中修改成 Shorten command line 下拉选项



**现象**: 执行Main方法报错

**错误信息**: 

```text
Failed to execute goal org.codehaus.mojo:exec-maven-plugin:3.0.0:exec (default-cli) on project bp-web: Command execution failed.
```
**思路**：

因为我勾选了 Delegate IDE build/run actions to Maven, 导致运行main方法有问题

可以在pom里添加 `exec-maven-plugin` 插件
```yaml
<plugins>
    <plugin>
        <groupId>org.codehaus.mojo</groupId>
        <artifactId>exec-maven-plugin</artifactId>
        <version>1.6.0</version>
        <configuration>
            <classpathScope>test</classpathScope>
        </configuration>
    </plugin>
</plugins>
```



