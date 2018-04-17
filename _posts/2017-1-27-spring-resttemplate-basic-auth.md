---
layout: post
title: Elegant Basic Auth with the Spring RestTemplate
last_modified_at: "2016-01-27"
category: "Java"
---

Communication via HTTP calls is a very common task for Spring applications in times of service oriented and microservice architectures. To secure services from unwanted access, HTTP Basic Access Authentication is a simple and sufficient (assuming usage of HTTPS) strategy.

We probably want to use the `RestTemplate` being provided by Spring directly. ~~Unfortunately it doesn't offer support for any kind of authentication out of the box.~~ The Spring community on the Internet already found solutions to overcome this circumstance, of course. But most of them didn't make me completely happy, so here is mine.

**_Important note_**: In the meanwhile I found out about `RestTemplateBuilder.basicAuthorization` which makes this blog post superfluous. [See the method docs](http://docs.spring.io/spring-boot/docs/current/api/org/springframework/boot/web/client/RestTemplateBuilder.html#basicAuthorization-java.lang.String-java.lang.String-){:target="_blank" rel="noopener noreferrer"} on how to use it.

Feel free to be happy with that and not read further. Anyway, I kept my solution it here in case you're still interested.

## Concept

Spring provides us the `ClientHttpRequestInterceptor` interface with which we can intercept a `HttpRequest` to manipulate headers, payload and more. I find it very useful for this use case since we want to append a header for every Basic Auth requiring request.

To provide a special `RestTemplate` that uses this interceptor, a class like `BasicAuthRestTemplate` adds some functionality by extending from `RestTemplate`.

## Implementation

At first we're gonna implement the required interceptor. As promised, very simple.

{% highlight java %}
public class BasicAuthInterceptor implements ClientHttpRequestInterceptor {

    private String username;
    private String password;

    public BasicAuthInterceptor(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public ClientHttpResponse intercept(HttpRequest httpRequest, byte[] bytes, ClientHttpRequestExecution clientHttpRequestExecution) throws IOException {
        HttpHeaders headers = httpRequest.getHeaders();
        headers.add(HttpHeaders.AUTHORIZATION, encodeCredentialsForBasicAuth(username, password));

        return clientHttpRequestExecution.execute(httpRequest, bytes);
    }

    public static String encodeCredentialsForBasicAuth(String username, String password) {
        return "Basic " + new Base64().encodeToString((username + ":" + password).getBytes());
    }
}
{% endhighlight %}

It knows the needed credentials for the realm and how to encode and append them to represent the wanted `Authorization` header.

Now we gonna use it in our slightly smarter `BasicAuthRestTemplate`. Internally it simply uses `setRequestFactory` to overwrite the factory that creates the resulting HTTP requests.

``` java
public class BasicAuthRestTemplate extends RestTemplate {

    private String username;
    private String password;

    public BasicAuthRestTemplate(String username, String password) {
        super();
        this.username = username;
        this.password = password;
        addAuthentication();
    }

    private void addAuthentication() {
        if (StringUtils.isEmpty(username)) {
            throw new RuntimeException("Username is mandatory for Basic Auth");
        }

        List<ClientHttpRequestInterceptor> interceptors = Collections.singletonList(new BasicAuthInterceptor(username, password));
        setRequestFactory(new InterceptingClientHttpRequestFactory(getRequestFactory(), interceptors));
    }
}
```

Short, beautiful and not complex.

## Usage

It's just as easy to use as the plain old `RestTemplate`.

``` java
BasicAuthRestTemplate restTemplate = new BasicAuthRestTemplate("user", "password");
ResponseEntity<String> result = restTemplate.getForEntity("http://google.de", String.class);
```
