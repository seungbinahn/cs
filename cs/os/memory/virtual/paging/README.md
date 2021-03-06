# Virtual Memory
1. 프로세스가 메모리를 요구할 때 가상 주소로만 요구하도록 하는 개념
2. OS는 가상 주소를 물리 주소로 변경하여 메모리에 접근
3. 물리주소는 필요에 따라 실시간으로 할당 되고 해제 됨

가상 주소는 각프로세스에 대해 private함 => 격리된 주소 공간 제공
운영체제가 번환 작업을 대신 수행해주무로 간단한 처리 가능

## 장점
1. 사용자에게 물리 주소를 공개하지 않고 가상 공간의 추상화 인터페이스를 제공함
2. 프로그램은 물리 메모리보다 더 큰 공간을 사용할 수 있음
3. 여러 프로그램이 동시에 더 많이 동작할 수 있음
4. 메모리, adderss의 공유가 쉬움
5. 프로세스의 생성과 보호가 효율적

## 단점
성능 감소

## 페이징
실제 데이터는 메모리 여기저기에 흩어져 있고, 필요한 만큼만 페이지로 가져와서 사용  
기본적으로 4KB 단위로 잘라서 사용    
virtual memory에는 프로세스가 현재 사용중인 페이지를 보유       
물리 메모리는 데이터와 코드를 프레임 단위로 나누어 둠    
페이지 테이블 : 각 페이지에 해당하는 프레임 번호 저장  

프로세스를 실행하는데 n개의 페이지가 필요하면, 물리 메모리에는 n개의 프리 프레임이 있어야 함   
프로세스 마다 페이지 테이블 생성  

주소를 페이지 크기로 분류하면 앞주소는 페이지번호(vpn), 뒷 번호는 offset으로 사용할 수 있다.   
이를 통해 pagetable을 탐색하면 물리 주소를 빠르게 찾을 수 있다.

지금 당장 쓰지 않는 프레임은 Disk에 저장한다. 따라서 메모리가 작더라도 많은 프로세스를 동시에 수행할 수 있다.
어떤 페이지는 메모리에 상주하고, 어떤 페이지는 디스크에 저장 되어있다. 이는 페이지 테이블에 표시되어있다.

페이지 테이블의 비트 구성 : V R M Prot Page Frame Number(PFN)  
V (Valid bit) : 페이지가 메모리에 있거나 디스크에 있음을 구별   
R (Reference bit) : 페이지가 최근에 참조 되었는가를 나타냄    
M (Modify bit) : 수정 여부를 나타냄
Prot (Protection bit) : operations의 보호 비트, Read, write, execute 가능 여부를 나타냄  
    
### 장점
물리 메모리를 쉽게 할당 할 수 있음 : 단순히 메모리가 필요하면 프리 리스트에서 가져오면 됨    
외부 단편화가 없음  
페이지를 out 시키기 편함(모든 페이지가 같고, V비트를 통해 빠르게 표현)     
다른 프로세스의 메모리를 볼 수 없으므로 메모리를 보호할 수 있음
페이지를 공유하기 쉬움    

### 단점
내부 단편화 발생 : 공간이 할당되어으므로 빈 공간을 낭비하게 됨    
성능 이슈 : 페이지 테이블과 메모리를 참조해야 하므로 2번의 엑세스 발생
=> TLB 를 통해 빠르게 탐색  
메모리 낭비 : 페이지 테이블을 저장하기 위해 메모리가 필요하다 

### 요구 페이징
필요한 순가에만 메모리에 올리는 기법, I/O의 횟수를 줄이고 메모리를 덜 사용할 수 있음. 
따라서 응답속도가 증가하며 더 많은 user를 수용할 수 있다.

페이지에 생긴 변화는 dirty를 확인해서 변화가 생겼을 때만 디스크에 기록하면 된다.

#### page fault
요구한 페이지가 evicted(디스크에 있는) paeg인 경우  
운영체제는 페이지 폴트 핸들러로 응답한다.
: 디스크에서 페이지를 찾고, 메모리로 올린다. 그리고 페이지 테이블을 업데이트 한다.    
페이지 replacement 알고리즘도 존재한다.

페이지 폴트 핸들링 과정
1. 페이지가 invalid : 디스크에 있으므로 트랩 발생
2. OS의 페이지 핸들러 코드 수행
3. 페이지가 디스크에 있으면 위치를 찾아 메인 메모리의 빈 프레임에 올림
4. 페이지 테이블 리셋

지역성
1. Temporal(시간) locality : 한번 엑세스 된 영역은 다시 한번 엑세스 될 확률이 높다.
2. Spatial(공간) locality : 요청된 영역의 주변 영역을 엑세스 할 가능성이 높다.     
따라서 페이지가 한번 메모리에 올라오면 자주 hit한다.

Cole miss(falut) : 프로세스가 처음 시작할 때는 모든 페이지가 디스크에 있으므로 계속 fault가 발생한다.

## 목적
큰 연속적인 주소공간을 프로세스에게 제공함

## TLBs(cache로 구현된 하드웨어)
Translation Lookaside BUffer(TLB, hardware)
VA를 PA로 빠르게 변환해주는 것     

지역성에 의해 cache에 조금만 저장해두어도 hit율이 높음

### Direct Mapped Cache
정해진 위치(나머지 연산 등으로 계산하여)에만 저장되는 캐시, 구현이 쉽지만 충돌이 자주 발생함

### Set associative
일정 캐시를 묶어서 조금 더 유연하게 저장 가능하도록 하는 기법 : tag를 이용한 탐색 영역이 증가함

cf) Fully associative : 모든 캐시를 찾아보아야 함