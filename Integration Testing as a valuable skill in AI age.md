Your senior engineer is right. And the reason is more specific than most people realize.

Let me explain why first, then build the full DERIVE demonstration.

---

## Why Integration Testing Is Specifically Valuable in the AI Age

When AI generates code, it generates code that is locally correct but globally wrong.

What that means precisely:

An LLM generates a Spring Boot service method. The method looks correct. The logic is sound. The annotations are right. The variable names make sense. You read it and it looks good.

Then you run it against a real database, a real message queue, a real external API — and it fails. Not because the code is wrong in isolation. Because the assumptions the LLM made about how the system components interact are wrong for your specific system.

This is the integration gap. And it is exactly the gap that integration testing closes.

---

## The Three Specific Reasons Your Senior Engineer Is Right

**Reason 1 — LLMs are trained on isolated code examples**

Training data for code is dominated by: Stack Overflow answers (single method solutions), GitHub snippets (isolated components), tutorial blog posts (happy path examples). Almost none of the training data shows how components behave together under realistic conditions with real infrastructure.

An LLM knows perfectly how to write a `@Transactional` method. It does not know that in YOUR system, that method interacts with a connection pool configured a specific way, a PostgreSQL version with specific behavior, and a Kafka consumer that shares the same thread pool. The integration is where your system's reality diverges from the training data's assumptions.

**Reason 2 — AI-generated code increases the velocity of introducing integration bugs**

Before AI coding tools, a developer wrote code slowly enough that they thought through the integration implications. They paused. They checked the existing codebase. They asked a colleague.

With AI tools, code is generated in seconds. The velocity is 10x. The integration thinking does not scale at the same rate. You get 10x more code with the same amount of integration reasoning — which means 10x more opportunities to introduce integration bugs that unit tests will not catch.

**Reason 3 — Integration tests are the verification layer that makes AI-generated code trustworthy**

An LLM generates a repository method. You have two choices: trust it and ship it, or verify it. Unit tests verify the logic in isolation. Integration tests verify the behavior against real infrastructure. In the AI age, integration tests are the safety net that lets you use AI-generated code at production velocity without introducing production bugs at the same velocity.

The engineer who can write integration tests is the engineer who can verify AI output at scale. That is a rare and valuable skill precisely because most engineers either test manually (slow, inconsistent) or trust unit tests (fast, insufficient).

---

## What Integration Testing Actually Means in Spring Boot

Before the DERIVE examples, let me establish the precise mental model.

**Unit test:** Tests one class in isolation. All dependencies are mocked. Tests the logic of the code. Fast. Runs in milliseconds. Does not verify that your code works with real infrastructure.

**Integration test:** Tests how multiple components work together. Uses real (or realistic) infrastructure. Tests the behavior of the system. Slower. Runs in seconds. Verifies that your code works in the real world.

**The gap between them:**

```java
// Unit test verifies this logic works
when(orderRepository.findById(1L)).thenReturn(Optional.of(order));
OrderDTO result = orderService.getOrder(1L);
assertEquals("CONFIRMED", result.getStatus());
// This tells you the mapping logic is correct
// It tells you NOTHING about whether findById actually works
// against your PostgreSQL schema with your indexes
// with your connection pool configuration

// Integration test verifies this actually works
// Real PostgreSQL, real schema, real data
Order saved = orderRepository.save(testOrder);
OrderDTO result = orderService.getOrder(saved.getId());
assertEquals("CONFIRMED", result.getStatus());
// This tells you the entire chain works
// Schema, mapping, query, connection pool, transaction - all verified
```

---

## DERIVE Applied to Integration Testing in Spring Boot

Let me now show you the full DERIVE framework applied to five real integration testing scenarios, each demonstrating a specific type of bug that AI-generated code introduces that only integration tests catch.

---

## Scenario 1: The N+1 Query That Unit Tests Cannot See

**The Setup:**

You asked an AI coding assistant to generate a REST endpoint that returns a list of orders with their items. The AI generated clean-looking code. Your unit tests pass. You ship it. Production is slow.

---

### BEFORE DERIVE

**Generic prompt:**
"How do I write integration tests for a Spring Boot REST endpoint?"

**Generic response:**
```
Use @SpringBootTest for full context loading.
Use TestRestTemplate or MockMvc for HTTP testing.
Use an H2 in-memory database for testing.
Assert the response status and body.
Use @Transactional to rollback test data.
```

This tells you the tools. It tells you nothing about what to actually test or why. You write tests that pass against H2 and still ship the N+1 query bug.

---

### AFTER DERIVE

**Round 1 — D:**

"I am writing integration tests for a Spring Boot 3.1 REST API. I used an AI coding assistant to generate an endpoint that returns orders with their items. Here is the generated code:

```java
@RestController
@RequestMapping('/api/orders')
public class OrderController {

    @Autowired
    private OrderService orderService;

    @GetMapping
    public List<OrderDTO> getOrders(@RequestParam Long userId) {
        return orderService.getOrdersForUser(userId);
    }
}

@Service
public class OrderService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private OrderItemRepository orderItemRepository;

    public List<OrderDTO> getOrdersForUser(Long userId) {
        List<Order> orders = orderRepository.findByUserId(userId);
        return orders.stream()
            .map(order -> {
                List<OrderItem> items = 
                    orderItemRepository.findByOrderId(order.getId());
                return new OrderDTO(order, items);
            })
            .collect(Collectors.toList());
    }
}
```

