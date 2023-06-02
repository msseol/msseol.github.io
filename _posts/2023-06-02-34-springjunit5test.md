---
layout: post
title: "Spring Rest API Junit5 테스트"
date: 2023-06-02 00:00:02
categories: BACKEND
tags: spring test junit junit5
cover: spring.png
---

<i class="fa-regular fa-circle-check" style="margin-right:0.7rem"></i>_Junit5 Test 관련 샘플링 메모 저장_

---

최근엔 테스트코드를 작성 안하는 부서로 와서 테스트와 거의 멀어져버렸던 것 같다. 과거에는 Junit4 + Jmeter를 주로 썼다면 요새는 Jmeter만으로 통합테스트만 하곤 했었다.  
무뎌지기 전에 부랴부랴 **자바와 JUnit을 활용한 실용주의 단위 테스트** 같은 책도 사서 보기도 하고.. Junit5 기반의 테스트 예시를 작성하며 기억을 잃지 않으려고 메모를 남겼다.

---

### Service 코드

```java
@Service
public class EmployeeServiceImpl implements EmployeeService {
      private final EmployeeRepository employeeRepository;
      EmployeeServiceImpl(EmployeeRepository employeeRepository) {
            this.employeeRepository = employeeRepository;
      }

      @Override
      public List<Employee> findAll() {
            return this.employeeRepository.findAll();
      }
      @Override
      public Optional<Employee> findById(int id) {
            return this.employeeRepository.findById(id);
      }
      @Override
      public Employee save(Employee emp) {
            return this.employeeRepository.save(emp);
      }
      @Override
      public void deleteById(int id) {
            this.employeeRepository.deleteById(id);
      }
}
```

### Controller 코드

```java
 @RestController
      @RequestMapping("/api/employees")
      public class EmployeeApi {
      private final EmployeeService employeeService;
      EmployeeApi(EmployeeService employeeService) {
            this.employeeService = employeeService;
      }

      @GetMapping
      List<Employee> getAll() {
            return this.employeeService.findAll();
      }
      @GetMapping(value = "/{id}")
      Optional<Employee> getById(@PathVariable int id) {
            return this.employeeService.findById(id);
      }
      @PostMapping
      Employee add(@RequestBody Employee emp) {
            return this.employeeService.save(emp);
      }
      @PutMapping
      Employee update(@RequestBody Employee emp) {
            return this.employeeService.save(emp);
      }
      @DeleteMapping(value = "/{id}")
      void delete(@PathVariable int id) {
            this.employeeService.deleteById(id);
      }
}
```

위와 같은 예시 구현에서의 JUnit 5 및 Mockito를 사용하여 이를 단위 테스트해보자.

#### 1. Repository Layer 테스트

```java
@Test
void findAll_should_return_employee_list() {
      // When
      List<Employee> employees = this.employeeRepository.findAll();
      // Then
      assertEquals(4, employees.size());
}

@Test
void findById_should_return_employee() {
      // When
      Optional<Employee> employee = this.employeeRepository.findById(2);
      // Then
      assertTrue(employee.isPresent());
}

@Test
void save_should_insert_new_employee() {
      // Given
      Employee newEmployee = new Employee();
      newEmployee.setFirstName("FIRST_NAME");
      newEmployee.setLastName("LAST_NAME");
      // When
      Employee persistedEmployee = this.employeeRepository.save(newEmployee);
      // Then
      assertNotNull(persistedEmployee);
      assertEquals(5, persistedEmployee.getId());
}

@Test
void save_should_update_existing_employee() {
      // Given
      Employee existingEmployee = new Employee();
      existingEmployee.setId(3);
      existingEmployee.setFirstName("FIRST_NAME");
      existingEmployee.setLastName("LAST_NAME");
      // When
      Employee updatedEmployee = this.employeeRepository.save(existingEmployee);
      // Then
      assertNotNull(updatedEmployee);
      assertEquals("FIRST_NAME", updatedEmployee.getFirstName());
      assertEquals("LAST_NAME", updatedEmployee.getLastName());
}

@Test
void deleteById_should_delete_employee() {
      // When
      this.employeeRepository.deleteById(2);
      Optional<Employee> employee = this.employeeRepository.findById(2);
      // Then
      assertFalse(employee.isPresent());
}
```

