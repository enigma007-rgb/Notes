This is the right question to ask. Most learning plans fail because they optimize for completeness. You need to optimize for leverage.

Before DERIVE, one honest reframe:

---

## The Real Problem You're Describing

You're not confused because there's too much to learn. You're confused because nobody has told you **which 20% produces 80% of the value**. Every tutorial treats everything as equally important. It isn't.

Here's the honest map:

```
What exists in Java/Spring/Go: ~1000 concepts
What production systems actually use daily: ~80 concepts  
What interviews actually test: ~60 concepts
Overlap between both: ~50 concepts

That overlap is what you need.
Everything else is reference material —
learn it when you need it, not before.
```

---

## D — Destroy the Default Path

The default path for "what should I learn" produces: a comprehensive curriculum covering every feature of every technology. Every roadmap you'll find online does this. It's useless because it doesn't tell you what to skip.

**The destruction:**

```
I am not asking for a comprehensive curriculum.
I am asking for the minimum viable knowledge set 
that handles the maximum number of production 
situations.

The constraint: if I learn only this and nothing 
else, I should be able to:
1. Build and maintain production backend systems
2. Debug real incidents without being helpless
3. Pass senior-level interviews at good companies

Anything outside this set is out of scope.
Tell me what's in the set and why everything 
else is excluded.
```

---

## E — Eliminate the Assumptions

```
Do NOT assume I need to know everything before 
I can build anything.

Do NOT assume interview knowledge and production 
knowledge are separate. Show me where they overlap —
that overlap is the highest-leverage learning.

Do NOT assume framework knowledge replaces 
fundamentals knowledge. Show me exactly which 
fundamentals are load-bearing for everything else.

Do NOT assume equal importance. Force-rank every 
concept: critical, important, or reference.

Do NOT give me topics. Give me questions that 
production systems ask of engineers — the concepts 
are the answers to those questions.
```

---

## R — Reach the Long Tail

```
You are a senior engineer who has:
- Interviewed 200+ Java and Go candidates
- Debugged production incidents at scale
- Onboarded junior engineers to production codebases

From that experience:
What are the 5 questions you can ask a candidate 
that immediately reveal whether they understand 
production systems vs. just tutorial knowledge?

What are the concepts that consistently separate 
engineers who can handle incidents from those who can't?

What did you wish you had learned earlier that 
would have made the first 2 years faster?
```

---

## I — Insert Reasoning Before the Map

Before any topic list, the reasoning that generates it:

**The three filters every concept must pass:**

```
Filter 1 — Production frequency
Does this concept appear in real systems weekly?
If no → reference material, not core learning.

Filter 2 — Failure cost  
When an engineer doesn't understand this, 
what breaks and how bad is it?
High failure cost = must know deeply.
Low failure cost = know it exists, look up details.

Filter 3 — Interview signal
Does this concept appear in interviews because 
it reveals real understanding, or because 
interviewers are lazy and test trivia?
Real understanding signals = learn deeply.
Trivia = learn the answer, move on.
```

Every concept below passed all three filters.

---

## The Map — Organized by What Production Systems Actually Ask

Not by technology. By the question the system is asking of you.

---

### QUESTION 1: "How does data move through your system?"

**This question underlies**: every API you build, every bug you debug, every performance problem you investigate.

---

**Java/Spring — HTTP Request Lifecycle**

```
The concept:
A request enters your Spring Boot app.
Walk through every layer it touches before 
a response goes back.

Why it's load-bearing:
Every production bug lives somewhere in this chain.
If you can't trace the chain, you can't find the bug.

The chain:
HTTP Request
  → Embedded Tomcat (thread assignment)
  → DispatcherServlet (Spring's front controller)
  → Filter Chain (security, logging, CORS)
  → HandlerMapping (which controller method?)
  → ArgumentResolver (deserialize request body)
  → Interceptor (preHandle)
  → Controller method
  → Service layer
  → Repository layer
  → Database
  ← Result mapped back
  → Interceptor (postHandle)
  → MessageConverter (serialize response)
  → Filter Chain (reverse)
  ← HTTP Response
```

**Real scenario — the interview question:**

*"A POST /orders endpoint works fine in development. In production it intermittently returns 500. No exception in your logs. What do you check and in what order?"*

The answer requires knowing this chain. The bug is in a filter (request body already consumed), or an interceptor (transaction not started), or a MessageConverter (serialization failing silently). Without the chain, you're guessing.