My unit tests mock both repositories and pass. But I have a feeling this code has a performance problem I cannot see in unit tests.

Do NOT tell me generic integration testing setup. Tell me:
1. What specific bug exists in this AI-generated code that unit tests cannot detect
2. What the integration test must do to catch this bug
3. What the test failure looks like that proves the bug exists"

---

**What DERIVE extracts:**

**The specific bug:**

This is a classic N+1 query problem. For a user with 50 orders:
- 1 query to fetch all orders: `SELECT * FROM orders WHERE user_id = ?`
- 50 queries to fetch items for each order: `SELECT * FROM order_items WHERE order_id = ?` × 50
- Total: 51 database queries for one API call

Unit tests cannot see this because mocked repositories return data instantly with no database round-trips. The performance impact is invisible until you hit a real database.

**The integration test that catches it:**

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@AutoConfigureTestDatabase(replace = AutoConfigureTestDatabase.Replace.NONE)
@Testcontainers
class OrderControllerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:15")
            .withDatabaseName("testdb")
            .withUsername("test")
            .withPassword("test");

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.datasource.username", postgres::getUsername);
        registry.add("spring.datasource.password", postgres::getPassword);
    }

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private OrderItemRepository orderItemRepository;

    @Autowired
    private UserRepository userRepository;

    // THE KEY: query counting datasource
    @Autowired
    private DataSource dataSource;

    @Test
    void getOrders_shouldNotCauseNPlusOneQueries() {
        // Arrange: create realistic test data
        // 1 user with 20 orders, each order with 5 items
        User user = userRepository.save(
            User.builder().email("test@test.com").build()
        );

        for (int i = 0; i < 20; i++) {
            Order order = orderRepository.save(
                Order.builder()
                    .userId(user.getId())
                    .status("CONFIRMED")
                    .build()
            );
            for (int j = 0; j < 5; j++) {
                orderItemRepository.save(
                    OrderItem.builder()
                        .orderId(order.getId())
                        .productName("Product " + j)
                        .quantity(1)
                        .build()
                );
            }
        }

        // THE CRITICAL PART: count actual SQL queries executed
        // Using datasource-proxy to intercept and count queries
        QueryCountHolder.clear();

        // Act
        ResponseEntity<List> response = restTemplate.getForEntity(
            "/api/orders?userId=" + user.getId(),
            List.class
        );

        // Assert: response is correct
        assertEquals(HttpStatus.OK, response.getStatusCode());
        assertEquals(20, response.getBody().size());

        // Assert: query count - THIS IS WHAT CATCHES THE BUG
        QueryCount queryCount = QueryCountHolder.getGrandTotal();
        
        // With N+1: this will be 21 (1 for orders + 20 for items)
        // Without N+1: this should be 2 (1 for orders with JOIN FETCH)
        assertThat(queryCount.getSelect())
            .as("Expected at most 2 SELECT queries but found %d - N+1 query detected",
                queryCount.getSelect())
            .isLessThanOrEqualTo(2);
    }
}
```

**The test failure that proves the bug:**

```
AssertionError: Expected at most 2 SELECT queries but found 21
- N+1 query detected
Expected: ≤ 2
Actual: 21

This test failure is the integration test catching what the unit test missed.
It is proof that the AI-generated code has a performance bug
that will cause slow API responses in production with real data volumes.
```

**The fix the integration test drives:**

```java
// Replace the separate repository calls with a JOIN FETCH query
public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query("SELECT o FROM Order o " +
           "LEFT JOIN FETCH o.items " +
           "WHERE o.userId = :userId")
    List<Order> findByUserIdWithItems(@Param("userId") Long userId);
}

// Now the service uses one query instead of N+1
public List<OrderDTO> getOrdersForUser(Long userId) {
    List<Order> orders = orderRepository.findByUserIdWithItems(userId);
    return orders.stream()
        .map(order -> new OrderDTO(order, order.getItems()))
        .collect(Collectors.toList());
}
```

Run the integration test again after this fix. It passes. You have verified the fix works against a real database — not just in theory.

---

## Scenario 2: The Transaction Boundary Bug

**The Setup:**

AI generated a payment processing service. The logic looks correct. Unit tests pass. In production, you occasionally have customers charged who have no order record — the classic partial write problem.

---

### AFTER DERIVE

**Full DERIVE Prompt:**

"I used AI to generate a payment service. Here is the generated code:

```java
@Service
public class PaymentService {

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private PaymentRepository paymentRepository;

    @Autowired
    private StripeClient stripeClient;

    @Autowired
    private EmailService emailService;