#### 2. Service Layer 테스트

서비스 레이어에서는 외부접근(DB Repository)하는 항목에 대해선 Mocking 하여 테스트 환경을 격리 시킨다.

```java
@ExtendWith(MockitoExtension.class) // Mokito 추가
class EmployeeServiceUnitTest {
      @Mock
      private EmployeeRepository employeeRepository; // Repository Mocking

      @InjectMocks
      private EmployeeServiceImpl employeeService;
}
```

> @Mock을 사용하면 EmployeeRepository 모의를 만들고 주입할 수 있다.

> @InjectMocks는 서비스 EmployeeServiceImpl 의 인스턴스를 생성하여 테스트할 수 있도록 하는 데 사용된다.

_테스트 예시_

```java
@Test
void findAll_should_return_employee_list() {
      // Given
      Employee employee = this.buildTestingEmployee();
      // When
      when(employeeRepository.findAll()).thenReturn(List.of(employee));
      List<Employee> employees = this.employeeService.findAll();
      // Then
      assertEquals(1, employees.size());
      verify(this.employeeRepository).findAll();
}

@Test
void findById_should_return_employee() {
      // Given
      Employee employee = this.buildTestingEmployee();
      // When
      when(employeeRepository.findById(1)).thenReturn(Optional.of(employee));
      Optional returnedEmployee = this.employeeService.findById(1);
      // Then
      assertEquals(employee.getId(), returnedEmployee.get()
            .getId());
      verify(this.employeeRepository).findById(1);
}

@Test
void save_should_insert_new_employee() {
      // Given
      Employee employee = this.buildTestingEmployee();
      // When
      this.employeeService.save(employee);
      // Then
      verify(this.employeeRepository).save(employee);
}

@Test
void deleteById_should_delete_employee() {
      // When
      this.employeeService.deleteById(1);
      // Then
      verify(this.employeeRepository).deleteById(1);
}

private Employee buildTestingEmployee() {
      Employee employee = new Employee();
      employee.setId(1);
      employee.setFirstName("FIRST_NAME");
      employee.setLastName("LAST_NAME");
      return employee;
}
```

#### 3. Controller Layer (Rest Endpoint) 테스트

스프링에서는 Controller 테스트 도구로 **@WebMvcTest**를 사용하여 테스트 도구를 제공한다.

```java
 @WebMvcTest(EmployeeApi.class)
class EmployeeRestApiTests {
      @MockBean
      private EmployeeService employeeService; // 서비스 Mocking

      @Autowired
      private MockMvc mockMvc;
}
```

> MockMvc는 Spring Framework에서 제공하는 테스트용 라이브러리로, 웹 애플리케이션의 컨트롤러를 테스트하는 데 사용된다. MockMvc를 사용하면 실제 서버를 구동하지 않고도 컨트롤러의 동작을 모의(mock)하여 테스트할 수 있다.

| Method      | Desc                                                                                                                                                  |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| perform()   | MockMvc를 통해 HTTP 요청을 실행합니다. GET, POST, PUT, DELETE 등의 HTTP 메서드를 사용할 수 있으며, 요청 경로, 파라미터, 헤더 등을 설정할 수 있습니다. |
| andExpect() | HTTP 응답을 검증합니다. 상태 코드, 응답 본문의 내용, 헤더, 모델 속성 등을 확인할 수 있습니다.                                                         |
| andReturn() | 컨트롤러의 실행 결과를 받아옵니다. 컨트롤러의 반환 값, 모델 객체 등을 확인할 수 있습니다.                                                             |

_테스트 예시_

