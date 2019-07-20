---
tags: snippet
---
Code Snippet 1. Spring + Mybatis
===

테스트 용이성과 객체간의 커뮤니케이션을 중요시 하고 있습니다.
다만 Spring Boot + JPA 환경이 아닌, Spring + MyBatis 기반의 코드 입니다. 따라서 Repository 대신 DAO 같은 용어가 나오거나, MockMvc 를 직접 설정 해줘야 하는 차이가 있습니다. 
> Spring Boot + JAP 환경 에서의 스니펫은 2부를 통해 설명 드리겠습니다.

코드 설명 시작 하겠습니다.

# CODE
## CouponListController 
특정 유저의 쿠폰함 목록 Controller 입니다.
객체에 너무 많은 역할을 주지 않기 위해 CouponController 가 아닌, CouponListController 로 명명 하였습니다.
> CouponUsedController 등이 추가로 존재 합니다.

최근 @Autowired 를 이용한 필드 자동 주입 보다는 생성자 주입을 권고 하고 있습니다.
저 역시도 실천하고 있고, 덕분에 @Autowired 생략이 가능하며 테스트도 자연스러워 지고 있습니다.


```java=
@Slf4j
@RestController
@RequestMapping(produces = MediaType.APPLICATION_JSON_VALUE, consumes = MediaType.APPLICATION_JSON_VALUE)
public class CouponListController {

   private ApiUserService userService;
   private UserCouponService userCouponService;

   CouponListController(ApiUserService userService, UserCouponService couponService) {
      this.userService = userService;
      this.userCouponService = couponService;
   }

   @GetMapping("/api/users/{uk}/coupons")
   public ResponseEntity coupons(@PathVariable String uk){

      Integer userId = userService.getUser(uk)
            .orElseThrow(()->new NotFoundUserException(uk));

      return ResponseEntity.ok(couponService.getUserCouponResponse(userId));
   }
}
```
### 특징
1. `produces, consumes` 를 통해 MediaType을 제한 하여 명시적인 요청에만 대응 하도록 하였습니다.
2. 테스트 용이성을 위해 생성자로 의존성을 주입 받을수 있게 하였고 package-private 제한자로 필요한 만큼의 액세스만 허용 했습니다.
3. RESTful 한 URI 를 만들기 위해, user 하위의 coupons 라는 resource 를 표현할수 있도록 URI를 명시적으로 만들었습니다.
4. Java8 람다식을 적극적으로 활용 하였습니다.
5. `NotFoundUserExcpetion` 처럼 명확한 예외 처리를 하였습니다.
7. 응답 status code 를 200 만 편향 되게 사용하고 Body 의 메세지를 통해 결과를 커뮤니케이션 하는 경우를 종종 보곤 합니다. `ResponseEntity` 를 활용하여 여러가지 응답코드를 사용 하고 있습니다.
> 409 CONFLICT, 401 UNAUTHORIZED 등을 모두 활용 합니다.
> ExcpetionHandler 쪽에 등록 하였으나 생략 하겠습니다.
9. return 되는 userCouponService.getUserCouponResponse(); 는 DTO 를 반환하도록 하였습니다.
> DTO 를 만들어 presentation layer 와 service layer 를 반드시 격리 시키고 있습니다.


## UserCouponService

쿠폰함의 쿠폰목록을 조회하는 Service 입니다.
사용 가능한 쿠폰과, 사용 불가능한 쿠폰 목록을 동시에 응답 주기로 약속 하여 2개의 List를 DTO 로 생성하는 과정 입니다.

