# 10장 ISP 인터페이스 분리 원칙

- 사용하지 않는 기능을 가지고 있는 경우, 해당 기능을 사용하지 않더라도 그 기능이 변화됨에 따라 재배포 등의 문제가 발생할 수 있다.

## ISP와 인터페이스

- 일반적으로, 필요 이상으로 많은 걸 포함하는 모듈에 의존하는 것은 해로운 일이다.
- 소스코드뿐 아니라 아키텍처 측면에서도 많은 부분을 담당하게 된다면 불필요한 부분이 발생할 수 있다.

