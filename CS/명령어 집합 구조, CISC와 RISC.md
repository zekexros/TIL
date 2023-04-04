# 명령어 집합 구조, CISC와 RISC

<img width="775" alt="CleanShot 2023-04-04 at 09 59 08@2x" src="https://user-images.githubusercontent.com/42647277/229659224-4082594d-283a-49e6-82fb-ea62c7235060.png">

## 명령어 집합

- 명령어 집합(구조): CPU가 이해할 수 있는 명령어들의 모음

  <img width="670" alt="CleanShot 2023-04-04 at 10 01 18@2x" src="https://user-images.githubusercontent.com/42647277/229659387-56d118eb-0882-4785-83a5-ed4e39dd0737.png">

  - 인텔의 CPU는 일반적으로 "X86(X86-64)"명령어 집합을, 애플의 CPU는 일반적으로 "ARM"명령어 집합을 따릅니다

    <img width="527" alt="CleanShot 2023-04-04 at 10 02 59@2x" src="https://user-images.githubusercontent.com/42647277/229659564-efc6201c-419c-411f-a3e9-6ac11387fd7f.png">

- 명령어 집합(구조): CPU의 언어인 셈

  <img width="594" alt="CleanShot 2023-04-04 at 10 04 38@2x" src="https://user-images.githubusercontent.com/42647277/229659743-d0d799af-2ad1-481c-a66b-6d5a12cd9a07.png">

  <img width="728" alt="CleanShot 2023-04-04 at 10 05 58@2x" src="https://user-images.githubusercontent.com/42647277/229659921-01738d97-5bf3-4ab4-afcc-6cafa6015390.png">

  - 명령어가 달라지만 그에 대한 나비효과로 많은 것들이 달라진다
  - 명령어 해석 방식, 레지스터의 종류와 개수, 파이프라이닝의 용이성 등등

- 명령어 집합의 두 축: CISC & RISC

<br/>

## CISC(Complex Instruction Set Computer)

- 복잡한 명령어 집합을 활용하는 컴퓨터(CPU)

- x86, x86-64는 CISC 기반 명령어 집합 구조

- 복잡하고 다양한 명령어 활용

- 명령어의 형태와 크기가 다양한 가변 길이 명령어를 활용

  <img width="612" alt="CleanShot 2023-04-04 at 10 08 12@2x" src="https://user-images.githubusercontent.com/42647277/229660142-51a0ec61-1964-4286-8f6b-9c0ada4f15e2.png">

- 다양하고 강력한 명령어를 활용

- 상대적으로 적은 수의 명령어로도 프로그램을 실행할 수 있다.

  <img width="562" alt="CleanShot 2023-04-04 at 10 09 07@2x" src="https://user-images.githubusercontent.com/42647277/229660228-d257db6f-2446-450e-bca8-83aaaec36936.png">

- 메모리를 최대한 아끼며 개발해야 했던 시절에 인기가 높았으나 <u>명령어 파이프라이닝이 불리하다</u>는 치명적인 단점

  <img width="479" alt="CleanShot 2023-04-04 at 10 13 28@2x" src="https://user-images.githubusercontent.com/42647277/229660754-74fcac0f-2393-40cf-a71f-025105669cda.png">

- 명령어가 워낙 복잡하고 다양한 기능을 제공하는 탓에 <u>명령어의 크기와 실행되기까지의 시간</u>이 일정하지 않음

- 복잡한 명령어 때문에 <u>명령어 하나를 실행하는 데에 여러 클럭 주기</u> 필요

- 대다수의 복잡한 명령어는 사용 빈도가 낮다

  <img width="484" alt="CleanShot 2023-04-04 at 10 13 35@2x" src="https://user-images.githubusercontent.com/42647277/229660797-06a7bfca-8f88-4ef4-8ac2-cc3e22b1d373.png">

<br/>

## RISC(Reduced Instruction Set Computer)

- 명령어의 종류가 적고, 짧고 규격화된 명령어 사용

  <img width="597" alt="CleanShot 2023-04-04 at 10 15 44@2x" src="https://user-images.githubusercontent.com/42647277/229661080-3535a432-e171-4733-a8f0-0cb1b3351e7e.png">

  <img width="665" alt="CleanShot 2023-04-04 at 10 16 21@2x" src="https://user-images.githubusercontent.com/42647277/229661158-29064443-46b3-4ec2-9910-3b7688872cc7.png">

- 메모리 접근 최소화(load와 store), 레지스터 십분 활용

- 다만 명령어 종류가 CISC보다 적기에 더 많은 명령어로 프로그램을 동작시킴

  <img width="511" alt="CleanShot 2023-04-04 at 10 17 34@2x" src="https://user-images.githubusercontent.com/42647277/229661322-1603d3db-7589-4a3d-83dc-7673b2d4923d.png">

  - x86-64(CISC)
  - arm(RISC)

<br/>

## 정리

<img width="648" alt="CleanShot 2023-04-04 at 10 18 17@2x" src="https://user-images.githubusercontent.com/42647277/229661450-71513e4c-d870-4711-a99a-6fd1bcf71808.png">

- 하지만 최근의 CISC기반의 CPU들은 이러한 단점들을 극복한 형태(명령어들을 잘게 쪼개어 1클러 내외로 수행)들을 내어 효율성을 올림

<br/>

## 출처

- https://www.inflearn.com/course/lecture?courseSlug=혼자-공부하는-컴퓨터구조-운영체제&unitId=149162