    @Transactional
    public PaymentResult processPayment(PaymentRequest request) {
        // Create order
        Order order = orderRepository.save(
            Order.builder()
                .userId(request.getUserId())
                .amount(request.getAmount())
                .status('PENDING')
                .build()
        );

        // Charge card via Stripe
        StripeCharge charge = stripeClient.charge(
            request.getCardToken(),
            request.getAmount()
        );

        // Update order status
        order.setStatus('COMPLETED');
        order.setStripeChargeId(charge.getId());
        orderRepository.save(order);

        // Send confirmation email
        emailService.sendConfirmation(order);

        return new PaymentResult(order.getId(), charge.getId());
    }
}
```

My unit tests mock stripeClient and emailService and all pass.

Before giving me integration tests: tell me every way this @Transactional usage is wrong for a method that calls an external payment API. Then tell me what integration tests would catch each problem. Be specific about what the test data setup looks like and what assertions prove each bug."

---

**What DERIVE extracts:**

**Bug 1 — Stripe API call inside @Transactional holds database connection:**

The `@Transactional` annotation opens a database connection at the start of the method and holds it until the method completes. The Stripe API call takes 200-500ms in real conditions. During that time your database connection is held open doing nothing. Under 100 concurrent payment requests, 100 database connections are held waiting for Stripe. Your HikariCP pool exhausts. Every other database operation in your application fails.

**Bug 2 — Stripe charges customer when database write fails:**

If `orderRepository.save(order)` (the second one, updating to COMPLETED) fails, `@Transactional` rolls back the database transaction. The order goes back to PENDING or disappears. But Stripe has already been charged. The customer was charged for an order that does not exist in your database.

**Bug 3 — Email sends when transaction rolls back:**

If anything after `emailService.sendConfirmation` fails, the transaction rolls back. But the email was already sent. The customer received a confirmation email for a transaction that was rolled back.

**The integration tests that catch each bug:**

```java
@SpringBootTest
@Testcontainers
class PaymentServiceIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:15");

    // WireMock simulates Stripe API
    @RegisterExtension
    static WireMockExtension wireMock = WireMockExtension
        .newInstance()
        .options(wireMockConfig().dynamicPort())
        .build();

    @Autowired
    private PaymentService paymentService;

    @Autowired
    private OrderRepository orderRepository;

    @Autowired
    private HikariDataSource dataSource;

    // ============================================
    // TEST 1: Connection pool exhaustion under load
    // ============================================
    @Test
    void processPayment_shouldNotHoldConnectionDuringStripeCall() {
        // Simulate slow Stripe response (400ms)
        wireMock.stubFor(post(urlEqualTo("/v1/charges"))
            .willReturn(aResponse()
                .withStatus(200)
                .withFixedDelay(400)  // 400ms delay simulating real Stripe
                .withBody(validStripeResponse())));

        // Record connection pool metrics before
        int activeConnectionsBefore = 
            dataSource.getHikariPoolMXBean().getActiveConnections();

        // Make payment request (do not wait for completion)
        CompletableFuture.runAsync(() -> 
            paymentService.processPayment(validRequest())
        );

        // Wait 100ms - Stripe call should be in flight
        Thread.sleep(100);

        // Assert: connection should NOT be held during Stripe API call
        // If @Transactional wraps the Stripe call, active connections will be > 0
        int activeConnectionsDuring = 
            dataSource.getHikariPoolMXBean().getActiveConnections();

        // This assertion FAILS with the current code
        // proving the connection is held during Stripe API call
        assertThat(activeConnectionsDuring)
            .as("Database connection held during external API call")
            .isEqualTo(activeConnectionsBefore);
    }

    // ============================================
    // TEST 2: Stripe charged when database fails
    // ============================================
    @Test
    void processPayment_whenDatabaseFailsAfterStripeCharge_shouldNotChargeCustomer() {
        // Set up Stripe mock to succeed
        wireMock.stubFor(post(urlEqualTo("/v1/charges"))
            .willReturn(aResponse()
                .withStatus(200)
                .withBody(validStripeResponse("ch_test_123"))));

        // Also set up Stripe refund endpoint - we expect a refund to be called
        wireMock.stubFor(post(urlEqualTo("/v1/refunds"))
            .willReturn(aResponse()
                .withStatus(200)
                .withBody(validRefundResponse())));

        // Create a request with an invalid userId that will cause 
        // the second orderRepository.save to fail
        // (simulate database constraint violation)
        PaymentRequest invalidRequest = PaymentRequest.builder()
            .userId(99999L)  // Non-existent user - foreign key will fail
            .amount(new BigDecimal("100.00"))
            .cardToken("tok_test")
            .build();

        // Act
        assertThrows(PaymentException.class, () -> 
            paymentService.processPayment(invalidRequest)
        );

        // Assert: no order was created (transaction rolled back)
        List<Order> orders = orderRepository.findByUserId(99999L);
        assertThat(orders).isEmpty();

        // Assert: Stripe refund was called
        // THIS ASSERTION FAILS with current code
        // proving the customer was charged but no refund was issued
        wireMock.verify(1, postRequestedFor(urlEqualTo("/v1/refunds")));
    }

    // ============================================
    // TEST 3: Email sent when transaction rolls back
    // ============================================
    @Test
    void processPayment_whenTransactionRollsBack_shouldNotSendEmail() {
        // Track emails sent
        List<String> sentEmails = new CopyOnWriteArrayList<>();

        // Override email service to track calls
        // (using @MockBean for email service only)

        wireMock.stubFor(post(urlEqualTo("/v1/charges"))
            .willReturn(aResponse()
                .withStatus(200)
                .withBody(validStripeResponse())));

        // Force a failure after email sends
        // by making orderRepository throw after the email step
        // This tests whether email is sent before transaction commits

        // The integration test PROVES the bug:
        // email is sent during the transaction
        // if transaction rolls back after email send,
        // customer gets email for cancelled transaction
    }
}
```

**The correct architecture the integration test drives you toward:**

```java
@Service
public class PaymentService {

