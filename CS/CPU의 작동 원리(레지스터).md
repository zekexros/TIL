# CPU의 작동 원리(레지스터)

<img width="389" alt="CleanShot 2023-03-21 at 22 01 12@2x" src="https://user-images.githubusercontent.com/42647277/226613631-c028e2a3-a994-4e1b-a291-623fda153ef9.png">

- **레지스터**는 CPU 내부의 작은 임시저장장치
- 프로그램 속 명령어와 데이터는 실행 전후로 레지스터에 저장
- CPU 내부에는 다양한 레지스터들이 있고 각기 다른 역할을 갖는다

<br/>

### 반드시 알아야 할 레지스터(과정을 이해하고싶다면 영상 참고하기)

`CPU 종류마다 레지스터 종류는 다르다`

1. 프로그램 카운터
   - 메모리에서 가져올 명령어의 주소(메머리에서 읽어 들일 명령어의 주소)
   - Instruction Pointer(명령어 포인터)라고 부르는 CPU도 있음
2. 명령어 레지스터
   - 해석할 명령어(방금 메모리에서 읽어 들인 명령어)
3. 메모리 주소 레지스터
   - 메모리의 주소
   - CPU가 읽어 들이고자 하는 주소를 주소 버스로 보낼 때 거치는 레지스터
4. 메모리 버퍼 레지스터
   - 메모리와 주고받을 값(데이터와 명령어)
   - CPU가 정보를 데이터 버스로 주고받을 때 거치는 레지스터

<img width="673" alt="CleanShot 2023-03-21 at 22 12 10@2x" src="https://user-images.githubusercontent.com/42647277/226616406-24ecc33f-d9b8-4fa0-bf29-e123b067e0fa.png">

<img width="667" alt="CleanShot 2023-03-21 at 22 12 16@2x" src="https://user-images.githubusercontent.com/42647277/226616402-7a9f3206-4c17-4fed-9dd1-26c459f50282.png">

<img width="645" alt="CleanShot 2023-03-21 at 22 13 59@2x" src="https://user-images.githubusercontent.com/42647277/226616784-7df0e9d5-0f10-4fb4-abd9-319329368030.png">

<img width="469" alt="CleanShot 2023-03-21 at 22 16 05@2x" src="https://user-images.githubusercontent.com/42647277/226617335-f2063b2d-3feb-4f2b-8d76-a208730df808.png">

- 순차적인 실행 흐름이 끊기는 경우
  - 특정 메모리 주소로 실행 흐름을 이동하는 명령어 실행 시
    - e.g. JUMP, CONDITIONAL JUMP, CALL, RET
  - 인터럽트 발생 시
  - ETC . .

<br/>

5. 플래그 레지스터

   - 연산 결과 또는 CPU 상태에 대한 부가적인 정보

6. 범용 레지스터

   - 다양하고 일반적인 상황에서 자유롭게 사용

7. 스택 포인터

   <img width="554" alt="CleanShot 2023-03-21 at 22 19 39@2x" src="https://user-images.githubusercontent.com/42647277/226618192-9b11a1bb-6e65-48b4-ab2f-fece1ad44bee.png">

   - 스택의 꼭대기 가리킴
   - 스택 주소 지정 방식: 스택과 스택 포인터를 이용한 주소 지정 방식

   - 스택 포인터: 스택의 꼭대기를 라기키는 레지스터(스택이 어디까지 차 있는지에 대한 표시)

8. 베이스 레지스터

   <img width="716" alt="CleanShot 2023-03-21 at 22 21 43@2x" src="https://user-images.githubusercontent.com/42647277/226618744-e762a598-5753-4a19-b9df-d061441d6b35.png">

   - 기준 주소 저장

   - 변위 주소 지정 방식: 오퍼랜드 필드의 값(변위)과 특정 레지스터의 값을 더하여 유효 주소 얻기

     - 상대 주소 지정 방식: 오퍼랜드 필드의 값(변위)과 프로그램 카운터의 값을 더하여 유효 주소 얻기

       <img width="722" alt="CleanShot 2023-03-21 at 22 24 14@2x" src="https://user-images.githubusercontent.com/42647277/226619442-e699a986-30cd-4e4e-8f7f-8155c8f94a8a.png">

     - 베이스 레지스터 주소 지정 방식: 오퍼랜드 필드의 값(변위)과 베이스 레지스터의 값을 더하여 유효 주소 얻기

       <img width="893" alt="CleanShot 2023-03-21 at 22 26 38@2x" src="https://user-images.githubusercontent.com/42647277/226620103-fada1438-342c-4c1c-919e-555c31b4eb43.png">

<br/>

## 출처

- https://www.inflearn.com/course/lecture?courseSlug=혼자-공부하는-컴퓨터구조-운영체제&unitId=149158

