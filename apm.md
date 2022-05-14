# Jennifer

WAS가 GC를 하는 동안에는 죽은 것으로 판단함

- ibm jdk는 그냥 가다가 메모리가 limit에 다다르면 full gc를 하는 패턴
- sun jdk는 계속해서 작게 gc를 하다가 limit에 다다르면 full gc를 하는 패턴

자체 덤프를 내는 부분이 있는데

- 설정은 구성 관리 - number_of_dump_trigger 값