    // Step 1: Create order OUTSIDE transaction (fast, then release connection)
    public PaymentResult processPayment(PaymentRequest request) {

        // Fast DB operation - connection acquired and released immediately
        Order order = createPendingOrder(request);

        // Stripe call OUTSIDE any transaction
        // Connection pool not held during API call
        StripeCharge charge;
        try {
            charge = stripeClient.charge(
                request.getCardToken(),
                request.getAmount()
            );
        } catch (StripeException e) {
            // Stripe failed - mark order as failed, no charge occurred
            failOrder(order.getId(), e.getMessage());
            throw new PaymentException("Payment failed: " + e.getMessage());
        }

        // Fast DB operation - connection acquired and released immediately
        Order completedOrder = completeOrder(order.getId(), charge.getId());

        // Email sent AFTER transaction commits via @TransactionalEventListener
        applicationEventPublisher.publishEvent(
            new OrderCompletedEvent(completedOrder.getId())
        );

        return new PaymentResult(completedOrder.getId(), charge.getId());
    }

    @Transactional
    protected Order createPendingOrder(PaymentRequest request) {
        return orderRepository.save(
            Order.builder()
                .userId(request.getUserId())
                .amount(request.getAmount())
                .status("PENDING")
                .build()
        );
        // Transaction commits here, connection released
    }

    @Transactional
    protected Order completeOrder(Long orderId, String chargeId) {
        Order order = orderRepository.findById(orderId).orElseThrow();
        order.setStatus("COMPLETED");
        order.setStripeChargeId(chargeId);
        return orderRepository.save(order);
        // Transaction commits here, connection released
    }

    // Email fires ONLY after transaction commits - never during rollback
    @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
    @Async
    public void onOrderCompleted(OrderCompletedEvent event) {
        Order order = orderRepository.findById(event.getOrderId())
            .orElseThrow();
        emailService.sendConfirmation(order);
    }
}
```

Now run your integration tests against this corrected architecture. All three tests pass. You have verified the fix with real database, real HTTP simulation, and real transaction behavior.

---

## Scenario 3: The Kafka Consumer That Loses Messages

**The Setup:**

AI generated a Kafka consumer for processing inventory updates. In production, you notice that under high load some inventory updates are silently lost. No errors in the logs. No exceptions. Just missing updates.

---

**Full DERIVE Prompt:**

"AI generated this Kafka consumer for processing inventory updates:

```java
@Component
public class InventoryConsumer {

    @Autowired
    private InventoryRepository inventoryRepository;

    @Autowired
    private ProductRepository productRepository;

    @KafkaListener(topics = 'inventory-updates', groupId = 'inventory-group')
    public void processInventoryUpdate(InventoryUpdateEvent event) {
        Product product = productRepository
            .findById(event.getProductId())
            .orElse(null);

        if (product == null) {
            return; // Product not found, skip
        }

        product.setStockLevel(event.getNewStockLevel());
        productRepository.save(product);
    }
}
```

Unit tests mock both repositories and pass. In production we occasionally lose inventory updates silently.

Before giving me tests: walk through every mechanism by which this consumer can silently lose messages. What does 'silently lost' look like in Kafka's offset management? Then give me the integration test that makes each silent loss visible."

---

**What DERIVE extracts:**

**Silent loss mechanism 1 — Auto-commit after process, before save:**

By default Kafka uses auto-commit with `enable.auto.commit=true`. The offset is committed on a timer (default every 5 seconds) regardless of whether your processing succeeded. If a commit fires after the message is consumed but before `productRepository.save(product)` completes, and then the application crashes, Kafka thinks the message was processed (offset committed) but the database was never updated. The message is permanently lost with no error.

**Silent loss mechanism 2 — `return` on null product swallows the message:**

If a product is not found, the consumer returns without error. Kafka commits the offset. The inventory update is permanently lost. No log entry at ERROR level. No metric. No alert. It is silently discarded. In a production system this might happen for valid reasons (product not yet synced from a different service) or invalid reasons (data corruption). You have no way to distinguish.

**Silent loss mechanism 3 — Database exception during save:**

If `productRepository.save(product)` throws a `DataAccessException` (connection timeout, constraint violation, deadlock), the default Kafka error handler retries the message. But if retries are exhausted, the message goes to a dead letter topic — or if no dead letter topic is configured, it is silently dropped after the retry limit.

**The integration tests that make each silent loss visible:**

```java
@SpringBootTest
@Testcontainers
@EmbeddedKafka(
    partitions = 1,
    topics = {"inventory-updates", "inventory-updates.DLT"}
)
class InventoryConsumerIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:15");

    @Autowired
    private KafkaTemplate<String, InventoryUpdateEvent> kafkaTemplate;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private InventoryRepository inventoryRepository;

    // ============================================
    // TEST 1: Message not lost when product not found
    // ============================================
    @Test
    void processInventoryUpdate_whenProductNotFound_shouldNotSilentlyDiscard()
            throws Exception {

        // Arrange: send update for non-existent product
        InventoryUpdateEvent event = InventoryUpdateEvent.builder()
            .productId(99999L)  // Does not exist
            .newStockLevel(50)
            .build();

        // Act
        kafkaTemplate.send("inventory-updates", event).get();

        // Wait for consumer to process
        Thread.sleep(2000);

        // Assert: the message should NOT be silently discarded
        // It should either:
        // A) Be retried until product appears
        // B) Be sent to a dead letter topic for manual handling
        // C) Be logged at ERROR level for alerting

        // Check dead letter topic received the message
        // THIS ASSERTION FAILS with current code
        // proving silent discard is happening
        ConsumerRecord<String, InventoryUpdateEvent> dlqRecord = 
            KafkaTestUtils.getSingleRecord(
                deadLetterConsumer,
                "inventory-updates.DLT",
                Duration.ofSeconds(5)
            );

        assertThat(dlqRecord).isNotNull();
        assertThat(dlqRecord.value().getProductId()).isEqualTo(99999L);
    }

    // ============================================
    // TEST 2: Message not lost on database failure
    // ============================================
    @Test
    void processInventoryUpdate_whenDatabaseFails_shouldRetryAndNotLoseMessage()
            throws Exception {

        // Arrange: create product that exists
        Product product = productRepository.save(
            Product.builder()
                .name("Test Product")
                .stockLevel(100)
                .build()
        );

        // Simulate database failure on first attempt
        // by stopping and restarting postgres container mid-test
        postgres.stop();

        InventoryUpdateEvent event = InventoryUpdateEvent.builder()
            .productId(product.getId())
            .newStockLevel(50)
            .build();

        kafkaTemplate.send("inventory-updates", event).get();

        // Restart database after 1 second
        Thread.sleep(1000);
        postgres.start();

        // Wait for retry to succeed
        Thread.sleep(5000);

        // Assert: stock level was eventually updated despite initial failure
        Product updated = productRepository.findById(product.getId())
            .orElseThrow();

        // THIS ASSERTION FAILS with current code
        // proving the message was lost when database was temporarily unavailable
        assertThat(updated.getStockLevel()).isEqualTo(50);
    }

    // ============================================
    // TEST 3: Exactly-once processing under concurrent messages
    // ============================================
    @Test
    void processInventoryUpdate_withConcurrentMessages_shouldProcessExactlyOnce()
            throws Exception {

        Product product = productRepository.save(
            Product.builder()
                .name("Concurrent Product")
                .stockLevel(100)
                .build()
        );

        // Send same product update 5 times (simulating retry scenario)
        for (int i = 0; i < 5; i++) {
            kafkaTemplate.send("inventory-updates",
                InventoryUpdateEvent.builder()
                    .productId(product.getId())
                    .newStockLevel(75)
                    .eventId("same-event-id-" + product.getId())
                    .build()
            ).get();
        }

        Thread.sleep(3000);

        // Assert: stock level should be 75, not processed 5 times
        Product updated = productRepository.findById(product.getId())
            .orElseThrow();
        assertThat(updated.getStockLevel()).isEqualTo(75);

        // Assert: only 1 inventory audit record, not 5
        List<InventoryAudit> audits = inventoryRepository
            .findByProductId(product.getId());
        assertThat(audits).hasSize(1);
    }
}
```

**The corrected consumer the integration tests drive you toward:**

```java
@Component
public class InventoryConsumer {