**The code that makes it concrete:**

```java
// Custom filter — runs before every request
@Component
@Order(1)
public class RequestLoggingFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain chain) 
            throws ServletException, IOException {
        
        String requestId = UUID.randomUUID().toString();
        MDC.put("requestId", requestId); // attach to all logs in this thread
        
        long start = System.currentTimeMillis();
        
        try {
            chain.doFilter(request, response); // proceed down the chain
        } finally {
            long duration = System.currentTimeMillis() - start;
            log.info("Request completed: {} {} {} {}ms",
                request.getMethod(), request.getRequestURI(),
                response.getStatus(), duration);
            MDC.clear(); // clean up thread-local state
        }
    }
}
```

**What MDC.put() teaches:** Spring Boot handles requests on threads from a pool. Thread-local state (MDC) is the mechanism for attaching a requestId to every log line from that request. Without clearing it in `finally`, the next request on the same thread inherits the previous request's ID. This is a real production logging bug that corrupts distributed tracing.

---

**Go — HTTP Request Lifecycle**

```
The concept:
Same question, different runtime.

The chain:
HTTP Request
  → net/http listener (goroutine per connection)
  → ServeMux / Gin router (path matching)
  → Middleware chain (each middleware wraps the next)
  → Handler function
  → Service layer
  → Database/cache
  ← Response written to ResponseWriter
```

**The code that makes it concrete:**

```go
// Middleware in Go — the pattern everything builds on
func RequestIDMiddleware(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
        
        requestID := uuid.New().String()
        
        // Attach to context — flows through entire call chain
        ctx := context.WithValue(r.Context(), "requestID", requestID)
        r = r.WithContext(ctx)
        
        // Set in response header
        w.Header().Set("X-Request-ID", requestID)
        
        start := time.Now()
        
        // Wrap ResponseWriter to capture status code
        wrapped := &statusRecorder{ResponseWriter: w, statusCode: 200}
        
        next.ServeHTTP(wrapped, r) // proceed down the chain
        
        log.Printf("requestId=%s method=%s path=%s status=%d duration=%s",
            requestID, r.Method, r.URL.Path, 
            wrapped.statusCode, time.Since(start))
    })
}

type statusRecorder struct {
    http.ResponseWriter
    statusCode int
}

func (r *statusRecorder) WriteHeader(code int) {
    r.statusCode = code
    r.ResponseWriter.WriteHeader(code)
}
```

**What this teaches:** Go middleware is function wrapping — each middleware is a function that takes a handler and returns a handler. The `next.ServeHTTP()` call is where the chain proceeds. Code before it runs on the way in. Code after it runs on the way out. This pattern is the foundation for every Go middleware you'll write or read.

---

### QUESTION 2: "How does your system stay correct when things go wrong?"

**This question underlies**: transaction management, error handling, data consistency — the concepts that separate systems that work from systems that work correctly.

---

**Java/Spring — Transaction Management**

```
The concept:
Multiple database operations that must all succeed 
or all fail together.

Why it's load-bearing:
Every financial operation, every order placement, 
every state change with side effects.
Get this wrong and you have partial data — 
the hardest category of production bug to fix.
```

**The three things you must know deeply:**

**Thing 1 — What @Transactional actually does:**

```java
@Service
public class OrderService {
    
    @Transactional  // Spring wraps this method in a proxy
    public Order placeOrder(String userId, List<String> productIds) {
        
        // All of these happen in ONE database transaction
        Order order = orderRepository.save(new Order(userId));
        inventoryService.decrementStock(productIds); // calls repository
        paymentService.chargeUser(userId, order.total()); // calls repository
        
        // If ANY of these throw RuntimeException:
        // ALL database changes are rolled back automatically
        // The proxy catches the exception and calls connection.rollback()
        
        return order;
    }
}
```

**Thing 2 — The self-invocation trap (most common Spring transaction bug):**

```java
@Service
public class OrderService {
    
    public void processOrders(List<String> orderIds) {
        for (String id : orderIds) {
            processSingleOrder(id); // BUG: calling own method
        }
    }
    
    @Transactional
    public void processSingleOrder(String orderId) {
        // This @Transactional is SILENTLY IGNORED
        // because the call goes through 'this' (real object)
        // not through the Spring proxy
        // No transaction. No rollback on failure.
    }
}

// Fix: inject self, or extract to separate bean
@Service
public class OrderService {
    
    @Autowired
    private OrderService self; // Spring injects the proxy, not 'this'
    
    public void processOrders(List<String> orderIds) {
        for (String id : orderIds) {
            self.processSingleOrder(id); // goes through proxy — works
        }
    }
    
    @Transactional
    public void processSingleOrder(String orderId) {
        // Transaction works correctly now
    }
}
```

