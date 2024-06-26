마이크로 서비스에서 문제가 발생한 위치를 디버깅하는것은 어려운일

## 11.1 스프링 클라우드 슬루스와 상관관계 ID

- 상관관계 ID가 존재하지 않는다면 상관관계 ID를 생성해서 서비스 호출에 삽입한다.
- 아웃바운드 서비스 호출에 대한 상관관계 ID 전파를 관리하여 트랜잭션에 대한 상관관계 ID가 자동으로 추가되도록 한다.
    - **서비스 A**는 **서비스 B**에 HTTP 요청을 보낼 때, HTTP 헤더에 `X-Correlation-ID: abc123`를 포함시켜 보낸다.
    - **서비스 B**는 요청을 수신하여 `X-Correlation-ID` 헤더에서 상관관계 ID를 읽어들이고, 이 ID를 사용하여 자신의 작업을 추적한다.

```jsx
@Configuration
public class FeignConfig {
    
    @Bean
    public RequestInterceptor requestInterceptor() {
        return requestTemplate -> {
            String correlationId = MDC.get("X-Correlation-ID");
            if (correlationId != null) {
                requestTemplate.header("X-Correlation-ID", correlationId);
            }
        };
    }
}

@Component
public class CorrelationIdFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)
            throws ServletException, IOException {
        String correlationId = request.getHeader("X-Correlation-ID");
        if (correlationId == null) {
            correlationId = UUID.randomUUID().toString();
        }
        MDC.put("X-Correlation-ID", correlationId);
        response.setHeader("X-Correlation-ID", correlationId);
        try {
            filterChain.doFilter(request, response);
        } finally {
            MDC.remove("X-Correlation-ID");
        }
    }
}
```
- 서비스 호출의 추적 정보를 집킨 분산 추적 플랫폼에 발행한다.

## 11.1.1 라이선싱 및 조직 서비스에 스프링 클라우드 슬루스 추가

1. 슬루스 메이븐 의존성을 추가한다. 
- 유입되는 모든 서비스 호출을 검사하여 스프링 클라우드 슬루스의 추적 정보가 호출에 포함되었는지 확인한다.

## 11.1.2 스프링 클라우드 슬루스의 추적분석

- 모든것이 정상적으로 설정되면 서비스 애플리케이션 코드에서 출력되는 모든 로그 문에 스프링 클라우드 슬루스의 추적 정보가 포함된다.
- http://localhost:8072/organization/v1/organization/e839ee96-28….
    - organization id 발급 주체는 누구지?

```jsx
애플리케이션 이름 | 추적 ID | 스팬 ID | 집킨 전송 여부 를 나타낸다. 
```
예시
```jsx
[my-application,1a2b3c4d5e6f7g8h,1a2b3c,zipkin] INFO com.example.MyClass - This is a log message.
```

추적 ID 가 상관관계 ID에 해당하는 용어로 트랜잭션 전체에서 고유한 번호다. 

스팬 ID 는 트랜잭션을 시각화하는데 관련이 많다. 

## 11.2 로그 수집과 스프링 클라우드 슬루스

- 수집된 로그를 중앙에서 관리하게 하게끔 할 수 있다.
    - 엘라스틱 서치
    - 로그 스태시
    - 키바나 등등..

## 11.2.1 동작하는 스프링 클라우드 슬루스

- 스프링 클라우드 슬루스와 ELK 스택으로 동일한 아키텍처 구현하는 방법
1. 각 컨테이너는 로깅 데이터를 로그스태시 어펜더에 기록한다.
2. 로그스태시는 스태시 데이터를 수집, 변환하고 일레스틱서치로 전송
3. 검색 가능한 형식으로 인덱싱하고 저장하므로 나중에 키바나에서 쿼리
4. 키바나는 인덱스 패턴을 사용해 일레스틱서치에서 데이터를 검색

## 11.2.2 서비스에서 로그백 구성

- 로그스태시 인코더 추가
    - 로그 메시지를 Logstash가 이해할 수 있는 JSON 형식으로 인코딩
- 로그스태시 TCP 어펜더 생성
    - 구조화된 로그를 생성하여, 로그 데이터를 더 쉽게 검색하고 분석할 수 있도록 한다. 