```java=
@Slf4j
@Service
public class UserCouponService {

   private String imageServerHost;
   private CouponDAO couponDAO;

   public UserCouponService(CouponDAO couponDAO, @Value("#{config['image.server.host']}") String imageServerHost) {
      this.couponDAO = couponDAO;
      this.imageServerHost = imageServerHost;
   }

   public UserCouponsResponseDto getUserCouponResponse(Integer userId) {

      List<MappedCoupon> mappedCoupons = couponDAO.getMappedCouponsByUserId(userId);

      List<MappedCouponDto> usableCouponsDto = mappedCoupons.stream()
            .filter(MappedCoupon::isUsable)
            .map(this::convertDto)
            .collect(toList());

      List<MappedCouponDto> notUsableCouponsDto = mappedCoupons.stream()
            .filter(MappedCoupon::isNotUsable)
            .map(this::convertDto)
            .collect(toList());

      return UserCouponsResponseDto.create(usableCouponsDto, notUsableCouponsDto);
   }

   private MappedCouponDto convertDto(MappedCoupon mc) {
      return new MappedCouponDto(mc, imageServerHost);
   }
}
```
### 특징
1. Controller와 달리 생성자를 public 으로 했는데, Service는 여러 패키지 에서 호출 될 수 있기 때문 입니다. 마찬가지로 테스트를 위해 필수 의존객체들을 생성자를 통해 주입받습니다.
2. `@Value("#{config['image.server.host']}")` property를 필드에 작성할 수도 있었지만 테스트를 위해 생성자 주입 받도록 하였습니다.
3. DAO에서 반환 된 mappedCoupons 를, 2개의 List로 분리 하였습니다. 
> 하나의 for문으로 처리 할 수도 있지만 'for문 안에 2개의 관점이 함께 있다면, 2개로 분리하라' 라는 `클린코더스-백명석` 님의 말을 참고 하였습니다.
4. stream 내부에서 DAO에서 받은 Entity 정보를 convertDto 로 치환하고 있습니다. 이때 imageServerHost를 주입하여 최종 생성되는 URL 을 바꿔 줍니다.
4. `UserCouponsResponseDto.create(dto,dto);`  를 통해 2개의 dto를 조합 하여 하나의 Response용 DTO를 만들고 있습니다.

## ResponseDTO

```java=
@Getter
@ToString
@NoArgsConstructor(access = AccessLevel.PRIVATE)
public class UserCouponsResponseDto {

   private List<MappedCouponDto> usableCoupons;
   private int usableCouponCount;
   private List<MappedCouponDto> noUsableCoupons;
   private int noUsableCouponCount;

   public static UserCouponsResponseDto create(List<MappedCouponDto> usableCoupons, List<MappedCouponDto> noUsableCoupons) {
      UserCouponsResponseDto dto = new UserCouponsResponseDto();
      dto.usableCoupons = usableCoupons;
      dto.noUsableCoupons = noUsableCoupons;
      dto.usableCouponCount = usableCoupons.size();
      dto.noUsableCouponCount = noUsableCoupons.size();
      return dto;
   }
}
```

### 특징
1. `@NoArgsConstructor(access = AccessLevel.PRIVATE)` 는 외부 생성자를 통해 해당 DTO 를 생성하지 못하도록 하였습니다.
2. `public static create()` 메소드로만 Dto를 생성 하도록 강제 하였습니다. 생성자에서 모든 응답 필드들의 값을 set 합니다.
> create() 를 별도로 만든 이유는, `new UserCouponsResponseDto(dto,dto);` 보다 좀더 명시적이며 Method Reference 식 으로도 쓸수 있어 create() 로 객체간 커뮤니케이션 하도록 하였습니다.


# TEST
## CouponListControllerTest

dao 결과에 의존하는 Integration Test 보다는, Unit Test를 선호 하는 편 입니다.
Unit Test 예시를 스니펫 하였습니다.