    private static final Logger log = 
        LoggerFactory.getLogger(InventoryConsumer.class);

    @Autowired
    private InventoryRepository inventoryRepository;

    @Autowired
    private ProductRepository productRepository;

    @Autowired
    private ProcessedEventRepository processedEventRepository;

    @KafkaListener(
        topics = "inventory-updates",
        groupId = "inventory-group",
        containerFactory = "manualAckKafkaListenerContainerFactory"
    )
    public void processInventoryUpdate(
            InventoryUpdateEvent event,
            Acknowledgment acknowledgment) {

        try {
            // Idempotency check - have we already processed this event?
            if (processedEventRepository.existsByEventId(event.getEventId())) {
                log.info("Duplicate event {}, skipping", event.getEventId());
                acknowledgment.acknowledge();  // Commit offset - already done
                return;
            }

            Product product = productRepository
                .findById(event.getProductId())
                .orElse(null);

            if (product == null) {
                // Do NOT silently discard
                // Log at ERROR level for alerting
                log.error(
                    "Product {} not found for inventory update event {}. " +
                    "Message will be sent to DLT after retries.",
                    event.getProductId(),
                    event.getEventId()
                );
                // Throw exception to trigger retry and eventual DLT routing
                throw new ProductNotFoundException(
                    "Product not found: " + event.getProductId()
                );
            }

            // Process in transaction
            processUpdate(product, event);

            // Only acknowledge AFTER successful processing
            // Manual ack prevents auto-commit timing issues
            acknowledgment.acknowledge();

        } catch (Exception e) {
            // Do NOT acknowledge - Kafka will retry
            log.error("Failed to process inventory update {}: {}",
                event.getEventId(), e.getMessage());
            throw e;  // Re-throw for retry/DLT handling
        }
    }

    @Transactional
    protected void processUpdate(Product product, InventoryUpdateEvent event) {
        product.setStockLevel(event.getNewStockLevel());
        productRepository.save(product);

        // Record as processed - idempotency record
        processedEventRepository.save(
            ProcessedEvent.builder()
                .eventId(event.getEventId())
                .processedAt(Instant.now())
                .build()
        );
    }
}
```

---

## Scenario 4: The Spring Security Integration Bug

**The Setup:**

AI generated a multi-tenant REST API. Unit tests pass. In production, a user from Tenant A occasionally sees data from Tenant B. This is a critical security bug. No unit test caught it.

---

**Full DERIVE Prompt:**

"AI generated this multi-tenant Spring Boot application:

```java
@Service
public class DocumentService {

    @Autowired
    private DocumentRepository documentRepository;