LogstaEncoder 로 형식을 맞춘 애플리케이션 로그를 확인해볼 수 있다. 

## 11.2.3 도커에서 ELK 스택 애플리케이션 정의 및 실행

ELK 스택 컨테이너를 설정하려면 두 단계를 따라야 한다. 

- 로그스태시 구성 파일
- 도커 구성 안에 ELK 스택 애플리케이션 정의

로그 스태시 구성 파일

```jsx 
input { //수신
	tcp {
		port => 5000
		codec => json_lines
	}
}

filter { //로그 데이터를 필터링 
	mutate {
		add_tag => [ "manningPublications" ]
	}
}

output { //로그 데이터를 전송할 위치 
	elasticsearch {
		hosts => "elasticsearch:9200"
		}
}
```

이걸 docker-compose.yml 파일에 ELK 도커 항목에 추가해줄 수 있다. 

```jsx
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.13.2
    container_name: elasticsearch
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - "9200:9200"
    volumes:
      - esdata:/usr/share/elasticsearch/data

  logstash:
    image: docker.elastic.co/logstash/logstash:7.13.2
    container_name: logstash
    volumes:
      - ./logstash/pipeline:/usr/share/logstash/pipeline
    ports:
      - "5000:5000"

  kibana:
    image: docker.elastic.co/kibana/kibana:7.13.2
    container_name: kibana
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
    ports:
      - "5601:5601"

volumes:
  esdata:
    driver: local
```

## 11.2.5 키바나에서 스프링 클라우드 슬루스의 추적 ID 검색

- 한 트랜잭션과 관련된 모든 로그 항목을 쿼리하려면 추적 ID를 가지고와서 키바나의 Discover 스크린에서 쿼리해야 한다.
- 스프링 클라우드 게이트웨이 필터를 작성하여 HTTP 응답에 추적 ID를 삽입하는것이다.

응답 필터를 이용해 스프링 클라우드 슬루스의 추적 ID 추가하기

```jsx
@Configuration
public class ResponseFilter {
	final Logger logger = LoggerFactory.getLogger(ResponseFilter.class);
	
	~~
	
	~~
	
	@Bean
	public GlobalFilter postGlobalFilter() {
		return (exchange, chain) -> {
			return chain.filter(exchange)
				.then(Mono.fromRunnable(()-> {
						String traceId = 
							tracer.currentSpan()
								.context()
								.traceIdString(); //슬루스 추적 ID의 응답헤더에 스팬 추가
```

## 11.3 집킨을 사용한 분산 추적

- 집킨은 서비스 호출 사이의 트랜잭션을 추적할 수 있는 분산 추적 플랫폼

## 11.3.1 스프링 클라우드 슬루스와 집킨 의존성 설정

## 11.3.2 집킨 연결을 위한 서비스 구성 설정

- 스프링 클라우드 컨피그 서버의 저장소에 있는 각 서비스의 구성파일에 설정된다.
- 집킨과 통신에 사용되는 URL을 정의하는 스프링 프로퍼티인 spring.zipkin.baseUrl 을 정의한다.

```jsx
zipkin.baseUrl: http://zipkin:9411
```

## 11.3.3 집킨 서버 구성

- zipkin + 엘라스틱 서치로 구성한다.
- 인메모리 데이터베이스 제한된 데이터를 보유하고 집킨 서버가 종료되거나 장애가 발생하면 데이터가 손실된다.

## 11.3.4 추적 레벨 설정

- spring.sleuth.sampler.percentage
- 값이 0이면 스프링 클라우드 슬루스가 집킨에 트랜잭션을 전송하지 않는다.
- 값이 .5이면 스프링 클라우드 슬루스가 전체 트랜잭션의 50%를 전송한다.
- 값이 1이면 스프링 클라우드 슬루스가 전체 트랜잭션 100%를 전송한다.

## 11.3.5 집킨으로 트랜잭션 추적

*스팬 트랜잭션 내 개별 작업 또는 로직 의미 

Spring Cloud Sleuth를 사용하는 애플리케이션에서 트레이싱 데이터를 Zipkin으로 전송하려면 이 설정을 포함