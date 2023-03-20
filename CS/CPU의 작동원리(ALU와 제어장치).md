# CPU의 작동원리(ALU와 제어장치)

<img width="848" alt="CleanShot 2023-03-20 at 23 40 09@2x" src="https://user-images.githubusercontent.com/42647277/226374148-02a8d208-4c23-438c-8ccd-798208a9f4b8.png">

<br/>

### ALU

<img width="727" alt="CleanShot 2023-03-20 at 23 41 08@2x" src="https://user-images.githubusercontent.com/42647277/226374402-141b1c94-1a8f-4c8a-bb34-1875a50ad6af.png">

<img width="760" alt="CleanShot 2023-03-20 at 23 44 19@2x" src="https://user-images.githubusercontent.com/42647277/226375341-20f1a3ec-8c35-4688-8b6c-c34a45f84a10.png">

<img width="713" alt="CleanShot 2023-03-20 at 23 44 31@2x" src="https://user-images.githubusercontent.com/42647277/226375404-e0ec5e0a-0f01-4352-be31-20d9d4d9ce06.png">

- 계산을 하기 위해서는 **피연산자**와 **수행할 연산**이 필요
- ALU는 레지스터로부터 피연산자를 받아들이고, 제어장치로부터 제어 신호를 받아들입니다.
- 연산 결과가 레지스터에 담기엔 너무 클 때 플래그 레지스터를 활용하는데 이를 오버플로우라고 한다.

![CleanShot 2023-03-20 at 23.47.08@2x](/Users/zeke/Library/Application Support/CleanShot/media/media_CcJSyJHjoA/CleanShot 2023-03-20 at 23.47.08@2x.png)

- 플래그에는 위와 같은 종류들이 있다.

<img width="678" alt="CleanShot 2023-03-20 at 23 48 42@2x" src="https://user-images.githubusercontent.com/42647277/226376697-be95ab42-30e5-4286-b70d-0b7de89f5ff7.png">

- 그리고 위와 같이 표시한다고 볼 수 있다.

<br/>

### 제어장치(가 받아들이는 정보)

- 클럭

  <img width="934" alt="CleanShot 2023-03-20 at 23 50 17@2x" src="https://user-images.githubusercontent.com/42647277/226377250-173da000-111c-47f6-a338-442ca406bd63.png">

  <img width="871" alt="CleanShot 2023-03-20 at 23 50 46@2x" src="https://user-images.githubusercontent.com/42647277/226377420-32fc3721-d27c-465a-a63a-a62ed3e6710d.png">

  - 클럭이란 컴퓨터의 모든 부품을 일사불란하게 움직일 수 있게 하는 시간 단위



- 해석할 명령어

  <img width="917" alt="CleanShot 2023-03-20 at 23 52 20@2x" src="https://user-images.githubusercontent.com/42647277/226377914-ca5febf1-efbe-4957-b236-35103982fd1f.png">



- 플래그

  <img width="943" alt="CleanShot 2023-03-20 at 23 54 18@2x" src="https://user-images.githubusercontent.com/42647277/226378501-40b8dd77-f29b-4662-81b8-fd0f8cfb30a8.png">



- 제어 신호

  <img width="925" alt="CleanShot 2023-03-20 at 23 55 02@2x" src="https://user-images.githubusercontent.com/42647277/226378727-547b607f-afc8-4f10-97a3-a5cb4331e397.png">



- 제어장치의 내보내는 정보

  <img width="952" alt="CleanShot 2023-03-20 at 23 55 42@2x" src="https://user-images.githubusercontent.com/42647277/226378942-57ab6dbd-f714-498e-a848-7a60b0b9cd99.png">

  - 레지스터에게 저장하라는 제어 신호를 내보내기
  - ALU에게 연산하라는 제어 신호를 내보내기
  - 메모리를 읽거나 쓰라는 제어 신호를 내보내기
  - 입출력장치를 읽거나 쓰거나 테스트하라는 제어 신호를 내보내기



## 출처

- https://www.inflearn.com/course/lecture?courseSlug=혼자-공부하는-컴퓨터구조-운영체제&unitId=149157

