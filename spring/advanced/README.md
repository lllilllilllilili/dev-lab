## 로그 추적기
- 병목 발생
  - 로그를 통해서 확인 필요
  - 개발자가 문제가 발생한 다음에 로그 남겨서 심고..
  - 미리 남겨둔다면 손쉽게 빨리 찾을 수 있다.
- 고객으로부터 요청이 오면 
  - MVC 
  - 잘만들어두면 다른 개발자가 편함
  - 모니터링 툴 도입하면 끝나지만.. 
- 애플리케이션의 흐름을 변경하면 안됌
  - 비즈니스 로직은 변경하면 안됌
- HTTP 요청 구분
  - id가 있는데 랜덤
  - 고객이 여러명 서버 요청 오면 최소한 구분을 할 수 있어야 한다. 
  - HTTP 요청 단위로 특정 단위로 남겨서 명확하게 구분이 가능해야함
  - id = 디비 트랜잭션 id이다. 
  - 처음 들어와서 끝날때까지가 하나의 트랜잭션이 됀다.