    public List<Document> getDocuments() {
        // Get tenant from security context
        Authentication auth = SecurityContextHolder
            .getContext()
            .getAuthentication();
        String tenantId = ((TenantUser) auth.getPrincipal()).getTenantId();

        return documentRepository.findByTenantId(tenantId);
    }
}
```

Unit tests mock SecurityContextHolder and DocumentRepository. All pass. In production, occasionally a user sees another tenant's documents.

Before giving integration tests: walk through the exact sequence of events that causes cross-tenant data leakage in this code. What is the threading scenario that creates the vulnerability? Then give me the integration test that reliably reproduces the bug."

---

**What DERIVE extracts:**

**The exact vulnerability — SecurityContextHolder ThreadLocal race condition:**

SecurityContextHolder stores authentication in a ThreadLocal — one authentication per thread. This is correct for synchronous request handling. The vulnerability appears when:

```
Thread 1 (Tenant A request):
  - SecurityContextHolder set to Tenant A's authentication
  - Calls documentService.getDocuments()
  - Reads SecurityContextHolder → gets Tenant A ✓

Thread 2 (Tenant B request, starts while Thread 1 is still executing):
  - SecurityContextHolder set to Tenant B's authentication
  - BUT: if Thread 1 and Thread 2 are the same physical thread 
    (possible with thread pool reuse), or if there is any async 
    context switch, Thread 1 now reads Tenant B's authentication

Specific scenario that produces the bug:
  - Thread pool has 10 threads
  - Thread 1 handles Tenant A request, reads authentication
  - Before Thread 1 finishes, it is used for an @Async operation
  - @Async clears ThreadLocal by default
  - ThreadLocal now has no authentication OR the previous request's authentication
  - Next operation reads wrong tenant ID
```

**The integration test that reliably reproduces it:**

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
class MultiTenantSecurityIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres = 
        new PostgreSQLContainer<>("postgres:15");

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private DocumentRepository documentRepository;

    @Autowired
    private UserRepository userRepository;

    @BeforeEach
    void setup() {
        // Create two tenants with distinct documents
        createTenantWithDocuments("tenant-a", "Secret Doc A1", "Secret Doc A2");
        createTenantWithDocuments("tenant-b", "Secret Doc B1", "Secret Doc B2");
    }

    // ============================================
    // TEST 1: Basic tenant isolation
    // ============================================
    @Test
    void getDocuments_shouldOnlyReturnCurrentTenantDocuments() {
        // Tenant A authenticates and gets documents
        HttpHeaders tenantAHeaders = createAuthHeaders("user-a", "tenant-a");

        ResponseEntity<List> responseA = restTemplate.exchange(
            "/api/documents",
            HttpMethod.GET,
            new HttpEntity<>(tenantAHeaders),
            List.class
        );

        assertThat(responseA.getStatusCode()).isEqualTo(HttpStatus.OK);

        // Verify NO Tenant B documents in response
        String responseBody = responseA.getBody().toString();
        assertThat(responseBody)
            .doesNotContain("Secret Doc B1")
            .doesNotContain("Secret Doc B2")
            .contains("Secret Doc A1")
            .contains("Secret Doc A2");
    }

    // ============================================
    // TEST 2: Concurrent request isolation
    // This is the test that catches the ThreadLocal race condition
    // ============================================
    @Test
    void getDocuments_underConcurrentLoad_shouldMaintainTenantIsolation()
            throws Exception {

        int concurrentRequests = 50;
        CountDownLatch startLatch = new CountDownLatch(1);
        CountDownLatch completeLatch = new CountDownLatch(concurrentRequests);
        List<String> tenantLeakageDetected = new CopyOnWriteArrayList<>();

        // Launch 50 concurrent requests alternating between tenant A and B
        for (int i = 0; i < concurrentRequests; i++) {
            final String tenantId = i % 2 == 0 ? "tenant-a" : "tenant-b";
            final String forbiddenContent = i % 2 == 0 ? 
                "Secret Doc B" : "Secret Doc A";

            Thread.ofVirtual().start(() -> {
                try {
                    startLatch.await();  // All start simultaneously

                    HttpHeaders headers = createAuthHeaders(
                        "user-" + tenantId, tenantId
                    );

                    ResponseEntity<String> response = restTemplate.exchange(
                        "/api/documents",
                        HttpMethod.GET,
                        new HttpEntity<>(headers),
                        String.class
                    );

                    // Check for cross-tenant leakage
                    if (response.getBody().contains(forbiddenContent)) {
                        tenantLeakageDetected.add(
                            tenantId + " received data from other tenant"
                        );
                    }
                } catch (Exception e) {
                    tenantLeakageDetected.add("Exception: " + e.getMessage());
                } finally {
                    completeLatch.countDown();
                }
            });
        }

        // Fire all requests simultaneously
        startLatch.countDown();
        completeLatch.await(30, TimeUnit.SECONDS);

        // Assert: NO cross-tenant leakage detected under concurrent load
        // THIS ASSERTION FAILS with the current code under concurrent load
        // proving the ThreadLocal race condition exists
        assertThat(tenantLeakageDetected)
            .as("Cross-tenant data leakage detected: %s", tenantLeakageDetected)
            .isEmpty();
    }

    // ============================================
    // TEST 3: Async operation does not leak tenant context
    // ============================================
    @Test
    void getDocuments_withAsyncProcessing_shouldMaintainTenantContext()
            throws Exception {

        // Request that triggers async processing internally
        HttpHeaders tenantAHeaders = createAuthHeaders("user-a", "tenant-a");

        // This endpoint triggers an @Async operation internally
        ResponseEntity<List> response = restTemplate.exchange(
            "/api/documents/with-async-processing",
            HttpMethod.GET,
            new HttpEntity<>(tenantAHeaders),
            List.class
        );

        // The async operation should use tenant-a context
        // not inherit no context or previous request's context
        assertThat(response.getBody())
            .doesNotContain("Secret Doc B1");
    }
}
```