```java
@Test
void should_return_employee_list() throws Exception {
      Employee employee = this.buildTestingEmployee();
      when(employeeService.findAll()).thenReturn(List.of(employee));

      mockMvc.perform(get("/api/employees"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$", hasSize(1)))
            .andExpect(jsonPath("$[0].id", is(1)))
            .andExpect(jsonPath("$[0].firstName", is("FIRST_NAME")))
            .andExpect(jsonPath("$[0].lastName", is("LAST_NAME")));
}

@Test
void should_return_employee() throws Exception {
      Employee employee = this.buildTestingEmployee();
      when(employeeService.findById(2)).thenReturn(Optional.of(employee));

      mockMvc.perform(get("/api/employees/2"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.id", is(1)))
            .andExpect(jsonPath("$.firstName", is("FIRST_NAME")))
            .andExpect(jsonPath("$.lastName", is("LAST_NAME")));
}

@Test
void should_add_new_employee() throws Exception {
      Employee employee = this.buildTestingEmployee();
      when(employeeService.save(any(Employee.class))).thenReturn(employee);

      mockMvc.perform(post("/api/employees")
            .contentType(MediaType.APPLICATION_JSON)
            .content("{ \"firstName\": \"FIRST_NAME\", \"lastName\": \"LAST_NAME\" }"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.firstName", is("FIRST_NAME")))
            .andExpect(jsonPath("$.lastName", is("LAST_NAME")));
}

@Test
void should_update_existing_employee() throws Exception {
      Employee employee = this.buildTestingEmployee();
      when(employeeService.save(any(Employee.class))).thenReturn(employee);

      mockMvc.perform(put("/api/employees")
            .contentType(MediaType.APPLICATION_JSON)
            .content("{ \"id\": 1 , \"firstName\": \"FIRST_NAME\", \"lastName\": \"LAST_NAME\" }"))
            .andExpect(status().isOk());
}

@Test
void should_remove_employee() throws Exception {
      mockMvc.perform(delete("/api/employees/1"))
            .andExpect(status().isOk());

}
```

#### 부록 : TestRestTemplate

**TestRestTemplate**를 통해 Service Endpoint로 직접 호출하는 테스트를 수행할 수 있다. (Jmeter나, Intellij의 http가 더 편하긴 하다 개인적으로)

_기본 설정_

```java
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
class EmployeeRestApiV2Tests {
      @Autowired
      private TestRestTemplate restTemplate;

      @LocalServerPort
      private int randomServerPort; // Runtime에 생성된 포트 주입
}
```

_테스트 예시_

```java
 @Test
void should_return_employee_list() {
      Employee[] employees = restTemplate.getForObject("http://localhost:" + randomServerPort + "/api/employees", Employee[].class);

      assertEquals(4, employees.length);
      assertEquals("Azhrioun", employees[0].getFirstName());
}

@Test
void should_return_employee() {
      Employee employee = restTemplate.getForObject("http://localhost:" + randomServerPort + "/api/employees/4", Employee.class);

      assertEquals(4, employee.getId());
      assertEquals("Stella", employee.getFirstName());
      assertEquals("Sherman", employee.getLastName());
}

@Test
void should_add_new_employee() throws Exception {
      Employee newEmployee = new Employee();
      newEmployee.setFirstName("Adrien");
      newEmployee.setLastName("Miller");
      ResponseEntity response = restTemplate.postForEntity("http://localhost:" + randomServerPort + "/api/employees", newEmployee, Employee.class);

      assertEquals(200, response.getStatusCodeValue());
      assertEquals("Adrien", response.getBody()
            .getFirstName());
      assertEquals("Miller", response.getBody()
            .getLastName());
      }

@Test
void should_update_existing_employee() throws Exception {
      Employee updatedEmployee = new Employee();
      updatedEmployee.setId(1);
      updatedEmployee.setLastName("Abdo");
      HttpEntity requestUpdate = new HttpEntity<>(updatedEmployee);
      ResponseEntity response = restTemplate.exchange("http://localhost:" + randomServerPort + "/api/employees", HttpMethod.PUT, requestUpdate, Employee.class);

      assertEquals("Abdo", response.getBody()
            .getLastName());
}

@Test
void should_remove_employee() throws Exception {
      restTemplate.delete("http://localhost:" + randomServerPort + "/api/employees/2");
      Employee employee = restTemplate.getForObject("http://localhost:" + randomServerPort + "/api/employees/2", Employee.class);

      assertNull(employee);
}
```

