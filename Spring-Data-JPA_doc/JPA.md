# Spring Data JPA: Полное Руководство

## 📌 Введение
Spring Data JPA — это часть экосистемы Spring, предоставляющая удобный способ работы с базами данных через ORM (Object-Relational Mapping). Основана на JPA (Java Persistence API) и Hibernate.

## 🔹 Установка Spring Data JPA
Добавьте зависимости в `pom.xml` (Maven):

```xml
<dependencies>
    <!-- Spring Boot JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    
    <!-- Драйвер базы данных (пример: PostgreSQL) -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

### 🔍 **Как это работает?**
Spring Boot автоматически подтягивает нужные зависимости и настраивает Hibernate как провайдер JPA. После этого мы можем создавать сущности и управлять ими через `JpaRepository`.

## 🔹 Конфигурация `application.properties`
Настроим подключение к базе данных:

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=postgres
spring.datasource.password=secret
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
```

### 🔍 **Как это работает?**
- `spring.datasource.url` — URL подключения к базе.
- `spring.datasource.username/password` — учетные данные.
- `spring.jpa.hibernate.ddl-auto=update` — Hibernate автоматически обновляет схему БД при запуске.

## 🔹 Создание Entity-класса (Таблица)
```java
import jakarta.persistence.*;

@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    private String email;

    public Student() {}
    public Student(String name, String email) {
        this.name = name;
        this.email = email;
    }

    // Геттеры и сеттеры
}
```

### 🔍 **Как это работает?**
- `@Entity` — указывает, что класс представляет таблицу в БД.
- `@Table(name = "students")` — имя таблицы в БД.
- `@Id` и `@GeneratedValue` — настройка автоинкрементного первичного ключа.

### 🔍 **Как типы данных переводятся в БД?**
| Тип в Java  | Тип в БД (PostgreSQL) |
|------------|----------------------|
| `Long`     | `BIGSERIAL` (если `@GeneratedValue`) или `BIGINT` |
| `String`   | `VARCHAR` |
| `Integer`  | `INTEGER` |
| `Double`   | `DOUBLE PRECISION` |
| `Boolean`  | `BOOLEAN` |
| `LocalDate`| `DATE` |

## 🔹 Создание Репозитория
Spring Data JPA позволяет легко работать с таблицами через `JpaRepository`:

```java
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface StudentRepository extends JpaRepository<Student, Long> {
    Student findByEmail(String email);
}
```

### 🔍 **Как это работает?**
- `JpaRepository<Student, Long>` — базовый интерфейс, предоставляющий CRUD-методы.
- `findByEmail(String email)` — Spring Data JPA сам создаёт SQL-запрос на основе названия метода.

## 🔹 Создание Сервиса
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import java.util.List;

@Service
public class StudentService {
    private final StudentRepository studentRepository;

    @Autowired
    public StudentService(StudentRepository studentRepository) {
        this.studentRepository = studentRepository;
    }

    public List<Student> getAllStudents() {
        return studentRepository.findAll();
    }

    public void addStudent(Student student) {
        studentRepository.save(student);
    }
}
```

## 🔹 Создание Контроллера
```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/students")
public class StudentController {
    private final StudentService studentService;

    @Autowired
    public StudentController(StudentService studentService) {
        this.studentService = studentService;
    }

    @GetMapping
    public List<Student> getStudents() {
        return studentService.getAllStudents();
    }

    @PostMapping
    public void addStudent(@RequestBody Student student) {
        studentService.addStudent(student);
    }
}
```

## 🔹 Запуск приложения
Запустите Spring Boot-приложение:
```bash
mvn spring-boot:run
```

Теперь API работает на `http://localhost:8080/students`.

## 🔹 CRUD-операции
| Метод  | URL | Описание |
|--------|-----------------|-----------------------------|
| `GET`  | `/students`      | Получить всех студентов    |
| `POST` | `/students`      | Добавить нового студента   |

## 🔹 Заключение
Spring Data JPA позволяет легко работать с базами данных, сокращая объем кода. Оно автоматически создаёт SQL-запросы, управляет транзакциями и предоставляет мощный API для работы с данными.

🚀 **Готово! Теперь ваше приложение работает с базой данных!**

