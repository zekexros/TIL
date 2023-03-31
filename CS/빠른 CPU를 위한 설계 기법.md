# 빠른 CPU를 위한 설계 기법



### 클럭

- 클럭 신호가 빠르게 반복되면 CPU를 비롯한 컴퓨터 부품들은 그만큼 빠른 박자에 맞춰 움직일까요?
  - 꼭 그런건 아니지만 일반적으로는 YES

<br/>

- 클럭 속도: 헤르츠(Hz) 단위로 측정

- 헤르츠(Hz): 1초에 클럭이 반복되는 횟수

- 클럭이 '똑-딱-'하고 1초에 한 번 반복되면 1Hz

- 클럭이 1초에 100번 반복되면 100Hz

  <img width="426" alt="CleanShot 2023-03-31 at 10 55 27@2x" src="https://user-images.githubusercontent.com/42647277/229003147-f60985b1-559d-4fbb-888b-7a536f8ccd81.png">

- 클럭 신호를 마냥 높이면 CPU가 엄청 빨라지겠구나?

  - 필요 이상으로 클럭을 높이면 발열이 심해짐

<br/>

### 코어와 멀티 코어

- 클럭 속도를 늘리는 방법을 제외한 빠른 CPU를 위한 방법들
  - 코어 수를 늘리는 방법
  - 스레드 수를 늘리는 방법

<br/>

- 코어란?

  - 현대적인 관점에서 'CPU'라는 용어를 재해석 해야 함

  - '명령어를 실행하는 부품?'

  - 전통적으로 '명령어를 실행하는 부품'은 원칙적으로 하나만 존재

  - 그러나 오늘날 CPU에는 '명령어를 실행하는 부품'이 여러 개 존재

  - '명령어를 실행하는 부품'을 코어라는 용어로 사용

    <img width="650" alt="CleanShot 2023-03-31 at 11 01 08@2x" src="https://user-images.githubusercontent.com/42647277/229003918-a58afbc1-7375-4a7d-8a8d-72b035300cba.png">

    <img width="643" alt="CleanShot 2023-03-31 at 11 01 46@2x" src="https://user-images.githubusercontent.com/42647277/229003976-709f5a58-1154-4cfb-abc4-2455fdc81933.png">

- 코어를 두 개, 세 개, 100개 늘리면 이에 비례하여 연산 속도도 빨라질까?
  - 항상 그렇지는 않다

<br/>

- 스레드란 '실행 흐름의 단위'

![CleanShot 2023-03-31 at 11.05.06@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_aEc0xQLfTo/CleanShot 2023-03-31 at 11.05.06@2x.png)

- 하드웨어 스레드

  - 하나의 코어가 동시에 처리하는 명령어 단위

  - 하이퍼스레딩: 인텔의 멀티스레드 기술

    <img width="376" alt="CleanShot 2023-03-31 at 11 05 58@2x" src="https://user-images.githubusercontent.com/42647277/229004456-66a99b70-0497-4a06-b1dc-36d030ffa32e.png">
    <img width="612" alt="CleanShot 2023-03-31 at 11 06 02@2x" src="https://user-images.githubusercontent.com/42647277/229004466-274f8a88-d3d2-4214-87ad-4fc8f0eafd5e.png">

  - 멀티스레드 프로세서를 실제로 설계하는 일은 매우 복잡하지만, 가장 큰 핵심은 레지스터

    - 레지스터 세트가 하나의 코어 내부에 여러개가 있으면 하나의 코어가 여러 명령어를 동시에 처리할 수 있다.

    - 하나의 코어가 여러 레지스터 세트를 갖고있다면 멀티 스레드 프로세스를 설계할 수 있다(레지스터만 필요하다는 뜻이 아닌 핵심이라는 의미)

      <img width="810" alt="CleanShot 2023-03-31 at 11 13 44@2x" src="https://user-images.githubusercontent.com/42647277/229005441-54cf9cd5-a55a-4fbe-8cdf-c2640dca6b9b.png">

  - 하드웨어 스레드는 <u>논리 프로세서</u>라고도 부른다

    <img width="604" alt="CleanShot 2023-03-31 at 11 15 28@2x" src="https://user-images.githubusercontent.com/42647277/229005686-e104de4f-fdeb-40a3-b102-b84ab6f4f607.png">

<br/>

- 소프트웨어 스레드

  - 하나의 프로그램에서 독립적으로 실행되는 단위

    <img width="518" alt="CleanShot 2023-03-31 at 11 07 59@2x" src="https://user-images.githubusercontent.com/42647277/229004708-1383196b-3b16-4794-9634-fe4124c9b2c1.png">

  - 다음과 같이 3가지 기능을 모두 동시에 실행할 때 스레드의 모습

    1. 사용자로부터 입력받은 내용을 화면에 보여 주는 기능

    2. 사용자가 입력한 내용이 맞춤법에 맞는지 검사하는 기능

    3. 사용자가 입력한 내용을 수시로 저장하는 기능

       <img width="478" alt="CleanShot 2023-03-31 at 11 10 20@2x" src="https://user-images.githubusercontent.com/42647277/229005004-3df9761b-f0f5-4576-a689-569075fb3924.png">

  - <u>1코어 1스레드 CPU도 여러 소프트웨어적 스레드를 만들 수 있다</u>

<br/>

## 출처

- https://www.inflearn.com/course/lecture?courseSlug=혼자-공부하는-컴퓨터구조-운영체제&unitId=149160