#### 부록: WireMock

Server To Server 호출을 테스트 할때, 호출 대상 서버의 트랜젝션을 호출 서버에서 관리하기가 어렵고 개발되지 않은 환경일 수도 있다.  
또한 테스트의 분리원칙 특성 상 테스트 코드는 호출 서버의 코드만 검증되도록 이루어져야 할 것이다. 이럴때 사용할 수 있는게 WireMock인데 가상의 응답을 주는 서버를 띄우는 것이라 보면 된다.
*(크롤러 만들다가 테스트 할 대상이 마땅찮아 있었는데, 천재 개발자 L모님이 알려주셨다. 흐흐)*

**pom.xml**

```xml
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-contract-stub-runner</artifactId>
      <version>3.1.6</version><!--your-->
</dependency>
```

_테스트 코드_

```java
@SpringBootTest
@AutoConfigureMockMvc
@AutoConfigureWireMock(port = 8090) // 가상 서버의 포트
public class WireMockServerTest {
      @Test
      @DisplayName("Mockserver 테스트")
      void crawlingMock() throws Exception {
            ResultActions actions =
                  mockMvc.perform(
                        get("/api/crawler") // 특정 대상 서버를 호출하여 크롤링하는 예시 API
                              .contentType(MediaType.APPLICATION_JSON)
                              .accept(MediaType.APPLICATION_JSON)
                              .characterEncoding("UTF-8")
                              .param("targetURLs", "http://localhost:8090/sample-url")); // WireMock으로 설정한 가상의 서버 및 Endpoint

            actions
                  .andExpect(status().isOk())
                  .andExpect(jsonPath("Status").value(200))
                  .andExpect(jsonPath("Merge").value("Aa0b1c3d9eghilMmnoPprSstuvy"));
      }
}
```

1. src/test/resources 디렉토리에 mappings 디렉토리를 생성한다.
2. mappings 디렉토리에 mock.json 파일을 작성한다.

_src/test/resources/mappings/mock.json_

```json
{
  "request": {
    "method": "GET",
    "url": "/sample-url" // 가상의 endpoint
  },
  "response": {
    // 가상의 response 설정
    "headers": {
      "Content-Type": "application/xml"
    },
    "status": 200,
    "body": "<html><body><img src='Apple.png'/>Public Static void main 900103 Minsu</body></html>"
  }
}
```

> mock.json 경로는 `src/test/` 폴더에 있어야 한다. test 경로가 아닌 resource에 생성했다가 안되었었다 -\_-

#### 참고 용어

<span class="text-danger">**GIVEN**</span>은 테스트를 실행하기 위해 필요한 초기 상태를 설정하는 단계입니다. 즉, 테스트를 수행하기 위해 필요한 **모든 사전 조건을 설정하는 단계**입니다. 이 단계에서는 테스트 대상 객체를 생성하고, 초기 데이터를 준비하고, 의존성을 주입하는 등의 작업을 수행합니다.

<span class="text-info">**WHEN**</span>은 실제로 테스트 대상 **객체의 동작을 실행하는 단계**입니다. 이 단계에서는 테스트할 메서드를 호출하고, 필요한 인자를 전달하며, 특정 동작을 수행합니다. 이 단계에서는 테스트 대상 객체의 동작이 예상대로 동작하는지를 확인하는 것이 중요합니다.

<span class="text-success">**THEN**</span>은 테스트의 **결과를 확인하는 단계**입니다. 이 단계에서는 예상된 결과가 실제로 발생했는지를 검증합니다. 예를 들어, 특정 값이 예상한 값과 일치하는지, 예외가 발생하는지 여부 등을 확인합니다. 이 단계에서는 테스트 대상 객체의 상태나 반환값을 검사하여 예상한 결과가 맞는지를 판별합니다.

[참조][ref]

[ref]: https://devwithus.com/spring-boot-rest-api-unit-testing/
