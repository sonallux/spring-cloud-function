### Introduction

Spring Cloud Function is a framework that promotes the notion that any business requirement could be realized as one of the `java.util.Supplier`, `java.util.Function` or `java.util.Consumer` while providing a set of abstractions to execute such functions in different context using provided set of Spring Boot adapter auto-configurations. For example, the same function implementation could be realized as Message Handler or REST endpoint or AWS Lambda and many more without touching its code, by simply adding one of the provided dependencies to your POM or Gradle script.

While you can find more details on Spring Cloud Function and all of its features [here](https://spring.io/projects/spring-cloud-function), this example is about AWS Lambdas and Spring support for it.

While AWS provides a set of  interfaces to implement various type of handlers (`RequestHandler`, `RequestStreamHandler` etc), you may want to keep your implementations POJO-like or you may have an existing legacy code that you would want to turn into AWS lambda. Regardless of the case Spring Cloud Function AWS adapter can help.
This example demonstrates how a simple Java Function managed as Spring Bean can be easily turned into and deployed as AWS Lambda. 

#### Basic Example

The example is very trivial, so here is the code:

```
@SpringBootApplication
public class FunctionConfiguration {
	
	@Bean
	public Function<String, String> uppercase() {
		return value -> value.toUpperCase();
	}
}
```

As you can see it's a standard Spring Boot application that has no code-level dependencies on AWS. In fact outside of Spring configuration annotations such as `@Bean` and `@SpringBootApplication`, it has no code-level dependencies on Spring either. While implementation of this function is trivial, you can imagine a more involved example with complex types, callbacks to legacy code etc (see next section for references to such samples). 

So, once deployed as AWS Lambda you can send a String and receive the uppercased version of it back.

* Build *

```
./mvnw clean package
```

** Deploy **

Combination of pre-configured [Spring Boot maven plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/) and [Apache Maven Shade plugin](https://maven.apache.org/plugins/maven-shade-plugin/) will produce a deployable JAR file. 


NOTE:  _Since the intention of this example to be as trivial as possible we do not provide a deploy script (i.e., [SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-cli-command-reference-sam-deploy.html)) to ensure there is nothing else that needs to be learned if this is read by a new AWS user, and instead recommend using AWS Dashboard functionality to deploy and test it. Some of the more complex examples listed in the next section have deploy scripts to simplify the process._ 

Now all you need to do is deploy (upload) the JAR file located in the `target` directory of the project to AWS.

NOTE: You will find two JAR files (actually three where one has additional extension `.original`). The AWS deployable artifact is the one that ends with `-aws.jar`. For example, in our case it's `function-sample-aws-2.0.0.RELEASE-aws.jar`.

When ask about _handler_ you specify `org.springframework.cloud.function.adapter.aws.FunctionInvoker::handleRequest` which is a generic request handler provided by the Spring Cloud Function. 
You can also find more details deployment procedures in this quick [getting started](https://docs.spring.io/spring-cloud-function/docs/3.2.9/reference/html/aws.html#_getting_started) paragraph available in Spring Cloud Function documentation. 

#### Other Examples