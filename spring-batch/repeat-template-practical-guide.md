1) 실전에서 쓰는 포인트
   RepeatTemplate은 보통 이런 상황에서 씁니다.

외부 API 호출을 짧게 재시도하고 싶을 때
“성공할 때까지”가 아니라 “N번 or T초까지만” 반복하고 싶을 때
단순 retry보다 반복 종료 조건을 세밀하게 제어하고 싶을 때
2) 실전 예시 코드 (API 호출 안정화)
   @Bean
   public ItemProcessor<Order, OrderResult> orderProcessor(ExternalOrderClient client) {
   return order -> {
   RepeatTemplate template = new RepeatTemplate();

        // 1) 최대 3번 반복
        SimpleCompletionPolicy countPolicy = new SimpleCompletionPolicy(3);
        // 2) 최대 2초까지만 반복
        TimeoutTerminationPolicy timeoutPolicy = new TimeoutTerminationPolicy(2000);

        CompositeCompletionPolicy composite = new CompositeCompletionPolicy();
        composite.setPolicies(new CompletionPolicy[]{countPolicy, timeoutPolicy});
        template.setCompletionPolicy(composite);

        // 예외 허용 횟수 (초과 시 최종 실패)
        template.setExceptionHandler(new SimpleLimitExceptionHandler(2));

        AtomicReference<OrderResult> resultRef = new AtomicReference<>();

        template.iterate(context -> {
            try {
                OrderResult result = client.send(order); // 외부 API 호출
                resultRef.set(result);
                return RepeatStatus.FINISHED; // 성공하면 즉시 종료
            } catch (TemporaryApiException e) {
                Thread.sleep(200); // 짧은 backoff
                return RepeatStatus.CONTINUABLE; // 재시도
            }
        });

        if (resultRef.get() == null) {
            throw new IllegalStateException("주문 전송 실패: 재시도 한도 초과");
        }
        return resultRef.get();
   };
   }
3) 운영에서 중요한 기준
   CONTINUABLE는 반복 계속, FINISHED는 즉시 종료
   TimeoutTerminationPolicy는 “n초 후 1번 실행”이 아니라 “n초 동안 반복 허용”
   println은 반복문에서 금지 수준으로 느림 -> 로그 레벨/샘플링 사용
   재시도 대상 예외를 명확히 분리 (TemporaryApiException vs 비즈니스 예외)
4) 언제 RepeatTemplate 대신 다른 걸 쓸까?
   단순 재시도/스킵이면 Step의 faultTolerant().retry().skip()가 더 간단합니다.
   RepeatTemplate은 “종료 조건이 복합적일 때” 쓰는 게 가장 효과적입니다.

원하면 다음으로, 네 프로젝트 기준 RepeatConfiguration을 “실전형(외부 API + 재시도 + 로깅)” 버전으로 바로 예시 코드 만들어줄게요.