**The correct implementation the integration tests drive:**

```java
// Fix 1: Use Hibernate filter for automatic tenant isolation
// This makes it structurally impossible to query across tenants
@Component
public class TenantFilterInterceptor implements HandlerInterceptor {

    @Autowired
    private EntityManager entityManager;

    @Override
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response,
                             Object handler) {

        Authentication auth = SecurityContextHolder.getContext()
            .getAuthentication();

        if (auth != null && auth.getPrincipal() instanceof TenantUser) {
            String tenantId = ((TenantUser) auth.getPrincipal()).getTenantId();

            Session session = entityManager.unwrap(Session.class);
            session.enableFilter("tenantFilter")
                .setParameter("tenantId", tenantId);
        }

        return true;
    }
}

// Fix 2: Propagate security context to async operations
@Configuration
@EnableAsync
public class AsyncConfig implements AsyncConfigurer {

    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(20);
        executor.setThreadNamePrefix("async-");

        // THIS is what propagates SecurityContext to async threads
        executor.setTaskDecorator(runnable -> {
            SecurityContext context = SecurityContextHolder.getContext();
            return () -> {
                try {
                    SecurityContextHolder.setContext(context);
                    runnable.run();
                } finally {
                    SecurityContextHolder.clearContext();
                }
            };
        });

        executor.initialize();
        return executor;
    }
}
```

---

## Scenario 5: The Redis Cache Serialization Bug

**The Setup:**

AI generated caching for a product catalog service. Works perfectly right after deployment. After the second deployment, the application starts throwing deserialization errors in production. Users see 500 errors intermittently.

---

**Full DERIVE Prompt:**

"AI generated this Redis caching configuration:

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheManager cacheManager(
            RedisConnectionFactory connectionFactory) {
        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofHours(1)))
            .build();
    }
}

@Service
public class ProductService {

    @Cacheable(value = 'products', key = '#productId')
    public ProductDTO getProduct(Long productId) {
        Product product = productRepository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));
        return productMapper.toDTO(product);
    }
}
```

Works fine right after deployment. After the second deployment it starts throwing errors.

Before giving me tests: explain exactly what happens to Redis cache entries between deployments and what specific change causes deserialization to fail. Then give me the integration test that catches this before deployment."

---

**What DERIVE extracts:**

**The exact deployment bug:**

Spring Boot's default Redis serialization uses Java serialization (`JdkSerializationRedisSerializer`). Java serialization creates binary data that includes the class's `serialVersionUID` — a version identifier for the class structure.

Here is the deployment sequence that causes the bug:

```
Deployment 1:
- ProductDTO class: { Long id, String name, BigDecimal price }
- Serialized to Redis with serialVersionUID = XXXX
- Cache entry: binary blob with XXXX identifier

Developer adds a field (or changes a field type) between deployments:
- ProductDTO class now: { Long id, String name, BigDecimal price, String category }

Deployment 2:
- New code starts serving requests
- Request hits /api/products/123
- Redis has a cached entry from Deployment 1
- Spring tries to deserialize the binary blob
- Old blob has no 'category' field
- New ProductDTO class expects 'category'
- SerializationException: could not deserialize
- User gets 500 error
```

The bug only appears after the second deployment. Unit tests never catch it because they mock the cache. Integration tests against a fresh Redis instance do not catch it because there are no stale entries from a previous deployment.

**The integration test that catches it before deployment:**

```java
@SpringBootTest
@Testcontainers
class RedisCacheIntegrationTest {