```java=
public class CouponListControllerTest {

   private MockMvc mockMvc;
   private ApiUserService mockUserService = mock(ApiUserService.class);

   @Before
   public void mockMvcSetup() {
      mockMvc = MockMvcBuilders
            .standaloneSetup(new CouponListController(mockUserService))
            .setControllerAdvice(new ApiExceptionHandler())
            .alwaysDo(print())
            .build();
   }

   @Test
   public void 파라미터_없으면_404() throws Exception {
      getPerform("/api/users//" + "/coupons").andExpect(status().isNotFound());
   }

   @Test
   public void 유저키_파라미터있을시_200() throws Exception {

      // given
      String userKey = "있는유저키";

      // when
      int expectReturnUserId = 10000;
      when(mockUserService.getUser(userKey)).thenReturn(Optional.of(expectReturnUserId));

      // then
      getPerform("/api/users/"+userKey + "/coupons").andExpect(status().isOk());
   }


   @Test
   public void 없는유저인경우_401_UNAUTHORIZED() throws Exception {

      // given
      String notFoundUser = "없는유저키";

      // when
      when(mockUserService.getUser(notFoundUser)).thenThrow(new NotFoundUserException(notFoundUser));

      // then
      getPerform("/api/users/" + notFoundUser + "/coupons").andExpect(status().isUnauthorized());
   }

   private ResultActions getPerform(String url) throws Exception {
      return mockMvc.perform(get(url).contentType(MediaType.APPLICATION_JSON_UTF8_VALUE).accept(MediaType.APPLICATION_JSON_UTF8_VALUE));
   }
}
```

### 특징
1. given - when - then 구조를 명시 합니다.
2. perform(get(url)) 코드가 반복되고 있어, getPerform() 으로 method Extract 하였습니다.
3. print() 도 반복되어 alwaysDo() 로 작성했습니다.
4. mock 객체를 적극 활용 하여 presentation layer 가 집중 해야 할 대상만 집중적으로 테스트 할 수 있도록 하였습니다.


### UserCouponServiceTest
`createUsedFixture()` 외 fixture 를 생성하는 코드는 생략 하였습니다.
> 참고로 메소드 화 한 이유는, 다른 테스트에서도 활용할수 있어 중복을 막기 위해 Extract Method 하였습니다.

또한 여러 상황을 고려한 테스트들이 너무 많아, 지면 절약상 생략 하였습니다.

```java=
@Slf4j
public class UserCouponServiceTest {

   private String imageServerHost = "https://www.test";
   private UserCouponService couponService;
   private CouponDAO couponDAO;

   @Before
   public void setup(){
      couponDAO = mock(CouponDAO.class);
      couponService = new UserCouponService(couponDAO, imageServerHost);
      when(couponDAO.getMappedCouponsByUserId(anyInt()))
            .thenReturn(Arrays.asList(createUsedFixture(), createExpiredFixture(), createUsableFixture()));
   }


   @Test
   public void getUserCouponResponse_사용불가목록은_2개이다(){

      // given - @Before

      // when
      UserCouponsResponseDto userCouponResponse = couponService.getUserCouponResponse(anyInt());

      List<MappedCouponDto> noUsableCoupons = userCouponResponse.getNoUsableCoupons();
      int noUsableCouponCount = userCouponResponse.getNoUsableCouponCount();

      // then
      assertThat(noUsableCoupons).hasSize(2);
      assertThat(noUsableCouponCount).isEqualTo(2);
   }

   @Test
   public void 사용불가목록2개_아이템별_검증(){

      // given - @Before

      // when
      List<MappedCouponDto> noUsableCoupons = couponService.getUserCouponResponse(anyInt()).getNoUsableCoupons();

      // then1. 0번째 아이템은 기간도 만료 됐고, View 도 하였다.
      assertThat(noUsableCoupons.get(0).isUsed()).isTrue();
      assertThat(noUsableCoupons.get(0).isExpired()).isTrue();
      assertThat(noUsableCoupons.get(0).getRwdImgUrl()).startsWith(imageServerHost);
      assertThat(noUsableCoupons.get(0).getIntroMsgs()).hasSize(5);

      // then2. 1번째 아이템은 기간은 만료 되었지만, View 하지는 않았다.
      assertThat(noUsableCoupons.get(1).isUsed()).isFalse();
      assertThat(noUsableCoupons.get(1).isExpired()).isTrue();
      assertThat(noUsableCoupons.get(1).getRwdImgUrl()).startsWith(imageServerHost);
      assertThat(noUsableCoupons.get(1).getIntroMsgs()).hasSize(5);
   }

    // 이하 생략...
}
```