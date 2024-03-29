# 외부 API 테스트

## 문제 상황
- `GitHub OAuth` 를 통해 회원이 로그인을 진행하는 로직을 단위 테스트 진행하던 중 문제가 발생했다.
- 로그인을 하게 되면, 우선 요청을 통해 해당 유저의 `Token` 값을 받아오고 그 `Token` 을 활용하여 `User` 의 정보를 가져와 `JWT` 을 생성하여 해당 웹앱에 로그인을 할 수 있는 `JWT` 와 `User` 의 정보를 반환해주는 방식으로 로그인이 진행되고 있다.
- 서비스 로직에 `WebClient` 를 통해 외부 `API` 요청으로 `User` 의 정보와 `User` 의 정보를 요청할 수 있는 `Token` 을 가져오는 로직이 있었는데, 여기서 `Test` 코드에서 임의로 설정한 `Url` 로 `API` 요청을 하는 경우 잘못된 `Url` 요청이라는 에러가 발생하고 있다.

### 문제 원인
- 테스트를 진행할때 실제 `API` 를 사용하지 못하므로, 임의의 `Url` 로 `API` 요청 시 존재하지 않는 `Url` 을 통한 `API` 요청으로 에러가 발생

### 에러 문구

![webClientError](https://user-images.githubusercontent.com/78953393/199882875-9ae6813d-8103-4434-ae61-6bf841fa518c.png)

## 문제 코드

### Token 을 요청하는 코드
```java
@Service
@RequiredArgsConstructor
public class OAuthService {
    private ResponseOauthTokenDto requestToken(String code, OauthProvider provider) {
        
        return webClient.post()
                .uri(provider.getTokenUrl())
                .accept(MediaType.APPLICATION_JSON)
                .bodyValue(tokenRequest(code, provider))
                .retrieve()
                .bodyToMono(ResponseOauthTokenDto.class)
                .block();
    }
}
```
- `User` 의 정보를 요청할 수 있는 `Token` 을 받아오는 코드이다.
- 여기서 `API` `Url` 를 통해 데이터를 요청하여 값을 받고 있는데, 테스트를 진행할때는 실제 `API` 요청을 보내지 않다 보니 문제가 발생하고 있다.

### User 정보를 요청 하는 코드
```java
@Service
@RequiredArgsConstructor
public class OAuthService {
    private ResponseUserDto getUserProfile(
            OauthProvider provider, 
            ResponseOauthTokenDto tokenResponse
        ) {
                
        return webClient.get()
                .uri(provider.getUserInfoUrl())
                .header("Authorization", "token " + tokenResponse.getAccessToken())
                .retrieve()
                .bodyToMono(ResponseUserDto.class)
                .block();
    }
}
```
- `외부 API` 를 통해 `UserProfile` 데이터를 가져오는 코드이다.
- 위와 동일한 문제가 발생하고 있다.

### 테스트 코드
```java
@DisplayName("비즈니스 로직 - OAuth")
@ExtendWith(MockitoExtension.class)
class OAuthServiceTest {
    
    @InjectMocks
    private OauthService sut;
    
    @Mock
    private MemberRepository memberRepository;
    
    private ObjectMapper mapper = new ObjectMapper();
    
    @DisplayName("유저가 OAuth 회원가입 요청을 하면, 유저 정보를 반환해 준다.")
    @Test
    void 유저_회원가입_성공() throws Exception {
        //given
        String code = "code";
        OauthProvider oauthProvider = createOAuthProvider();
        ResponseOauthTokenDto responseOauthTokenDto = createResponseOAuthTokenDto();
        ResponseUserDto responseUserDto = createRequestUserDto();
        
        given(inMemoryProviderRepository.getProvider()).willReturn(oauthProvider);
        given(memberRepository.save(member)).willReturn(member);
        
        //when
        ResponseLoginDto loginDto = sut.signup(code);
        
        
        //then
        SoftAssertions.assertSoftly(softly -> {
            softly.assertThat(loginDto.getAccessToken())
                    .as("외부 API 요청을 통해 생성된 accessToken 과 로그인 결과 반환되는 accessToken 은 동일해야 한다.")
                    .isEqualTo(responseOauthTokenDto.getAccessToken());
            softly.assertThat(loginDto.getName())
                    .as("외부 API 요청을 통해 받아온 username 과 반환되는 username 은 동일해야 한다.")
                    .isEqualTo(responseUserDto.getName());
        });
        then(memberRepository).should().save(member);
    }
}
```
- 처음 작성한 테스트
- 위에 코드로 테스트를 진행하면, `WebClient` 를 통해 `API` 에 요청을 진행할때 임의로 만들 `OAuthProvider()` 에 설정해둔 `Url` 로 요청을 하게되는데 이 경우 당연하게도 없는 `Url` 로 요청을 하는것이기 때문에 에러가 발생하게 된다.

## 문제 해결 방법
1. 임의의 테스트용 `MockWebServer` 를 사용해서, 그쪽으로 API 요청을 하고 임의의 `MockWebServer` 에 우리가 원하는 `Response` 가 반환되도록 설정하여 해결하는 방법
2. `WebClient` 를 `Mocking` 하여 사용 해결하는 방법

### MockWebServer 사용
- 우선 여러 설정도 필요하고 처음 사용하면 조금은 복잡하고 어렵지만, `Spring Team` 또한 이 방법을 사용하여 `Test` 를 진행한다고 하며 권장하는 방법이라고 한다.

```java
implementation 'com.squareup.okhttp3:okhttp:4.10.0'
testImplementation 'com.squareup.okhttp3:mockwebserver:4.10.0'
```
- 우선 `MockWebServer` 을 사용하기 위해 `build.gradle` 에 의존성을 추가해줘야 한다.
>https://mvnrepository.com 사이트에서 okhttp 와 mockwebserver 를 검색해서 추가하면 된다.
>https://square.github.io/okhttp/ 공식 사이트 문서도 읽어 보면 좋을것 같다.
- ~~여담으로 라이브러리 자체가 kotlin 으로 되어있어, 내부 코드를 보고 싶었는데 이해하기 어려웠다.~~

```java
@DisplayName("비즈니스 로직 - OAuth")
@ExtendWith(MockitoExtension.class)
class OAuthServiceTest {
    
    private static MockWebServer mockBackEnd;
    
    @InjectMocks
    private OauthService sut;
    
    ...
    
    @BeforeAll
    static void setUp() throws IOException {
        mockBackEnd = new MockWebServer();
        mockBackEnd.start();
    }

    @AfterAll
    static void tearDown() throws IOException {
        mockBackEnd.shutdown();
    }

    void init() {
        String baseUrl = String.format("http://localhost:%s",
                mockBackEnd.getPort());
        sut = OauthService.byTest(inMemoryProviderRepository, memberRepository, jwtTokenProvider, redisTemplate, baseUrl);
    }
}
```
- 테스트 코드에서도 추가적인 설정이 필요하다.
- `setUp()`: 모든 테스트가 실행되기전 `MockWebServer` 를 생성하고, `Server` 를 `Start` 해줘야한다.
- `tearDown()`: 모든 테스트가 종료되면 `MockWebServer` 를 종료해줘야 한다.
- `init()`: 테스트가 실행되기전 `WebClient` 에 `BaseUrl` 을 설정해주기 위해 테스트 전용 생성자를 통해 `OAuthService` 를 생성 하여 `sut` 에 넣어준다.

```java
@DisplayName("유저가 OAuth 회원가입 요청을 하면, 유저 정보를 반환해 준다.")
@Test
void 유저_회원가입_성공() throws Exception {
    ...
    //Object Mapper 를 사용해 responseDto 를 변환
    String responseOAuthTokenDtoToString = mapper.writeValueAsString(responseOauthTokenDto);
    String responseUserDtoToString = mapper.writeValueAsString(responseUserDto);
    
    //Token API Response 설정
    mockBackEnd.enqueue(
            new MockResponse()
                    .setBody(responseOAuthTokenDtoToString)
                    .addHeader("Content-Type", MediaType.APPLICATION_JSON)
    );
    // User Info API Response 설정
    mockBackEnd.enqueue(
            new MockResponse()
                    .setBody(responseUserDtoToString)
                    .addHeader("Content-Type", MediaType.APPLICATION_JSON)
    );
    ...
}
```
- 테스트 메서드 내부에 `WebClient` 를 통해 `API` 요청 시 반환해줄 `Response` 를 `enqueue()` 메서드를 사용해 설정해준다.
- `APPLICATION_JSON` 타입으로 각각 `responseOAuthTokenDtoToString` 와 `responseUserDtoToString` 이 `body` 에 실려 반환 된다.

```java
@DisplayName("비즈니스 로직 - OAuth")
@ExtendWith(MockitoExtension.class)
class OAuthServiceTest {
    
    @InjectMocks
    private OauthService sut;
    
    @Mock
    private MemberRepository memberRepository;
    
    private ObjectMapper mapper = new ObjectMapper();
    
    ...
    
    @DisplayName("유저가 OAuth 회원가입 요청을 하면, 유저 정보를 반환해 준다.")
    @Test
    void 유저_회원가입_성공() throws Exception {
        //given
        init();
        String code = "code";
        OauthProvider oauthProvider = createOAuthProvider();
        ResponseOauthTokenDto responseOauthTokenDto = createResponseOAuthTokenDto();
        ResponseUserDto responseUserDto = createRequestUserDto();
        String responseOAuthTokenDtoToString = mapper.writeValueAsString(responseOauthTokenDto);
        String responseUserDtoToString = mapper.writeValueAsString(responseUserDto);
        Member member = responseUserDto.toMemberByTest();
        
        mockBackEnd.enqueue(
                new MockResponse().setBody(responseOAuthTokenDtoToString)
                        .addHeader("Content-Type", MediaType.APPLICATION_JSON)
        );
        mockBackEnd.enqueue(
                new MockResponse().setBody(responseUserDtoToString)
                        .addHeader("Content-Type", MediaType.APPLICATION_JSON)
        );
        
        given(inMemoryProviderRepository.getProvider()).willReturn(oauthProvider);
        given(memberRepository.save(member)).willReturn(member);
        
        //when
        ResponseLoginDto loginDto = sut.signup(code);
        
        //then
        SoftAssertions.assertSoftly(softly -> {
            softly.assertThat(loginDto.getAccessToken())
                    .as("외부 API 요청을 통해 생성된 accessToken 과 로그인 결과 반환되는 accessToken 은 동일해야 한다.")
                    .isEqualTo(responseOauthTokenDto.getAccessToken());
            softly.assertThat(loginDto.getName())
                    .as("외부 API 요청을 통해 받아온 username 과 반환되는 username 은 동일해야 한다.")
                    .isEqualTo(responseUserDto.getName());
        });
        then(memberRepository).should().save(member);
    }
}
```
- 완성된 테스트 코드
- 우리가 설정해둔 임의의 `Response` 가 잘 반환되는걸 확인할 수 있다.

```
[2d74c81b] HTTP POST http://localhost:49416/tokenUrl

[2d74c81b] [25a822f1-1, L:/127.0.0.1:49417 - R:localhost/127.0.0.1:49416] Response 200 OK

[5b8572df] HTTP GET http://localhost:49416/userInfoUrl

[5b8572df] [25a822f1-2, L:/127.0.0.1:49417 - R:localhost/127.0.0.1:49416] Response 200 OK
```
- 추가적으로 테스트 실행 후 로그를 살펴보면 위에 처럼 우리가 설정해준 Url 로 요청을 보내고 200 OK 를 반환받는걸 확인 할 수 있다.

### WebClient Mocking 사용
- 우선 이방법은 권장되지 않는 방법이다.
- 여러 이유가 있는데 우선 `WebClient` 를 구현하고 있는 `DefaultWebClient` 에서 `WebClient` 동작에 사용되는 모든 메서드를 `Mocking` 해줘야한다.
- 서비스에서 `WebClient` 가 어떻게 사용되는지 세부 구현 내용을 전부 알아야 하기 때문에 좋지 못한 테스트 방법이 된다.

```java
when(webClientMock.get())
  .thenReturn(requestHeadersUriSpecMock);
  
when(requestHeadersUriMock.uri(oauthProvider.getTokenURl))
  .thenReturn(requestHeadersSpecMock);
  
when(requestHeadersMock.retrieve())
  .thenReturn(responseSpecMock);
  
when(responseMock.bodyToMono(ResponseOauthTokenDto.class))
  .thenReturn(responseOauthTokenDto);
```
- 이정도의 `Mocking` 이 필요하며, `MockWebServer` 를 사용하는 방식에 비해 간단하다 생각 할 수 있지만 테스트가 여러개의 경우 매번 이런 `Mocking` 은 매우 번거로울수 있다.
- 테스트 케이스가 몇개 없고, 추가적인 외부 라이브러리를 사용이 어려울때 사용하면 좋을것 같다.

## 정리

### 사용
- 테스트 케이스가 매우 적거나, 외부 라이브러리를 사용하지 못하는 특별한 경우에는 `WebClient` 를 `Mocking` 하여 사용하는것이 좋을것 같다. 그 외 경우에는 초기 설정이 조금은 복잡하지만 `MockWebServer` 를 사용할것 같다.

### 느낀점
- 항상 외부 `API` 를 사용하는 로직을 테스트할때 많은 어려움이 있는데, 또 다른 방법을 배운것 같아 좋았다.

---
## REFERENCE
[Mocking a WebClient in Spring](https://www.baeldung.com/spring-mocking-webclient)
[OkHttp](https://square.github.io/okhttp/)