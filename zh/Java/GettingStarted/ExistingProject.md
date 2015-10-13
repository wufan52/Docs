云代码 SDK支持 JDK6, 7, 8，推荐使用JDK8。

##安装Maven

Eclipse:

1.	点击"Help" >> "Install New Software.."
2.	在"Work with"中输入：`http://download.eclipse.org/technology/m2e/releases`，在列表中选择"Maven Integration for Eclipse"，即可安装Maven插件。

##安装 MaxLeap Command Line Tools（MLC）
####Linux 和 Mac OSX
下述命令将把名为"MLC"的工具安装至`/usr/local/bin/lcc`目录。完成后，您可直接在Terminal中使用 MLC。

*	Git获取

	进入目录/usr/local/bin，运行git命令获取：
		
	```java
	cd /usr/local/bin
	git clone https://gitlab.ilegendsoft.com/zcloudsdk/zcc.git
	```

##	安装SDK

### 添加云代码至已有的项目
####配置pom.xml

* 获取云代码 SDK
* 获取测试插件JUnit
* 获取编译打包插件

```Java
	//添加依赖，获取云代码 SDK及JUnit测试插件
    <dependencies>
        <dependency>
            <groupId>com.maxleap</groupId>
            <artifactId>cloud-code-test</artifactId>
            <version>2.4.0</version>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>
    </dependencies>
	
	//获取编译打包插件
    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-dependency-plugin</artifactId>
                <executions>
                    <execution>
                        <id>copy-mod-dependencies-to-target</id>
                        <phase>process-classes</phase>
                        <goals>
                            <goal>copy-dependencies</goal>
                        </goals>
                        <configuration>
                            <outputDirectory>target/lib</outputDirectory>
                            <includeScope>compile</includeScope>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <descriptors>
                        <descriptor>src/main/assembly/mod.xml</descriptor>
                    </descriptors>
                </configuration>
                <executions>
                    <execution>
                        <id>assemble</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
          <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>3.0</version>
            <configuration>
              <source>1.8</source>
              <target>1.8</target>
            </configuration>
          </plugin>
        </plugins>
    </build>
```

####配置打包规则

在/src/main/assembly中新建mod.xml文件，并在其中添加如下配置：

```Java
	<?xml version="1.0" encoding="UTF-8"?>
	<assembly xmlns="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2"
	          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	          xsi:schemaLocation="http://maven.apache.org/plugins/maven-assembly-plugin/assembly/1.1.2 http://maven.apache.org/xsd/assembly-1.1.2.xsd">

	    <id>mod</id>
	    <formats>
	        <format>zip</format>
	    </formats>
	    <includeBaseDirectory>false</includeBaseDirectory>
	    <fileSets>
	        <fileSet>
	            <outputDirectory>/config</outputDirectory>
	            <directory>src/main/resources/config</directory>
	            <includes>
	                <include>**</include>
	            </includes>
	        </fileSet>
	        <fileSet>
	            <outputDirectory>/cloud/public</outputDirectory>
	            <directory>src/main/resources/public</directory>
	            <includes>
	                <include>**</include>
	            </includes>
	        </fileSet>
	        <fileSet>
	            <outputDirectory>/cloud/lib</outputDirectory>
	            <directory>target</directory>
	            <includes>
	                <include>${project.artifactId}-${project.version}.jar</include>
	            </includes>
	        </fileSet>
	        <fileSet>
	            <outputDirectory>/cloud/lib</outputDirectory>
	            <directory>target/lib</directory>
	            <excludes>
                    <exclude>jackson-*.jar</exclude>
                    <exclude>log4j-*.jar</exclude>
                    <exclude>slf4j-*.jar</exclude>
                    <exclude>cloud-code-*.jar</exclude>
                    <exclude>sdk-data-api*.jar</exclude>
                    <exclude>junit-*.jar</exclude>
                </excludes>
	        </fileSet>
	    </fileSets>
	</assembly>
```

请注意：如果您选择将打包配置文件放在其他路径下，您则需要更新pom.xml文件中的以下部分，将`src/main/assembly/mod.xml`替换为您自定义的路径：