**Thing 3 — Propagation behaviors (the two you actually use):**

```java
// REQUIRED (default): join existing transaction or create new one
// Use for: 99% of transactional methods
@Transactional(propagation = Propagation.REQUIRED)
public void placeOrder() { ... }

// REQUIRES_NEW: always create new transaction, suspend existing
// Use for: audit logging, notification sending —
// things that must commit even if outer transaction rolls back
@Transactional(propagation = Propagation.REQUIRES_NEW)
public void logAuditEvent(String event) {
    // This commits independently of whatever called it
    // Even if outer transaction rolls back, this log is saved
    auditRepository.save(new AuditLog(event));
}

// Real scenario where this matters:
@Transactional  // outer transaction
public void placeOrder(Order order) {
    orderRepository.save(order);
    auditService.logAuditEvent("ORDER_PLACED"); // REQUIRES_NEW
    paymentService.charge(order); // throws exception
    // outer transaction rolls back — order not saved
    // audit log IS saved — logged that attempt was made
}
```

**Interview question this answers:**

*"An order is placed, payment fails, but the audit log shows the order succeeded. How do you fix this without losing the audit record?"*

Answer requires knowing REQUIRES_NEW propagation. Without it, you either lose the audit record (REQUIRED) or you have to design around it awkwardly.

---

**Go — Error Handling as a System**

```
The concept:
Go has no exceptions. Every error is a value.
This forces you to think about errors as 
part of your API contract, not exceptional cases.

Why it's load-bearing:
Every function call has an error return.
How you handle, wrap, and propagate errors 
determines whether your system is debuggable
or opaque.
```

**The three levels you must understand:**

```go
// Level 1: Basic — loses context
func getUser(id string) (*User, error) {
    user, err := db.QueryRow(id)
    if err != nil {
        return nil, err  // which id? what operation? unknown.
    }
    return user, nil
}

// Level 2: Wrapped — preserves context chain
func getUser(id string) (*User, error) {
    user, err := db.QueryRow(id)
    if err != nil {
        return nil, fmt.Errorf("getUser %s: %w", id, err)
        // %w wraps for errors.Is() / errors.As() unwrapping
    }
    return user, nil
}

// Level 3: Typed — enables programmatic handling
var ErrUserNotFound = errors.New("user not found")

type ValidationError struct {
    Field   string
    Message string
}
func (e *ValidationError) Error() string {
    return fmt.Sprintf("%s: %s", e.Field, e.Message)
}

func getUser(id string) (*User, error) {
    if id == "" {
        return nil, &ValidationError{Field: "id", Message: "required"}
    }
    user, err := db.QueryRow(id)
    if errors.Is(err, sql.ErrNoRows) {
        return nil, fmt.Errorf("getUser: %w", ErrUserNotFound)
    }
    if err != nil {
        return nil, fmt.Errorf("getUser %s: %w", id, err)
    }
    return user, nil
}

// At the HTTP handler — different errors → different responses
func handleGetUser(w http.ResponseWriter, r *http.Request) {
    user, err := getUser(r.PathValue("id"))
    if err != nil {
        var valErr *ValidationError
        switch {
        case errors.As(err, &valErr):
            http.Error(w, valErr.Message, http.StatusBadRequest)
        case errors.Is(err, ErrUserNotFound):
            http.Error(w, "not found", http.StatusNotFound)
        default:
            slog.Error("getUser failed", "error", err)
            http.Error(w, "internal error", http.StatusInternalServerError)
        }
        return
    }
    json.NewEncoder(w).Encode(user)
}
```

---

### QUESTION 3: "How does your system handle more load?"

**This question underlies**: concurrency, connection pooling, caching — the concepts that determine whether your system scales or collapses.

---

**Java/Spring — Concurrency and Thread Model**

```
What you must know:
Spring Boot's default model is one thread per request.
Each thread can handle exactly one request at a time.
Tomcat's default thread pool: 200 threads.
200 concurrent requests max before queuing begins.

Why this matters:
If your handler does slow I/O (database, external API),
that thread is blocked — doing nothing, holding a 
resource. At scale, this is where systems collapse.
```

