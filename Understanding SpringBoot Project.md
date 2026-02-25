# Learning Spring Boot Through Real Projects — First Principles Approach

This is honestly the **best way to learn** — tutorials give you toy problems, real projects give you real judgment. Here's a structured way to do it.

---

## Phase 1: Understand the Project Before Touching It

### 1. Start with the "Skeleton Read"
Before reading any code, understand the **shape** of the project:

```
- What does this application DO? (Read README, check endpoints)
- What is it connected to? (DB, message queues, external APIs)
- How is it deployed? (Docker, cloud, bare metal)
```

### 2. Read `pom.xml` / `build.gradle` First
Every dependency tells a story. Ask yourself:

- `spring-boot-starter-web` → it's an HTTP API
- `spring-boot-starter-data-jpa` → it talks to a relational DB
- `spring-security` → authentication/authorization exists
- `spring-kafka` / `rabbitmq` → async messaging
- `lombok` → boilerplate is reduced (watch for `@Data`, `@Builder`)

This tells you the **technology landscape** before reading a single class.

### 3. Map the Entry Points
Spring Boot apps have clear entry points. Find them:

```java
// 1. The main class
@SpringBootApplication  ← start here

// 2. All Controllers (HTTP entry points)
@RestController, @Controller

// 3. Scheduled jobs
@Scheduled

// 4. Event listeners
@EventListener, @KafkaListener, @RabbitListener
```

These are the **doors** through which all behavior enters the system.

### 4. Trace a Single Request End-to-End
Pick ONE simple API endpoint and trace it completely:

```
HTTP Request
    → Controller method
        → Service method
            → Repository / DB call
                → Response back
```

Draw this on paper. Do this for 3-4 endpoints until the pattern clicks.

---

## Phase 2: First Principles Thinking on Spring Boot

### Understand the Core Mechanism: IoC + DI
Everything in Spring Boot flows from one idea:

> **"You don't create objects. You declare what you need, and Spring wires it."**

```java
// You don't do: UserService service = new UserService(new UserRepo());
// Spring does it for you because of:
@Service        ← "I am a bean, manage me"
@Autowired      ← "I need this bean injected"
```

Ask for every class: **"Who creates this? Who uses it? Why is it a bean?"**

### Understand the Layered Architecture (Why it exists)
```
Controller  → Handles HTTP, input validation, response format
Service     → Business logic, transactions, orchestration
Repository  → Data access ONLY
Entity      → Database table representation
DTO         → What goes in/out of the API (not the Entity itself)
```

**First principle:** Each layer has ONE responsibility. Violations of this = bugs and mess.

When reading code, ask: *"Is this class doing what its layer is supposed to do?"* You'll spot problems immediately.

### Understand `application.properties` / `application.yml`
This file is the **brain of configuration**. Read it entirely:
- What DB is it connecting to?
- Are there multiple profiles? (`dev`, `prod`, `test`)
- What ports, timeouts, feature flags exist?

---

## Phase 3: Adding New Features Without Breaking Existing Code

### The Golden Rule: **Open/Closed Principle**
> Open for extension, closed for modification.

Concretely this means:

```java
// DON'T modify existing service methods
// DO add new methods or new service classes

// DON'T change existing DTOs used by working endpoints
// DO create new DTOs for new endpoints
```

### Step-by-Step Feature Addition Process

**Step 1 — Write the contract first (API design)**
```
What endpoint will this be? GET /users/{id}/orders
What does it accept? Path variable: userId
What does it return? List of OrderDTO
What can go wrong? User not found, no orders
```

**Step 2 — Check what already exists**
Before writing anything, search:
- Is there already a `UserService`? → Add a method there, don't make `UserService2`
- Is there already an `Order` entity and `OrderRepository`? → Reuse them
- Is there a common exception handler (`@ControllerAdvice`)? → Use it

**Step 3 — Follow the existing patterns exactly**
Look at how an existing similar feature is built and **mirror the style**:
```java
// If existing code does this for error handling:
throw new ResourceNotFoundException("User not found");

// Your new code should do the same, not:
return ResponseEntity.status(404).body("not found");
```
Consistency IS correctness in team codebases.

**Step 4 — Write in layers, bottom-up**
```
1. Entity changes (if needed) or confirm existing entity works
2. Repository — add query method
3. Service — add business logic method
4. DTO — create request/response classes
5. Controller — wire it all together
```

**Step 5 — Test in isolation**
- Write a test for your service method alone (mock the repo)
- Then test the controller alone (mock the service)
- This proves your code doesn't affect existing behavior

---

## Phase 4: Real Projects to Use

