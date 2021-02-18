# 리눅스 작동법

![bookcover](./img/book-cover.jpg)

---
## 목차 

[1장. 조감도](#1장-조감도)

2장. 기본 명령어와 디렉터리 계층 구조

3장. 디바이스

4장. 디스크와 파일 시스템

5장. 리눅스 커널 부팅 방법

6장. 사용자 공간 시동 방법

7장. 시스템 설정: 로깅, 시스템 시간, 일괄 작업과 사용자

8장. 프로세스와 리소스 활용

9장. 네트워크와 그 설정에 대한 이해

10장. 네트워크 응용프로그램과 서비스

11장. 셸 스크립트 소개

12장. 네트워크를 거쳐 파일 옮기기

13장. 사용자 환경

14장. 리눅스 데스크톱에 대한 조망

15장. 개발 툴

16장. C 소스 코드로 소프트웨어 컴파일하는 기본적인 방법

17장. 기초를 바탕으로 길제 구축하기

---
## 1장. 조감도

- 운영체제가 동작하는 방법을 추상(abstraction)으로 이해하자
    - 추상? 대부분의 상세 부분을 무시하는 것

### 1.1 리눅스 시스템의 추상화 레벨과 레이어

- 레이어(layer) or 레벨(level)
    - 해당 구성요소가 사용자와 하드웨어 사이의 어디에 위치하는지에 따라 구성요소를 구분(그룹으로 분류)하는 것

- 리눅스 시스템 세가지 주요 레벨
    ![리눅스레벨](./img/1/linux_level.png)

- 커널과 사용자 프로세스 동작 방식 차이점
    - 커널 
        - 커널 모드(kernel mode)에서 동작
        - 프로세서나 주기억 장치에 제한 없이 접근 가능
        - 커널 공간(kernal space)
            - 커널만 접근 가능한 영역
    - 사용자 프로세스
        - 사용자 모드(user mode)에서 동작
        - (아주 적은) 메모리 공간의 일부와 안전한 CPU작업에만 접근할수 있도록 제한됨
        - 사용자 공간
            - 사용자 프로세스가 접근할 수 있는 주기억 공간의 일부
        - 만약 프로세스가 실수하더라도 결과가 제한적이고 커널에서 해결 가능

### 1.2 하드웨어: 주기억 장치에 대한 이해

- 주기억 장치(main memory)
    - 0과 1(비트)들을 위한 커다란 저장 공간(집합체)
    - 실행 중인 커널과 프로세스가 상주하는 공간
    - 주변 장치의 모든 입출력이 비트 묶음의 형태로 주기억 장치로 흘러들어가게 됨

- CPU
    - 메모리상의 운영자
    - 명령들과 메모리의 데이터를 읽고 메모리에 다시 데이터를 기록

- 상태(state)
    - 비트의 특정한 배열
        - 예시) 메모리에 4비트 -> 0110,0001,1011 세가지 다른 종류의 상태를 표현함
    - 상태에 대해 추상적 용어를 사용
        - 비트를 사용하여 상태를 묘사하는 것이 아니라 어떤 것이 무엇을 했는지, 그 순간 어떤 동작을 하고 있는지를 묘사
        - 단 하나의 프로세스가 메모리에서 수백만의 비트로 구성되기 때문에 추상적 용어 사용이 더 쉬움 

### 1.3 커널

- 커널이 실행하는 거의 모든 작업은 주기억 장치를 위주로 진행
- 커널은 메모리를 여러 개의 구획으로 분리하고, 구획에 대한 상태 정보를 항상 보유해야함
- 각 프로세스는 메모리에 각자의 구획을 갖고, 커널은 각 프로세스가 각각의 구획을 유지를 하는지 확인

- 커널의 유지 책임(아래 네가지 일반 시스템 영역내 작업 유지)
    - 프로세스 
        - 커널은 어떤 프로세스가 CPU의 사용을 허용받았는지 알고 있어야 함 
    - 메모리 
        - 커널은 모든 메모리를 지속적으로 파악해야함
            - 현재 특정 프로세스에 할당된 메모리가 무엇인지
            - 프로세스 간 공유할 수 있는 메모리는 무엇인지
            - 할당되지 않은 것은 무엇인지
    - 장치 드라이버
        - 커널은 (디스크와 같은) 하드웨어와 프로세스 사이의 인터페이스로 동작
        - 하드웨어를 운영하는 것은 커널의 일
    - 시스템 콜과 지원
        - 프로세스는 보통 커널과 소통을 하는데 시스템 콜(system call)을 사용

