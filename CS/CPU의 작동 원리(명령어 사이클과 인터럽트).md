# CPU의 작동원리(명령어 사이클과 인터럽트)



## 명령어 사이클

- 프로그램 속 명령어들은 일정한 주기가 반복되며 실행

- 이 주기를 <u>명령어 사이클</u>이라고 함

- 메모리에 저장된 명령어를 실행하려면?

  <img width="379" alt="CleanShot 2023-03-30 at 20 42 28@2x" src="https://user-images.githubusercontent.com/42647277/228825343-ec88c727-54b1-49de-b82c-9fa6cdedea07.png">

  - 인출 사이클: 가장 먼저 CPU로 갖고 와야 한다

  

  <img width="368" alt="CleanShot 2023-03-30 at 20 43 12@2x" src="https://user-images.githubusercontent.com/42647277/228825486-cf45ea05-e13f-4b3a-b3a5-ecee37526595.png">

  - 실행 사이클
  - 인출-실행-인출-실행 반복~~

  

  - 그런데 CPU로 명령어를 가지고 와도 바로 실행이 불가능한 경우도 있다.

    <img width="640" alt="CleanShot 2023-03-30 at 20 44 49@2x" src="https://user-images.githubusercontent.com/42647277/228825823-4d99009f-bdce-48eb-87c7-5cdf86504bf5.png">

    <img width="602" alt="CleanShot 2023-03-30 at 20 45 17@2x" src="https://user-images.githubusercontent.com/42647277/228825916-5de0e673-dc70-4ed7-9141-64a2b6552f96.png">

  

<br/>

## 인터럽트

<img width="435" alt="CleanShot 2023-03-30 at 20 40 15@2x" src="https://user-images.githubusercontent.com/42647277/228824933-25a30c6a-d77a-4400-a30a-5f568f9272bc.png">

- 인터럽트: 방해하다, 중단시키다
- 'CPU가 꼭 주목해야 할 때', 'CPU가 얼른 처리해야 할 다른 작업이 생겼을 때' 발생
- "강대리, 이거 급한거니까 지금 하던 일 멈추고 이것부터 처리해주게"

<img width="530" alt="CleanShot 2023-03-30 at 20 48 09@2x" src="https://user-images.githubusercontent.com/42647277/228826611-4ff5f3ec-630a-4da1-bccc-bbd22d3da894.png">

<br/>

### 동기 인터럽트(예외)

<img width="482" alt="CleanShot 2023-03-30 at 20 50 08@2x" src="https://user-images.githubusercontent.com/42647277/228827071-d95ef275-1417-4cf2-8344-fd446dc714e4.png">

- CPU가 예기치 못한 상황을 접했을 때 발생

  <img width="466" alt="CleanShot 2023-03-30 at 20 48 56@2x" src="https://user-images.githubusercontent.com/42647277/228826776-e542f13b-f53e-4b1c-9f63-9669b8e21f16.png">

<br/>

### 비동기 인터럽트(하드웨어 인터럽트)

- 주로 입출력장치에 의해 발생

- 알림(세탁기 완료 알림, 전자레인지 조리 알림)과 같은 역할

  <img width="691" alt="CleanShot 2023-03-30 at 20 51 09@2x" src="https://user-images.githubusercontent.com/42647277/228827301-157d0b23-debc-4649-8554-2e3de624dee0.png">

  <img width="379" alt="CleanShot 2023-03-30 at 20 52 25@2x" src="https://user-images.githubusercontent.com/42647277/228827589-b271787f-235c-491a-93a8-6d8f5c80c990.png">

- <u>입출력 작업 도중에도 효율적으로 명령어를 처리</u>하기 위해 하드웨어 인터럽트 사용

-  입출력장치는 CPU에 비해 느림, 인터럽트가 없다면 CPU는 프린트 완료 여부를 확인하기 위해 주기적으로 확인해야 한다

  <img width="416" alt="CleanShot 2023-03-30 at 20 53 50@2x" src="https://user-images.githubusercontent.com/42647277/228827938-f72281b8-4b69-4507-abcf-4c11be130873.png">

<br/>

### 하드웨어 인터럽트의 처리 순서

- 인터럽트의 종류를 막론하고 인터럽트 처리 순서는 대동소이하다
  1. 입출력장치는 CPU에 <u>인터럽트 요청 신호</u>를 보냅니다.
  2. CPU는 실행 사이클이 끝나고 명령어를 인출하기 전 항상 인터럽트 여부를 확인합니다.
  3. CPU는 인터럽트 요청을 확인하고 <u>인터럽트 플래그</u>를 통해 현재 인터럽트를 받아들일 수 있는지 여부를 확인합니다.
  4. 인터럽트를 받아들일 수 있다면 CPU는 지금까지의 작업을 백업합니다.
  5. CPU는 <u>인터럽트 벡터</u>를 참조하여 <u>인터럽트 서비스 루틴</u>을 실행합니다.
  6. 인터럽트 서비스 루틴 실행이 끝나면 4에서 백업해 둔 작업을 복구하여 실행을 재개합니다.

