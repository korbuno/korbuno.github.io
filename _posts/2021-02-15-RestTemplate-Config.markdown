---
layout: post
title: RestTemplate 설정과 pdf파일 요청하기
date: 2021-02-15 23:31:23 +0900
category: Spring
---
# RestTemplate란?
> 동기식 HTTP 요청을 생성해주며, exchange와 execute 메서드 외에도 공용 HTTP method 별 템플릿을 제공한다.    
**주의) Spring 5.0 부터 RestTemplate 클래스는 유지 관리 모드에 있으며, 향후 변경 및 버그에 대한 사소한 요청만 허용한다. RestTemplate 대신 WebClient를 사용하는 것을 권장한다. [API상세 출처](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/web/client/RestTemplate.html)**

원문

*** 

NOTE: As of 5.0 this class is in maintenance mode, with only minor requests for changes and bugs to be accepted going forward. Please, consider using the org.springframework.web.reactive.client.WebClient which has a more modern API and supports sync, async, and streaming scenarios.

***   
   
## 들어가기 전에    
위 원문을 보기 전에 RestTemplate로 설정 및 사용했다가 WebClient로 변경하는 피드백을 받았다. 프로젝트 Spring 버전에 따라 API 상세 문서를 확인하고 사용하도록 하자.

## 설정

```java
import java.util.concurrent.TimeUnit;

import org.apache.http.client.HttpClient;
import org.apache.http.impl.client.DefaultConnectionKeepAliveStrategy;
import org.apache.http.impl.client.HttpClientBuilder;
import org.apache.http.impl.client.HttpClients;
import org.apache.http.impl.conn.PoolingHttpClientConnectionManager;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.client.ClientHttpRequestFactory;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;
import org.springframework.web.servlet.HandlerInterceptor;

@Configuration
public class RestTemplateConfig implements HandlerInterceptor {

	private final int maxTotal;
	private final int maxPerRoute;
	private final int readTimeOut;
	private final int connectTimeOut;

	public RestTemplateConfig(@Value("${?.resttemplate.max-total}") int maxTotal,
			@Value("${?.resttemplate.max-per-route}") int maxPerRoute,
			@Value("${?.resttemplate.read-time-out}") int readTimeOut,
			@Value("${?.resttemplate.connect-time-out}") int connectTimeOut) {
		this.maxTotal = maxTotal;
		this.maxPerRoute = maxPerRoute;
		this.readTimeOut = readTimeOut;
		this.connectTimeOut = connectTimeOut;
	}
	
	@Bean
	public RestTemplate restTemplate(ClientHttpRequestFactory factory) {
		return new RestTemplate(factory);
	}

	@Bean
	public ClientHttpRequestFactory simpleClientHttpRequestFactory() {
		PoolingHttpClientConnectionManager pollingClientConnectionManager = new PoolingHttpClientConnectionManager(30,
				TimeUnit.SECONDS);
		pollingClientConnectionManager.setMaxTotal(maxTotal);
		pollingClientConnectionManager.setDefaultMaxPerRoute(maxPerRoute);
		HttpClientBuilder httpClientBuilder = HttpClients.custom();
		httpClientBuilder.setConnectionManager(pollingClientConnectionManager);
		httpClientBuilder.setKeepAliveStrategy(new DefaultConnectionKeepAliveStrategy());
		HttpClient httpClient = httpClientBuilder.build();
		HttpComponentsClientHttpRequestFactory clientHttpRequestFactory = new HttpComponentsClientHttpRequestFactory(
				httpClient);
		clientHttpRequestFactory.setReadTimeout(readTimeOut);
		clientHttpRequestFactory.setConnectTimeout(connectTimeOut);

		return clientHttpRequestFactory;
	}

}
```

## 설명

1. 19 ~ 32 line
    - 설정파일에서 property값으로 셋팅하는 경우이다. 바로 멤버변수에 @Value 어노테이션을 사용하지 않은 이유는, Spring 컨테이너에 종속되지 않기 위해서이다.
    - Spring 컨테이너에 종속되지 않으면 어떤점이 좋은가?
        - TDD 위주의 코드 작성 시, 컨테이너 없이도 사용 가능하다.
        - 그 외는 아직 경험해보지 못함.
2. 34 ~ 37 line
    - RestTemplate를 Bean으로 등록하며, 스프링 컨테이너에서 관리할 수 있다.
3. 39 ~ 55 line
    - ClientHttpRequestFactory를 Bean으로 등록하며, 59라인의 멤버변수로 할당한다.
    - 커넥션 풀을 설정한다.
        - MaxTotal : 최대 커넥션 수
        - DefaultMaxPerRoute : 호스트 IP + Port 의 조합의 수
        - KeepAliveStrategy : 커넥션 유지 전략
        - ReadTimeout : (클라이언트 <- 서버)의 응답 제한 시간
        - ConnectTimeout : (클라이언트 -> 서버)의 요청 제한시간

## pdf 파일 요청 및 스트리밍 반환

```java
// Controller 파일
@GetMapping("/api/{idx}")
ResponseEntity<StreamingResponseBody> report(@PathVariable("idx") Long idx) {

    StreamingResponseBody body = outputStream -> StreamUtils
            .copy(myService.getReportPdf(idx).getBody().getInputStream(), outputStream);

    return ResponseEntity.ok().contentType(MediaType.APPLICATION_PDF)
            .header(HttpHeaders.CONTENT_DISPOSITION, "inline; filename=picture.jpg;")
            .header(HttpHeaders.CACHE_CONTROL, "max-age=" + cacheMaxAge).body(body);
}

// ServiceImpl 파일
public ResponseEntity<Resource> getReportPdf(long idx) {
    try {
        HttpHeaders headers = new HttpHeaders();
        headers.set("Accept", MediaType.APPLICATION_PDF_VALUE);

        UriComponentsBuilder builder = UriComponentsBuilder.fromHttpUrl(innerApiUrl)
                .queryParam("idx", idx);

        HttpEntity<String> entity = new HttpEntity<String>(null, headers);

        ResponseEntity<Resource> response = restTemplate.exchange(builder.toUriString(), HttpMethod.GET, entity,
                Resource.class);

        return response;
    } catch (Exception e) {
        log.error("api서버 호출 실패");
    }

    return ResponseEntity.status(HttpStatus.BAD_GATEWAY).build();
}
```

## 설명

1. 1 ~ 11 line
    - pdf 스트리밍을 반환한다.
2. 13 ~ 33 line
    - RestTemplate를 사용하여 url를 호출하고, 응답을 반환 받는다.

## RestTemplate 사용 시 주의점.

> 메모리 누수

RestTemplate는 굉장히 무거운 객체이다. 요청을 할 때마다 Connection을 만들게 되는데, 누적되면 거대한 메모리 누수가 발생한다.   
사용할 때는 서버의 스펙과 트래픽 용량에 맞추어 커넥션풀을 설정하여야 한다.   
또한, heap 메모리 관련 오류가 많이 발생한다면, 잘못된 수치의 설정을 사용하는 것으로 커넥션 수를 줄여주거나, heap메모리를 늘려주면 대처할 수 있다.