**The patterns that prevent collapse:**

```java
// Pattern 1: Don't block on slow I/O in the request thread
// BAD — blocks thread for duration of external call
@GetMapping("/user/{id}")
public UserProfile getUser(@PathVariable String id) {
    User user = userService.getUser(id);           // 50ms db call
    Activity activity = activityService.get(id);   // 100ms external API
    Score score = scoringService.getScore(id);      // 80ms external API
    // Thread blocked for 230ms sequential
    return buildProfile(user, activity, score);
}

// BETTER — parallel execution, thread blocked for max(50,100,80)ms
@GetMapping("/user/{id}")
public UserProfile getUser(@PathVariable String id) 
        throws ExecutionException, InterruptedException {
    
    CompletableFuture<User> userFuture = 
        CompletableFuture.supplyAsync(() -> userService.getUser(id));
    CompletableFuture<Activity> activityFuture = 
        CompletableFuture.supplyAsync(() -> activityService.get(id));
    CompletableFuture<Score> scoreFuture = 
        CompletableFuture.supplyAsync(() -> scoringService.getScore(id));
    
    CompletableFuture.allOf(userFuture, activityFuture, scoreFuture).join();
    // Thread blocked for ~100ms (the longest call) not 230ms
    
    return buildProfile(
        userFuture.get(), activityFuture.get(), scoreFuture.get()
    );
}
```

**Connection pool — the concept most engineers learn in an incident:**

```java
// application.properties — these defaults will hurt you
spring.datasource.hikari.maximum-pool-size=10      # default: 10
spring.datasource.hikari.minimum-idle=10
spring.datasource.hikari.connection-timeout=30000  # 30s before exception
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000

// The incident:
// 200 Tomcat threads, 10 HikariCP connections.
// 11th request that needs a database: waits 30 seconds.
// User sees 30 second response time, then an error.
// All because pool-size=10 with thread-pool=200.

// The rule: 
// Pool size should be tuned to your database's capacity,
// not to your thread count. Formula (from HikariCP docs):
// pool_size = (core_count * 2) + effective_spindle_count
// For a 4-core machine with SSD: (4*2) + 1 = 9
// For most apps: 10-20 is the right range, NOT 200.
```

**Interview question this answers:**

*"Your API response times are fine at 50 req/sec but blow up at 200 req/sec. Database CPU is at 5%. What's happening?"*

Answer: thread pool or connection pool exhaustion. Not database performance. This is the question that separates engineers who understand their runtime from those who only know the framework.

---

**Go — Goroutines and Context**

```
What you must know:
Go's concurrency model is the reason to use Go.
Goroutines are cheap (~2KB stack vs 1MB for threads).
You can have 100,000 goroutines where Java has 200 threads.
But goroutines leak if you don't manage their lifecycle.
Context is the mechanism for lifecycle management.
```

**The three patterns that cover most production cases:**

```go
// Pattern 1: Bounded worker pool — never spin unbounded goroutines
func processItems(ctx context.Context, items []Item) []Result {
    
    jobs := make(chan Item, len(items))
    results := make(chan Result, len(items))
    
    // Fixed number of workers — controls resource usage
    workerCount := 10
    var wg sync.WaitGroup
    
    for i := 0; i < workerCount; i++ {
        wg.Add(1)
        go func() {
            defer wg.Done()
            for item := range jobs {
                select {
                case <-ctx.Done():
                    return // exit if context cancelled
                default:
                    results <- process(item)
                }
            }
        }()
    }
    
    // Feed jobs
    for _, item := range items {
        jobs <- item
    }
    close(jobs) // signals workers: no more jobs
    
    // Wait then close results
    go func() {
        wg.Wait()
        close(results)
    }()
    
    var output []Result
    for r := range results {
        output = append(output, r)
    }
    return output
}

// Pattern 2: Context propagation — the contract every function must follow
// Every function that does I/O takes context as first parameter
func getUser(ctx context.Context, id string) (*User, error) {
    // Database call respects context cancellation/timeout
    row := db.QueryRowContext(ctx, "SELECT * FROM users WHERE id=$1", id)
    
    // HTTP call respects context
    req, _ := http.NewRequestWithContext(ctx, "GET", apiURL+id, nil)
    resp, err := httpClient.Do(req)
    
    // If ctx is cancelled (request timeout, client disconnect):
    // QueryRowContext returns context.DeadlineExceeded
    // httpClient.Do returns context.Canceled
    // The goroutine exits cleanly — no leak
}

// Pattern 3: Timeout at the boundary — every external call gets a deadline
func handleGetUser(w http.ResponseWriter, r *http.Request) {
    
    // Every handler gets a timeout — nothing runs forever
    ctx, cancel := context.WithTimeout(r.Context(), 3*time.Second)
    defer cancel() // always cancel to release resources
    
    user, err := getUser(ctx, r.PathValue("id"))
    if err != nil {
        if errors.Is(err, context.DeadlineExceeded) {
            http.Error(w, "timeout", http.StatusGatewayTimeout)
            return
        }
        http.Error(w, "error", http.StatusInternalServerError)
        return
    }
    json.NewEncoder(w).Encode(user)
}
```