```java
	<plugin>
		<artifactId>maven-assembly-plugin</artifactId>
		<configuration>
			<descriptors>
				<descriptor>src/main/assembly/mod.xml</descriptor>
			</descriptors>
		</configuration>
	</plugin>	
```

当然你也可以自己打包zip，只需按照我们的目录结构来打包你的应用即可
![imgCloudCodeStructure](../../../images/imgCloudcodeZipStructure.png)

#### 配置 global.json
在/src/main/resources/config（请确保此路径存在）中，添加global.json文件，并在其中添加如下配置：

```java
{
	"applicationName" : "HelloWorld",
	"applicationId": "YOUR_APPLICATION_ID",
	"applicationKey": "YOUR_MASTER_KEY",
	"lang" : "java",
	"javaMain": "Main",
	"packageHook" : "YOUR_HOOK_PACKAGE_NAME",
	"packageClasses" : "YOUR_ENTITY_PACKAGE_NAME",
	"version": "0.0.1"
}
```

根据创建应用时获取的key，修改下列键的值：

键|值|
------------|-------|
applicationName|MaxLeap应用名称
applicationId|Application ID
applicationKey|Master Key
javaMain|入口main函数类名
packageHook|Hook包名
packageClasses|Class实体包名
version|当前云代码项目版本号

### 定义一个简单的function

```Java
import com.maxleap.code.MLLoader;
import com.maxleap.code.Response;
import com.maxleap.code.impl.GlobalConfig;
import com.maxleap.code.impl.LoaderBase;
import com.maxleap.code.impl.Response;

public class Main extends LoaderBase implements Loader {
    @Override
    public void main(GlobalConfig globalConfig) {

    	//定义Cloud Function
        defineFunction("hello", request -> {
            Response<String> response = new Response<String>(String.class);
            response.setResult("Hello, " + request.parameter(Map.class).get("name") + "!");
            return response;
        });
    }
}
```

注意：

* Main class的main method是云代码项目启动的入口（在global.json中指定），需要继承LoaderBase并实现Loader接口，在main方法中需要注册所有的cloud function和job。

### 打包

在当前项目根目录下运行Maven命令：

`mvn package`

我们将在项目根目录下的target文件夹中发现 *xxx-1.0-SNAPSHOT-mod.zip* 文件，这便是我们想要的package.

## 云代码的上传及部署
1. 登录：lcc login <UserName>
2. 选择所要部署的目标应用，作为后续操作的上下文：lcc use <AppName>
3. 上传Package： lcc upload <PackageLocation>
4. 部署云代码：lcc deploy <VersionNumber>

**注意：**

*	这里的VersionNumber定义在您云代码项目中的global.json文件中（version字段的值）
* 	若您在部署之前，已经部署过某个版本的云代码，需要先卸载该版本的云代码，才能部署新版本。
*	使用lcc help来获取所有相关命令帮助，你也可以查看[lcc使用向导](ML_DOCS_GUIDE_LINK_PLACEHOLDER_JAVA)，以获取lcc的更多信息。

### 测试

通过 curl，我们向部署好的Cloud Function发送如下POST请求，以测试我们的Function是否部署成功：

```shell
curl -X POST \
-H "X-ML-AppId: YOUR_APPID" \
-H "X-ML-APIKey: YOUR_APIKEY" \
-H "Content-Type: application/json" \
-d '{"name":"David Wang"}' \
https://api.leap.as/functions/hello
```
此时，我们将得到如下结果：

```shell
Hello, David Wang!
```
表明测试通过，部署成功。

注意:

* X-ML-APIKey的值为应用的API KEY，而非云代码项目中使用的Master Key.

## 下一步
 至此，您已经完成MaxLeap SDK的安装与必要的配置。请移步至[云代码 SDK使用教程](ML_DOCS_GUIDE_LINK_PLACEHOLDER_JAVA)以获取 MaxLeap 云代码 SDK 的详细功能介绍以及使用方法。