#### 1.3.1 프로세스 관리

- 프로세스의 시작, 멈춤, 재개와 종료에 대한 것
- 문맥 전환(context switch)
    - 하나의 프로세스가 다른 프로세스를 위해 CPU에 대한 제어를 포기하는 행위
    - 유저입장에서는 프로세서들이 동시에 실행하는 것처럼 보이지만, 1개 코어로 실행할 경우 1초보다 짧은 시간 동안 CPU를 사용한 후 멈추고, 다른 프로세스가 CPU를 사용함
    - 커널이 책임짐
- 타임 슬라이스(time slice)
    - 각 시간의 조각
    - 중요한 연산을 하기에 충분한 시간을 프로세스에 부여
    - 조각들이 너무 작기 때문에 동시에 다수의 프로세스를 실행 하는것 처럼 보임(다중 작업 처리-multitask 능력)
- 문맥전환의 흐름
    1. CPU(실질적 하드웨어)는 내부 타이머를 기초로 하여 현재 프로세스를 중단하고 커널 모드로 전환. 그리고 커널에 통제권을 넘김
    2. 커널은 CPU와 메모리의 현재 상태를 기록. (방금 중지된 프로세스를 다시 재개하는데 필수적)
    3. 커널은 이전 타임 슬라이스 동안 발생해야 했던 작업들을 실행(입출력,여러 활동등으로부터 데이터를 수집하는 등의 작업)
    4. 커널은 다른 프로세스가 동작할 수 있도록 준비. 커널이 실행할 준비가 된 프로세스 목록을 분석하고 하나를 선택
    5. 커널은 이 새로운 프로세스를 위한 메모리 준비. 이어서 CPU 준비시킴
    6. 커널이 새로운 프로세스를 위한 타임 슬라이스가 얼마나 걸릴 것인지 CPU에게 알림
    7. 커널이 사용자 모드로 CPU를 전환하고, CPU 통제권을 프로세스에 넘김
- 커널은 문맥전환이 이뤄지는 동안 프로세스의 타임 슬라이스 사이 사이에 실행됨
- 다수의 CPU를 갖춘 시스템의 경우 좀더 복잡(현재 CPU에 대한 통제권을 커널이 포기할 필요가 없기 때문)

#### 1.3.2 메모리 관리

- 커널은 문맥 전환이 이뤄지는 동안 메모리를 관리하는 복잡한 작업 수행
- 수행 조건
    - 커널은 사용자 프로세스가 접근할 수 없는 전용 영역을 메모리 안에 보유하고 있어야 한다
    - 각 사용자 프로세스는 자신만의 메모리 구역이 있어야 한다
    - 하나의 사용자 프로세스는 다른 프로세스의 전용 영역에 접근할수 없다
    - 사용자 프로세스들은 메모리를 공유할 수 없다
    - 사용자 프로세스에서 일부 메모리는 읽기 전용이 될 수 있다.
    - 시스템은 보조로 디스크 공간을 사용함으로써 물리적으로 존재하는 것보다 더 많은 메모리를 사용할 수 있다
- 가상 메모리(virtual memory) 라고 하여 메모리 접근을 계획할수 있게 해주는 메모리 관리 유닛(memory management unit, MMU)을 포함
- 커널은 각각의 프로세스가 머신(machine) 전체를 점유하고 있는 것처럼 동작하도록 해당 프로세스를 설정
    - 프로세스가 메모리의 일정 부분에 접근할 때 MMU는 그 접근을 가로채서 메모리 어드레스 맵(ememory address map, 메모리 주소 지도)을 사용하여 프로세스의 메모리 위치를 머신상의 실질적인 물리적 메모리 위치로 변환시킴
    - 커널은 여전히 이 메모리 어드레스 맵을 초기화시키고 지속적으로 유지하며 변경해야함

#### 1.3.3 장치 드라이버와 관리