---

### QUESTION 4: "How does your system stay fast?"

**This question underlies**: caching, database indexing, query optimization — the concepts that determine whether your system is fast enough to be useful.

---

**Java/Spring — Caching**

```java
// Spring Cache abstraction — same API, swappable backend
@Service
public class UserService {
    
    // Cache result — key is the method argument
    @Cacheable(value = "users", key = "#id")
    public User getUser(String id) {
        // Only called on cache MISS
        // On cache HIT: returns cached value, method body skipped
        return userRepository.findById(id).orElseThrow();
    }
    
    // Evict on update — cache must not serve stale data
    @CacheEvict(value = "users", key = "#user.id")
    public User updateUser(User user) {
        return userRepository.save(user);
    }
    
    // Update cache entry — evict and put in one operation
    @CachePut(value = "users", key = "#result.id")
    public User createUser(User user) {
        return userRepository.save(user);
    }
}

// Cache configuration with TTL (using Caffeine — in-memory)
@Configuration
@EnableCaching
public class CacheConfig {
    
    @Bean
    public CacheManager cacheManager() {
        CaffeineCacheManager manager = new CaffeineCacheManager("users");
        manager.setCaffeine(Caffeine.newBuilder()
            .maximumSize(10_000)          // max entries
            .expireAfterWrite(5, TimeUnit.MINUTES)  // TTL
            .recordStats());              // enable metrics
        return manager;
    }
}
```

**The cache invalidation problem — most common cache bug:**

```java
// Scenario:
// User updates their profile.
// Cache still serves old profile for 5 minutes.
// User sees their old data and thinks update failed.
// They try again. Now two updates have been submitted.

// The cache consistency contract you must choose:
// 1. Cache-aside: read from cache, write to DB + evict cache
//    Simple. Consistent on next read. Short staleness window.
//    Use for: most read-heavy data.

// 2. Write-through: write to DB + write to cache atomically
//    Complex. Always consistent. Write latency increases.
//    Use for: data that must never be stale.

// 3. Write-behind: write to cache immediately, DB async
//    Fast writes. Risk of data loss if cache fails.
//    Use for: non-critical data, analytics, counters.

// The interview answer:
// "What cache strategy would you use for user profile data?"
// Answer: cache-aside with short TTL (1-5 min) + @CacheEvict on update.
// Not write-through (overkill for profile data).
// Not write-behind (profile data is important enough to not risk losing).
```

---

**Go — Database Patterns**

```go
// Connection pool — the most important configuration
func NewDB(ctx context.Context, dsn string) (*pgxpool.Pool, error) {
    config, err := pgxpool.ParseConfig(dsn)
    if err != nil {
        return nil, fmt.Errorf("parse config: %w", err)
    }
    
    // These defaults will hurt you in production if not set
    config.MaxConns = 20
    config.MinConns = 5
    config.MaxConnLifetime = time.Hour      // prevent stale connections
    config.MaxConnIdleTime = 30 * time.Minute
    config.HealthCheckPeriod = time.Minute
    
    pool, err := pgxpool.NewWithConfig(ctx, config)
    if err != nil {
        return nil, fmt.Errorf("create pool: %w", err)
    }
    
    // Fail fast on startup if database unreachable
    if err := pool.Ping(ctx); err != nil {
        return nil, fmt.Errorf("ping database: %w", err)
    }
    
    return pool, nil
}

// The query pattern that prevents connection leaks
func getUser(ctx context.Context, pool *pgxpool.Pool, id string) (*User, error) {
    var user User
    
    err := pool.QueryRow(ctx,
        "SELECT id, name, email FROM users WHERE id = $1", id,
    ).Scan(&user.ID, &user.Name, &user.Email)
    
    if errors.Is(err, pgx.ErrNoRows) {
        return nil, ErrUserNotFound
    }
    if err != nil {
        return nil, fmt.Errorf("query user %s: %w", id, err)
    }
    
    return &user, nil
}
```

