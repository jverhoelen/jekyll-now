---
layout: post
title: Documenting Spring REST APIs with Swagger
last_modified_at: "2017-01-28"
category: "Java"
---

When working in a context where services communicate to each other over REST, it is quite helpful to properly document those APIs. Especially when services are developed and maintained by different teams it saves time and confusion to provide a single entry point for REST API documentation.

## Swagger Specification and Springfox

Independent of language or framework, the [Swagger Specification](http://swagger.io/specification/ "Language agnostic Swagger specification"){:target="_blank" rel="noopener noreferrer"} describes a format to describe REST APIs. It contains all aspects that REST can make use of and is agnostic to how to decided to build your interface.

[Springfox](http://springfox.io "Springfox, a Java Spring implementation of Swagger"){:target="_blank" rel="noopener noreferrer"} is an implementation of the Swagger Specification for Spring. It is awesome and easy to use. On top of the Swagger specification the Swagger-UI is built to provide a beautiful and convenient user interface to browse the API docs. Springfox provides both artifacts in separate dependencies, so you could use them without each other.

## Basic Implementation

Before starting to use Springfox, I saw some non-straightforward implementation tutorials on the web. I tried to simplify them while getting it working with a minimal approach to avoid unneeded code.

At first, we're going to include the latest dependencies for both into our `pom.xml`.

``` xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.6.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.6.0</version>
</dependency>
```

Then we define a separate `SwaggerConfig` class containing all our configuration regarding the generated JSON according to the Swagger Specification.

``` java
@Configuration
@EnableSwagger2
public class SwaggerConfig {

    private static final ModelRef ERROR = new ModelRef("Error");

    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                .apiInfo(apiInfo())
                .useDefaultResponseMessages(false)
                .globalResponseMessage(RequestMethod.GET,
                        Arrays.asList(
                                new ResponseMessageBuilder()
                                        .code(500)
                                        .message("Server error")
                                        .responseModel(ERROR)
                                        .build(),
                                new ResponseMessageBuilder()
                                        .code(400)
                                        .message("Bad request – wrong usage of the API")
                                        .responseModel(ERROR)
                                        .build(),
                                new ResponseMessageBuilder()
                                        .code(401)
                                        .message("No or invalid authentication")
                                        .responseModel(ERROR)
                                        .build(),
                                new ResponseMessageBuilder()
                                        .code(403)
                                        .message("Not permitted to access for users role")
                                        .responseModel(ERROR)
                                        .build(),
                                new ResponseMessageBuilder()
                                        .code(404)
                                        .message("Requested resource not available (anymore)")
                                        .responseModel(ERROR)
                                        .build()
                        ))
                .select()
                // ignore all REST endpoints that Spring Boot ships
                .apis(Predicates.not(RequestHandlerSelectors.basePackage("org.springframework.boot")))
                .build();
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .version("0.1")
                .title("my-rest-api")
                .description("Example REST API")
                .build();
    }
}
```

If you're not able to use Spring Boot and its auto-configuration, make sure you explicitly add resource handlers for the Swagger-UI.

``` java
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/webjars/**")
                .addResourceLocations("classpath:/META-INF/resources/webjars/");

        registry.addResourceHandler("swagger-ui.html")
                .addResourceLocations("classpath:/META-INF/resources/");

    }
}
```

When running the application, you should find the Swagger UI with very basic documentation of your REST controller definitions on `{host}/swagger-ui.html`.

If you want to take influence on some details of your endpoints' documentation, head to the next section.

## Customizing the Docs

Swagger already provides all information about endpoints with the above basic configuration. Furthermore it offers annotations that can be used to declare exceptional information on request definitions. This can be a special description of an error code, the resulting response, notes about a resource, authorization, parameters and everything else.

``` java
@ApiOperation(
    value = "All ingredients"
    notes = "Returns an array of all ingredients known in the system",
    tags = "food,ingredient")
@ApiResponse(code = 500, message = "Ingredient provider is currently not available")
@RequestMapping(method = RequestMethod.GET, path = "/ingredient")
public List<Ingredient> getAll(
        @ApiParam(value = "Can be used to filter the list by name") @RequestParam(required = false) String search) {
    ...
}
```

This request has a dedicated title, notes and some tags. It also has a special description for the response code `500`. Furthermore the `search` `@RequestParam` is being described a little more.

In total, there is pretty proud (list of Swagger Annotations)[https://github.com/swagger-api/swagger-core/wiki/Annotations "Java Swagger Annotations"]{:target="_blank" rel="noopener noreferrer"} you can make use of to describe your API a bit more, when needed.


## Conclusion

The Springfox implementation of Swagger integrates well with Spring (Boot). Without explicit definition of how request and response models have to look, Springfox pulls as much information from Springs request definitions. It combines them with global definitions such as error code or API metadata. Swagger-UI offers a convenient website to browse the REST API docs and even try out requests in real.

If you need to provide API docs to your own or other teams, there is no hassle in using it. It is not not invasive to your code, deployment and operations at all and ship out of two dependencies. Large manually maintained documentation files now cling to the past.
