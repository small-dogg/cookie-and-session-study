# 쿠키, 세션

Web은 Domain을 참조하지만, Domain은 Web을 참조해서는 안된다.

## Cookie를 사용하여 로그인 및 로그아웃 처리하기
 - 보안상 문제점이 있음
   - 쿠키값은 임의로 변경될 수 있음
   - 쿠키에 보관된 정보가 탈취될 수 있음
   - 쿠키를 사용하여 악의적인 요청을 계속 시도할 수 있음
 - 대안
   - 쿠키에 중요한 값을 노출하지않고, 사용자 별로 예측 불가능한 임의의 토큰을 노출하고, 토큰과 사용자를 맵핑해서 사용.
   - 토큰 값이 탈취되어도 사용할 수 없도록 만료시간을 짧게, 그리고 해킹이 의심될 경우 강제로 토큰을 제거함.

## Session을 사용하여 로그인 및 로그아웃 처리하기
 - 쿠키 값을 변조 가능 -> 예상 불가능한 복잡한 세션Id를 사용한다.
 - 쿠키에 보관하는 정보는 클라이언트 해킹시 털릴 가능성이 있다 -> 세션Id가 털려도 여기에는 중요한 정보가 없다.
 - 쿠키 탈취 후 사용 -> 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 세션의 만료시간을 짧게(예: 30분) 유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 세션을 강제로 제거하면 된다.

## HttpSession을 사용하여 로그인 및 로그아웃 처리하기
 - 위에서 구현한 Session을 통한 로그인 기능을 HttpSession으로 변경하여 처리

# 필터, 인터셉터

## 서블릿 필터

### 공통 관심 사항
- 로그인한 사용자만 상품 관리 페이지에 들어가야함.
- 현재는 로그인하지 않은 사용자가 URL로 접근할 수 있음.
---
- 때문에 모든 접근 로직에 로그인된 사용자를 식별하고 관리해야함.
- 웹과 관련된 공통관심사는 서블릿 필터 또는 스프링 인터셉터를 사용하는 것이 좋다.
- HTTP의 헤더나 URL 정보를 사용하여 처리한다.


### 서블릿 필터 소개
필터는 서블릿이 지원하는 수문장이다.

**필터 흐름**

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
```
필터를 적용하면 필터가 호출된 다음에 서블릿이 호출된다. 사용자 요청 로그를 남기는 요구사항이 있다면 필터를 사용하면된다.
참고로 필터는 특정 URL 패턴에 적용할 수 있다. '/*'을 입력하면 모든 요청에 필터가 적용된다.

**필터 제한**

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러
HTTP 요청 -> WAS -> 필터(적절하지 않은 요청이라 판단, 서블릿 호출 X)
```
옳바르지 않은 요청인 경우 필터를 통해 서블릿에 접근하지 않고 끝을 낼 수 있다.
로그인 처리시 유용하다.

**필터 체인**

```
HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러
```
필터는 체인으로 구성되는데 중간에 필터를 자유롭게 추가할 수 있다. 로그를 남기는 필터를 먼저 적용하고, 다음에 로그인 여부를 체크하는 필터를 만들 수 있다.

**필터 인터페이스**
```java
public interface Filter {
   public default void init(FilterConfig filterConfig) throws ServletException {
   }

   public void doFilter(ServletRequest request, ServletResponse response,
                        FilterChain chain) throws IOException, ServletException;

   public default void destroy() {
   }
}
```
필터 인터페이스를 구현하고 등록하면 서블릿 컨테이너가 필터를 싱글톤 객체로 생성하고, 관리한다.
- `init()`: 필터 초기화 메서드, 서블릿 컨테이너가 생성될 때 호출된다.
- `doFilter()`: 고객의 요청이 올 때 마다 해당 메서드가 호출된다. 필터의 로직을 구현하면 된다.
- `destory()`: 필터 종료 메서드, 서블릿 컨테이너가 종료될 때 호출된다.

## 스프링 인터셉터

스프링 인터셉터도 서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술이다.
서블릿 필터가 서블릿이 제공하는 기수이라면, 스프링 인터셉터는 스프링 MVC가 제공하는 기술이다.
둘다 웹과 관련된 공통 관심 사항을 처리하지만, 적용되는 순서와 범위, 그리고 사용방법이 다르다.

**스프링 인터셉터 흐름**

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러
```
- 스프링 인터셉터는 서블릿과 컨트롤러 사이에서 컨트롤러 호출 직전에 호출된다.
- 스프링 인터셉터는 스프링 MVC가 제공하는 기능이기 때문에 디스패처 서블릿 이후에 등장한다. 스프링 MVC의 시작점이 디스패처 서블릿이라고 생각하면 이해가 될 것이다.
- 스프링 인터셉터에도 URL 패턴을 적용할 수 있는데, 서블릿 URL 패턴과는 다르고, 매우 정밀하게 설정할 수있따.

**스프링 인터셉터 제한**

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러 // 로그인 사용자
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 (적절하지않은 요청이라 판단, 컨트롤러 호출x) // 비로그인 사용자
```
인터셉터에서 적절하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있다. 그래서 로그인 여부를 체크하기에 딱 좋다.

**스프링 인터셉터 체인**

```
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터1 -> 인터셉터2 -> 컨트롤러
```

스프링 인터셉터는 서블릿 필터와 비슷해 보이지만, 보다 편리하고 정교하게 다양한 기능을 제원한다.

**스프링 인터셉터 인터페이스**

스프링 인터셉터를 사용하려면 `HandlerInterceptor` 인터페이스를 구현하면 된다.

```java
public interface HandlerInterceptor {
   default boolean preHandle(HttpServletRequest request, HttpServletResponse
           response, Object handler) throws Exception {
   }

   default void postHandle(HttpServletRequest request, HttpServletResponse
           response, Object handler, @Nullable ModelAndView modelAndView)
           throws Exception {
   }

   default void afterCompletion(HttpServletRequest request, HttpServletResponse
           response, Object handler, @Nullable Exception ex) throws
           Exception {
   }
}

```
- 서블릿 필터의 경우 단순하게 `doFilter()` 하나만 제공되지만,
인터셉터는 컨트롤러 호출전(`preHandler()`), 호출 후(`postHandler()`), 요청 완료 이후(`afterCompletion()`)와 같이 단계적으로 잘 세분화되어 있다.
- 서블릿 필터의 경우 단순히 `request`, `response`만 제공했지만, 인터셉터는 어떤 컨트롤러(`handler`)가 호출되는지 호출정보도 받을 수 있다. 그리고 어떤 `modelAndView`가 반환되는지 응답 정보도 받을 수 있다.

**WebConfig - 인터셉터 등록**

WebMvcConfigurer의 addInterceptors를 오버라이딩하여 등록한다.
PathPatterns와 excludePathPatterns로 간결하게, Path를 제어할 수 있다.
```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
   @Override
   public void addInterceptors(InterceptorRegistry registry) {
      registry.addInterceptor(new LogInterceptor())
              .order(1)
              .addPathPatterns("/**")
              .excludePathPatterns("/css/**", "/*.ico", "/error");
   }
   //...
}
```

## ArgumentResolver 활용

HnadlerMethodArgumentResolver를 구현하여 애너테이션만으로 사용자의 로그인 검증 처리를 구현할 수 있다.