---

### QUESTION 5: "How does your system know who's making the request?"

**This question underlies**: authentication, authorization, security — concepts that appear in every production system and every interview.

---

**Java/Spring Security — The Essential Mental Model**

```
What you must know:
Spring Security is a filter chain.
Every request passes through it before reaching your controller.
Your job is to configure the chain, not understand every class.

The three concepts that cover 90% of use cases:
1. Authentication — who are you? (JWT validation)
2. Authorization — what can you do? (role checking)
3. Security context — how does Spring know who you are 
   for the rest of the request?
```

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // stateless API — no CSRF needed
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()     // public endpoints
                .requestMatchers("/admin/**").hasRole("ADMIN") // admin only
                .anyRequest().authenticated())               // everything else: must be logged in
            .addFilterBefore(
                jwtAuthFilter,  // your filter runs before Spring's auth
                UsernamePasswordAuthenticationFilter.class
            );
        return http.build();
    }
}

// JWT filter — the authentication mechanism
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain chain)
            throws ServletException, IOException {
        
        String header = request.getHeader("Authorization");
        
        if (header == null || !header.startsWith("Bearer ")) {
            chain.doFilter(request, response); // no token — proceed unauthenticated
            return;
        }
        
        String token = header.substring(7);
        
        try {
            Claims claims = jwtService.validateAndExtract(token);
            String userId = claims.getSubject();
            List<String> roles = claims.get("roles", List.class);
            
            // Store in SecurityContext — available to entire request
            UsernamePasswordAuthenticationToken auth =
                new UsernamePasswordAuthenticationToken(
                    userId,
                    null,
                    roles.stream()
                         .map(SimpleGrantedAuthority::new)
                         .collect(Collectors.toList())
                );
            
            SecurityContextHolder.getContext().setAuthentication(auth);
            
        } catch (JwtException e) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return;
        }
        
        chain.doFilter(request, response);
    }
}

// In your controller — access the authenticated user
@GetMapping("/profile")
public UserProfile getMyProfile() {
    String userId = (String) SecurityContextHolder.getContext()
        .getAuthentication().getPrincipal();
    return userService.getProfile(userId);
}
```

---

**Go — JWT Middleware**

```go
// Auth middleware — same concept, Go idiom
func AuthMiddleware(jwtSecret []byte) func(http.Handler) http.Handler {
    return func(next http.Handler) http.Handler {
        return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
            
            header := r.Header.Get("Authorization")
            if !strings.HasPrefix(header, "Bearer ") {
                http.Error(w, "unauthorized", http.StatusUnauthorized)
                return
            }
            
            tokenStr := strings.TrimPrefix(header, "Bearer ")
            
            claims, err := validateJWT(tokenStr, jwtSecret)
            if err != nil {
                http.Error(w, "unauthorized", http.StatusUnauthorized)
                return
            }
            
            // Attach to context — available downstream
            ctx := context.WithValue(r.Context(), userIDKey, claims.UserID)
            ctx = context.WithValue(ctx, rolesKey, claims.Roles)
            
            next.ServeHTTP(w, r.WithContext(ctx))
        })
    }
}

// In handler — retrieve from context
func handleGetProfile(w http.ResponseWriter, r *http.Request) {
    userID, ok := r.Context().Value(userIDKey).(string)
    if !ok {
        http.Error(w, "unauthorized", http.StatusUnauthorized)
        return
    }
    // use userID
}
```

---

### QUESTION 6: "How do you know when your system is broken?"

**This question underlies**: logging, monitoring, graceful shutdown — what separates systems you can operate from systems you just hope work.

---

**The Non-Negotiable Production Checklist**

Both Java/Spring and Go. Non-negotiable means: ship nothing without these.

```java
// Spring Boot — structured logging
// Never use System.out.println in production.
// Never use string concatenation in log statements.

// BAD
log.info("Processing order " + orderId + " for user " + userId);

