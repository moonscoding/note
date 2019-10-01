# LoadBalancing



> 하나의 인터넷 서비스가 발생하는 트래픽이 많으 때 여러 서버가 분산처리하여 서버의 로드율을 증가, 
> 부하량 , 속도저하등을 고려하여 적절히 분산처리하여 해결해주는 서비스



## 주요기능

###  NAT(Network Address Translation)

- 사설 IP주소를 공인 IP 주소로 바꾸는데 사용하는 통신망의 주소 변조기



### Tunneling

- 인터넷상에 눈에 보이지 않는 통로를 만들어 통신할 수 있게 하는 개념
- 데이터를 캡슐화하여 연결된 상호 간에만 캡슐화된 패킷을 구별해 캡슐화를 해제



### DSR(Dynamic Source Routing Protocol)

- 로드 밸런서 사용시 서버에서 클라이언트로 되돌아가는 경우 목적지 주소를 스위치의 IP 주소가 아닌 클라이언트의 IP 주소로 전달해서 `네트워크 스위치를 거치지 않고` 바로 클라이언트를 찾아가는 개념



###### 참고

- https://nesoy.github.io/articles/2018-06/Load-Balancer