| Project | What you learn |
|---|---|
| [Mall](https://github.com/macrozheng/mall) | E-commerce, full real-world complexity |
| [Spring PetClinic](https://github.com/spring-projects/spring-petclinic) | Official Spring sample, clean architecture |
| [Halo](https://github.com/halo-dev/halo) | Blogging platform, real production code |
| [eladmin](https://github.com/elunez/eladmin) | Admin system, Spring Security + JWT |
| [RuoYi](https://github.com/yangzongzhuan/RuoYi) | Enterprise patterns |

**Start with PetClinic** — it's clean, well-structured, and meant to be understood.

---

## The Mental Loop to Use Every Day

```
1. OBSERVE  → What does this code do? (read without judgment)
2. QUESTION → Why is it done this way? What problem does it solve?
3. PREDICT  → If I change X, what breaks?
4. VERIFY   → Run it, test it, confirm your mental model
5. EXTEND   → Add something small. Did the system behave as expected?
```

This loop, applied consistently, builds **real intuition** — not just syntax knowledge.

---

The biggest shift in mindset: **you're not learning Spring Boot, you're learning how to think about systems that happen to be built with Spring Boot.** The framework is just the vocabulary; the thinking is the skill.

-----

# Deep Dive: Understanding & Extending Spring Boot Projects From Scratch

Let's use **Spring PetClinic** as our real project. It's clean, production-patterned, and free.

---

## STEP 0: Setup Before Anything

```bash
# Clone the project
git clone https://github.com/spring-projects/spring-petclinic.git
cd spring-petclinic

# Run it FIRST before reading any code
./mvnw spring-boot:run

# Open browser: http://localhost:8080
# Click around. Use it as a USER first.
```

**Why run first?** You need to know what the app DOES before understanding HOW it does it. Most people skip this and get lost in code immediately.

Write down what you see:
```
- There are Owners
- Owners have Pets
- Pets have Visits (vet visits)
- There are Vets with Specialities
```

Now you have a **mental model** of the domain. Code will make sense now.

---

## STEP 1: Read `pom.xml` — Understand the Tech Stack

Open `pom.xml`. Read every dependency and ask WHY it's there:

```xml
<!-- This tells you: It's a web app with REST/MVC endpoints -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- This tells you: It talks to a relational database using JPA/Hibernate -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- This tells you: H2 in-memory DB for development/testing -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- This tells you: Thymeleaf = server-side HTML templating (not a REST API) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

<!-- This tells you: Input validation (@NotBlank, @Size etc) is used -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

**After reading pom.xml, you now know:**
```
Stack:        Spring MVC (not REST API - it serves HTML pages)
Database:     H2 (in-memory, resets on restart) via JPA/Hibernate
Templating:   Thymeleaf (Java + HTML mixed)
Validation:   Bean Validation API
```

This shapes everything. You won't go looking for `@RestController` and JSON — it uses `@Controller` and HTML views.

---

## STEP 2: Read `application.properties` — Understand Configuration

```properties
# src/main/resources/application.properties

# Tells you: JPA will show you every SQL query it runs (great for learning)
spring.jpa.show-sql=true

# Tells you: Database schema is created from schema.sql file
spring.sql.init.mode=always

# Tells you: Which datasource (H2 here)
spring.datasource.url=jdbc:h2:mem:petclinic

# Tells you: App runs on port 8080
server.port=8080
```

Also look for:
```
src/main/resources/
    ├── application.properties     ← main config
    ├── db/
    │   ├── h2/schema.sql          ← DB table creation scripts
    │   └── h2/data.sql            ← Sample data inserted on startup
```

**Open `schema.sql`** — this is the most important file to understand the data model:

```sql
CREATE TABLE owners (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  first_name VARCHAR(30),
  last_name  VARCHAR_IGNORECASE(30),
  address    VARCHAR(255),
  city       VARCHAR(80),
  telephone  VARCHAR(20)
);

CREATE TABLE pets (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  name       VARCHAR(30),
  birth_date DATE,
  type_id    INTEGER NOT NULL,
  owner_id   INTEGER NOT NULL   -- FK to owners
);

CREATE TABLE visits (
  id          INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  pet_id      INTEGER NOT NULL,  -- FK to pets
  visit_date  DATE,
  description VARCHAR(255)
);

CREATE TABLE vets (
  id         INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  first_name VARCHAR(30),
  last_name  VARCHAR_IGNORECASE(30)
);
```

**Draw this on paper:**
```
owners (1) ──── (many) pets (1) ──── (many) visits
                   |
                pet_types

vets (many) ──── (many) specialties
```

Now you understand the entire data model WITHOUT reading a single Java class.

---

## STEP 3: Understand the Folder Structure

```
src/main/java/org/springframework/samples/petclinic/
│
├── PetClinicApplication.java          ← ENTRY POINT (main method)
│
├── owner/                             ← Everything about owners
│   ├── Owner.java                     ← Entity (maps to DB table)
│   ├── OwnerRepository.java           ← DB queries
│   ├── OwnerController.java           ← HTTP routes
│   ├── Pet.java                       ← Entity
│   ├── PetRepository.java
│   ├── PetController.java
│   ├── Visit.java                     ← Entity
│   └── VisitController.java
│
├── vet/                               ← Everything about vets
│   ├── Vet.java
│   ├── VetRepository.java
│   └── VetController.java
│
└── system/                            ← App-level config, error pages
    └── CrashController.java
```

**Pattern you notice:** Each feature/domain has its own folder with Entity + Repository + Controller grouped together. This is called **Package by Feature** (not by layer). It means all Owner-related code lives together.

---

## STEP 4: Read an Entity — Understanding the Data Layer

Open `Owner.java`:

```java
@Entity                              // ← Tells JPA: this class = a DB table
@Table(name = "owners")              // ← Maps to 'owners' table specifically
public class Owner extends Person {  // ← Inherits firstName, lastName from Person

    @Column(name = "address")
    @NotBlank                        // ← Validation: can't be empty
    private String address;

    @Column(name = "city")
    @NotBlank
    private String city;

    @Column(name = "telephone")
    @NotBlank
    @Digits(fraction = 0, integer = 10)  // ← Must be a number, max 10 digits
    private String telephone;

    // ← ONE owner has MANY pets
    // mappedBy = "owner" means: Pet.java has a field called 'owner'
    // cascade = ALL means: if you delete owner, pets get deleted too
    // fetch = EAGER means: when you load Owner, pets load immediately
    @OneToMany(mappedBy = "owner", cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    private List<Pet> pets = new ArrayList<>();

    // getter, setter, helper methods...
}
```

Now open `Pet.java` to see the other side of the relationship:

```java
@Entity
@Table(name = "pets")
public class Pet extends NamedEntity {  // ← Inherits id, name

    @Column(name = "birth_date")
    private LocalDate birthDate;

    // MANY pets belong to ONE type
    @ManyToOne
    @JoinColumn(name = "type_id")   // ← FK column in pets table
    private PetType type;

    // MANY pets belong to ONE owner
    @ManyToOne
    @JoinColumn(name = "owner_id")  // ← FK column in pets table
    private Owner owner;            // ← This is what mappedBy="owner" refers to

    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER)
    @JoinColumn(name = "pet_id")
    private List<Visit> visits = new ArrayList<>();
}
```

**How these connect to `schema.sql`:**
```
Java: @JoinColumn(name = "owner_id")
SQL:  owner_id INTEGER NOT NULL   -- in pets table

Java: @OneToMany(mappedBy = "owner")
SQL:  The FK is on the pets side, owners table has no FK column
```

---

## STEP 5: Read the Repository — Understanding Data Access

Open `OwnerRepository.java`:

```java
public interface OwnerRepository extends JpaRepository<Owner, Integer> {
    // JpaRepository gives you for FREE:
    // save(owner), findById(id), findAll(), delete(owner), count()...

    // Custom query using method name conventions:
    // Spring reads this method name and builds SQL automatically:
    // SELECT * FROM owners WHERE last_name LIKE '%name%'
    @Query("SELECT DISTINCT owner FROM Owner owner 
            left join fetch owner.pets 
            WHERE owner.lastName LIKE :lastName%")
    List<Owner> findByLastName(@Param("lastName") String lastName);

}
```

**The magic here:** You don't write SQL. You either:
1. Use built-in methods (`findById`, `save`, `findAll`)
2. Write method names Spring translates to SQL (`findByLastName`)
3. Write JPQL (Java-flavored SQL) with `@Query`

**First principle:** Repository = the ONLY place that talks to the DB. No other class should have SQL or DB logic.

---

## STEP 6: Read the Controller — Understanding HTTP Layer

Open `OwnerController.java`:

```java
@Controller                              // ← Handles HTTP, returns HTML views
public class OwnerController {

    private final OwnerRepository owners; // ← Injected by Spring (no 'new')

    // Constructor injection - Spring sees this constructor and
    // automatically provides the OwnerRepository bean
    public OwnerController(OwnerRepository clinicService) {
        this.owners = clinicService;
    }

    // ─────────────────────────────────────────────
    // ROUTE: GET /owners/new  → Show empty form
    // ─────────────────────────────────────────────
    @GetMapping("/owners/new")
    public String initCreationForm(Map<String, Object> model) {
        Owner owner = new Owner();
        model.put("owner", owner);          // ← Send empty Owner to the HTML form
        return "owners/createOrUpdateOwnerForm"; // ← Thymeleaf template name
    }

    // ─────────────────────────────────────────────
    // ROUTE: POST /owners/new  → Process form submission
    // ─────────────────────────────────────────────
    @PostMapping("/owners/new")
    public String processCreationForm(@Valid Owner owner, BindingResult result) {
        // @Valid triggers the @NotBlank, @Digits validations on Owner fields
        // BindingResult holds any validation errors

        if (result.hasErrors()) {
            return "owners/createOrUpdateOwnerForm"; // ← Go back to form with errors
        }

        this.owners.save(owner);             // ← Save to DB via repository
        return "redirect:/owners/" + owner.getId(); // ← Redirect to owner's page
    }

    // ─────────────────────────────────────────────
    // ROUTE: GET /owners?lastName=Smith  → Search
    // ─────────────────────────────────────────────
    @GetMapping("/owners")
    public String processFindForm(@RequestParam(defaultValue = "") String lastName,
                                   Owner owner, BindingResult result,
                                   Map<String, Object> model) {

        List<Owner> results = this.owners.findByLastName(lastName);

        if (results.isEmpty()) {
            result.rejectValue("lastName", "notFound", "not found");
            return "owners/findOwners";  // ← Back to search page
        }
        else if (results.size() == 1) {
            return "redirect:/owners/" + results.get(0).getId(); // ← Direct to owner
        }
        else {
            model.put("selections", results);
            return "owners/ownersList";  // ← Show list of matches
        }
    }
}
```

**Trace what happens when you search for an owner:**
```
1. Browser: GET /owners?lastName=Smith
2. Spring routes to: processFindForm(lastName="Smith")
3. Calls: owners.findByLastName("Smith")  [Repository]
4. Repository runs SQL: SELECT * FROM owners WHERE last_name LIKE 'Smith%'
5. Returns List<Owner>
6. Controller decides: 0 results? show error. 1 result? redirect. Many? show list.
7. Thymeleaf renders HTML and sends to browser
```

---

## STEP 7: Now Add a New Feature — Step by Step

### Feature: "Add a phone number search to find owners by telephone"

**Step 1 — Design it first (on paper)**
```
Endpoint:   GET /owners/search/phone?telephone=1234567890
Returns:    List of owners with that phone number
New or mod: New endpoint, new repository method
Risk:       Zero — we're adding, not changing anything
```

**Step 2 — Add the Repository method**

Open `OwnerRepository.java` and ADD (don't change existing):

```java
public interface OwnerRepository extends JpaRepository<Owner, Integer> {

    // EXISTING - don't touch
    @Query("SELECT DISTINCT owner FROM Owner owner 
            left join fetch owner.pets 
            WHERE owner.lastName LIKE :lastName%")
    List<Owner> findByLastName(@Param("lastName") String lastName);

    // NEW - added below, existing code untouched
    // Spring auto-generates: SELECT * FROM owners WHERE telephone = ?
    List<Owner> findByTelephone(String telephone);
}
```

**Step 3 — Add the Controller method**

Open `OwnerController.java` and ADD a new method (don't touch existing methods):

```java
// ADD this method to existing OwnerController class
// Existing methods are completely untouched

@GetMapping("/owners/search/phone")
public String findByPhone(@RequestParam String telephone,
                           Map<String, Object> model) {

    // Input validation
    if (telephone == null || telephone.isBlank()) {
        model.put("error", "Please enter a phone number");
        return "owners/findByPhone";  // template we'll create
    }

    List<Owner> results = this.owners.findByTelephone(telephone);

    if (results.isEmpty()) {
        model.put("message", "No owners found with that phone number");
    } else {
        model.put("owners", results);
    }

    return "owners/findByPhone";
}
```

**Step 4 — Create the Thymeleaf Template**

Create `src/main/resources/templates/owners/findByPhone.html`:

```html
<!DOCTYPE html>
<html xmlns:th="https://www.thymeleaf.org"
      th:replace="~{fragments/layout :: layout (~{::body},'owners')}">
<body>
    <div class="container xd-container">
        <h2>Find Owner by Phone</h2>

        <!-- Search Form -->
        <form method="get" action="/owners/search/phone">
            <div class="form-group">
                <label>Phone Number:</label>
                <input type="text" name="telephone" class="form-control"
                       placeholder="Enter phone number"/>
            </div>
            <button type="submit" class="btn btn-primary">Search</button>
        </form>

        <!-- Error Message -->
        <div th:if="${error}" class="alert alert-danger">
            <p th:text="${error}"></p>
        </div>

        <!-- No Results -->
        <div th:if="${message}" class="alert alert-info">
            <p th:text="${message}"></p>
        </div>

        <!-- Results Table -->
        <table th:if="${owners}" class="table table-striped">
            <thead>
                <tr>
                    <th>Name</th>
                    <th>Phone</th>
                    <th>City</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody>
                <tr th:each="owner : ${owners}">
                    <td th:text="${owner.firstName + ' ' + owner.lastName}"></td>
                    <td th:text="${owner.telephone}"></td>
                    <td th:text="${owner.city}"></td>
                    <td>
                        <a th:href="@{/owners/{id}(id=${owner.id})}">View</a>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</body>
</html>
```

**Step 5 — Test it**
```bash
# Run the app
./mvnw spring-boot:run

# Visit in browser:
http://localhost:8080/owners/search/phone?telephone=6085551023

# Should show owners with that phone number
```

**What you changed:**
```
OwnerRepository.java  → Added 1 method  (existing method untouched)
OwnerController.java  → Added 1 method  (existing methods untouched)
findByPhone.html      → New file created (nothing existing touched)
```

**Zero risk of breaking existing features.**

---

## STEP 8: A More Complex Feature — "Vet Appointment Booking"

This teaches you how features span multiple layers.

**Design first:**
```
Feature:   Owner can book a vet appointment for their pet
Data need: New table: appointments (id, pet_id, vet_id, date, reason)
Endpoints: GET  /appointments/new?petId=1  → show booking form
           POST /appointments/new          → save appointment
           GET  /appointments?ownerId=1    → view my appointments
```

**Step 1 — Add DB table in `schema.sql`:**
```sql
-- ADD at the bottom, never modify existing tables
CREATE TABLE appointments (
  id          INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
  pet_id      INTEGER NOT NULL,
  vet_id      INTEGER NOT NULL,
  appt_date   DATE NOT NULL,
  reason      VARCHAR(255),
  FOREIGN KEY (pet_id) REFERENCES pets(id),
  FOREIGN KEY (vet_id) REFERENCES vets(id)
);
```

**Step 2 — Create the Entity:**
```java
// NEW FILE: src/.../owner/Appointment.java

@Entity
@Table(name = "appointments")
public class Appointment {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    // Many appointments can be for one pet
    @ManyToOne
    @JoinColumn(name = "pet_id")
    private Pet pet;

    // Many appointments can be with one vet
    @ManyToOne
    @JoinColumn(name = "vet_id")
    private Vet vet;

    @Column(name = "appt_date")
    @DateTimeFormat(pattern = "yyyy-MM-dd")
    private LocalDate apptDate;

    @Column(name = "reason")
    @NotBlank
    private String reason;

    // getters and setters
}
```

**Step 3 — Create the Repository:**
```java
// NEW FILE: AppointmentRepository.java

public interface AppointmentRepository extends JpaRepository<Appointment, Integer> {

    // Find all appointments for a specific pet
    List<Appointment> findByPetId(Integer petId);

    // Find all appointments for all pets belonging to an owner
    @Query("SELECT a FROM Appointment a WHERE a.pet.owner.id = :ownerId")
    List<Appointment> findByOwnerId(@Param("ownerId") Integer ownerId);

    // Find all appointments with a specific vet
    List<Appointment> findByVetId(Integer vetId);
}
```

**Step 4 — Create the Controller:**
```java
// NEW FILE: AppointmentController.java
// Completely new file = zero risk to existing code

@Controller
public class AppointmentController {

    private final AppointmentRepository appointments;
    private final PetRepository pets;
    private final VetRepository vets;

    // Spring injects all three automatically
    public AppointmentController(AppointmentRepository appointments,
                                  PetRepository pets,
                                  VetRepository vets) {
        this.appointments = appointments;
        this.pets = pets;
        this.vets = vets;
    }

    // Show booking form
    @GetMapping("/appointments/new")
    public String showBookingForm(@RequestParam Integer petId,
                                   Map<String, Object> model) {

        Pet pet = pets.findById(petId)
                      .orElseThrow(() -> new RuntimeException("Pet not found"));

        model.put("appointment", new Appointment());
        model.put("pet", pet);
        model.put("vets", vets.findAll());  // All available vets
        return "appointments/bookingForm";
    }

    // Process form submission
    @PostMapping("/appointments/new")
    public String processBooking(@Valid Appointment appointment,
                                  BindingResult result,
                                  @RequestParam Integer petId,
                                  Map<String, Object> model) {

        if (result.hasErrors()) {
            model.put("vets", vets.findAll());
            return "appointments/bookingForm";
        }

        Pet pet = pets.findById(petId).orElseThrow();
        appointment.setPet(pet);
        appointments.save(appointment);

        // Redirect to owner's page after booking
        return "redirect:/owners/" + pet.getOwner().getId();
    }

    // View all appointments for an owner
    @GetMapping("/appointments")
    public String viewAppointments(@RequestParam Integer ownerId,
                                    Map<String, Object> model) {

        List<Appointment> ownerAppointments = appointments.findByOwnerId(ownerId);
        model.put("appointments", ownerAppointments);
        return "appointments/list";
    }
}
```

---

## STEP 9: How Everything Is Connected — The Big Picture

```
HTTP Request arrives
        │
        ▼
┌───────────────────┐
│   DispatcherServlet│  ← Spring's front controller, routes ALL requests
└───────────────────┘
        │
        ▼
┌───────────────────┐
│    Controller     │  ← Receives request, validates input
│  @GetMapping      │    Returns view name OR redirect
│  @PostMapping     │
└───────────────────┘
        │ calls
        ▼
┌───────────────────┐
│    Repository     │  ← Talks to DB using JPA/Hibernate
│  JpaRepository    │    Translates Java calls to SQL
│  @Query           │
└───────────────────┘
        │ queries
        ▼
┌───────────────────┐
│    Database       │  ← H2 in dev, MySQL/Postgres in prod
│  (H2/MySQL)       │
└───────────────────┘
        │
        ▼ returns Entity objects back up the chain
┌───────────────────┐
│  Thymeleaf Engine │  ← Merges Java objects + HTML templates
│  (View Layer)     │    Produces final HTML
└───────────────────┘
        │
        ▼
   HTML Response sent to browser
```

**Spring Beans and Dependency Injection — How wiring works:**
```
When app starts, Spring scans all classes:

@Entity       → "Register this as a JPA entity (table mapping)"
@Repository   → "Create one instance, make it available for injection"
@Controller   → "Create one instance, handle HTTP routes"
@Service      → "Create one instance, holds business logic"
@Component    → "Create one instance, generic bean"

When Spring sees:
public OwnerController(OwnerRepository owners) { ... }

It thinks:
"OwnerController needs an OwnerRepository. 
 I have one. I'll inject it automatically."

This is why you never write: new OwnerRepository()
Spring manages all object creation.
```

---

## STEP 10: Checklist Before Adding Any Feature

```
□ 1. Do I understand what the feature needs to DO? (Write it in plain English)
□ 2. Does a similar feature already exist I can learn from?
□ 3. Do I need a new DB table, or can I use existing ones?
□ 4. What are the HTTP endpoints? (URL, method, inputs, outputs)
□ 5. What repository methods do I need? (new file or add to existing?)
□ 6. Do I need a new Controller or add to existing one?
□ 7. What can go wrong? (not found, invalid input, empty results)
□ 8. Am I creating new files vs modifying existing ones? (prefer new)
□ 9. After adding: do ALL existing pages still work?
□ 10. Does my code follow the same patterns as existing code?
```

---

## The Core Mental Model — Remember This Always

```
Entity     = A row in a database table        (WHAT is stored)
Repository = The librarian                    (HOW you get/save it)
Controller = The receptionist                 (WHO handles requests)
Template   = The presentation layer           (HOW it looks)

Spring IoC = You declare what you need,
             Spring figures out how to provide it.
             You never use 'new' for Spring beans.
```

Start with PetClinic. Run it → read `pom.xml` → read `schema.sql` → trace one request from URL to DB and back. Once that click happens, everything else is just repetition of the same pattern at different scales.

-----

# Ensuring New Features Work Without Breaking Existing Code

## The Core Problem First

When you add complex features, there are 3 types of risk:

```
1. DIRECT BREAK    → Your new code changes existing behavior
2. INDIRECT BREAK  → Your new code shares resources (DB, beans) with existing code
3. SILENT BREAK    → Existing code still runs but returns wrong data
```

Testing is how you **catch all three before users do.**

---

## The Testing Pyramid — Start Here

```
                    /\
                   /  \
                  / E2E \          ← Few, slow, test full user flows
                 /────────\
                /Integration\      ← Medium, test layers together
               /──────────────\
              /   Unit Tests   \   ← Many, fast, test one thing at a time
             /──────────────────\
```

**Rule:** Write many unit tests, some integration tests, few E2E tests.

---

## PART 1: Unit Testing — Test Each Piece in Isolation

### What is a Unit Test?
Test ONE class/method in isolation. Fake (mock) everything it depends on.

### Setup — Add to `pom.xml`
```xml
<!-- Already included in Spring Boot starter, but confirm it's there -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <!-- Includes: JUnit 5, Mockito, AssertJ automatically -->
</dependency>
```

---

### Real Example: Testing the Appointment Feature We Built

**The Appointment Controller we wrote:**
```java
@Controller
public class AppointmentController {

    private final AppointmentRepository appointments;
    private final PetRepository pets;
    private final VetRepository vets;

    @PostMapping("/appointments/new")
    public String processBooking(@Valid Appointment appointment,
                                  BindingResult result,
                                  @RequestParam Integer petId,
                                  Map<String, Object> model) {
        if (result.hasErrors()) {
            model.put("vets", vets.findAll());
            return "appointments/bookingForm";
        }
        Pet pet = pets.findById(petId).orElseThrow();
        appointment.setPet(pet);
        appointments.save(appointment);
        return "redirect:/owners/" + pet.getOwner().getId();
    }
}
```

**Now write a unit test for this:**

```java
// src/test/java/.../AppointmentControllerTest.java

@ExtendWith(MockitoExtension.class)   // ← Use Mockito, no Spring context needed
class AppointmentControllerTest {

    // @Mock creates a FAKE version of these — no real DB, no real Spring
    @Mock
    private AppointmentRepository appointmentRepository;

    @Mock
    private PetRepository petRepository;

    @Mock
    private VetRepository vetRepository;

    // @InjectMocks creates REAL AppointmentController
    // but injects the FAKE repositories into it
    @InjectMocks
    private AppointmentController controller;

    // ─────────────────────────────────────────────────────
    // TEST 1: Happy path — valid appointment gets saved
    // ─────────────────────────────────────────────────────
    @Test
    void processBooking_ValidAppointment_SavesAndRedirects() {

        // ARRANGE — set up the scenario
        Owner owner = new Owner();
        owner.setId(5);

        Pet pet = new Pet();
        pet.setId(1);
        pet.setOwner(owner);  // pet belongs to owner with id=5

        Appointment appointment = new Appointment();
        appointment.setReason("Annual checkup");
        appointment.setApptDate(LocalDate.now().plusDays(3));

        BindingResult result = mock(BindingResult.class);
        Map<String, Object> model = new HashMap<>();

        // Tell fake repository: when asked for pet with id=1, return our pet
        when(petRepository.findById(1)).thenReturn(Optional.of(pet));

        // Tell fake result: no errors
        when(result.hasErrors()).thenReturn(false);

        // ACT — call the actual method
        String viewName = controller.processBooking(appointment, result, 1, model);

        // ASSERT — verify the outcomes
        assertEquals("redirect:/owners/5", viewName);  // redirects to owner 5

        // Verify save() was called ONCE with our appointment
        verify(appointmentRepository, times(1)).save(appointment);

        // Verify the pet was set on the appointment
        assertEquals(pet, appointment.getPet());
    }

    // ─────────────────────────────────────────────────────
    // TEST 2: Invalid input — form has errors, goes back to form
    // ─────────────────────────────────────────────────────
    @Test
    void processBooking_ValidationErrors_ReturnsToForm() {

        // ARRANGE
        Appointment appointment = new Appointment(); // empty/invalid
        BindingResult result = mock(BindingResult.class);
        Map<String, Object> model = new HashMap<>();

        List<Vet> allVets = Arrays.asList(new Vet(), new Vet());

        when(result.hasErrors()).thenReturn(true);  // simulate validation failure
        when(vetRepository.findAll()).thenReturn(allVets);

        // ACT
        String viewName = controller.processBooking(appointment, result, 1, model);

        // ASSERT
        assertEquals("appointments/bookingForm", viewName);  // stays on form
        assertEquals(allVets, model.get("vets"));            // vets are in model

        // CRITICAL: verify save was NEVER called when there are errors
        verify(appointmentRepository, never()).save(any());
    }

    // ─────────────────────────────────────────────────────
    // TEST 3: Pet not found — should throw exception
    // ─────────────────────────────────────────────────────
    @Test
    void processBooking_PetNotFound_ThrowsException() {

        // ARRANGE
        BindingResult result = mock(BindingResult.class);
        when(result.hasErrors()).thenReturn(false);

        // Tell fake repo: pet with id=999 does NOT exist
        when(petRepository.findById(999)).thenReturn(Optional.empty());

        // ASSERT + ACT together — expect an exception to be thrown
        assertThrows(RuntimeException.class, () -> {
            controller.processBooking(new Appointment(), result, 999, new HashMap<>());
        });

        // Verify nothing was saved
        verify(appointmentRepository, never()).save(any());
    }
}
```

**Run it:**
```bash
./mvnw test -Dtest=AppointmentControllerTest
```

---

### Testing the Service Layer (Business Logic)

If you add a service class (recommended for complex features):

```java
@Service
public class AppointmentService {

    private final AppointmentRepository appointmentRepo;
    private final VetRepository vetRepo;

    public AppointmentService(AppointmentRepository appointmentRepo,
                               VetRepository vetRepo) {
        this.appointmentRepo = appointmentRepo;
        this.vetRepo = vetRepo;
    }

    // Complex business rule:
    // A vet can't have more than 5 appointments on the same day
    public Appointment bookAppointment(Appointment appointment) {
        long apptCount = appointmentRepo
            .countByVetIdAndApptDate(
                appointment.getVet().getId(),
                appointment.getApptDate()
            );

        if (apptCount >= 5) {
            throw new VetFullyBookedException(
                "This vet is fully booked on " + appointment.getApptDate()
            );
        }

        return appointmentRepo.save(appointment);
    }
}
```

**Test the business rule:**

```java
@ExtendWith(MockitoExtension.class)
class AppointmentServiceTest {

    @Mock
    private AppointmentRepository appointmentRepo;

    @Mock
    private VetRepository vetRepo;

    @InjectMocks
    private AppointmentService service;

    @Test
    void bookAppointment_VetHasCapacity_SavesAppointment() {
        // ARRANGE
        Vet vet = new Vet(); vet.setId(1);
        Appointment appt = new Appointment();
        appt.setVet(vet);
        appt.setApptDate(LocalDate.of(2026, 3, 10));

        // Vet only has 2 appointments that day (under limit of 5)
        when(appointmentRepo.countByVetIdAndApptDate(1, appt.getApptDate()))
            .thenReturn(2L);
        when(appointmentRepo.save(appt)).thenReturn(appt);

        // ACT
        Appointment saved = service.bookAppointment(appt);

        // ASSERT
        assertNotNull(saved);
        verify(appointmentRepo).save(appt);
    }

    @Test
    void bookAppointment_VetFullyBooked_ThrowsException() {
        // ARRANGE
        Vet vet = new Vet(); vet.setId(1);
        Appointment appt = new Appointment();
        appt.setVet(vet);
        appt.setApptDate(LocalDate.of(2026, 3, 10));

        // Vet already has 5 appointments — fully booked
        when(appointmentRepo.countByVetIdAndApptDate(1, appt.getApptDate()))
            .thenReturn(5L);

        // ASSERT: exception must be thrown, nothing saved
        assertThrows(VetFullyBookedException.class,
            () -> service.bookAppointment(appt));

        verify(appointmentRepo, never()).save(any());
    }

    @Test
    void bookAppointment_VetAtCapacityEdgeCase_ThrowsException() {
        // Edge case: exactly AT the limit (4 appointments → 5th should work,
        //            but a 6th should fail)
        Vet vet = new Vet(); vet.setId(1);
        Appointment appt = new Appointment();
        appt.setVet(vet);
        appt.setApptDate(LocalDate.of(2026, 3, 10));

        when(appointmentRepo.countByVetIdAndApptDate(1, appt.getApptDate()))
            .thenReturn(4L);  // 4 existing, 5th should succeed
        when(appointmentRepo.save(appt)).thenReturn(appt);

        assertDoesNotThrow(() -> service.bookAppointment(appt));
    }
}
```

---

## PART 2: Integration Testing — Test Layers Together With Real DB

Unit tests test logic. Integration tests test that your code works with a **real database.**

```java
// @SpringBootTest loads the FULL Spring context
// @Transactional rolls back DB changes after each test (keeps DB clean)
@SpringBootTest
@Transactional
class AppointmentRepositoryIntegrationTest {

    @Autowired
    private AppointmentRepository appointmentRepo;

    @Autowired
    private PetRepository petRepo;

    @Autowired
    private VetRepository vetRepo;

    @Test
    void findByOwnerId_ReturnsCorrectAppointments() {

        // ARRANGE — use real data from data.sql (loaded on startup)
        // PetClinic's data.sql has owner with id=1, pet with id=1
        Pet pet = petRepo.findById(1).orElseThrow();

        Vet vet = vetRepo.findById(1).orElseThrow();

        Appointment appt = new Appointment();
        appt.setPet(pet);
        appt.setVet(vet);
        appt.setApptDate(LocalDate.now().plusDays(1));
        appt.setReason("Check up");
        appointmentRepo.save(appt);

        // ACT — call the custom query we wrote
        List<Appointment> results = appointmentRepo.findByOwnerId(
            pet.getOwner().getId()
        );

        // ASSERT — the appointment we just saved should appear
        assertFalse(results.isEmpty());
        assertTrue(results.stream()
            .anyMatch(a -> a.getReason().equals("Check up")));
    }

    @Test
    void countByVetIdAndApptDate_CountsCorrectly() {

        Pet pet = petRepo.findById(1).orElseThrow();
        Vet vet = vetRepo.findById(1).orElseThrow();
        LocalDate testDate = LocalDate.of(2026, 6, 15);

        // Save 3 appointments on the same date
        for (int i = 0; i < 3; i++) {
            Appointment a = new Appointment();
            a.setPet(pet); a.setVet(vet);
            a.setApptDate(testDate);
            a.setReason("Visit " + i);
            appointmentRepo.save(a);
        }

        // ACT
        long count = appointmentRepo.countByVetIdAndApptDate(vet.getId(), testDate);

        // ASSERT
        assertEquals(3, count);
    }
}
```

---

## PART 3: Testing HTTP Layer — MockMvc

This tests your Controller endpoints without starting a real server:

```java
@WebMvcTest(AppointmentController.class)  // Only loads web layer, not full context
class AppointmentControllerMvcTest {

    @Autowired
    private MockMvc mockMvc;  // Simulates HTTP requests

    // Mock the dependencies since @WebMvcTest doesn't load them
    @MockBean
    private AppointmentRepository appointmentRepository;

    @MockBean
    private PetRepository petRepository;

    @MockBean
    private VetRepository vetRepository;

    @Test
    void getBookingForm_ValidPetId_ReturnsForm() throws Exception {

        // ARRANGE
        Owner owner = new Owner(); owner.setId(1);
        Pet pet = new Pet(); pet.setId(1); pet.setOwner(owner);
        pet.setName("Buddy");

        when(petRepository.findById(1)).thenReturn(Optional.of(pet));
        when(vetRepository.findAll()).thenReturn(Collections.emptyList());

        // ACT + ASSERT
        mockMvc.perform(get("/appointments/new").param("petId", "1"))
            .andExpect(status().isOk())                          // HTTP 200
            .andExpect(view().name("appointments/bookingForm"))  // correct template
            .andExpect(model().attributeExists("pet"))           // pet in model
            .andExpect(model().attributeExists("vets"));         // vets in model
    }

    @Test
    void postBooking_ValidData_RedirectsToOwner() throws Exception {

        Owner owner = new Owner(); owner.setId(5);
        Pet pet = new Pet(); pet.setId(1); pet.setOwner(owner);

        when(petRepository.findById(1)).thenReturn(Optional.of(pet));

        mockMvc.perform(post("/appointments/new")
                .param("petId", "1")
                .param("reason", "Annual checkup")
                .param("apptDate", "2026-06-15")
                .param("vet.id", "1"))
            .andExpect(status().is3xxRedirection())              // redirect
            .andExpect(redirectedUrl("/owners/5"));              // to owner's page
    }

    @Test
    void postBooking_EmptyReason_StaysOnForm() throws Exception {

        mockMvc.perform(post("/appointments/new")
                .param("petId", "1")
                .param("reason", "")   // blank = validation error
                .param("apptDate", "2026-06-15"))
            .andExpect(status().isOk())
            .andExpect(view().name("appointments/bookingForm")); // back to form
    }
}
```

---

## PART 4: Regression Testing — Proving Existing Features Still Work

This is the most important part. After adding your feature, run tests on **existing features**:

```java
// Test that the EXISTING owner search still works after your changes
@SpringBootTest
@AutoConfigureMockMvc
@Transactional
class ExistingOwnerFeaturesRegressionTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void existingOwnerSearch_StillWorks() throws Exception {
        mockMvc.perform(get("/owners").param("lastName", "Davis"))
            .andExpect(status().isOk())
            .andExpect(view().name("owners/ownersList"));
    }

    @Test
    void existingOwnerCreation_StillWorks() throws Exception {
        mockMvc.perform(post("/owners/new")
                .param("firstName", "Test")
                .param("lastName", "User")
                .param("address", "123 Main St")
                .param("city", "Anytown")
                .param("telephone", "1234567890"))
            .andExpect(status().is3xxRedirection());
    }

    @Test
    void existingVetList_StillWorks() throws Exception {
        mockMvc.perform(get("/vets.html"))
            .andExpect(status().isOk())
            .andExpect(view().name("vets/vetList"));
    }

    @Test
    void existingPetCreation_StillWorks() throws Exception {
        mockMvc.perform(post("/owners/1/pets/new")
                .param("name", "TestPet")
                .param("birthDate", "2020-01-01")
                .param("type.id", "1"))
            .andExpect(status().is3xxRedirection());
    }
}
```

Run this **before** your feature (to confirm all pass) and **after** (to catch breaks).

---

## PART 5: The Full Safety Workflow

Here is the exact process to follow for every complex feature:

```
BEFORE writing any code:
────────────────────────
1. Run ALL existing tests → they must all pass (this is your baseline)
   ./mvnw test

2. Note the test count:
   "Tests run: 47, Failures: 0, Errors: 0"


WHILE writing your feature:
────────────────────────────
3. Write the test FIRST (TDD) before the implementation
   - Forces you to think about behavior before code
   - Failing test = clear goal to reach

4. Write the minimum code to make the test pass

5. Refactor the code while keeping tests green


AFTER writing your feature:
────────────────────────────
6. Run your new tests → all must pass

7. Run ALL existing tests AGAIN → same count must pass
   If any existing test fails → your new code broke something → fix it

8. Manually test the new feature in the browser

9. Manually test existing features in the browser
   (same ones your regression tests cover)
```

---

## PART 6: What to Test for Complex Features — Checklist

```
For every new feature, write tests covering:

□ HAPPY PATH
  → Normal input, everything works, expected output

□ EMPTY / NULL INPUT
  → What happens when required fields are missing?

□ INVALID INPUT
  → Wrong format, negative numbers, past dates, etc.

□ NOT FOUND
  → What if the entity being referenced doesn't exist?

□ BOUNDARY / EDGE CASES
  → Exactly at the limit (vet has exactly 5 appointments)
  → One over the limit (vet has exactly 6)
  → Empty lists (owner has no pets)

□ SIDE EFFECTS
  → Was save() called? Was it called only once?
  → Was delete() NOT called when it shouldn't be?

□ EXISTING FEATURES (regression)
  → Run all old tests after your change
```

---

## PART 7: One Command to Run Everything

```bash
# Run all tests (unit + integration) at once
./mvnw test

# Run a specific test class
./mvnw test -Dtest=AppointmentServiceTest

# Run a specific test method
./mvnw test -Dtest=AppointmentServiceTest#bookAppointment_VetFullyBooked_ThrowsException

# Run all tests and see coverage report
./mvnw test jacoco:report
# Open: target/site/jacoco/index.html
```

---

## The Mental Model — One Sentence Each

```
Unit Test        = "Does my logic work in perfect conditions?"
Integration Test = "Does my code work with the real database?"
MockMvc Test     = "Does my HTTP layer return the right responses?"
Regression Test  = "Did I break anything that was already working?"
Mock             = "A fake object that pretends to be the real thing"
@Transactional   = "Roll back all DB changes after each test — keep DB clean"
```

The key shift: **don't think of tests as extra work. They ARE the proof that your feature works.** Without them, you only know your feature works when you manually click through the browser — which is slow, incomplete, and doesn't catch regressions.

------

# Understanding Microservices Projects From First Principles

## Start With The Core Question: Why Does This Architecture Exist?

Before reading a single line of code, understand the problem microservices solve:

```
MONOLITH (Single Spring Boot app):
─────────────────────────────────
Everything in one app:
  UserService + OrderService + PaymentService + NotificationService
       │
  ONE deployment, ONE database, ONE codebase

Problem when it grows:
  - Change payment logic → redeploy ENTIRE app
  - Payment service crashes → ENTIRE app goes down
  - Need to scale orders (Black Friday) → must scale EVERYTHING
  - 50 developers editing same codebase → merge conflicts hell


MICROSERVICES (Multiple small apps):
─────────────────────────────────────
Each service = its own Spring Boot app + its own database:
  [User Service]   [Order Service]   [Payment Service]   [Notification Service]
       │                 │                  │                      │
   users_db          orders_db          payments_db           (no DB, just sends emails)

Benefits:
  - Change payment → redeploy ONLY payment service
  - Payment crashes → only payments affected, rest still runs
  - Scale ONLY order service on Black Friday
  - Teams own their service independently
```

**First principle:** Every complexity in a microservices project exists because of this trade-off — independence in exchange for distributed system problems.

---

## The New Problems Microservices Create

```
Monolith:   UserService calls OrderService.getOrders() → simple Java method call
Microservice: User Service needs to call Order Service → HTTP call across network

New problems you'll see in the codebase:
┌─────────────────────────────────────────────────────────┐
│  1. SERVICE DISCOVERY   How does Service A find Service B?     │
│  2. API GATEWAY         Single entry point for all clients     │
│  3. COMMUNICATION       REST (sync) or Events (async)?         │
│  4. FAULT TOLERANCE     What if a service is down?             │
│  5. DISTRIBUTED CONFIG  Each service needs config              │
│  6. TRACING             Follow a request across services       │
│  7. AUTH                Who handles login across services?     │
└─────────────────────────────────────────────────────────┘
```

Every tool and pattern you see in the codebase is solving one of these 7 problems. When you see an unfamiliar class or config, ask: **which of these 7 is this solving?**

---

## STEP 1: Understand the Project From The Outside

Use this real project: `https://github.com/GoogleCloudPlatform/microservices-demo` or Spring's own `spring-petclinic-microservices`.

```bash
git clone https://github.com/spring-petclinic/spring-petclinic-microservices
cd spring-petclinic-microservices
ls -la
```

**What you see:**
```
spring-petclinic-microservices/
│
├── spring-petclinic-api-gateway/          ← Single entry point
├── spring-petclinic-config-server/        ← Central configuration
├── spring-petclinic-discovery-server/     ← Service registry (Eureka)
├── spring-petclinic-customers-service/    ← Owns: owners, pets
├── spring-petclinic-visits-service/       ← Owns: visits
├── spring-petclinic-vets-service/         ← Owns: vets
├── spring-petclinic-admin-server/         ← Monitoring dashboard
│
├── docker-compose.yml                     ← Run all services together
└── pom.xml                                ← Parent POM (shared dependencies)
```

**First thing to draw on paper:**
```
                        ┌─────────────────┐
Browser/Mobile ────────▶│   API Gateway   │:8080
                        └────────┬────────┘
                                 │ routes to
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                   ▼
    ┌──────────────────┐ ┌──────────────┐ ┌──────────────────┐
    │ Customers Service│ │ Vets Service │ │  Visits Service  │
    │      :8081       │ │    :8083     │ │      :8082       │
    └────────┬─────────┘ └──────┬───────┘ └────────┬─────────┘
             │                  │                   │
         customers_db        vets_db            visits_db


    ┌──────────────────┐    ┌──────────────────────┐
    │  Config Server   │    │  Discovery Server    │
    │     :8888        │    │  (Eureka)  :8761     │
    └──────────────────┘    └──────────────────────┘
```

This diagram tells you more than 2 hours of reading code.

---

## STEP 2: Read the Infrastructure Services First

These are not business logic — they are the glue holding everything together.

### 2A — Config Server (Solves: Distributed Configuration)

```bash
cd spring-petclinic-config-server
cat src/main/resources/application.yml
```

```yaml
server:
  port: 8888

spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/spring-petclinic/spring-petclinic-microservices-config
          # ↑ All config lives in a SEPARATE git repo
          # Each service pulls its config from here on startup
```

```java
@SpringBootApplication
@EnableConfigServer          // ← This ONE annotation makes it a config server
public class ConfigServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(ConfigServerApplication.class, args);
    }
}
```

**What this means:**
```
Without Config Server:
  customers-service/src/main/resources/application.yml  ← hardcoded config
  visits-service/src/main/resources/application.yml     ← hardcoded config
  vets-service/src/main/resources/application.yml       ← hardcoded config

  Problem: Change DB password → update and redeploy ALL services

With Config Server:
  One git repo holds all config
  Each service fetches its config at startup from Config Server
  Change DB password → update one file, services refresh automatically
```

### 2B — Discovery Server (Solves: Service Discovery)

```bash
cd spring-petclinic-discovery-server
cat src/main/java/.../EurekaServerApplication.java
```

```java
@SpringBootApplication
@EnableEurekaServer      // ← Makes this a service registry
public class EurekaServerApplication { ... }
```

```yaml
# application.yml
server:
  port: 8761

eureka:
  client:
    registerWithEureka: false  # Don't register with itself
    fetchRegistry: false
```

**What Eureka does:**
```
Without Discovery:
  Customers service wants to call Visits service
  It needs to know: "Visits is at http://192.168.1.45:8082"
  Problem: IP address changes in cloud, multiple instances exist

With Eureka:
  Every service on startup: "Hey Eureka, I'm visits-service, I'm at port 8082"
  Customers service asks: "Hey Eureka, where is visits-service?"
  Eureka responds: "It's at these addresses: [instance1, instance2]"
  Customers service calls it without hardcoded URLs

Visit: http://localhost:8761 after startup → see all registered services
```

---

## STEP 3: Read a Business Service

```bash
cd spring-petclinic-customers-service
```

**Read pom.xml:**
```xml
<dependencies>
    <!-- Standard Spring Boot web -->
    <dependency>spring-boot-starter-web</dependency>
    <dependency>spring-boot-starter-data-jpa</dependency>

    <!-- Connects to Config Server for its config -->
    <dependency>spring-cloud-starter-config</dependency>

    <!-- Registers itself with Eureka -->
    <dependency>spring-cloud-starter-netflix-eureka-client</dependency>

    <!-- Exposes health/metrics endpoints -->
    <dependency>spring-boot-starter-actuator</dependency>
</dependencies>
```

**Read bootstrap.yml** (loads before application.yml):
```yaml
spring:
  application:
    name: customers-service    # ← Its name in Eureka registry

  config:
    import: optional:configserver:http://localhost:8888
    # ↑ "Go get my config from Config Server at startup"
```

**Read the main class:**
```java
@SpringBootApplication
@EnableDiscoveryClient      // ← "Register me with Eureka"
public class CustomersServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(CustomersServiceApplication.class, args);
    }
}
```

**Read the Controller:**
```java
@RestController             // ← Note: REST not @Controller — returns JSON not HTML
@RequestMapping("/owners")
public class OwnerResource {

    private final OwnerRepository ownerRepository;

    @GetMapping
    public List<Owner> getAllOwners() {
        return ownerRepository.findAll();
    }

    @GetMapping("/{ownerId}")
    public Owner getOwner(@PathVariable int ownerId) {
        return ownerRepository.findById(ownerId)
                              .orElseThrow(() -> new ResourceNotFoundException("Owner not found"));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Owner createOwner(@Valid @RequestBody Owner owner) {
        return ownerRepository.save(owner);
    }
}
```

**Key difference from monolith:**
```
Monolith Controller:     returns String (view name) or ModelAndView
Microservice Controller: returns @ResponseBody (JSON) — other services consume it
```

---

## STEP 4: Understand How Services Talk To Each Other

This is the hardest concept. There are two patterns:

### Pattern 1: Synchronous (REST via Feign Client)

When Service A needs data from Service B **right now**:

```
API Gateway receives: GET /owners/1/pets
API Gateway asks: "I need to show owner + their visits"

Step 1: Gateway calls Customers Service → GET /owners/1
Step 2: Gateway calls Visits Service   → GET /visits?ownerId=1
Step 3: Gateway combines responses
Step 4: Returns combined JSON to browser
```

**How the API Gateway does this with Feign:**

```java
// In API Gateway — declaring a client for Customers Service
@FeignClient(name = "customers-service")  // ← "customers-service" = Eureka name
public interface CustomersServiceClient {

    // This interface method = HTTP call to customers-service
    // Feign generates the actual HTTP call automatically
    @GetMapping("/owners/{ownerId}")
    Owner getOwner(@PathVariable int ownerId);

    @GetMapping("/owners")
    List<Owner> getAllOwners();
}
```

```java
// In API Gateway Controller — using the Feign client
@RestController
public class ApiGatewayController {

    @Autowired
    private CustomersServiceClient customersClient;  // Feign-generated HTTP client

    @Autowired
    private VisitsServiceClient visitsClient;

    @GetMapping("/api/gateway/owners/{ownerId}")
    public OwnerDetails getOwnerDetails(@PathVariable int ownerId) {

        // This is actually an HTTP call to customers-service
        Owner owner = customersClient.getOwner(ownerId);

        // This is an HTTP call to visits-service
        List<Visit> visits = visitsClient.getVisitsForPets(
            owner.getPets().stream()
                 .map(Pet::getId)
                 .collect(toList())
        );

        return new OwnerDetails(owner, visits);
    }
}
```

**What this looks like end-to-end:**
```
Browser → GET /api/gateway/owners/1
    │
    ▼
API Gateway (8080)
    ├──HTTP──▶ customers-service (8081): GET /owners/1
    │                  │
    │              returns Owner JSON
    │
    ├──HTTP──▶ visits-service (8082): GET /visits?petIds=1,2
    │                  │
    │              returns Visits JSON
    │
    combines both → returns OwnerDetails JSON
    │
    ▼
Browser receives combined response
```

### Pattern 2: Asynchronous (Events via Message Queue)

When Service A doesn't need to wait for Service B:

```
Example: Owner books appointment
  → Save appointment (immediate, synchronous)
  → Send confirmation email (can happen later, asynchronous)

If email fails, booking should NOT fail
This is where async messaging helps
```

```java
// In Appointments Service — publishes an event
@Service
public class AppointmentService {

    @Autowired
    private RabbitTemplate rabbitTemplate;  // or KafkaTemplate

    public Appointment bookAppointment(Appointment appointment) {
        Appointment saved = appointmentRepo.save(appointment);

        // Fire and forget — don't wait for notification service
        AppointmentBookedEvent event = new AppointmentBookedEvent(
            saved.getId(),
            saved.getOwner().getEmail(),
            saved.getApptDate()
        );

        // Publish to RabbitMQ exchange
        rabbitTemplate.convertAndSend(
            "appointments.exchange",
            "appointment.booked",    // routing key
            event
        );

        return saved;  // Return immediately, don't wait for email
    }
}
```

```java
// In Notification Service — listens for the event
@Service
public class NotificationListener {

    @RabbitListener(queues = "appointment.notifications")
    public void handleAppointmentBooked(AppointmentBookedEvent event) {
        // This runs whenever a new event arrives in the queue
        emailService.sendConfirmation(
            event.getOwnerEmail(),
            event.getAppointmentDate()
        );
    }
}
```

**The key mental model:**
```
SYNCHRONOUS (Feign/REST):
  Service A ──calls──▶ Service B ──response──▶ Service A
  A WAITS for B. If B is down, A fails.
  Use when: You need the response to continue

ASYNCHRONOUS (RabbitMQ/Kafka):
  Service A ──publishes event──▶ [Message Queue] ◀──consumes── Service B
  A does NOT wait. If B is down, event stays in queue until B recovers.
  Use when: You don't need the response immediately
```

---

## STEP 5: Understand the API Gateway

```bash
cd spring-petclinic-api-gateway
cat src/main/resources/application.yml
```

```yaml
spring:
  cloud:
    gateway:
      routes:
        # Rule 1: requests to /api/customer/** go to customers-service
        - id: customers-service
          uri: lb://customers-service    # lb:// = load balanced via Eureka
          predicates:
            - Path=/api/customer/**
          filters:
            - StripPrefix=2              # Remove /api/customer from path

        # Rule 2: requests to /api/vet/** go to vets-service
        - id: vets-service
          uri: lb://vets-service
          predicates:
            - Path=/api/vet/**
          filters:
            - StripPrefix=2

        # Rule 3: requests to /api/visit/** go to visits-service
        - id: visits-service
          uri: lb://visits-service
          predicates:
            - Path=/api/visit/**
          filters:
            - StripPrefix=2
```

**What this means:**
```
Client sends:   GET /api/customer/owners/1
Gateway sees:   /api/customer/** → route to customers-service
Gateway strips: /api/customer prefix
Gateway sends:  GET /owners/1 → to customers-service

Client never knows customers-service exists
Client only talks to the gateway
```

**Gateway also handles:**
```
- Authentication  (check JWT token before forwarding)
- Rate limiting   (block 1000+ requests/minute from same IP)
- Logging         (log every request in one place)
- SSL termination (HTTPS handled here, services use HTTP internally)
```

---

## STEP 6: Understand Fault Tolerance (Circuit Breaker)

What happens when one service is down?

```
Without Circuit Breaker:
  Customer calls API Gateway
  Gateway calls Visits Service
  Visits Service is DOWN
  Gateway waits... 30 second timeout
  Gateway returns 500 error after 30 seconds
  All users experience 30-second hangs

With Circuit Breaker (Resilience4j):
  First 5 calls to Visits Service fail
  Circuit OPENS — stops trying to call Visits Service
  For next 60 seconds: immediately return fallback response
  After 60 seconds: try one call (half-open state)
  If it works: circuit CLOSES, normal operation resumes
```

```java
// In API Gateway or service calling another service
@Service
public class VisitsServiceClient {

    @Autowired
    private WebClient.Builder webClientBuilder;

    // @CircuitBreaker wraps this method
    // If visits-service fails repeatedly → calls getVisitsFallback instead
    @CircuitBreaker(name = "visits-service", fallbackMethod = "getVisitsFallback")
    public List<Visit> getVisitsForPet(int petId) {
        return webClientBuilder.build()
            .get()
            .uri("http://visits-service/visits?petId=" + petId)
            .retrieve()
            .bodyToFlux(Visit.class)
            .collectList()
            .block();
    }

    // Fallback: return empty list instead of crashing
    // User sees "No visits found" instead of 500 error
    public List<Visit> getVisitsFallback(int petId, Exception ex) {
        System.out.println("Visits service down, returning empty: " + ex.getMessage());
        return Collections.emptyList();
    }
}
```

**Circuit Breaker States:**
```
CLOSED (normal):
  All calls go through normally
  Failure counter tracks failures
        │
        │ failure rate > threshold (e.g. 50%)
        ▼
OPEN (broken):
  All calls immediately return fallback
  No calls to broken service (prevents cascade failure)
        │
        │ after wait duration (e.g. 60 seconds)
        ▼
HALF-OPEN (testing):
  Allow limited calls through to test if service recovered
        │
        ├── success → back to CLOSED
        └── failure → back to OPEN
```

---

## STEP 7: How to Add a Feature to a Microservice Project

### Feature: "Add an Appointment Service"

**Step 1 — Decide: new service or extend existing?**
```
Questions to ask:
  Does this data belong to an existing service domain?
  → Visits = vet visits (already exists)
  → Appointments = scheduled future visits (new concept)

  Will this service be deployed/scaled independently?
  → Yes, during booking spikes we want to scale appointments only

Decision: NEW SERVICE — spring-petclinic-appointments-service
```

**Step 2 — Create the new service (copy structure from existing):**
```bash
cp -r spring-petclinic-visits-service spring-petclinic-appointments-service
cd spring-petclinic-appointments-service

# Update pom.xml artifactId
# Update bootstrap.yml spring.application.name
# Clear out Visits entities/repos/controllers
```

**Step 3 — Register with infrastructure:**

```yaml
# bootstrap.yml
spring:
  application:
    name: appointments-service   # ← Eureka will know it by this name

  config:
    import: optional:configserver:http://localhost:8888
```

```java
@SpringBootApplication
@EnableDiscoveryClient           // ← Register with Eureka automatically
public class AppointmentsServiceApplication { ... }
```

**Step 4 — Add route in API Gateway:**
```yaml
# In api-gateway application.yml — ADD this route
- id: appointments-service
  uri: lb://appointments-service
  predicates:
    - Path=/api/appointment/**
  filters:
    - StripPrefix=2
```

**Step 5 — Call Customers Service from Appointments Service:**

```java
// Appointments service needs owner info → call customers-service
@FeignClient(name = "customers-service")
public interface CustomersClient {
    @GetMapping("/owners/{ownerId}")
    Owner getOwner(@PathVariable int ownerId);
}
```

```java
@RestController
@RequestMapping("/appointments")
public class AppointmentController {

    private final AppointmentRepository appointmentRepo;
    private final CustomersClient customersClient;  // calls customers-service

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Appointment createAppointment(@Valid @RequestBody AppointmentRequest request) {

        // Verify owner exists by calling customers-service
        Owner owner = customersClient.getOwner(request.getOwnerId());
        // If owner doesn't exist, Feign throws FeignException → handle it

        Appointment appointment = new Appointment();
        appointment.setOwnerId(request.getOwnerId());
        appointment.setPetId(request.getPetId());
        appointment.setApptDate(request.getApptDate());
        appointment.setReason(request.getReason());

        return appointmentRepo.save(appointment);
    }
}
```

---

## STEP 8: The Complete Mental Map

```
┌─────────────────────────────────────────────────────────────────────┐
│                         MICROSERVICE LANDSCAPE                       │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  INFRASTRUCTURE LAYER (not business logic):                          │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────────────┐ │
│  │ Config Server  │  │ Eureka Server  │  │    API Gateway         │ │
│  │ "central       │  │ "phone book    │  │ "single door for       │ │
│  │  settings"     │  │  for services" │  │  all clients"          │ │
│  └────────────────┘  └────────────────┘  └────────────────────────┘ │
│                                                                       │
│  BUSINESS LAYER (your features live here):                           │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │  Customers   │  │    Vets      │  │   Visits     │              │
│  │  Service     │  │   Service    │  │   Service    │              │
│  │  (owns data) │  │ (owns data)  │  │ (owns data)  │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
│                                                                       │
│  COMMUNICATION:                                                       │
│  Sync  → Feign Client (REST) → use when you NEED the response       │
│  Async → RabbitMQ/Kafka      → use when you DON'T need to wait      │
│                                                                       │
│  RESILIENCE:                                                          │
│  Circuit Breaker → if a service fails, stop calling it, use fallback │
└─────────────────────────────────────────────────────────────────────┘
```

---

## The 5 Questions to Ask When Reading Any Microservice Codebase

```
1. TOPOLOGY
   "How many services exist? Draw them. What does each own?"

2. COMMUNICATION
   "How do services talk? REST (Feign) or Events (Kafka/Rabbit)?"
   Look for: @FeignClient, RabbitTemplate, KafkaTemplate

3. DISCOVERY
   "How do services find each other?"
   Look for: @EnableDiscoveryClient, Eureka, Consul

4. RESILIENCE
   "What happens when a service is down?"
   Look for: @CircuitBreaker, @Retry, fallbackMethod

5. ENTRY POINT
   "Where does a request enter the system?"
   Look for: API Gateway routes, @RequestMapping in gateway
```

Every complex thing you see in a microservice project is answering one of these 5 questions. Once you map each component to its question, the entire project becomes readable.


-------

# Adding Complex Features to Microservices Without Breaking Existing Code

## The Core Mindset First

```
Monolith fear:    "Will my change break this method?"
Microservice fear: "Will my change break OTHER SERVICES 
                   that depend on mine?"

The blast radius is bigger in microservices because:
  - Other services call YOUR API
  - They depend on YOUR response structure
  - They depend on YOUR events
  - A DB schema change can break YOUR service
    which breaks EVERY service that calls you
```

---

## The Golden Rules Before Writing Any Code

```
Rule 1: NEVER change an existing API contract
        (URL, request body, response body)
        → Other services are built against it

Rule 2: NEVER rename or delete a database column directly
        → Your running service reads that column

Rule 3: NEVER change an existing event structure
        → Other services consume those events

Rule 4: ADD don't MODIFY
        → New endpoint > modifying existing endpoint
        → New field > renaming existing field
        → New event > changing existing event

Rule 5: Make changes backward compatible
        → Old callers must still work during transition
```

---

## PART 1: Safe API Changes — The Contract Problem

### The Problem Visualized

```
Customers Service exposes:
GET /owners/{id}
Response:
{
  "id": 1,
  "firstName": "John",
  "lastName": "Doe",
  "telephone": "1234567890"
}

API Gateway calls this endpoint and maps the response.
Visits Service calls this endpoint to get owner info.
Frontend calls this endpoint directly.

You want to ADD: address, city fields to response
You want to RENAME: telephone → phoneNumber
```

### Safe Way — Add Fields, Never Remove/Rename

```java
// EXISTING Owner response class — DO NOT TOUCH existing fields
public class OwnerResponse {
    private Integer id;
    private String firstName;
    private String lastName;
    private String telephone;     // ← NEVER rename this to phoneNumber

    // SAFE: Add new fields — old callers ignore fields they don't know
    private String address;       // ← new, safe to add
    private String city;          // ← new, safe to add

    // SAFE: Add phoneNumber as ALIAS alongside telephone
    // Old callers use telephone, new callers use phoneNumber
    private String phoneNumber;   // ← new alias

    // In your service method, set BOTH
    // owner.setTelephone(dbTelephone);
    // owner.setPhoneNumber(dbTelephone);  // same value, both fields present
}
```

### If You Must Remove/Rename — Use API Versioning

```java
// Keep OLD endpoint exactly as is
@GetMapping("/owners/{id}")
public OwnerResponseV1 getOwner(@PathVariable int id) {
    Owner owner = ownerRepo.findById(id).orElseThrow();
    return mapToV1(owner);  // old structure, telephone field
}

// Add NEW versioned endpoint for new callers
@GetMapping("/v2/owners/{id}")
public OwnerResponseV2 getOwnerV2(@PathVariable int id) {
    Owner owner = ownerRepo.findById(id).orElseThrow();
    return mapToV2(owner);  // new structure, phoneNumber field
}
```

```java
// V1 response — never change this
public class OwnerResponseV1 {
    private Integer id;
    private String firstName;
    private String lastName;
    private String telephone;    // old field name
}

// V2 response — new structure for new callers
public class OwnerResponseV2 {
    private Integer id;
    private String firstName;
    private String lastName;
    private String phoneNumber;  // renamed field
    private String address;      // new field
    private String city;         // new field
}
```

**Migration path:**
```
Week 1: Deploy V2 endpoint alongside V1
Week 2: Update all callers (API Gateway, other services) to use V2
Week 3: Mark V1 as @Deprecated
Week 6: Remove V1 (after confirming zero traffic)
```

---

## PART 2: Safe Database Changes — The Schema Problem

### The Most Dangerous Thing in Microservices

```
You have running service → reading column "telephone"
You rename column to "phone_number" in DB
→ Service crashes immediately on next query
→ All services calling you get 500 errors
→ Production is down
```

### Use Expand-Contract Pattern (The Only Safe Way)

**Scenario:** Rename `telephone` to `phone_number` in owners table

```
PHASE 1 — EXPAND (add new, keep old)
PHASE 2 — MIGRATE (copy data)  
PHASE 3 — SWITCH (code uses new column)
PHASE 4 — CONTRACT (remove old column)

Each phase = separate deployment
Never do all 4 in one deployment
```

**Use Flyway for DB migrations (already in most Spring Boot projects):**

```
src/main/resources/db/migration/
  V1__initial_schema.sql
  V2__add_phone_number_column.sql      ← Phase 1
  V3__copy_telephone_to_phone_number.sql ← Phase 2
  V4__make_phone_number_not_null.sql   ← Phase 3 (after code switch)
  V5__drop_telephone_column.sql        ← Phase 4 (weeks later)
```

**Phase 1 — V2__add_phone_number_column.sql:**
```sql
-- ADD new column, keep old column
-- Service still reads/writes telephone (nothing breaks)
ALTER TABLE owners ADD COLUMN phone_number VARCHAR(20);
```

**Phase 2 — V3__copy_telephone_to_phone_number.sql:**
```sql
-- Copy existing data to new column
UPDATE owners SET phone_number = telephone WHERE phone_number IS NULL;
```

**Phase 2 — Update Java entity to write to BOTH columns:**
```java
@Entity
@Table(name = "owners")
public class Owner {

    // Keep old column — still read by existing code
    @Column(name = "telephone")
    private String telephone;

    // New column — start writing to this too
    @Column(name = "phone_number")
    private String phoneNumber;

    // Synchronize them automatically
    public void setTelephone(String telephone) {
        this.telephone = telephone;
        this.phoneNumber = telephone;  // write to both
    }

    public void setPhoneNumber(String phoneNumber) {
        this.phoneNumber = phoneNumber;
        this.telephone = phoneNumber;  // write to both
    }
}
```

**Phase 3 — Switch code to use phoneNumber, deploy:**
```java
// After deploying code that uses phoneNumber:
// Run V4 migration to make phone_number NOT NULL
```

**V4__make_phone_number_not_null.sql:**
```sql
ALTER TABLE owners MODIFY phone_number VARCHAR(20) NOT NULL;
```

**Phase 4 — V5__drop_telephone_column.sql (weeks later):**
```sql
-- ONLY after confirming zero code references telephone
ALTER TABLE owners DROP COLUMN telephone;
```

---

## PART 3: Safe Event Changes — The Message Contract Problem

### The Problem

```
Appointments Service publishes:
{
  "eventType": "APPOINTMENT_BOOKED",
  "appointmentId": 123,
  "ownerId": 1,
  "petId": 5,
  "date": "2026-06-15"
}

Notification Service consumes this event and sends email.
Analytics Service consumes this event to update dashboards.

You want to add vetId and reason to the event.
You want to rename date → appointmentDate.
```

### Safe Event Evolution

**Rule: Events are append-only contracts. Add fields, never remove or rename.**

```java
// EXISTING event — never change field names
public class AppointmentBookedEvent {
    private Integer appointmentId;
    private Integer ownerId;
    private Integer petId;
    private LocalDate date;           // ← NEVER rename this

    // SAFE: Add new fields — consumers that don't know them just ignore them
    private Integer vetId;            // ← new
    private String reason;            // ← new
    private LocalDate appointmentDate;// ← new alias alongside 'date'
}
```

```java
// Publisher — set all fields including the new ones
AppointmentBookedEvent event = AppointmentBookedEvent.builder()
    .appointmentId(saved.getId())
    .ownerId(saved.getOwnerId())
    .petId(saved.getPetId())
    .date(saved.getApptDate())            // old field — keep it
    .appointmentDate(saved.getApptDate()) // new alias — add it
    .vetId(saved.getVetId())              // new field
    .reason(saved.getReason())            // new field
    .build();
```

```java
// Old consumer (Notification Service) — still works, ignores unknown fields
// Jackson's ObjectMapper ignores unknown fields by default
@RabbitListener(queues = "appointment.notifications")
public void handleEvent(AppointmentBookedEvent event) {
    // Uses event.getDate() — still works
    // Ignores event.getVetId() — doesn't know about it, doesn't care
    emailService.send(event.getOwnerId(), event.getDate());
}
```

**Ensure consumers ignore unknown fields:**
```java
// In each consumer service's config
@Bean
public ObjectMapper objectMapper() {
    return new ObjectMapper()
        .configure(DeserializationFeature.FAIL_ON_UNKNOWN_PROPERTIES, false);
        // ↑ CRITICAL: new fields in events won't crash old consumers
}
```

---

## PART 4: Adding a Complete Complex Feature — End to End

### Feature: "Appointment Reminder System"

```
What it does:
  - 24 hours before any appointment, send reminder to owner
  - Track whether reminder was sent (don't send twice)
  - Owner can opt out of reminders

Spans:
  - Appointments Service  (has appointment data)
  - Notification Service  (sends emails)
  - Customers Service     (has owner contact info)
  - New: Scheduler        (triggers reminder job)
```

### Step 1 — Design Without Touching Anything

```
Write this BEFORE opening any source file:

New things needed:
  1. DB: reminders table in appointments DB (tracks sent reminders)
  2. DB: opt_out column in customers DB owners table  
  3. API: PATCH /owners/{id}/notifications (opt-out endpoint)
  4. Service: ReminderScheduler (runs every hour, finds due reminders)
  5. Event: REMINDER_DUE (appointments → notification service)
  6. Handler: NotificationService listens for REMINDER_DUE event

Existing things I must NOT change:
  - All existing /owners/** endpoints in customers-service
  - All existing /appointments/** endpoints in appointments-service  
  - Existing APPOINTMENT_BOOKED event structure
  - Existing notification email handler
```

### Step 2 — Safe DB Changes (Expand Phase)

**In Customers Service — add opt-out:**

```sql
-- V6__add_reminder_opt_out.sql
-- Safe: adding nullable column with default, nothing breaks
ALTER TABLE owners 
ADD COLUMN reminder_opt_out BOOLEAN NOT NULL DEFAULT FALSE;
```

**In Appointments Service — add reminders tracking table:**

```sql
-- V4__create_reminders_table.sql
-- Safe: brand new table, zero impact on existing tables
CREATE TABLE appointment_reminders (
    id              INTEGER GENERATED BY DEFAULT AS IDENTITY PRIMARY KEY,
    appointment_id  INTEGER NOT NULL REFERENCES appointments(id),
    scheduled_at    TIMESTAMP NOT NULL,
    sent_at         TIMESTAMP,                    -- null = not yet sent
    status          VARCHAR(20) DEFAULT 'PENDING', -- PENDING, SENT, FAILED, OPTED_OUT
    created_at      TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Step 3 — New API Endpoint (Additive Only)

**In Customers Service — add opt-out endpoint:**

```java
// ADD to existing OwnerResource controller
// DO NOT modify any existing methods

@PatchMapping("/{ownerId}/notifications")
public ResponseEntity<Void> updateNotificationPreference(
        @PathVariable int ownerId,
        @RequestBody NotificationPreferenceRequest request) {

    Owner owner = ownerRepo.findById(ownerId)
        .orElseThrow(() -> new ResourceNotFoundException("Owner not found"));

    owner.setReminderOptOut(request.isOptOut());
    ownerRepo.save(owner);

    return ResponseEntity.noContent().build();  // 204 No Content
}
```

```java
// New DTO — new file, nothing existing affected
public class NotificationPreferenceRequest {
    private boolean optOut;
    // getter, setter
}
```

**Add route in API Gateway — additive change:**

```yaml
# ADD to existing gateway routes — don't touch existing routes
- id: customers-notifications
  uri: lb://customers-service
  predicates:
    - Path=/api/customer/*/notifications
  filters:
    - StripPrefix=2
```

### Step 4 — The Scheduler (New Component)

**Add to Appointments Service:**

```java
// NEW FILE — zero impact on existing code
@Component
public class ReminderScheduler {

    private final AppointmentRepository appointmentRepo;
    private final ReminderRepository reminderRepo;
    private final RabbitTemplate rabbitTemplate;

    // Runs every hour
    @Scheduled(fixedDelay = 3600000)
    public void scheduleReminders() {
        LocalDateTime now = LocalDateTime.now();
        LocalDateTime reminderWindow = now.plusHours(24);

        // Find appointments in next 24 hours with no reminder sent
        List<Appointment> upcoming = appointmentRepo
            .findUpcomingWithoutReminder(now, reminderWindow);

        upcoming.forEach(this::processReminder);
    }

    private void processReminder(Appointment appointment) {
        // Use DB as lock — only one instance processes each appointment
        boolean created = reminderRepo.createIfNotExists(
            appointment.getId(),
            LocalDateTime.now()
        );

        if (!created) return;  // Another instance already handling it

        try {
            ReminderDueEvent event = ReminderDueEvent.builder()
                .appointmentId(appointment.getId())
                .ownerId(appointment.getOwnerId())
                .petId(appointment.getPetId())
                .appointmentDate(appointment.getApptDate())
                .build();

            rabbitTemplate.convertAndSend(
                "appointments.exchange",
                "reminder.due",
                event
            );

            reminderRepo.markSent(appointment.getId());

        } catch (Exception e) {
            reminderRepo.markFailed(appointment.getId());
            log.error("Failed to publish reminder for appt {}", appointment.getId(), e);
        }
    }
}
```

**New repository for reminders:**

```java
// NEW FILE
public interface ReminderRepository extends JpaRepository<AppointmentReminder, Integer> {

    @Query("""
        SELECT a FROM Appointment a
        WHERE a.apptDate BETWEEN :from AND :to
        AND a.id NOT IN (
            SELECT r.appointmentId FROM AppointmentReminder r
            WHERE r.status IN ('SENT', 'OPTED_OUT')
        )
    """)
    List<Appointment> findUpcomingWithoutReminder(
        @Param("from") LocalDateTime from,
        @Param("to") LocalDateTime to
    );

    // Atomic insert — returns false if row already exists (prevents duplicates)
    @Modifying
    @Query(value = """
        INSERT INTO appointment_reminders (appointment_id, scheduled_at, status)
        SELECT :apptId, :scheduledAt, 'PENDING'
        WHERE NOT EXISTS (
            SELECT 1 FROM appointment_reminders WHERE appointment_id = :apptId
        )
    """, nativeQuery = true)
    int createIfNotExists(@Param("apptId") int apptId,
                          @Param("scheduledAt") LocalDateTime scheduledAt);
}
```

### Step 5 — Notification Service Handles New Event

**Add new listener — don't touch existing listener:**

```java
// EXISTING listener — DO NOT TOUCH
@RabbitListener(queues = "appointment.notifications")
public void handleAppointmentBooked(AppointmentBookedEvent event) {
    emailService.sendBookingConfirmation(event.getOwnerId(), event.getDate());
}

// NEW listener — added below, completely separate
@RabbitListener(queues = "appointment.reminders")
public void handleReminderDue(ReminderDueEvent event) {

    // Check opt-out preference by calling customers-service
    boolean optedOut = customersClient.isReminderOptOut(event.getOwnerId());

    if (optedOut) {
        // Publish back so appointments-service can mark it opted out
        rabbitTemplate.convertAndSend(
            "appointments.exchange",
            "reminder.opted.out",
            new ReminderOptedOutEvent(event.getAppointmentId())
        );
        return;
    }

    Owner owner = customersClient.getOwner(event.getOwnerId());
    emailService.sendReminder(owner.getEmail(), event.getAppointmentDate());
}
```

### Step 6 — Enable Scheduling (One-Line Change)

```java
// In AppointmentsServiceApplication.java
// This is the ONLY change to an existing file

@SpringBootApplication
@EnableDiscoveryClient
@EnableScheduling           // ← ADD this one annotation
public class AppointmentsServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(AppointmentsServiceApplication.class, args);
    }
}
```

---

## PART 5: Testing the Complex Feature

### Test the Scheduler in Isolation

```java
@ExtendWith(MockitoExtension.class)
class ReminderSchedulerTest {

    @Mock AppointmentRepository appointmentRepo;
    @Mock ReminderRepository reminderRepo;
    @Mock RabbitTemplate rabbitTemplate;

    @InjectMocks ReminderScheduler scheduler;

    @Test
    void scheduleReminders_PublishesEventForEachUpcoming() {
        // ARRANGE
        Appointment appt1 = buildAppointment(1, 1, LocalDate.now().plusHours(12));
        Appointment appt2 = buildAppointment(2, 2, LocalDate.now().plusHours(20));

        when(appointmentRepo.findUpcomingWithoutReminder(any(), any()))
            .thenReturn(Arrays.asList(appt1, appt2));

        when(reminderRepo.createIfNotExists(anyInt(), any()))
            .thenReturn(1);  // successfully created

        // ACT
        scheduler.scheduleReminders();

        // ASSERT — event published for each appointment
        verify(rabbitTemplate, times(2))
            .convertAndSend(eq("appointments.exchange"), eq("reminder.due"), any());
    }

    @Test
    void scheduleReminders_AlreadyProcessed_DoesNotPublishAgain() {
        Appointment appt = buildAppointment(1, 1, LocalDate.now().plusHours(12));

        when(appointmentRepo.findUpcomingWithoutReminder(any(), any()))
            .thenReturn(List.of(appt));

        when(reminderRepo.createIfNotExists(anyInt(), any()))
            .thenReturn(0);  // row already exists — another instance got it

        scheduler.scheduleReminders();

        // ASSERT — nothing published since another instance handled it
        verify(rabbitTemplate, never())
            .convertAndSend(anyString(), anyString(), any());
    }
}
```

### Regression Test — Existing Features Untouched

```java
@SpringBootTest
@AutoConfigureMockMvc
class ExistingAppointmentFeaturesRegressionTest {

    @Autowired MockMvc mockMvc;

    @Test
    void existingBookAppointment_StillWorks() throws Exception {
        mockMvc.perform(post("/appointments")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                      "ownerId": 1,
                      "petId": 1,
                      "vetId": 1,
                      "apptDate": "2026-06-15",
                      "reason": "Checkup"
                    }
                """))
            .andExpect(status().isCreated());
    }

    @Test
    void existingGetAppointment_StillWorks() throws Exception {
        mockMvc.perform(get("/appointments/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.appointmentId").exists());
    }

    @Test
    void existingOwnerEndpoints_UnchangedByNewOptOutEndpoint() throws Exception {
        // Verify existing owner endpoints completely unaffected
        mockMvc.perform(get("/owners/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.firstName").exists())
            .andExpect(jsonPath("$.telephone").exists()); // old field still present
    }
}
```

---

## PART 6: The Deployment Safety Checklist

```
BEFORE merging your feature branch:

□ API CONTRACTS
  □ No existing endpoint URLs changed
  □ No existing response fields removed or renamed
  □ No existing request fields made newly required
  □ New endpoints added at new paths

□ DATABASE
  □ Only additive migrations (no DROP COLUMN, no RENAME COLUMN)
  □ New columns have DEFAULT values or are nullable
  □ Flyway migration files numbered sequentially
  □ Migration tested on a copy of production data

□ EVENTS
  □ No existing event fields renamed or removed
  □ New event fields are optional (not required by consumers)
  □ Consumers configured with FAIL_ON_UNKNOWN_PROPERTIES=false
  □ New event types on new routing keys (not replacing old ones)

□ TESTS
  □ Unit tests for all new business logic
  □ Integration tests for new DB queries
  □ Regression tests for ALL existing endpoints still pass
  □ Contract tests pass (if using Pact or Spring Cloud Contract)

□ DEPLOYMENT ORDER
  □ Infrastructure changes first (DB migrations)
  □ Then producer services (services that publish events)
  □ Then consumer services (services that consume events)
  □ Then API Gateway route changes last
```

---

## The Mental Model — One Paragraph

Every microservice is a **contract with the outside world** — its API, its events, its database schema. Your job when adding features is to **extend the contract without breaking it**. Old callers must always keep working. You do this by adding new things (endpoints, fields, events, tables) rather than changing existing ones, using versioning when changes are unavoidable, and making DB changes in slow deliberate phases. Test the new feature AND regression-test the old features after every change.


---------


# Adding AI/ML Features to Microservices Through First Principles

## Start With The Core Question: What Problem Are You Actually Solving?

```
Most developers think: "Let me add AI to this project"
First principles thinking: "What PAIN exists that 
                            intelligence could remove?"

Wrong approach: "Add a chatbot because AI is trendy"
Right approach: "Owners struggle to describe symptoms accurately
                 → AI can help translate vague descriptions 
                   into structured medical observations"

The question is never "how do I add AI?"
The question is always "what decision or task 
                        is currently done poorly 
                        that a model could do better?"
```

---

## Step 1: Audit the Project for AI Opportunities

Go through every user interaction in PetClinic and ask:

```
For each feature ask 3 questions:

1. Is there a DECISION being made manually?
   → AI can assist or automate decisions

2. Is there UNSTRUCTURED DATA being entered?
   → AI can extract structure from it

3. Is there a PREDICTION that would add value?
   → AI can predict outcomes from patterns
```

**PetClinic audit:**

```
Feature                  Pain Point                    AI Solution
────────────────────────────────────────────────────────────────────────
Owner describes symptoms  Vague text, vet wastes time   Symptom classifier
                          reading unclear notes          → structured urgency score

Visit history             No pattern detection          Anomaly detector  
                          across all visits             → flag unusual patterns

Appointment booking       Random vet assignment         Smart vet matcher
                          based on availability only    → match by speciality + history

Search for owners         Exact match only              Semantic search
                          "Jon" doesn't find "John"     → fuzzy/intent search

Vet notes                 Free text, hard to query      Named entity extraction
                          "limping on left hind leg"    → {symptom, location, severity}

Appointment no-shows      No prediction, just happens   No-show predictor
                          slots wasted                  → flag high-risk bookings
```

**Pick ONE for each service. Start with the highest value, lowest complexity.**

For this guide we'll build **3 real features:**
```
1. Symptom Urgency Classifier    → Visits Service
2. Smart Vet Matcher             → Appointments Service  
3. Semantic Owner Search         → Customers Service
```

---

## Step 2: Understand What AI Actually IS in Code

```
Before adding any AI, understand what you're actually integrating:

TYPE 1: EXTERNAL AI API (OpenAI, Anthropic, Gemini)
─────────────────────────────────────────────────────
  You send text → their server runs model → you get response
  
  Pros:  No infrastructure, state of the art models, fast to build
  Cons:  Cost per call, latency, data leaves your system, dependency
  
  Use when: Complex reasoning, language understanding, generation
  Examples: Symptom analysis, generating summaries, chat


TYPE 2: EMBEDDED ML MODEL (own model in your service)
──────────────────────────────────────────────────────
  Your service loads a model file → runs inference locally
  
  Pros:  No external calls, fast, free after training, data stays local
  Cons:  Need training data, need to train/maintain model, more complex
  
  Use when: Classification, scoring, pattern matching on YOUR data
  Examples: No-show predictor trained on your history, vet matcher


TYPE 3: VECTOR DATABASE + EMBEDDINGS (semantic search)
───────────────────────────────────────────────────────
  Convert text to numbers (vectors) → store in vector DB
  → find similar items by mathematical distance
  
  Pros:  Powerful similarity search, works on your domain data
  Cons:  Need embedding model, vector DB infrastructure
  
  Use when: Search, recommendations, finding similar records
  Examples: Semantic owner search, similar past cases
```

---

## Step 3: Architecture — Where Does AI Live?

```
Option A: AI logic inside each microservice
────────────────────────────────────────────
  Visits Service ──calls OpenAI──▶ classifies symptoms
  Customers Service ──calls OpenAI──▶ semantic search

  Problem: Every service manages AI keys, retry logic, rate limits
           Duplicated AI infrastructure across services


Option B: Dedicated AI Service (Recommended)
─────────────────────────────────────────────
  All AI logic lives in one service
  Other services call it like any other microservice

  ┌─────────────────┐     ┌─────────────────┐
  │ Visits Service  │────▶│                 │────▶ OpenAI API
  └─────────────────┘     │   AI Service    │
  ┌─────────────────┐     │   :8085         │────▶ Local ML Model
  │Appt Service     │────▶│                 │
  └─────────────────┘     │                 │────▶ Vector DB
  ┌─────────────────┐     └─────────────────┘
  │Customers Service│────▶
  └─────────────────┘

  Benefits:
  - One place manages API keys, rate limits, caching
  - AI models swappable without touching other services
  - Easy to add circuit breaker on AI calls
  - Other services don't care HOW AI works, just get results
```

---

## Step 4: Build the AI Service

### Project Structure

```
spring-petclinic-ai-service/
├── pom.xml
├── src/main/java/.../ai/
│   ├── AiServiceApplication.java
│   │
│   ├── symptom/
│   │   ├── SymptomAnalysisController.java
│   │   ├── SymptomAnalysisService.java
│   │   └── dto/
│   │       ├── SymptomAnalysisRequest.java
│   │       └── SymptomAnalysisResponse.java
│   │
│   ├── vetmatch/
│   │   ├── VetMatchController.java
│   │   ├── VetMatchService.java
│   │   └── dto/...
│   │
│   ├── search/
│   │   ├── SemanticSearchController.java
│   │   ├── SemanticSearchService.java
│   │   └── dto/...
│   │
│   └── config/
│       ├── OpenAiConfig.java
│       └── CacheConfig.java
```

### pom.xml for AI Service

```xml
<dependencies>
    <!-- Standard microservice dependencies -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>

    <!-- Spring AI — unified interface to OpenAI, Anthropic, etc -->
    <dependency>
        <groupId>org.springframework.ai</groupId>
        <artifactId>spring-ai-openai-spring-boot-starter</artifactId>
        <version>1.0.0</version>
    </dependency>

    <!-- Caching — avoid calling OpenAI for same input twice -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-cache</artifactId>
    </dependency>

    <!-- Resilience — circuit breaker for AI API calls -->
    <dependency>
        <groupId>io.github.resilience4j</groupId>
        <artifactId>resilience4j-spring-boot3</artifactId>
    </dependency>
</dependencies>
```

### application.yml

```yaml
server:
  port: 8085

spring:
  application:
    name: ai-service

  ai:
    openai:
      api-key: ${OPENAI_API_KEY}    # from environment variable, never hardcode
      chat:
        options:
          model: gpt-4o-mini        # cheap, fast, good enough for classification
          temperature: 0.1          # low = consistent structured responses

  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=1000,expireAfterWrite=1h

resilience4j:
  circuitbreaker:
    instances:
      openai:
        failureRateThreshold: 50
        waitDurationInOpenState: 30s
        fallbackMethod: getFallbackResponse
```

---

## Feature 1: Symptom Urgency Classifier

### The First Principles Design

```
Problem:  Owner writes "my dog seems off today and didn't eat"
          Vet has no idea if this is urgent or routine

What we need:
  Input:  Free text symptom description + pet species + age
  Output: {
    urgencyLevel: "HIGH" | "MEDIUM" | "LOW",
    urgencyScore: 0-100,
    identifiedSymptoms: ["lethargy", "loss of appetite"],
    recommendedAction: "See vet within 24 hours",
    reasoning: "Loss of appetite combined with lethargy..."
  }

How AI solves it:
  The model has seen millions of medical texts
  It understands "seems off" = lethargy
  It understands that combination matters
  We don't need to code all these rules — model knows them
```

### The DTO Contracts

```java
// Request — what other services send us
public class SymptomAnalysisRequest {
    @NotBlank
    private String description;     // "my dog seems off and won't eat"

    @NotBlank
    private String petSpecies;      // "dog", "cat", "bird"

    private Integer petAgeYears;    // affects what's normal
    private Integer petId;          // for context, optional
}

// Response — what we return
public class SymptomAnalysisResponse {

    public enum UrgencyLevel { HIGH, MEDIUM, LOW, UNKNOWN }

    private UrgencyLevel urgencyLevel;
    private int urgencyScore;           // 0-100
    private List<String> identifiedSymptoms;
    private String recommendedAction;
    private String reasoning;
    private boolean aiGenerated;        // always flag AI responses
    private String disclaimer;          // always include medical disclaimer
}
```

### The Service

```java
@Service
@Slf4j
public class SymptomAnalysisService {

    private final ChatClient chatClient;   // Spring AI's unified chat interface

    public SymptomAnalysisService(ChatClient.Builder builder) {
        this.chatClient = builder.build();
    }

    @Cacheable(value = "symptomAnalysis",
               key = "#request.description + #request.petSpecies")
    @CircuitBreaker(name = "openai", fallbackMethod = "getFallback")
    public SymptomAnalysisResponse analyze(SymptomAnalysisRequest request) {

        // The prompt is your most important code
        // Think of it as a function specification for the AI
        String prompt = buildPrompt(request);

        String aiResponse = chatClient.prompt()
            .user(prompt)
            .call()
            .content();

        return parseResponse(aiResponse, request);
    }

    private String buildPrompt(SymptomAnalysisRequest request) {
        // CRITICAL: Ask for JSON output so we can parse it reliably
        // Give it a clear schema to follow
        // Give it context about what you need
        return """
            You are a veterinary triage assistant.
            Analyze the following pet symptom description and respond
            ONLY with valid JSON matching this exact schema:

            {
              "urgencyLevel": "HIGH" | "MEDIUM" | "LOW",
              "urgencyScore": <integer 0-100>,
              "identifiedSymptoms": [<string>, ...],
              "recommendedAction": <string>,
              "reasoning": <string, max 100 words>
            }

            Urgency rules:
            - HIGH (70-100): life-threatening, see vet immediately
            - MEDIUM (30-69): needs attention within 24-48 hours
            - LOW (0-29): monitor at home, routine checkup if persists

            Pet information:
            - Species: %s
            - Age: %s years
            - Description: "%s"

            Respond ONLY with the JSON object. No explanation outside JSON.
            """.formatted(
                request.getPetSpecies(),
                request.getPetAgeYears() != null ? request.getPetAgeYears() : "unknown",
                request.getDescription()
            );
    }

    private SymptomAnalysisResponse parseResponse(String aiJson,
                                                   SymptomAnalysisRequest req) {
        try {
            // Parse the JSON response from AI
            ObjectMapper mapper = new ObjectMapper();
            JsonNode node = mapper.readTree(aiJson);

            SymptomAnalysisResponse response = new SymptomAnalysisResponse();
            response.setUrgencyLevel(
                SymptomAnalysisResponse.UrgencyLevel.valueOf(
                    node.get("urgencyLevel").asText("UNKNOWN")
                )
            );
            response.setUrgencyScore(node.get("urgencyScore").asInt(0));
            response.setRecommendedAction(node.get("recommendedAction").asText());
            response.setReasoning(node.get("reasoning").asText());

            List<String> symptoms = new ArrayList<>();
            node.get("identifiedSymptoms").forEach(s -> symptoms.add(s.asText()));
            response.setIdentifiedSymptoms(symptoms);

            response.setAiGenerated(true);
            response.setDisclaimer(
                "This is AI-assisted triage only. " +
                "Always consult a qualified veterinarian for medical decisions."
            );

            return response;

        } catch (Exception e) {
            log.error("Failed to parse AI response: {}", aiJson, e);
            return getFallback(req, e);
        }
    }

    // Fallback when OpenAI is down or fails
    // NEVER let AI unavailability break your core feature
    public SymptomAnalysisResponse getFallback(SymptomAnalysisRequest req,
                                                Exception ex) {
        log.warn("AI analysis unavailable, returning safe fallback: {}", ex.getMessage());

        SymptomAnalysisResponse fallback = new SymptomAnalysisResponse();
        fallback.setUrgencyLevel(SymptomAnalysisResponse.UrgencyLevel.UNKNOWN);
        fallback.setUrgencyScore(50);  // middle ground when uncertain
        fallback.setRecommendedAction("Please consult your veterinarian to assess your pet.");
        fallback.setReasoning("Automated analysis temporarily unavailable.");
        fallback.setAiGenerated(false);
        fallback.setDisclaimer("Manual review required. AI analysis currently unavailable.");
        return fallback;
    }
}
```

### The Controller

```java
@RestController
@RequestMapping("/ai/symptoms")
public class SymptomAnalysisController {

    private final SymptomAnalysisService service;

    @PostMapping("/analyze")
    public ResponseEntity<SymptomAnalysisResponse> analyze(
            @Valid @RequestBody SymptomAnalysisRequest request) {

        SymptomAnalysisResponse response = service.analyze(request);
        return ResponseEntity.ok(response);
    }
}
```

---

## Feature 2: Smart Vet Matcher

### First Principles Design

```
Problem:  Appointments assigned to random available vet
          Owner's pet has diabetes → random vet assigned
          → Better vet: one who has seen diabetic pets before

What we need:
  Input:  Pet medical history + symptoms + available vets
  Output: Ranked list of vets with match reasoning

This is a SCORING problem, not a generation problem:
  For each vet, score how well they match this case
  Ranking based on:
    - Speciality match to symptoms
    - History of treating this pet
    - History with this species
    - Current workload
```

### The Feign Client to Fetch Data

```java
// AI Service calls other services to get context
@FeignClient(name = "vets-service")
public interface VetsServiceClient {
    @GetMapping("/vets")
    List<VetDto> getAllVets();
}

@FeignClient(name = "visits-service")
public interface VisitsServiceClient {
    @GetMapping("/visits/pet/{petId}")
    List<VisitDto> getVisitHistory(@PathVariable int petId);
}
```

### The Service

```java
@Service
public class VetMatchService {

    private final ChatClient chatClient;
    private final VetsServiceClient vetsClient;
    private final VisitsServiceClient visitsClient;

    public VetMatchResponse findBestVets(VetMatchRequest request) {

        // Step 1: Get all available vets (real data)
        List<VetDto> availableVets = vetsClient.getAllVets()
            .stream()
            .filter(v -> request.getAvailableVetIds().contains(v.getId()))
            .collect(toList());

        // Step 2: Get pet's visit history (real data)
        List<VisitDto> history = visitsClient.getVisitHistory(request.getPetId());

        // Step 3: Ask AI to rank vets using all this context
        String prompt = buildMatchingPrompt(request, availableVets, history);

        String aiResponse = chatClient.prompt()
            .user(prompt)
            .call()
            .content();

        return parseMatchResponse(aiResponse, availableVets);
    }

    private String buildMatchingPrompt(VetMatchRequest request,
                                        List<VetDto> vets,
                                        List<VisitDto> history) {

        // Serialize context as JSON for the AI
        String vetsJson = serializeVets(vets);
        String historyJson = serializeHistory(history);

        return """
            You are a veterinary appointment scheduling assistant.
            Rank the available vets for this appointment.

            Pet information:
            - Species: %s
            - Age: %d years
            - Current symptoms: %s

            Visit history (last 10 visits):
            %s

            Available vets:
            %s

            Respond ONLY with JSON:
            {
              "rankedVets": [
                {
                  "vetId": <integer>,
                  "matchScore": <integer 0-100>,
                  "matchReason": <string, max 50 words>,
                  "highlights": [<string>, ...]
                }
              ]
            }

            Ranking criteria:
            1. Speciality relevance to current symptoms
            2. Prior treatment of THIS pet (continuity of care)
            3. Experience with this species
            Order by matchScore descending.
            """.formatted(
                request.getPetSpecies(),
                request.getPetAgeYears(),
                request.getSymptomDescription(),
                historyJson,
                vetsJson
            );
    }
}
```

---

## Feature 3: Semantic Owner Search

### First Principles Design

```
Problem:  Current search: exact match on lastName
          "Jon" finds nothing if owner is "John"
          "the owner with the labrador" → no results

What we need:
  Convert owner records to vectors (numerical representations)
  Store in vector DB
  Convert search query to vector
  Find owners whose vector is closest to query vector

This is SIMILARITY SEARCH — not keyword matching
```

### How Vectors Work (First Principles)

```
Normal string:  "John Davis, telephone: 6085551023, city: Madison"

Vector:         [0.23, -0.87, 0.44, 0.12, -0.33, ...]
                 ↑ 1536 numbers that encode MEANING

Similar meaning = vectors close together in space
"Jon Davies Madison" → vector close to "John Davis Madison"
→ search finds the right owner

How similarity is measured:
  cosine_similarity(query_vector, owner_vector)
  → 1.0 = identical meaning
  → 0.0 = completely unrelated
  → rank owners by this score
```

### Embedding Service

```java
@Service
public class SemanticSearchService {

    private final EmbeddingModel embeddingModel;  // Spring AI
    private final VectorStore vectorStore;         // stores embeddings

    @FeignClient(name = "customers-service")
    interface CustomersClient {
        @GetMapping("/owners")
        List<OwnerDto> getAllOwners();
    }

    // Called when owners are created/updated
    // Index owner data as vectors
    public void indexOwner(OwnerDto owner) {
        // Create searchable text from owner's structured data
        String ownerText = buildOwnerText(owner);

        // Convert to vector using embedding model
        Document doc = new Document(
            ownerText,
            Map.of(
                "ownerId", owner.getId(),
                "firstName", owner.getFirstName(),
                "lastName", owner.getLastName()
            )
        );

        vectorStore.add(List.of(doc));
    }

    public List<OwnerSearchResult> search(String query, int topK) {

        // Convert query to vector and find similar owners
        List<Document> similar = vectorStore.similaritySearch(
            SearchRequest.query(query)
                         .withTopK(topK)
                         .withSimilarityThreshold(0.6)  // min 60% match
        );

        return similar.stream()
            .map(doc -> new OwnerSearchResult(
                (Integer) doc.getMetadata().get("ownerId"),
                (String) doc.getMetadata().get("firstName"),
                (String) doc.getMetadata().get("lastName"),
                doc.getScore()  // similarity score 0-1
            ))
            .collect(toList());
    }

    private String buildOwnerText(OwnerDto owner) {
        // Richer text = better semantic understanding
        return String.format(
            "Owner %s %s lives in %s at %s. " +
            "Pets: %s. Phone: %s.",
            owner.getFirstName(),
            owner.getLastName(),
            owner.getCity(),
            owner.getAddress(),
            owner.getPets().stream()
                 .map(p -> p.getName() + " the " + p.getType())
                 .collect(joining(", ")),
            owner.getTelephone()
        );
    }
}
```

---

## Step 5: How Other Services Consume AI Service

### Add Feign Client in Visits Service

```java
// In Visits Service — AI is just another service to call
@FeignClient(name = "ai-service")
public interface AiServiceClient {

    @PostMapping("/ai/symptoms/analyze")
    SymptomAnalysisResponse analyzeSymptoms(
        @RequestBody SymptomAnalysisRequest request
    );
}
```

### Use It in Visit Creation

```java
@Service
public class VisitService {

    private final VisitRepository visitRepo;
    private final AiServiceClient aiClient;  // injected Feign client

    public Visit createVisit(CreateVisitRequest request) {

        // Step 1: Save the visit (core feature — must always work)
        Visit visit = new Visit();
        visit.setPetId(request.getPetId());
        visit.setDescription(request.getDescription());
        visit.setVisitDate(LocalDate.now());
        Visit saved = visitRepo.save(visit);

        // Step 2: Try AI analysis (enhancement — optional)
        // CRITICAL: Wrapped in try-catch
        // If AI fails, visit creation still succeeds
        try {
            SymptomAnalysisRequest aiRequest = new SymptomAnalysisRequest();
            aiRequest.setDescription(request.getDescription());
            aiRequest.setPetSpecies(request.getPetSpecies());
            aiRequest.setPetAgeYears(request.getPetAgeYears());

            SymptomAnalysisResponse analysis = aiClient.analyzeSymptoms(aiRequest);

            // Save AI analysis alongside visit (in separate table — don't pollute visits table)
            saveVisitAnalysis(saved.getId(), analysis);

        } catch (Exception e) {
            // AI failure is logged but NEVER bubbles up to user
            log.warn("AI analysis failed for visit {}, continuing without it: {}",
                saved.getId(), e.getMessage());
        }

        return saved;
    }
}
```

---

## Step 6: The Critical Rules for AI in Production

### Rule 1: AI is Always Optional Enhancement

```java
// WRONG — AI failure breaks core feature
public Visit createVisit(CreateVisitRequest req) {
    SymptomAnalysis analysis = aiClient.analyze(req); // if this throws → visit not created
    Visit visit = visitRepo.save(buildVisit(req, analysis));
    return visit;
}

// RIGHT — core feature works regardless of AI
public Visit createVisit(CreateVisitRequest req) {
    Visit visit = visitRepo.save(buildVisit(req)); // always runs

    try {
        SymptomAnalysis analysis = aiClient.analyze(req);
        saveAnalysis(visit.getId(), analysis);
    } catch (Exception e) {
        log.warn("AI unavailable: {}", e.getMessage()); // log and move on
    }

    return visit; // always returns
}
```

### Rule 2: Always Store AI Inputs and Outputs

```sql
-- Never call AI without logging what went in and came out
CREATE TABLE ai_analysis_log (
    id              INTEGER PRIMARY KEY,
    service         VARCHAR(50),       -- 'symptom_analysis', 'vet_match'
    entity_id       INTEGER,           -- visit_id, appointment_id
    input_data      TEXT,              -- exact prompt/input sent
    output_data     TEXT,              -- exact response received
    model_used      VARCHAR(50),       -- 'gpt-4o-mini'
    latency_ms      INTEGER,           -- how long it took
    was_cached      BOOLEAN,
    was_fallback    BOOLEAN,           -- did it use fallback?
    created_at      TIMESTAMP
);
```

```java
@Service
public class AiLoggingService {

    private final AiAnalysisLogRepository logRepo;

    public void log(String service, Integer entityId,
                    String input, String output,
                    String model, long latencyMs,
                    boolean cached, boolean fallback) {

        AiAnalysisLog entry = new AiAnalysisLog();
        entry.setService(service);
        entry.setEntityId(entityId);
        entry.setInputData(input);
        entry.setOutputData(output);
        entry.setModelUsed(model);
        entry.setLatencyMs(latencyMs);
        entry.setWasCached(cached);
        entry.setWasFallback(fallback);
        logRepo.save(entry);
    }
}
```

**Why log everything:**
```
- Debug why AI gave a wrong answer
- Detect prompt injection attacks
- Measure accuracy over time
- Audit trail for medical-adjacent decisions
- Improve prompts using real input/output pairs
- Cost tracking (each API call costs money)
```

### Rule 3: Always Validate AI Output

```java
// AI can return anything — always validate before using
private SymptomAnalysisResponse parseAndValidate(String aiJson) {

    // Validation 1: Is it valid JSON?
    JsonNode node;
    try {
        node = mapper.readTree(aiJson);
    } catch (JsonProcessingException e) {
        log.error("AI returned invalid JSON: {}", aiJson);
        return getFallback();
    }

    // Validation 2: Are required fields present?
    if (!node.has("urgencyLevel") || !node.has("urgencyScore")) {
        log.error("AI response missing required fields: {}", aiJson);
        return getFallback();
    }

    // Validation 3: Is the score in valid range?
    int score = node.get("urgencyScore").asInt(-1);
    if (score < 0 || score > 100) {
        log.error("AI returned out-of-range score: {}", score);
        score = 50;  // safe default
    }

    // Validation 4: Is the urgency level a known value?
    String level = node.get("urgencyLevel").asText("UNKNOWN");
    if (!Set.of("HIGH", "MEDIUM", "LOW").contains(level)) {
        log.error("AI returned unknown urgency level: {}", level);
        level = "UNKNOWN";
    }

    // Build response with validated values
    SymptomAnalysisResponse response = new SymptomAnalysisResponse();
    response.setUrgencyScore(score);
    response.setUrgencyLevel(UrgencyLevel.valueOf(level));
    // ... rest of fields
    return response;
}
```

### Rule 4: Never Send Sensitive Data to External AI

```java
// WRONG — sending PII to OpenAI
String prompt = "Analyze symptoms for owner John Davis, 
                 phone 555-1234, address 123 Main St...";

// RIGHT — anonymize before sending
String prompt = "Analyze symptoms for pet (species: dog, age: 3 years). 
                 Symptoms: lethargy, not eating.";
// Owner name, phone, address NEVER leave your system
```

---

## Step 7: Testing AI Features

```java
@ExtendWith(MockitoExtension.class)
class SymptomAnalysisServiceTest {

    @Mock
    private ChatClient chatClient;

    @InjectMocks
    private SymptomAnalysisService service;

    @Test
    void analyze_ValidSymptoms_ReturnsHighUrgency() {
        // Mock the AI response — don't call real OpenAI in tests
        String mockAiResponse = """
            {
              "urgencyLevel": "HIGH",
              "urgencyScore": 85,
              "identifiedSymptoms": ["difficulty breathing", "blue gums"],
              "recommendedAction": "Emergency vet visit immediately",
              "reasoning": "Breathing difficulty with cyanosis is life-threatening"
            }
            """;

        when(chatClient.prompt()).thenReturn(mock(ChatClient.PromptSpec.class));
        // ... chain the mock calls

        SymptomAnalysisRequest request = new SymptomAnalysisRequest();
        request.setDescription("dog can't breathe, gums turning blue");
        request.setPetSpecies("dog");
        request.setPetAgeYears(5);

        SymptomAnalysisResponse response = service.analyze(request);

        assertEquals(UrgencyLevel.HIGH, response.getUrgencyLevel());
        assertTrue(response.getUrgencyScore() >= 70);
        assertNotNull(response.getDisclaimer()); // disclaimer always present
        assertTrue(response.isAiGenerated());
    }

    @Test
    void analyze_AiServiceDown_ReturnsFallback() {
        // Simulate OpenAI being down
        when(chatClient.prompt()).thenThrow(new RuntimeException("Connection refused"));

        SymptomAnalysisRequest request = new SymptomAnalysisRequest();
        request.setDescription("dog limping");
        request.setPetSpecies("dog");

        // Must not throw — must return safe fallback
        SymptomAnalysisResponse response = service.analyze(request);

        assertNotNull(response);
        assertEquals(UrgencyLevel.UNKNOWN, response.getUrgencyLevel());
        assertFalse(response.isAiGenerated()); // fallback is not AI generated
    }

    @Test
    void analyze_AiReturnsMalformedJson_ReturnsFallback() {
        // AI sometimes returns broken JSON
        String brokenResponse = "I think this pet needs urgent care...";
        // mock to return broken response
        // ...

        SymptomAnalysisResponse response = service.analyze(buildRequest());

        // Must handle gracefully
        assertNotNull(response);
        assertEquals(UrgencyLevel.UNKNOWN, response.getUrgencyLevel());
    }
}
```

---

## The Complete Mental Model

```
┌─────────────────────────────────────────────────────────────┐
│              AI IN MICROSERVICES — FIRST PRINCIPLES          │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  1. START WITH PAIN, NOT TECHNOLOGY                          │
│     Find decisions/tasks done poorly → AI solves those       │
│                                                               │
│  2. AI IS A SEPARATE SERVICE                                 │
│     Isolate all AI logic, keys, retry in one place           │
│                                                               │
│  3. AI IS ALWAYS OPTIONAL                                    │
│     Core feature works without AI                            │
│     AI failure = warning log, not user error                 │
│                                                               │
│  4. VALIDATE EVERYTHING                                       │
│     AI output is untrusted input — parse and validate        │
│     Same as validating user input                            │
│                                                               │
│  5. LOG EVERYTHING                                           │
│     Every AI call logged with input, output, latency         │
│     You can't improve what you can't measure                 │
│                                                               │
│  6. PROTECT PRIVACY                                          │
│     Anonymize before sending to external APIs                │
│     PII never leaves your infrastructure                     │
│                                                               │
│  7. CIRCUIT BREAK AI CALLS                                   │
│     If OpenAI is down, fail fast to fallback                 │
│     Don't let AI outage cascade to your services             │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

The central insight: **AI is just another microservice that happens to return probabilistic answers instead of deterministic ones.** All the same rules apply — contracts, fallbacks, testing, logging — plus one extra rule: never trust the output without validation.
