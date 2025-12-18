5조 Re_View 관련 문서입니다

소스코드 Git |  https://github.com/VenyVince/Re_View.git

---

EC2 서버 | http://3.115.212.100/


---

# API 테스트 예제 (React ↔ Spring Boot)

## 0. 첨언
아래 내용은 **Re:View** 팀원들을 위한 내용입니다. 
기본적으로 현재 저희 팀은 FE서버와 BE서버를 따로 두고있습니다.
각각 localhost 3000 / 8080입니다.

BE서버의 경우 바로 인텔리제이에서 프로젝트 시작(ReviewApplication 빌드)하면 켜지고
FE 서버의 경우 npm run View(frontend)라고 터미널에 입력하면 아마 동작할거에요. 
이외 명령어로 동작하는 경우에는 View또는 frontend의 src폴더와 동일한 위치에 있는 package.json을 수정하면 됩니다.

+ 아래 내용은 팀장이 정리한 내용을 바탕으로 GPT에게 정리해달라고 해서 자세하지 못한 부분이 있을 수도 있으니 5. 기타파일- 프로젝트 관련 - API 통신 정리.txt파일 확인해보시면 될 것 같습니다.

## 1. 개요

이 프로젝트는 **React**에서 **Spring Boot** API를 호출하여 데이터를 받아 화면에 표시하는 간단한 예제입니다.
예제에서는 **GET 요청**을 기준으로 설명하며, POST, PUT, PATCH, DELETE도 유사한 흐름입니다.

---

## 2. 프로젝트 구조

```
src/
 └─ main/
     └─ java/com/review/shop/
         ├─ controller/test/TestProductController.java
         ├─ service/TestProductService.java
         ├─ dto/TestProductResponseDTO.java
         └─ config/TestCorsConfig.java
frontend/
 └─ src/
     └─ TestProduct.js
```

---

## 3. 데이터 흐름 (GET 요청 기준)

1. **React → Spring Controller**

   * React에서 `fetch`로 `/api/test/products` 요청
   * CORS 설정으로 localhost:3000에서 요청 허용

2. **Controller → Service**

   * Controller에서 Service 호출 (`getTestProducts()`)

3. **Service → DTO**

   * 하드코딩된 리스트 생성
   * DTO(`TestProductResponseDTO`)에 담아 반환

4. **Spring Boot → JSON**

   * DTO가 JSON으로 변환되어 응답
   * React에서 JSON 받아 화면 표시

---

## 4. 코드 예제

### 4.1 CORS 설정 (`TestCorsConfig.java`)

```java
@Configuration
public class TestCorsConfig {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurer() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/test/**")
                        .allowedOrigins("http://localhost:3000") // React 개발서버 허용
                        .allowedMethods("*"); // 모든 HTTP 메소드 허용
            }
        };
    }
}
```

---

### 4.2 Controller (`TestProductController.java`)

```java
@RestController
@RequestMapping("/api/test/products")
public class TestProductController {

    @Autowired
    private TestProductService testProductService;

    @GetMapping
    public TestProductResponseDTO getProducts() {
        return testProductService.getTestProducts(); // Service 호출
    }
}
```

---

### 4.3 Service (`TestProductService.java`)

```java
@Service
public class TestProductService {

    public TestProductResponseDTO getTestProducts() {
        TestProductDTO p1 = new TestProductDTO(1L, "샴푸", "BrandA", 12000);
        TestProductDTO p2 = new TestProductDTO(2L, "린스", "BrandB", 15000);
        TestProductDTO p3 = new TestProductDTO(3L, "바디워시", "BrandC", 9000);

        return new TestProductResponseDTO(Arrays.asList(p1, p2, p3));
    }
}
```

---

### 4.4 DTO (`TestProductResponseDTO.java`)

```java
@Data
public class TestProductResponseDTO {
    private List<TestProductDTO> products;

    public TestProductResponseDTO(List<TestProductDTO> products) {
        this.products = products;
    }

    @Data
    @AllArgsConstructor
    public static class TestProductDTO {
        private Long id;      // 1L, 2L → JSON에서는 숫자만 표시됨
        private String name;
        private String brand;
        private double price;
    }
}
```

---

### 4.5 React 예제 (`TestProduct.js`)

```javascript
useEffect(() => {
  fetch("http://localhost:8080/api/test/products") // GET 요청
    .then(res => res.json())
    .then(data => console.log(data));
}, []);

{data.products?.map(p => (
  <div key={p.id}>
    {p.name} - {p.brand} ({p.price}원)
  </div>
))}
```

---

## 5. JSON 응답 예시

```json
{
  "products": [
    { "id": 1, "name": "샴푸", "brand": "BrandA", "price": 12000 },
    { "id": 2, "name": "린스", "brand": "BrandB", "price": 15000 },
    { "id": 3, "name": "바디워시", "brand": "BrandC", "price": 9000 }
  ]
}
```

* Java에서는 `1L, 2L`로 Long 타입 사용 가능
* JSON 변환 시 숫자(`1, 2`)만 표시 → 화면에 `L`은 안 나옴

---

## 6. 요약

| 단계             | 설명                         |
| -------------- | -------------------------- |
| 1️⃣ FE         | React에서 GET 요청 (`fetch`)   |
| 2️⃣ Controller | 요청을 받아 Service 호출          |
| 3️⃣ Service    | 하드코딩된 데이터 생성, DTO로 반환      |
| 4️⃣ JSON       | Spring Boot가 DTO → JSON 변환 |
| 5️⃣ FE         | React가 JSON 받아 화면에 렌더링     |

💡 **Tip**: POST, PUT, PATCH, DELETE도 Controller에서 메소드만 바꿔주면 동일한 흐름으로 동작합니다.

---