// GOOD — structured, queryable, no string allocation if level disabled
log.info("Processing order", 
    kv("orderId", orderId), 
    kv("userId", userId),
    kv("amount", order.getTotal()));

// Why structured: in your log aggregator (CloudWatch, Datadog, ELK),
// you can query: orderId = "ord-123" instead of grep-ing text.
// At 10,000 req/sec that query difference is minutes vs hours.
```

```java
// Spring Boot — graceful shutdown
// application.properties
server.shutdown=graceful
spring.lifecycle.timeout-per-shutdown-phase=30s

// What this does:
// SIGTERM received → stop accepting new requests
// Wait up to 30s for in-flight requests to complete
// Shutdown application
// Without this: SIGTERM kills process immediately
// In-flight requests get connection reset by peer
// Orders half-processed. Data inconsistency.
```

```go
// Go — graceful shutdown
func main() {
    srv := &http.Server{
        Addr:         ":8080",
        Handler:      router,
        ReadTimeout:  10 * time.Second,
        WriteTimeout: 30 * time.Second,
        IdleTimeout:  120 * time.Second,
    }
    
    go func() {
        if err := srv.ListenAndServe(); err != http.ErrServerClosed {
            slog.Error("server error", "error", err)
            os.Exit(1)
        }
    }()
    
    // Wait for interrupt
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    
    slog.Info("shutting down...")
    
    ctx, cancel := context.WithTimeout(context.Background(), 30*time.Second)
    defer cancel()
    
    if err := srv.Shutdown(ctx); err != nil {
        slog.Error("forced shutdown", "error", err)
    }
    
    // Close other resources AFTER server stops
    db.Close()
    
    slog.Info("shutdown complete")
}
```

```go
// Go — structured logging (slog, standard library since 1.21)
// Never use fmt.Println or log.Printf in production

slog.Info("order placed",
    "orderId", orderId,
    "userId", userId,
    "amount", order.Total,
    "duration_ms", time.Since(start).Milliseconds(),
)

slog.Error("payment failed",
    "orderId", orderId,
    "error", err,
    "provider", "razorpay",
)
```

---

## V — The Complete Priority Map

Everything above, force-ranked:

```
CRITICAL — learn deeply before anything else:
  Java: HTTP lifecycle, @Transactional (including traps),
        connection pool configuration, Spring Security filter chain

  Go:   Context propagation, error wrapping, goroutine lifecycle,
        graceful shutdown, connection pool configuration

IMPORTANT — learn well, not perfectly:
  Java: CompletableFuture basics, @Cacheable + eviction strategy,
        Spring profiles, exception handling with @ControllerAdvice

  Go:   Worker pool pattern, structured logging with slog,
        JWT middleware pattern, database query patterns with pgx

REFERENCE — know it exists, look up details when needed:
  Java: Spring Boot Actuator, Micrometer metrics, 
        @Async, Spring Events, Bean scopes beyond singleton

  Go:   pprof profiling, sync.Pool, 
        advanced channel patterns, reflect package

SKIP FOR NOW — real but not leverage-producing yet:
  Java: Reactive/WebFlux, Spring Batch, Spring Integration,
        Hibernate second-level cache, custom BeanPostProcessors

  Go:   CGo, unsafe package, custom runtime instrumentation,
        assembly optimization
```

---

## E — The Iterative Learning Protocol

Don't learn topics. Learn answers to production questions.

For every concept above, the learning loop is:

```
Step 1: What problem does this solve?
        (if you can't answer this, you'll forget it)

Step 2: What does it look like when it's wrong?
        (the failure mode teaches more than the happy path)

Step 3: Write the minimal code that demonstrates it
        (code you wrote > code you read, always)

Step 4: Break it deliberately
        (remove the @Transactional, remove the defer cancel(),
         set pool size to 1 — see what happens)

Step 5: Write the interview answer
        (one paragraph explaining the concept through 
         a production scenario — not a definition)
```

**The most important thing:**

The 50 concepts above are not 50 things to learn. They are 6 questions that production systems ask, with the specific knowledge that answers each one. Learn the questions first. The concepts become answers, not trivia.

When you're confused about what to learn next, ask: *which of the 6 questions do I understand least well?* Learn the concepts that answer that question. Stop when you can answer it with a real scenario. Move to the next question.

That's the entire learning strategy. DERIVE didn't give you a list. It gave you a filter. Apply the filter to anything new you encounter and you'll always know whether to learn it deeply, learn it exists, or skip it entirely.
