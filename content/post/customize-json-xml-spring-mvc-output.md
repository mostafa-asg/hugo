---
title: "5 ways to customize Spring MVC JSON/XML output"
date: 2018-04-01T22:34:30+04:30
draft: false
tags: [Spring MVC,REST]
---
To adhere to guidelines or requirements, API designers may want to control how
JSON/XML responses are formatted. **Spring Web** makes use of Jackson to perform
JSON/XML serialization. Therefore, to customize our output format, we must configure
the Jackson processor. Spring Web offers XML-based or Java-based approaches to
handling configuration. In this article, we will look at the Java-based configuration.

## Enable XML output
First of all, if you have created your project through
[https://start.spring.io/](https://start.spring.io/) you need add the following
dependency to your project because there is no XML serialization by default:
```
<dependency>
	<groupId>com.fasterxml.jackson.dataformat</groupId>
	<artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```
ok, now we can investigate the output customization process. Imagine we have this
REST controller:
```
@RestController
@RequestMapping("/users")
public class UsersResource {

    @RequestMapping("/{userID}")
    public User getByID(@PathVariable("userID") int userID){
        // I ignore fetching user from database
        // and just return a hardcoded user
        User user = new User("mostafa",null,18);
        return user;
    }

}
```
And User is defined like this:
```
public class User {

    private String firstName;
    private String lastName;
    private int age;

    public User(String firstName, String lastName, int age) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.age = age;
    }

    public String getFirstName() {
        return firstName;
    }

    public String getLastName() {
        return lastName;
    }

    public int getAge() {
        return age;
    }
}
```
If you start your service and call your service like this:
```
curl http://localhost:8080/users/15
```
You will get the output something like this:
```
{"firstName":"mostafa","lastName":null,"age":18}
```
Now let's back to our problem. How we can change the generated output? For example
change **firstName** to **first_name** and remove fields with null values like **lastName**.

## Solution 1> Use properties
You can configure the ObjectMapper and XmlMapper instances by using the environment.
Jackson provides an extensive suite of simple on/off features that can be used to
configure various aspects of its processing. For example, to enable pretty print, add this
line to the application.properties file:
```
spring.jackson.serialization.indent_output=true
```
Here is the valid properties:  
{{< fluid_imgs
        "center|/static/custom-json-xml/jackson-properties.png"
>}}


## Solution 2> Use Jackson2ObjectMapperBuilderCustomizer
You can customize the default **Jackson2ObjectMapperBuilder** using **Jackson2ObjectMapperBuilderCustomizer**.
Just create a new class annotated with **@Configuration** and return an instance of Jackson2ObjectMapperBuilderCustomizer
with **@Bean** annotation:
```
@Configuration
public class OutputConfiguration {
    @Bean
    public Jackson2ObjectMapperBuilderCustomizer customJson(){
        return builder -> {
            // human readable
            builder.indentOutput(true);
            // exclude null values
            builder.serializationInclusion(JsonInclude.Include.NON_NULL);
            // all lowercase with under score between words
            builder.propertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE);
        };
    }
}
```
## Solution 3> Use ObjectMapper
This method replace default ObjectMapper instance:
```
@Configuration
public class OutputConfiguration {
    @Bean
    @Primary
    public ObjectMapper customJson(){
        return new Jackson2ObjectMapperBuilder()
                   .indentOutput(true)
                   .serializationInclusion(JsonInclude.Include.NON_NULL)
                   .propertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE)
                   .build();
    }
}
```
## Solution 4> Use Jackson2ObjectMapperBuilder
Like the above code, but just return the builder:
```
@Configuration
public class OutputConfiguration {
   @Bean
    public Jackson2ObjectMapperBuilder customJson() {
        return new Jackson2ObjectMapperBuilder()
                .indentOutput(true)
                .serializationInclusion(JsonInclude.Include.NON_NULL)
                .propertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE);
    }
}
```
## Solution 5> Use MappingJackson2HttpMessageConverter
```
@Bean
    public MappingJackson2HttpMessageConverter customJson(){
        return new MappingJackson2HttpMessageConverter(
                new Jackson2ObjectMapperBuilder()
                        .indentOutput(true)
                        .serializationInclusion(JsonInclude.Include.NON_NULL)
                        .propertyNamingStrategy(PropertyNamingStrategy.SNAKE_CASE)
                        .build()
        );
} 
```
That's it. Do you know the other ways to customize JSON/XML output? If you know let me know.   
source: [Spring Mvc documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-spring-mvc.html#howto-customize-the-jackson-objectmapper)
