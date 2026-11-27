# Student Management REST API (Spring Boot)

A full-featured RESTful web service built incrementally across **Lab Experiments 3 to 9** using **Spring Boot**, **Spring Data JPA**, **MySQL**, **Bean Validation**, **JUnit / MockMvc**, and **Spring Security**.

---

## Tech Stack & Dependencies
- **Java & Spring Boot** (`spring-boot-starter-webmvc`, `spring-boot-devtools`)
- **Database & ORM:** MySQL 8.x, Spring Data JPA (`spring-boot-starter-data-jpa`), MySQL Connector/J
- **Validation:** Jakarta Bean Validation (`spring-boot-starter-validation`)
- **Security:** Spring Security (`spring-boot-starter-security`) with HTTP Basic Authentication
- **Testing:** JUnit 5, Spring Boot Test (`MockMvc`), and `cURL`

---

## Project Structure

```text
student-management/
├── src/
│   ├── main/
│   │   ├── java/com/example/ucs067/studentmanagement/
│   │   │   ├── config/
│   │   │   │   └── SecurityConfig.java          # HTTP Basic auth & endpoint rules (Lab 9)
│   │   │   ├── controller/
│   │   │   │   └── HelloController.java         # REST endpoints for Students & Courses
│   │   │   ├── exception/
│   │   │   │   └── GlobalExceptionHandler.java  # Centralized @Valid error handling (Lab 7)
│   │   │   ├── repository/
│   │   │   │   ├── StudentRepository.java       # JpaRepository + derived queries (Labs 4–5)
│   │   │   │   └── CourseRepository.java        # JpaRepository for Course entity (Lab 6)
│   │   │   ├── Course.java                      # @Entity owning @ManyToOne FK (Lab 6)
│   │   │   ├── Student.java                     # @Entity with @OneToMany & validation (Labs 3–8)
│   │   │   └── StudentManagementApplication.java
│   │   └── resources/
│   │       └── application.properties           # Port (8081), MySQL & Security config
│   └── test/
│       └── java/com/example/ucs067/studentmanagement/
│           └── StudentManagementApplicationTests.java # MockMvc integration tests (Lab 8)
├── pom.xml
└── README.md
```

---

## Lab-by-Lab Implementation Log

### Lab Experiment 3: In-Memory CRUD REST API
- Updated `Student.java` with a default constructor, parameterized constructor, and getters/setters to support Jackson JSON serialization/deserialization.
- Built initial CRUD endpoints (`GET`, `POST`, `PUT`, `DELETE`) in `HelloController` backed by an in-memory `ArrayList<Student>` with `ResponseEntity` HTTP status handling (`200 OK`, `404 Not Found`).

### Lab Experiment 4: MySQL Persistence & Spring Data JPA
- Configured MySQL database (`studentdb`) on port `3306` and added `spring-boot-starter-data-jpa` and `mysql-connector-j` to `pom.xml`.
- Mapped `Student` to the `students` table using `@Entity`, `@Table`, and `@Id`.
- Created `StudentRepository extends JpaRepository<Student, Integer>` and refactored `HelloController` to persist all CRUD operations directly in MySQL.

### Lab Experiment 5: Derived Query Methods, Pagination & Sorting
- Extended `StudentRepository` with Spring Data derived queries (`findByName`, `findByNameContaining`, `findByEmail`) without writing manual SQL.
- Added search endpoints (`/students/search/name/{name}`, `/students/search/name/contains/{keyword}`, `/students/search/email/{email}`).
- Implemented paginated retrieval using `Pageable` (`/students/page?page=0&size=2`) and alphabetical sorting using `Sort.by("name")` (`/students/sort/name`).

### Lab Experiment 6: One-to-Many Relationship (`Student` ↔ `Course`)
- Created the `Course` entity (`id`, `courseName`, `credits`) and `CourseRepository`.
- Configured bidirectional `@OneToMany(mappedBy = "student", cascade = CascadeType.ALL, orphanRemoval = true)` on `Student` and `@ManyToOne` on `Course` (maintaining the `student_id` foreign key).
- Resolved infinite JSON serialization loops using `@JsonManagedReference` and `@JsonBackReference`.
- Added endpoints to assign courses to students (`POST /students/{studentId}/courses`) and list all courses (`GET /courses`).

