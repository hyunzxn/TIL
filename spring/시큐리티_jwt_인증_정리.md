# 1. 들어가며

JWT 인증에 대해서는 많이 들어봤지만 항상 제대로 공부해본 적이 없었다. 그렇지만 늘 JWT를 이용한 인증방식에 대해서 제대로 공부를 해야 될 것 같았다. (인증은 모든 애플리케이션에 필수적인 요소이니까...)

이번에 공부를 해보니 JWT 자체에 대한 공부보다 스프링 시큐리티에 대한 공부를 더 많이 하게 된 것 같다.

이번 글은 JWT 인증을 어떻게 하는지에 대해서 다루기 보단 스프링 시큐리티와 JWT를 사용하여 인증 처리를 하는 과정에 있어 그 안에 녹아있는 개념에 대해서 정리하고자 하는 용도이다. 따라서 JWT 인증을 따라하기 위한 튜토리얼을 기대하는 분들은 조용히 뒤로가기를 해주시면 감사하겠다.

(초보자가 쓴 글이므로 틀린 부분에 대해서는 자유롭게 의견을 주시면 감사하겠습니다.)

<br>

# 2. JWT란?

## 2.1 JWT의 개념

JWT(Json Web Token)란 인증에 사용되는 웹 토큰으로서 JSON 방식을 사용하는 Claim 기반의 웹 토큰이다. 

**JWT 구조**

JWT는 Header, Payload, Signature 세 가지 부분으로 구성되어있다. JSON 형태인 각 부분은 모두 Base64Url로 인코딩되어 있다. JWT의 각 부분을 연결해주기 위해서 . 이 사용된다. JWT의 공식 홈페이지를 들어가면 가장 예시가 되는 JWT를 보면 이해가 더 쉽다.

<img src="https://github.com/hyunzxn/TIL/assets/100478841/7a835f2c-6bf0-48d4-8546-1e6b2e4a5e7f" alt="image-20230829192443381" style="zoom:30%;" />

왼쪽이 인코딩 된 JWT의 모습이고 오른쪽이 디코딩 된 부분으로서 Header, Payload, Signature 세 부분으로 나뉘어져 있는 것을 확인할 수 있다. 각각의 부분이 어떤 역할을 하는지 간단히 알아보면 다음과 같다.

- Header

  - alg: 어떤 알고리즘을 사용하여 Signature를 해싱할지를 명시하는 부분이다.

  - typ: 토큰의 타입을 지정하는 부분이다.

- Payload

  토큰에서 사용되는 주요 정보들이 저장되는 부분이다. 각각의 정보의 조각들을 클레임(Claim)이라고 한다. 여기에 토큰 발행시간, 토큰 만료시간 등 토큰에 관한 주요한 정보들이 여기에 담겨져 있다.

- Signature

  토큰을 인코딩하거나 유효성 검증을 할 때 사용하는 고유한 암호화 코드이다. 여기에 있는 서명으로 Header와 Payload에 있는 값을 각각 Base64Url로 인코딩한다.



JWT를 만들고 나면 이제 인증이 필요한 API에 대해서 요청을 보낼 때 HTTP Request Header에 로그인 시 발급받은 토큰을 담아야 한다. 서버는 이 토큰을 확인하여 유효한 토큰인 경우에만 요청에 대한 응답을 클라이언트에 보내주게 되는 것이다. 

JWT를 사용하게 되면 세션이 필요없게 되므로 DB에 부담이 덜 하다는 장점이 있다. 다만 JWT가 네트워크 통신 중에 탈취될 경우에 보안상 취약점을 가진다는 한계를 지닌다. 이를 극복하기 위해 엑세스 토큰과 리프레시 토큰으로 구분하여 활용하는 방식이 있다. 이번 글에서는 그 부분을 자세히 다루지 않을 것이다.



JWT에 대한 좋은 글들은 구글에 넘치도록 있으니 이 글에서는 이 정도로만 간단하게 정리하고자 한다.