- 보통 커널 모드에서만 접근 가능 (부적절한 접근으로 머신에 손상이 올수 있기 때문에)
- 장치 드라이버는 전통적으로 커널 영역(같은 장치라도 동일한 프로그래밍 인터페이스 갖는 경우가 드물기 때문)
- 장치 드라이버들은 SW 개발자들의 작업을 단순화하기 위해 사용자 프로세스에 동일한 인터페이스를 제공하기 위해 노력중

#### 1.3.4 시스템 호출과 지원

- 시스템 콜은 사용자 프로세스 단독으로는 잘할 수 없거나 전혀 할수 없는 특정한 작업을 수행
- 파일 읽고 쓰는 기능은 모두 시스템 콜과 관련

- fork()
    - 프로세스가 fork()를 호출할 때 커널은 프로세스와 거의 일치하는 복사본을 만들어낸다.
- exec()
    - 프로세스가 exec(program)를 호출할 때 커널은 program을 시작하여 현재 프로세스를 대신한다.

- 리눅스 시스템상의 모든 사용자 프로세스는 fork()의 결과로 시작
- 대다수의 경우 기존 프로세스의 복사본을 실행하지 않고 exec()를 실행하여 새로운 프로그램을 시작

- 예시(명령어 ls 실행)
    ![새로운 프로세스의 시작](./img/1/fork_exec.png)
    
    1. 터미널 윈도우에 ls를 입력한다면 윈도우 내부에서 실행되고 있는 셸이 fork()를 호출 -> 셸의 복사본 생성
    2. 셸의 새로운 복사본이 exec(ls)를 호출하여 ls 실행 

- 의사 장치(pseudodevices)
    - 커널은 전통적인 시스템 호출이외의 기능을 통해 사용자 프로세스들을 지원, 의사 장치는 그 중 하나
    - 사용자 프로세스의 입장에서는 장치처럼 보여짐
    - 실제로는 순전히 소프트웨어로 구현됨
    - 예시
        - 커널 난수(random number) 생성장치(/dev/random)
    - 기술적으로 의사 장치에 접근하는 사용자 프로세스는 여전히 장치를 여는 데 시스템 콜을 사용해야함. 피할 수 없음

### 1.4 사용자 공간

- 커널이 사용자 프로세스에게 할당하는 주요 메모리
- 프로세스 타입과 상호작용
![프로세스 타입과 상호작용](./img/1/user_interface.png)
    - 상위레벨
        - 사용자와 가장 가까운 레벨
        - 사용자들이 직접 제어하는 복잡한 작업 수행
    - 중위레벨
        - 하위 레벨보다 좀 더 큰 요소를 포함
        - 메일, 프린트, 데이터베이스 서비스
    - 하위레벨 :
        - 커널과 가까운 레벨 
        - 단순 작업, 복잡하지 않은 작업 수행 요소들로 구성
        - 네트워크 구성, 커뮤니케이션 버스...
- 일부 사용자 공간 요소들은 분류하기 어려움(작업이 복잡해서)
    - 웹, 데이터베이스 서버

### 1.5 사용자

- 리눅스 커널은 유닉스 사용자에 대한 전통적인 개념을 지원
- 사용자(user)
    - 프로세스들을 실행할 수 있고 파일들을 소유할 수 있는 독립체
    - 사용자 이름(username)과 관련 있음
    - 사용자 ID(userid)
        - 단순한 숫자로 구성된 식별자 
- 접근 권한과 한계를 지원하기 위해 존재
- 모든 사용자 공간 프로세스는 owner 라는 사용자를 갖고 있고, 프로세스들은 owner로써 실행된다
    - 소유 프로세스의 행동을 종료, 수정 가능
    - 단, 다른 사용자의 프로세스 방해는 할 수 없음
    - 파일 소유, 다른 사용자들과 파일들을 공유할것인지 선택가능
- root
    - 다른 사용자의 프로세스를 종료시키고 변경할 수 있음
    - 슈퍼 사용자(superuer) 라고 불림
    - 루트로 운영할 수 있는 사람에 대해 루트 접근 권한을 가지고 있다고 말함
    - 전통적 유닉스 시스템상에서 관리자(administrator) 라고 함
- 그룹(groups)
    - 다수의 사용자들로 구성된 세트
    - 목적
        - 사용자가 그룹 내의 다른 사용자들과 파일 접근을 공유할수 있도록 하려는 것