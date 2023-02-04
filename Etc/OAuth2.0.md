# OAuth 2.0

- **OAuth 2.0**(Open Authorization 2.0, OAuth2)은 인증 및 인가를 위한 개방형 표준 프로토콜입니다. 
- 이 프로토콜에서는 Third-Party 프로그램에게 리소스 소유자를 대신하여 리소스 서버에서 제공하는 자원에 대한 접근 권한을 위임하는 방식을 제공합니다.
- 2012년에 OAuth1.0을 대체하였고 지금은 온라인 인가(authorization)에 있어 업계 표준이 되었습니다.
- OAuth 2.0 사용을 통해 다음과 같은 이점을 얻을 수 있습니다.
  - 이 프로토콜은 SSL(Secure Sockets Layer)에 의존하여 웹 서버와 브라우저 간의 데이터가 비공개로 유지되도록 합니다.
  - 토큰을 사용하여 사용자 데이터에 대한 제한된 접근 권한을 부여합니다.
  - 구현하기 쉽고 강력한 인증을 제공합니다.
  - SSO(single sign on)을 사용합니다.
    - SSO를 사용하면 사용자가 한 세트의 로그인 자격 증명을 사용하여 여러 애플리케이션에 접근할 수 있습니다.

<br/>

## OAuth 2.0 주요 용어

| 용어               | 설명                                                         |
| ------------------ | ------------------------------------------------------------ |
| **Authentication** | 인증, 접근 자격이 있는지 검증하는 단계를 말합니다.           |
| **Authorization**  | 인가, 자원에 접근할 권한을 부여하는 것입니다. 인가가 완료되면 리소스 접근 권한이 담긴 Access Token이 클라이언트에게 부여됩니다. |
| **Access Token**   | 리소스 서버에게서 리소스 소유자의 보호된 자원을 획득할 때 사용되는 만료 기간이 있는 Token입니다. |
| **Refresh Token**  | Access Token 만료시 이를 갱신하기 위한 용도로 사용하는 Token입니다. Refresh Token은 일반적으로 Access Token보다 만료 기간이 깁니다. |

<br/>

## OAuth 2.0의 원리

- OAuth 2.0은 인가 프로토콜이지 인증 프로토콜이 아닙니다.
- 따라서 원격 API나 유저의 데이터와 같은 리소스들에 대한 접근 권한을 부여하는 수단으로 설계되었습니다.
- OAuth 2.0은 Access token을 사용합니다. 이 Access token은 최종 사용자를 대신하여 리소스에 접근할 수 있는 권한을 나타내는 데이터 조각입니다.
- OAuth 2.0은 access token에 대한 특정한 형식을 정의하지 않았습니다. 그러나 JWT형식이 자주 사용됩니다.
- 이를 통해 토큰 발급자들은 토큰 자체에 데이터를 포함할 수 있습니다.
- 또한 보안상의 이유로 access token에는 만료 시간이 있을 수 있습니다.

<br/>

## OAuth 2.0의 4가지 역할

| 시스템 요소              | 역할                                                         |
| ------------------------ | ------------------------------------------------------------ |
| **Resource** **Owner**   | 리소스 소유자 또는 사용자. 보호된 자원에 접근할 수 있는 자격을 부여해 주는 주체. OAuth2 프로토콜 흐름에서 클라이언트를 인증(Authorize)하는 역할을 수행합니다. 인증이 완료되면 권한 획득 자격(Authorization Grant)을 클라이언트에게 부여합니다. 개념적으로는 리소스 소유자가 자격을 부여하는 것이지만 일반적으로 권한 서버가 리소스 소유자와 클라이언트 사이에서 중개 역할을 수행하게 됩니다. |
| **Client**               | 보호된 자원을 사용하려고 접근 요청을 하는 애플리케이션입니다. |
| **Resource** **Server**  | 사용자의 보호된 자원을 호스팅하는 서버입니다.                |
| **Authorization Server** | 권한 서버. 인증/인가를 수행하는 서버로 클라이언트의 접근 자격을 확인하고 Access Token을 발급하여 권한을 부여하는 역할을 수행합니다. |

<br/>

## OAuth 2.0 Scopes

- Scopes는 OAuth 2.0에서 중요한 개념입니다.
- 리소스에 대한 접근 권한을 부여할 수 있는 이유를 정확히 지정하는데 사용됩니다.
- 허용 가능한 Scopes 값들과 관련된 리소스들은 리소스 서버에 따라 다릅니다.