(참고: https://mangkyu.tistory.com/56)

<br>

# 3. 스프링 시큐리티와 JWT로 인증 구현해보기

여기서 설명하는 모든 코드는 아래 깃허브 리포지토리에서 확인할 수 있다.

📌코드 확인: https://github.com/hyunzxn/JWT-Tutorial



스프링 시큐리티와 JWT를 활용하여 인증 처리를 하기 위해 사용되는 주요 개념들이 몇 가지 있다.

1. TokenProvider
2. JWT Custom Filter
3. Custom UserDetails & Custom UserDetailsService



이제부터 각각에 대해서 자세히 살펴보고자 한다.

## 3.1 TokenProvider

거창하게 TokenProvider라고 하였지만 사실 TokenProvider는 스프링 시큐리니 혹은 JWT가 공식적으로 제공하는 인터페이스나 클래스가 아니다. 개발자가 만든 클래스이다. 우선 코드를 보면 다음과 같다.

```java
@Slf4j
@Component
public class TokenProvider {

	private static final String AUTHORITIES_KEY = "auth";
	private static final String BEARER_TYPE = "bearer";
	private static final long ACCESS_TOKEN_EXPIRE_TIME = 1000 * 60 * 30; // 30분
	private static final long REFRESH_TOKEN_EXPIRE_TIME = 1000 * 60 * 60 * 24 * 7; // 7일

	private Key key;

	// application.yml에서 주입받은 secret 값을 base64 decode하여 key 변수에 할당
	public TokenProvider(@Value("${jwt.secret}") String secret) {
		byte[] keyBytes = Decoders.BASE64.decode(secret);
		this.key = Keys.hmacShaKeyFor(keyBytes);
	}

	// Authentication 객체에 포함되어 있는 권한 정보들을 담은 토큰을 생성
	public TokenDto generateTokenDto(Authentication authentication) {
		// 권한들 가져오기
		String authorities = authentication.getAuthorities().stream()
			.map(GrantedAuthority::getAuthority)
			.collect(Collectors.joining(","));
		long now = (new Date()).getTime();

		// Access Token 생성
		Date accessTokenExpiresIn = new Date(now + ACCESS_TOKEN_EXPIRE_TIME);
		String accessToken = Jwts.builder()
			.setSubject(authentication.getName())
			.claim(AUTHORITIES_KEY, authorities)
			.setExpiration(accessTokenExpiresIn)
			.signWith(key, SignatureAlgorithm.HS512)
			.compact();


		// Refresh Token 생성
		String refreshToken = Jwts.builder()
			.setExpiration(new Date(now + REFRESH_TOKEN_EXPIRE_TIME))
			.signWith(key, SignatureAlgorithm.HS512)
			.compact();

		return TokenDto.builder()
			.grantType(BEARER_TYPE)
			.accessToken(accessToken)
			.accessTokenExpiresIn(accessTokenExpiresIn.getTime())
			.refreshToken(refreshToken)
			.build();

	}

	// JWT토큰을 복호화하여 토큰에 들어있는 정보를 꺼냅니다.
	public Authentication getAuthentication(String accessToken) {
		// 토큰 복호화 : JWT의 body
		Claims claims = parseClaims(accessToken);

		if(claims.get(AUTHORITIES_KEY) == null) {
			throw new RuntimeException("권한 정보가 없는 토큰입니다.");
		}

		// 클레임에서 권한 정보 가져오기
		Collection<? extends GrantedAuthority> authorities =
			Arrays.stream(claims.get(AUTHORITIES_KEY).toString().split(","))
				.map(SimpleGrantedAuthority::new)
				.collect(Collectors.toList());

		// UserDetails 객체를 만들어서 Authentication 리턴
		UserDetails principal = new User(claims.getSubject(),"", authorities);
		return new UsernamePasswordAuthenticationToken(principal,"",authorities);
	}

	// 토큰을 검증하는 역할
	public boolean validateToken(String token) {
		try {
			Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(token);
			return true;
		}catch(io.jsonwebtoken.security.SecurityException | MalformedJwtException e) {
			log.info("잘못된 JWT 서면입니다.");
		}catch(ExpiredJwtException e) {
			log.info("만료된 JWT 토큰입니다.");
		}catch(UnsupportedJwtException e) {
			log.info("지원되지 않는 JWT토큰입니다.");
		}catch (IllegalArgumentException e) {
			log.info("JWT 토큰이 잘못되었습니다.");
		}
		return false;
	}

	private Claims parseClaims(String accessToken) {
		try{
			return Jwts.parserBuilder().setSigningKey(key).build().parseClaimsJws(accessToken).getBody();
		}catch (ExpiredJwtException e) {
			return e.getClaims();
		}
	}


}
```

TokenProvider의 역할을 정리해보자면 다음과 같다. 

- JWT 생성 
- JWT 로부터 정보를 추출하고 Authentication 객체 생성
- JWT 유효성 검증



**JWT생성**

JWT를 활용하는 인증 처리 과정이므로 당연히 JWT를 만드는 메서드가 필요할 것이다. 위 코드에서는 `generateTokenDto()` 메서드가 이 역할을 담당한다. 여기선 액세스 토큰과 리프레시 토큰 모두 생성하고 있다.



**JWT로부터 정보를 추출하고 Authentication 객체 생성**

TokenProvider의 가장 중요한 역할이다. 위 코드에서는 `getAuthentication()` 메서드가 이 역할을 담당한다. `getAuthentication()` 메서드는 로그인 비즈니스 로직이 담겨져 있는 `AuthService`와 JWT를 HTTP 요청 시 확인하는 `JwtFilter` 에서 호출된다.

코드를 보면 `UsernamePasswordAuthenticationToken` 이 나오는데 `UsernamePasswordAuthenticationToken` 클래스는 `AbstractAuthenticationToken` 클래스를 상속하는 클래스인데 `AbstractAuthenticationToken` 는 `Authentication` 인터페이스를 구현한 클래스이다.

이러한 이유로 `getAuthentication()`  메서드는  `UsernamePasswordAuthenticationToken`을 리턴하고 있는데  이 메서드의 리턴 타입이 `Authentication` 인 것을 확인할 수 있다.

코드를 확인해보면 `UserDetails principal = new User(claims.getSubject(),"", authorities);` 부분이 있다. 여기서의 `User`는 `org.springframework.security.core.userdetails` 패키지에 있는 User 클래스를 의미한다. User 클래스는 `UserDeatails`와 `CredentialsContainer`를 구현한다. 



**JWT 유효성 검증**

JWT의 유효성을 검증하는 메서드이다. 이 메서드도 로그인 비즈니스 로직이 담겨져 있는 `AuthService`와 JWT를 HTTP 요청 시 확인하는 `JwtFilter` 에서 호출된다. 



## 3.2 JWT Custom Filter

JWT Custom Filter는 HTTP 요청이 디스패처 서블릿을 거쳐 스프링 컨테이너에 도달하기 이전에 JWT에 관련된 로직을 수행하는 필터이다. 코드는 다음과 같다.

```java
@Slf4j
@RequiredArgsConstructor
public class JwtFilter extends OncePerRequestFilter {

	public static final String AUTHORIZATION_HEADER = "Authorization";
	public static final String BEARER_PREFIX = "Bearer";

	private final TokenProvider tokenProvider;

	@Override
	protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response,
		FilterChain filterChain) throws ServletException, IOException {

		String requestURI = request.getRequestURI();

		// 1. Request Header 에서 토큰을 꺼냄
		String jwt = resolveToken(request);

		// 2. validateToken 으로 토큰 유효성 검사, 정상 토큰이면 해당 토큰으로 Authentication을 가져와서 SecurityContext에 저장
		if (StringUtils.hasText(jwt) && tokenProvider.validateToken(jwt)) {
			Authentication authentication = tokenProvider.getAuthentication(jwt);
			SecurityContextHolder.getContext().setAuthentication(authentication);
			log.info("Security Context에 '{}' 인증 정보를 저장했습니다, uri: {}", authentication.getName(), requestURI);

		} else {
			log.debug("유효한 JWT 토큰이 없습니다, uri: {}", requestURI);
		}

		filterChain.doFilter(request, response);

	}

	// HttpServletRequest 객체의 Header에서 token을 꺼내는 역할을 수행
	private String resolveToken(HttpServletRequest request) {
		String bearerToken = request.getHeader(AUTHORIZATION_HEADER);
		if (StringUtils.hasText(bearerToken) && bearerToken.startsWith(BEARER_PREFIX)) {
			return bearerToken.substring(7);
		}
		return null;
	}
}
```

JWTFilter의 역할은 HTTP Request Header에 담겨있는 토큰을 꺼내서 확인하고 SecurityContext에 등록해주는 역할이다. 이렇게 SecurityContext에 넣어줘야 서버는 사용자가 인증이 됐다고 인식을 할 수 있다. 

로직을 간단하게 정리해보면 다음과 같다.

1. `Authentication authentication = tokenProvider.getAuthentication(jwt);` 

   토큰에서 정보를 추출해서 Authentication 객체를 생성한다.

2. `SecurityContextHolder.getContext().setAuthentication(authentication);`

   SecurityContextHolder에서 컨텍스트를 하나 꺼내서 거기에 1번에서 생성한 Authentication 객체를 집어넣어준다.

스프링 시큐리티를 사용해서 인증을 한다는 것은 결국 SecurityContextHolder Authentication 객체를 넣어준다는 것으로 이해할 수도 있다.



## 3.3 Custom UserDetails & Custom UserDetailsService

Custom UserDetails 과 Custom UserDetailService 를 살펴보기 이전에 UserDetails와 UserDetailService에 대해서 알아야 한다.



우선 UserDetails는 유저의 정보를 저장하는 곳이라고 이해하면 편한데 이에 대한 스프링 시큐리티 팀이 코드에 설명해 놓은 주석을 참고하면 이해가 바로 된다.

> They simply store user information which is later encapsulated into Authentication objects.

유저 정보를 저장하고 추후에 Authentication 객체를 만들 때 그 안에 들어가게 된다는 것이다. 



UserDetailsService는 UserDetails를 찾는 인터페이스라고 생각하면 된다. 이 인터페이스는 `loadUserByUsername(String username)` 메서드만을 가지고 있다. 



CustomUserDetails와 CustomUserDetailsService의 코드는 다음과 같다.

```java
@ToString
@Builder
@AllArgsConstructor
@Getter
public class CustomMemberDetails implements UserDetails {

	private Long id;
	private String email;
	private String password;
	private Collection<GrantedAuthority> role;

	@Override
	public Collection<? extends GrantedAuthority> getAuthorities() {
		return role;
	}

	@Override
	public String getPassword() {
		return this.password;
	}

	@Override
	public String getUsername() {
		return this.email;
	}

	@Override
	public boolean isAccountNonExpired() {
		return true;
	}

	@Override
	public boolean isAccountNonLocked() {
		return true;
	}

	@Override
	public boolean isCredentialsNonExpired() {
		return true;
	}

	@Override
	public boolean isEnabled() {
		return true;
	}
}
```

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class CustomUserDetailService implements UserDetailsService {

	private final MemberRepository memberRepository;

	@Override
	public UserDetails loadUserByUsername(String username) {
		Member member = memberRepository.findByEmail(username)
			.orElseThrow(() -> new UsernameNotFoundException(username + "을/를 데이터베이스에서 찾을 수 없습니다."));

		// DB 에 Member 값이 존재한다면 CustomMemberDetails 객체로 만들어서 리턴
		GrantedAuthority grantedAuthority = new SimpleGrantedAuthority(member.getRole().toString());

		return CustomMemberDetails.builder()
			.id(member.getId())
			.email(member.getEmail())
			.password(member.getPassword())
			.role(Collections.singleton(grantedAuthority))
			.build();
	}
}
```

(참고: 코드에서 내가 만든 유저 관련 엔티티의 이름이 Member라 관련된 부분을 모두 User -> Member로 수정해줬다.)



코드를 간단하게 정리해보자면 CustomUserDetailsService에서 유저 이름으로 멤버를 찾아서 그 멤버의 정보를 CustomMeberDetails 객체를 생성할 때 넣어주면서 UserDetails 객체를 생성하는 것이다.



> ❗️내가 만난 에러!
>
> 로직을 다 구현하고 실제 HTTP 요청을 보냈더니 계속해서 401 에러가 발생했다. 이유를 한참 삽질하다가 알게 됐는데 UserDetails를 구현할 때 오버라이드하는 메서드들 중 isAccountNonExpired(), isAccountNonLocked(), isCredentialsNonExpired(), isEnabled() 이게 디폴트 값이 false인데 이걸 true로 해줘야 정상적으로 인증 처리가 된다.





# 4. 로그인 해보기

지금까지 JWT를 생성하고 이것을 Filter에서 검증하고 DB에서 인증된 사용자를 조회하는 것에 대한 것을 정리해봤다. 이제 실제 로그인을 할 때 어떤 로직이 있는지 살펴보고자 한다. 회원가입 및 로그인 로직은 `AuthService` 로직에서 담당하고 있다.



```java
@Service
@RequiredArgsConstructor
public class AuthService {

	private final AuthenticationManager authenticationManager;
	private final MemberRepository memberRepository;
	private final PasswordEncoder passwordEncoder;
	private final TokenProvider tokenProvider;
	private final RefreshTokenRepository refreshTokenRepository;

	/**
	 * 회원가입
	 */
	@Transactional
	public MemberResponseDto signUp(MemberRequestDto memberRequestDto) {
		if(memberRepository.existsByEmail(memberRequestDto.getEmail())) {
			throw new RuntimeException("이미 가입되어 있는 스튜디오입니다.");
		}

		Member member = memberRequestDto.member(passwordEncoder);
		return MemberResponseDto.memberResponseDto(memberRepository.save(member));
	}

	/**
	 * 로그인
	 */
	@Transactional
	public TokenDto login(MemberRequestDto memberRequestDto) {
		// 1. ID(email)/PW 기반으로 AuthenticationToken 생성
		UsernamePasswordAuthenticationToken authenticationToken = memberRequestDto.toAuthentication();

		// 2. 실제 검증 로직(사용자 비밀번호 체크)
		// authenticate 메서드가 실행이 될 때 CustomUserDetailsService 에서 만든 loadUserByUsername 메서드가 실행됨
		Authentication authentication = authenticationManager.authenticate(authenticationToken);

		// 3. 인증 정보를 기반으로 JWT토큰 생성
		TokenDto tokenDto = tokenProvider.generateTokenDto(authentication);

		// 4. RefreshToken 저장
		RefreshToken refreshToken = RefreshToken.builder()
			.key(authentication.getName())
			.value(tokenDto.getRefreshToken())
			.build();

		refreshTokenRepository.save(refreshToken);

		// 5. 토큰 발급
		return tokenDto;
	}

	/**
	 * 토큰 재발급
	 */
	@Transactional
	public TokenDto refresh(TokenRequestDto tokenRequestDto) {
		// 1. Refresh Token 검증 (validateToken() : 토큰 검증)
		if(!tokenProvider.validateToken(tokenRequestDto.getRefreshToken())) {
			throw new RuntimeException("Refresh Token이 유효하지 않습니다.");
		}

		// 2. Access Token에서 Studio ID 가져오기
		Authentication authentication = tokenProvider.getAuthentication(tokenRequestDto.getAccessToken());

		// 3. 저장소에서 Studio ID를 기반으로 Refresh Token값 가져옴
		RefreshToken refreshToken = refreshTokenRepository.findByKey(authentication.getName())
			.orElseThrow(() -> new RuntimeException("로그아웃 된 사용자입니다."));

		// 4. Refresh Token 일치 여부
		if (!refreshToken.getValue().equals(tokenRequestDto.getRefreshToken())) {
			throw new RuntimeException("토큰의 유저 정보가 일치하지 않습니다.");
		}

		// 5. 새로운 토큰 생성
		TokenDto tokenDto = tokenProvider.generateTokenDto(authentication);

		// 6. 저장소 정보 업데이트
		RefreshToken newRefreshToken = refreshToken.updateValue(tokenDto.getRefreshToken());
		refreshTokenRepository.save(newRefreshToken);

		// 토큰 발급
		return tokenDto;
	}
}
```

이 코드에서 로그인 관련된 코드만 보면 되는데 흐름을 정리해보면 다음과 같다

1. 유저가 입력한 아이디와 비밀번호를 기반으로 `UsernamePasswordAuthenticationToken` 생성
2. AuthenticationManager를 활용해서 인증 처리하고 Authentication 객체 생성
3. 생성한 Authentication 객체를 바탕으로 JWT 생성
4. JWT 리턴
5. 로그인 성공적!



`UsernamePasswordAuthenticationToken` 역시 `AbstractAuthenticationToken` 을 상속한 토큰이다. AuthenticationManager는 꼭 사용하지 않고 AuthenticationManagerBuilder를 사용해도 되지만 본인은 AuthenticationManager를 직접 만들어 빈으로 등록해서 그걸 주입받아서 사용했다.





# 5. Spring Security 관련 개념들

이하에서 정리한 개념들은 스프링 시큐리티 공식문서를 참고했다.



## 5.1 Authentication

`Authentication`은 `Principal`과 `Serializable`을 상속한 인터페이스로서 스프링 시큐리티에서 인증객체의 역할을 담당한다. AuthenticationManager가 인증 처리를 해줄 대상 객체가 되는 것이 이 Authentication이다.

`Authentication`은 다음과 같은 정보를 가지고 있다. 

- principal: 사용자를 식별한다. 사용자 이름/비밀번호로 인증할 때는 보통 `UserDetails` 인스턴스가 이에 해당된다.
- credentials: 주로 비밀번호이다. 대부분은 유출되지 않도록 인증이 완료되면 바로 비운다.
- authorities: 사용자에게 부여된 권한인데 `GrantedAuthority`로 추상화된다.



## 5.2 AuthenticationManager

인증을 처리하는 인터페이스이다. `authenticate(Authentication authentication)` 메서드를 가지고 있다. AuthenticationManager를 구현한 많은 구현체들이 있는데 그 중 제일 많이 사용되는 것은 `ProviderManager`이다.



## 5.3 ProviderManager

`ProviderManager`는 다수의 `AuthenticationProvider`를 리스트로 갖고 있다. 그리고 `ProviderManager`는 동작을 가지고 있는 이 `AuthenticationProvider` 리스트에 모두 위임한다. 

`AuthenticationProvider`에 여러 종류가 있는데 대표적으로 이름/비밀번호 기반 인증을 처리하는 `DaoAuthenticationProvider`가 있고 JWT 기반 인증을 처리하는 `JwtAuthenticationProvider`가 있다. 

s