<br/>

- 인터럽트 요청 신호

  <img width="387" alt="CleanShot 2023-03-30 at 20 59 03@2x" src="https://user-images.githubusercontent.com/42647277/228829092-08a589a8-d087-4688-9e9b-79f5c3f57e5d.png">

- CPU가 인터럽트 요청을 받아들이려면?

  <img width="435" alt="CleanShot 2023-03-30 at 21 00 03@2x" src="https://user-images.githubusercontent.com/42647277/228829282-6110404d-2cf3-4b00-9190-7320a5e87f9b.png">

  - 인터럽트 플래그를 통해 현재 인터럽트를 받아들일 수 있는지 확인

  - 간혹 매우 긴급한 인터럽트 요청이 들어올 수 있는데 그 때는 인터럽트 플래그를 확인하지 않고 요청을 받아들임, 즉 모든 인터럽트를 인터럽트 플래그로 막을 수 있는 것은 아님

    <img width="778" alt="CleanShot 2023-03-30 at 21 02 25@2x" src="https://user-images.githubusercontent.com/42647277/228829784-2bbfd6be-ca65-46f5-954c-8e25afced096.png">

- CPU가 인터럽트를 받아들이기로 했다면 인터럽트 서비스 루틴 실행

  <img width="462" alt="CleanShot 2023-03-30 at 21 06 09@2x" src="https://user-images.githubusercontent.com/42647277/228830616-f5fdfe25-cb5e-418c-b2fe-07e56f4a69a0.png">

  - 인터럽트 서비스 루틴이란, 인터럽트가 발생했을 때 해당 인터럽트를 어떻게 처리하면 되는지 적혀있는 **프로그램**
    - "키보드가 인터럽트 요청을 보내면 이렇게 행동해야 한다"
    - "마우스가 인터럽트 요청을 보내면 이렇게 행동해야 한다
  - 인터럽트 서비스 루틴도 프로그램이기에 메모리에 저장

  <img width="193" alt="CleanShot 2023-03-30 at 21 04 45@2x" src="https://user-images.githubusercontent.com/42647277/228830289-5f8c7497-6374-4550-85cf-6dcd4f5ece52.png">

- 인터럽트 벡터: 각각의 인터럽트를 구분하기 위한 정보

  - CPU가 해당 인터럽트 서비스 루틴의 시작 주소를 알기 위해서 인터럽트 벡터를 필요로 한다.

  <img width="518" alt="CleanShot 2023-03-30 at 21 07 17@2x" src="https://user-images.githubusercontent.com/42647277/228830867-750facc6-c39c-4da5-ba7c-8cee09c6be2b.png">

<br/>

- 'CPU가 인터럽트를 처리한다' ==  '인터럽트 서비스 루틴을 실행하고 본래 수행하던 작업으로 다시 되돌아온다. 그리고 인터럽트의 시작 주소는 벡터를 통해 알 수 있다.'

<br/>

- 인터럽트 처리를 위해 메모리의 스택영역에 기존에 작업하던 내용들을 백업한다.

  <img width="520" alt="CleanShot 2023-03-30 at 21 14 43@2x" src="https://user-images.githubusercontent.com/42647277/228832605-9e597bf8-73fb-40c4-a1dd-9b692009b897.png">
  <img width="514" alt="CleanShot 2023-03-30 at 21 14 50@2x" src="https://user-images.githubusercontent.com/42647277/228832631-0c943fd9-697a-431a-bfa8-c84ce910c1ec.png">

<br/>

- 하드웨어 인터럽트의 처리 순서 정리

  <img width="624" alt="CleanShot 2023-03-30 at 21 16 20@2x" src="https://user-images.githubusercontent.com/42647277/228832971-f70c8e83-144e-445b-a29f-0cbf8aa5ca67.png">

<br/>

- 명령어 사이클

  <img width="554" alt="CleanShot 2023-03-30 at 21 17 24@2x" src="https://user-images.githubusercontent.com/42647277/228833277-6082c907-7826-4283-af7d-1d11f63d2e89.png">



<br/>

## 출처

- https://www.inflearn.com/course/lecture?courseSlug=혼자-공부하는-컴퓨터구조-운영체제&unitId=149159