### Lab Experiment 7: Input Validation & Centralized Exception Handling
- Added `spring-boot-starter-validation` and enforced constraints on `Student` fields (`@NotBlank`, `@Email`).
- Enabled request body validation via `@Valid` on `POST /students` and `PUT /students/{id}`.
- Created `GlobalExceptionHandler` (`@RestControllerAdvice` + `@ExceptionHandler(MethodArgumentNotValidException.class)`) to return clean `400 Bad Request` field-level JSON error maps instead of stack traces.

### Lab Experiment 8: Auto-Increment Schema Migration, cURL & MockMvc Automated Testing
- Migrated `students.id` to `AUTO_INCREMENT` (`@GeneratedValue(strategy = GenerationType.IDENTITY)`) while preserving the `course.student_id` foreign key constraint.
- Verified full CRUD lifecycle and status codes (`200`, `400`, `404`) via `cURL`.
- Added automated integration tests in `StudentManagementApplicationTests` using `@SpringBootTest` and `@AutoConfigureMockMvc` to verify endpoint responses and `400 Bad Request` validation payloads.

### Lab Experiment 9: API Authentication with Spring Security
- Added `spring-boot-starter-security` and configured local credentials (`admin` / `admin123`) in `application.properties`.
- Created `SecurityConfig` defining a `SecurityFilterChain` bean with HTTP Basic authentication (`httpBasic(withDefaults())`), disabling CSRF for stateless API testing, permitting unauthenticated `GET /hello`, and securing all `/students` and `/courses` endpoints (`401 Unauthorized` without valid credentials).

---

## API Endpoints Reference

| Method | Endpoint | Auth Required | Description |
| :--- | :--- | :---: | :--- |
| `GET` | `/hello` | No | Health/greeting check (`Hello, Spring Boot!`) |
| `GET` | `/students` | Yes | Retrieve all students and their enrolled courses |
| `GET` | `/students/{id}` | Yes | Retrieve a specific student by ID |
| `GET` | `/students/page?page=0&size=2` | Yes | Retrieve paginated list of students |
| `GET` | `/students/sort/name` | Yes | Retrieve students sorted alphabetically by name |
| `GET` | `/students/search/name/{name}` | Yes | Exact-match search by student name |
| `GET` | `/students/search/name/contains/{keyword}` | Yes | Substring search by student name |
| `GET` | `/students/search/email/{email}` | Yes | Search student by email |
| `POST` | `/students` | Yes | Create a new validated student |
| `PUT` | `/students/{id}` | Yes | Update an existing student's name and email |
| `DELETE` | `/students/{id}` | Yes | Delete a student (cascades to enrolled courses) |
| `GET` | `/courses` | Yes | Retrieve all courses |
| `POST` | `/students/{studentId}/courses` | Yes | Create a course and assign it to a student |

---

## Setup & Running Locally

1. **Create the MySQL Database:**
   ```sql
   CREATE DATABASE studentdb;
   ```
2. **Configure `src/main/resources/application.properties`:**
   ```properties
   spring.application.name=student-management
   server.port=8081

   spring.datasource.url=jdbc:mysql://localhost:3306/studentdb
   spring.datasource.username=root
   spring.datasource.password=YOUR_MYSQL_PASSWORD
   spring.jpa.hibernate.ddl-auto=update
   spring.jpa.show-sql=true

   spring.security.user.name=admin
   spring.security.user.password=admin123
   ```
3. **Run the Application:**
   ```bash
   ./mvnw spring-boot:run
   ```
4. **Test with `cURL`:**
   ```bash
   # Public endpoint
   curl -i http://localhost:8081/hello

   # Authenticated GET
   curl -i -u admin:admin123 http://localhost:8081/students

   # Authenticated POST
   curl -i -u admin:admin123 -X POST http://localhost:8081/students \
     -H "Content-Type: application/json" \
     -d "{\"name\":\"Karan Verma\",\"email\":\"karan@example.com\"}"
   ```