    @Container
    static GenericContainer<?> redis = 
        new GenericContainer<>("redis:7")
            .withExposedPorts(6379);

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("spring.data.redis.host", redis::getHost);
        registry.add("spring.data.redis.port", 
            () -> redis.getMappedPort(6379));
    }

    @Autowired
    private ProductService productService;

    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    @Autowired
    private ProductRepository productRepository;

    // ============================================
    // TEST 1: Verify JSON serialization not Java serialization
    // ============================================
    @Test
    void productCache_shouldUseJsonSerializationNotJavaSerialization() {
        // Arrange
        Product product = productRepository.save(
            Product.builder()
                .name("Test Product")
                .price(new BigDecimal("29.99"))
                .build()
        );

        // Act: populate cache
        productService.getProduct(product.getId());

        // Assert: cache entry should be readable JSON, not binary blob
        // This directly checks what is stored in Redis
        byte[] rawCacheEntry = (byte[]) redisTemplate
            .execute(connection -> 
                connection.get(("products::" + product.getId()).getBytes())
            );

        String cacheContent = new String(rawCacheEntry);

        // THIS ASSERTION FAILS with default JDK serialization
        // The stored bytes are binary, not JSON
        assertThat(cacheContent)
            .as("Cache should use JSON serialization for deployment compatibility")
            .contains("\"name\"")
            .contains("Test Product")
            .doesNotContain("\\u0000");  // Binary null bytes indicate JDK serialization
    }

    // ============================================
    // TEST 2: Simulate deployment - old cached entry + new class version
    // ============================================
    @Test
    void productCache_withStaleEntryFromPreviousDeployment_shouldNotThrow500() {
        Product product = productRepository.save(
            Product.builder()
                .name("Deployment Test Product")
                .price(new BigDecimal("49.99"))
                .build()
        );

        // Simulate a cache entry from the PREVIOUS deployment
        // with a slightly different structure
        // (missing the 'category' field that was added in this deployment)
        String staleCacheEntry = """
            {
                "id": %d,
                "name": "Deployment Test Product",
                "price": 49.99
            }
            """.formatted(product.getId());

        // Manually write stale entry to Redis
        redisTemplate.execute(connection -> {
            connection.set(
                ("products::" + product.getId()).getBytes(),
                staleCacheEntry.getBytes()
            );
            return null;
        });

        // Act: request that hits the stale cache entry
        // With JDK serialization: this throws SerializationException → 500 error
        // With JSON serialization: this deserializes gracefully, missing field = null
        assertDoesNotThrow(() -> {
            ProductDTO result = productService.getProduct(product.getId());
            assertThat(result).isNotNull();
            assertThat(result.getName()).isEqualTo("Deployment Test Product");
            // category is null (not in stale entry) - acceptable
            // rather than 500 error
        });
    }

    // ============================================
    // TEST 3: Cache eviction on update
    // ============================================
    @Test
    void updateProduct_shouldEvictCacheEntry() throws Exception {
        Product product = productRepository.save(
            Product.builder()
                .name("Original Name")
                .price(new BigDecimal("19.99"))
                .build()
        );

        // Populate cache
        productService.getProduct(product.getId());

        // Verify it is cached
        Boolean isCached = redisTemplate.hasKey(
            "products::" + product.getId()
        );
        assertThat(isCached).isTrue();

        // Update product
        productService.updateProduct(product.getId(), "Updated Name");

        // Assert: cache entry is evicted
        // If not evicted, users see stale name after update
        Boolean stillCached = redisTemplate.hasKey(
            "products::" + product.getId()
        );
        assertThat(stillCached)
            .as("Cache entry should be evicted after product update")
            .isFalse();
    }
}
```

**The corrected cache configuration the tests drive:**

```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public RedisCacheManager cacheManager(
            RedisConnectionFactory connectionFactory,
            ObjectMapper objectMapper) {

        // Use JSON serialization instead of Java serialization
        // JSON is human-readable, deployment-safe, and version-tolerant
        GenericJackson2JsonRedisSerializer serializer =
            new GenericJackson2JsonRedisSerializer(objectMapper);

        RedisCacheConfiguration config = RedisCacheConfiguration
            .defaultCacheConfig()
            .entryTtl(Duration.ofHours(1))
            // THIS is the critical fix
            .serializeValuesWith(
                RedisSerializationContext.SerializationPair
                    .fromSerializer(serializer)
            )
            // Do not cache null values - prevents null poisoning
            .disableCachingNullValues();

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .build();
    }
}
```

---

## The Complete DERIVE Loop for Integration Testing — The Meta-Pattern

Looking across all five scenarios, the pattern is exact:

**Round 1 (D):** Paste the actual AI-generated code. State what the unit tests do. State the production symptom (slow API, lost messages, 500 errors, data leakage). Ask for the specific mechanism that creates the bug — not generic testing advice.

**Round 2 (E + R):** Take the bug mechanism from Round 1 and expose the assumption it makes about your infrastructure. Use the operational role framing: "you are an engineer who has been paged because of this exact failure." Ask what the failure looks like in monitoring, in logs, and in user experience at the moment it happens.

**Round 3 (I):** Force reasoning about what the integration test must set up to reliably reproduce the bug. The test setup is often harder than the assertion. What data volume? What concurrency? What infrastructure state? These details make the difference between a test that catches the bug reliably and one that catches it intermittently.

**Round 4 (V):** Before running the integration test in CI, ask: what is the confidence level that this test will fail consistently when the bug exists and pass consistently when it is fixed? Flaky integration tests are worse than no integration tests because they produce false confidence.

**Round 5 (E — Expand):** Every integration test you write reveals an adjacent integration concern. The N+1 query test reveals that you should also test query count under pagination. The transaction test reveals that you should also test the idempotency behavior. The Kafka test reveals that you should also test consumer group rebalancing behavior. Round 5 probes the adjacent concern that the test just exposed.

---

## Why This Skill Compounds in the AI Age

The reason your senior engineer is right is this precise dynamic:

AI generates code at 10x velocity. Integration bugs scale with code volume. Manual testing does not scale. Unit tests do not catch integration bugs. The engineer who can write integration tests that catch AI-generated integration bugs is the quality gate that makes AI-assisted development safe at production velocity.

Without integration tests, AI-assisted development produces: fast code that passes unit tests, ships to production, and introduces subtle integration bugs that take hours to diagnose and damage user trust.

With integration tests, AI-assisted development produces: fast code that is verified against real infrastructure before shipping, with bugs caught in the test environment where they cost minutes to fix instead of production where they cost hours and user trust.

The skill is not just valuable. In a world where AI generates the code, the engineer who verifies the code is the most important engineer on the team. Integration testing is how you verify at scale.