<br/>

## Access Token and Authorization Code

- resource owner가 접근 권한을 얻은 후에도 authorization server는 access token을 직접 반환하지 않을 수 있습니다.
- 보안을 강화하기 위해 access token을 대신하여 authorization code를 반환할 수 있고 이는 access token으로 교환됩니다.
- 추가로, Authorization Server는 access token과 함께 refresh token을 발급할 수도 있습니다.
- access token과는 달리 refresh token은 일반적으로 만료 시간이 길며 만료되면 새 access token으로 교환할 수 있습니다. 따라서 클라이언트에서는 refresh token에 대해 안전하게 저장해야 합니다.

<br/>

## Obtaining Authorization

- OAuth2 프로토콜에서는 다양한 클라이언트 환경에 적합하도록 권한 부여 방식에 따른 프로토콜을 4가지 종류로 구분하여 제공하고 있습니다.

1. Authorization Code Grant│ 권한 부여 승인 코드 방식

   - **권한 부여 승인을 위해 자체 생성한 Authorization Code를 전달하는 방식으로 많이 쓰이고** **기본이 되는 방식**입니다. 간편 로그인 기능에서 사용되는 방식으로 클라이언트가 사용자를 대신하여 특정 자원에 접근을 요청할 때 사용되는 방식입니다. 보통 타사의 클라이언트에게 보호된 자원을 제공하기 위한 인증에 사용됩니다.

     ![img](https://postfiles.pstatic.net/MjAyMzAyMDNfMTA5/MDAxNjc1Mzg2ODIyMTk2.p2TqHr3sSdI5wcVsE01Hn95R-LpblY6qk1-TJ8dmz2cg.opYLjtRDwQu1F4-9SwreEr_df0fc-b1bgCZo2iyZRTIg.JPEG.mds_datasecurity/000.jpg?type=w966)

   - 권한 부여 승인 요청 시 response_type을 code로 지정하여 요청합니다. 이후 클라이언트는 권한 서버에서 제공하는 로그인 페이지를 브라우저를 띄워 출력합니다. 이 페이지를 통해 사용자가 로그인을 하면 권한 서버는 권한 부여 승인 코드 요청 시 전달받은 redirect_url로 Authorization Code를 전달합니다. Authorization Code는 권한 서버에서 제공하는 API를 통해 Access Token으로 교환됩니다. 

   - Request & Response 예시

     | **Step 1: Authorization** |                                                              |
     | ------------------------- | ------------------------------------------------------------ |
     | **Request**               | (GET)/authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fc |
     | **Response**              | https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA&state=xyz |
     
     <br/>
     
     | **Step 2: Access Token**  |                                                              |
     |---|---|
     | **Request**               | (POST) /tokenAuthorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JWContent-Type: application/x-www-form-urlencoded grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb |
     | **Response**              | {           "access_token":"2YotnFZFEjr1zCsicMWpAA",           "token_type":"example",           "expires_in":3600,           "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",           "example_parameter":"example_value"} |
     | 특이사항                  | Authorization Code 획득 후 해당 Code로 Access Token 획득     |

   <br/>

   <br/>

2. Implicit Grant │ 암묵적 승인 방식

- **자격증명을 안전하게 저장하기 힘든 클라이언트(ex: JavaScript등의 스크립트 언어를 사용한 브라우저)에게 최적화된 방식**입니다. 

- 암시적 승인 방식에서는 권한 부여 승인 코드 없이 바로 Access Token이 발급 됩니다. Access Token이 바로 전달되므로 만료기간을 짧게 설정하여 누출의 위험을 줄일 필요가 있습니다. 

- Refresh Token 사용이 불가능한 방식이며, 이 방식에서 권한 서버는 client_secret를 사용해 클라이언트를 인증하지 않습니다. Access Token을 획득하기 위한 절차가 간소화되기에 응답성과 효율성은 높아지지만 Access Token이 URL로 전달된다는 단점이 있습니다.

  ![img](https://postfiles.pstatic.net/MjAyMDEyMjNfMjMz/MDAxNjA4NzAyMjQxOTg2.GTTG4UxxFQrWdRcrJHeQqoZODvOsdLwH70iTaJKsGJUg.wRwxtAmdLuOzscjqOjrm5MOaSF3gVJ-Q1wysvprXMTkg.PNG.mds_datasecurity/image.png?type=w966)	

- 권한 부여 승인 요청 시 response_type을 token으로 설정하여 요청합니다. 이후 클라이언트는 권한 서버에서 제공하는 로그인 페이지를 브라우저를 띄워 출력하게 되며 로그인이 완료되면 권한 서버는 Authorization Code가 아닌 Access Token를 redirect_url로 바로 전달합니다. 

- Request & Response 예시

  | **Request**  | (GET)/authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz&redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb |
  | ------------ | ------------------------------------------------------------ |
  | **Response** | http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA&state=xyz&token_type=example&expires_in=3600 |
  | **특이사항** | Authorize 요청 시 url로 Access Token이 바로 전달됨           |

<br/>

<br/>

3. Resource Owner Password Credentials Grant │ 자원 소유자 자격증명 승인 방식

- **간단하게 username, password로 Access Token을 받는 방식**입니다. 

- 클라이언트가 타사의 외부 프로그램일 경우에는 이 방식을 적용하면 안됩니다. 자신의 서비스에서 제공하는 어플리케이션일 경우에만 사용되는 인증 방식입니다. Refresh Token의 사용도 가능합니다. 

  ![img](https://postfiles.pstatic.net/MjAyMDEyMjNfMTQ4/MDAxNjA4NzAyMjYwNjY0.9NccTFwC2vvPezXtsufXP5jaltyM4f3T0Szk7Ykqe50g.4XIlCvZQnGvXWvEdHNImR53EvUs72joB3dqsambiwX4g.PNG.mds_datasecurity/image.png?type=w966)

- 위와 같이 흐름은 간단합니다. 제공하는 API를 통해 username, password을 전달하여 Access Token을 받는 것입니다. 중요한 점은 이 방식은 권한 서버, 리소스 서버, 클라이언트가 모두 같은 시스템에 속해 있을 때 사용되어야 하는 방식이라는 점입니다.

- Request & Response 예시

  | **Request**  | (POST) /tokenAuthorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JWContent-Type: application/x-www-form-urlencoded grant_type=password&username=johndoe&password=A3ddj3w |
  | ------------ | ------------------------------------------------------------ |
  | **Response** | {           "access_token":"2YotnFZFEjr1zCsicMWpAA",           "token_type":"example",           "expires_in":3600,           "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",           "example_parameter":"example_value"} |
  | **특이사항** | Username, Password로 Access Token 획득                       |

<br/>

<br/>

4. Client Credentials Grant │클라이언트 자격증명 승인 방식

- **클라이언트의 자격증명만으로 Access Token을 획득하는 방식**입니다. 

- OAuth2의 권한 부여 방식 중 가장 간단한 방식으로 클라이언트 자신이 관리하는 리소스 혹은 권한 서버에 해당 클라이언트를 위한 제한된 리소스 접근 권한이 설정되어 있는 경우 사용됩니다.

- 이 방식은 자격증명을 안전하게 보관할 수 있는 클라이언트에서만 사용되어야 하며, Refresh Token은 사용할 수 없습니다.

  ![img](https://postfiles.pstatic.net/MjAyMDEyMjNfMjIx/MDAxNjA4NzAyNDQwMzQ5.86I4OrUs0bHxoJQKBbFcPkLuXxfEf1slgI_fnF40ja8g.9a4JaG7DQf8pGcyuRmEkpSbGdbff6hETz01uGPwiwRog.PNG.mds_datasecurity/image.png?type=w966)

- Request & Response 예시

  | **Request**  | (POST) /tokenAuthorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JWContent-Type: application/x-www-form-urlencoded grant_type=client_credentials |
  | ------------ | ------------------------------------------------------------ |
  | **Response** | {           "access_token":"2YotnFZFEjr1zCsicMWpAA",           "token_type":"example",           "expires_in":3600,           "example_parameter":"example_value"} |
  | **특이사항** | 클라이언트 자격증명만으로 Access Token 획득                  |

  

## 출처

- https://blog.naver.com/mds_datasecurity/222182943542
- https://auth0.com/intro-to-iam/what-is-oauth-2
- https://www.clowder.com/post/why-your-organization-should-be-using-oauth-2.0