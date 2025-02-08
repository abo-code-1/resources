# Подключение к базе данных в Spring Boot и работа с Spring Data JPA

## 1️⃣ Как Spring Boot понимает, что нужно подключиться к БД?
Spring Boot использует **Spring Boot AutoConfiguration** — механизм, который сканирует классы на наличие `@Configuration`, аннотаций `@Entity`, `@Repository`, `@Service`, `@Component` и автоматически конфигурирует контекст приложения.

### 🔹 Ключевые механизмы:
1. Spring загружает `application.properties` и ищет настройки `spring.datasource.*`.
2. Если найден URL базы данных (`spring.datasource.url`), Spring Boot подключает `DataSourceAutoConfiguration`.
3. Автоматически создается `DataSource` (источник данных).
4. Подключается `EntityManager` через Hibernate для управления сущностями.

---

## 2️⃣ Как Spring Boot конфигурирует подключение к БД?
В `application.properties` указываются параметры:

```properties
# Настройки источника данных (DataSource)
spring.datasource.url=jdbc:postgresql://localhost:5432/mydb
spring.datasource.username=myuser
spring.datasource.password=mypassword
spring.datasource.driver-class-name=org.postgresql.Driver

# Настройки Hibernate (JPA)
spring.jpa.database-platform=org.hibernate.dialect.PostgreSQLDialect
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
```

### 🔹 Что делает Spring Boot при старте?
1. Ищет `spring.datasource.url`. Если он найден, включается **`DataSourceAutoConfiguration`**.
2. Определяет, какой **драйвер базы данных** использовать (`spring.datasource.driver-class-name`).
3. Создает `DataSource` с указанными параметрами.
4. Если подключен `spring-boot-starter-data-jpa`, Spring загружает **JPA и Hibernate**.
5. Hibernate автоматически управляет созданием и обновлением таблиц (`spring.jpa.hibernate.ddl-auto`).

---

## 3️⃣ Как это работает под капотом?
### **Автоматическая конфигурация в Spring Boot**
Spring Boot загружает классы `DataSourceAutoConfiguration` и `JpaRepositoriesAutoConfiguration`.

### **1️⃣ `DataSourceAutoConfiguration`**
```java
@Configuration
@ConditionalOnClass({ DataSource.class, EmbeddedDatabaseType.class })
@EnableConfigurationProperties(DataSourceProperties.class)
public class DataSourceAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean
    public DataSource dataSource(DataSourceProperties properties) {
        return properties.initializeDataSourceBuilder().build();
    }
}
```
### **2️⃣ `JpaRepositoriesAutoConfiguration`**
```java
@Configuration
@ConditionalOnClass(EntityManagerFactory.class)
@EnableJpaRepositories(basePackages = "com.example.repository")
public class JpaRepositoriesAutoConfiguration {
    // Автоматически подключает JPA-репозитории
}
```

---

## 4️⃣ Как Spring создает `DataSource` и `EntityManager`
### **🔹 `DataSource` создается через `DataSourceBuilder`**
```java
@Bean
@ConditionalOnMissingBean
public DataSource dataSource(DataSourceProperties properties) {
    return properties.initializeDataSourceBuilder().build();
}
```

### **🔹 `EntityManager` создается через `LocalContainerEntityManagerFactoryBean`**
```java
@Bean
public LocalContainerEntityManagerFactoryBean entityManagerFactory(DataSource dataSource) {
    LocalContainerEntityManagerFactoryBean emf = new LocalContainerEntityManagerFactoryBean();
    emf.setDataSource(dataSource);
    emf.setPackagesToScan("com.example.model"); // Где искать @Entity
    emf.setJpaVendorAdapter(new HibernateJpaVendorAdapter());
    return emf;
}
```

---

## 5️⃣ Как Spring определяет, какие классы являются сущностями?
Spring сканирует классы в указанном пакете и находит **классы с аннотацией `@Entity`**.

Пример сущности:
```java
import jakarta.persistence.*;

@Entity
@Table(name = "users")
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;
}
```

---

## 6️⃣ Как Spring понимает, что `@Repository` работает с БД?
Если интерфейс **наследует `JpaRepository`**, Spring **автоматически создает бин**, который реализует этот интерфейс.

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    User findByUsername(String username);
}
```

✔️ `JpaRepository` наследует `CrudRepository`  
✔️ Spring создает `UserRepository` с готовыми методами  
✔️ Запрос `findByUsername("John")` **автоматически преобразуется в SQL**  

---

## 📌 Итог: Как Spring Boot подключается к БД?
1. **Spring Boot загружает `application.properties`**
2. **Находит `spring.datasource.url`** → включает `DataSourceAutoConfiguration`
3. **Создает `DataSource`** и регистрирует его как Spring Bean
4. **Сканирует классы `@Entity`** и подключает Hibernate (`JpaRepositoriesAutoConfiguration`)
5. **Spring автоматически создает `JpaRepository`** и обрабатывает SQL-запросы
6. **Приложение готово к работе с БД!**

🔹 **Ключевая магия Spring Boot** → **Автоматическая конфигурация** + **Spring Beans**  
🔹 **Hibernate берет на себя преобразование объектов в SQL**  
🔹 **Не нужно вручную настраивать `EntityManager` или `DataSource`**  

⚡ **Результат**: минимальные настройки, максимум гибкости! 🚀


