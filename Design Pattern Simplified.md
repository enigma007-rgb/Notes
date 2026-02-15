https://freedium-mirror.cfd/https://medium.com/womenintechnology/stop-memorizing-design-patterns-use-this-decision-tree-instead-e84f22fca9fa
---

I'll fetch and analyze this article about design patterns for you.This is an excellent article about choosing design patterns pragmatically. Let me break down the key concepts and structure in depth:

## Core Philosophy

The article's fundamental premise is that design patterns fail not because they're wrong, but because they're applied at inappropriate moments or as substitutes for identifying the actual problem. Instead of memorizing patterns, the author advocates describing the specific friction you're trying to eliminate first.

## The Three-Question Framework

The decision tree organizes all patterns around **one core question: where is the pain coming from?** This then branches into three categories:

### 1. **Creating Objects** (Creational Patterns)
**When to use:** Object creation has become complex with too many parameters, unclear defaults, or scattered "which implementation?" logic.

**Key Patterns:**

- **Singleton** - Use only when you genuinely need exactly one instance. The author warns against using it just for "easy access" since it hides dependencies and complicates testing. Best for stateless or safely-shareable objects like read-only configs or loggers.

- **Builder** - Apply when constructors accumulate optional arguments and configuration combinations matter. The goal isn't aesthetic chaining but making object creation explicit with early validation. Example: instead of `Request.new(url, method, headers, body, timeout, retry_count, cache, auth)`, you get readable, validated construction.

- **Factory Method** - Centralize creation decisions when you're choosing implementations based on context (file type, provider, feature flags). Works when a base class defines a contract and subclasses decide concrete types.

- **Abstract Factory** - For families of related objects that must match (like provider-specific client + mapper + validator).

- **Prototype** - When cloning a configured object is cheaper than rebuilding it, especially with expensive initialization.

### 2. **Structuring Objects** (Structural Patterns)
**When to use:** Code is correct but awkward because boundaries are unclear, external interfaces leak into application logic, or composition is difficult.

**Key Patterns:**

- **Adapter** - Bridge incompatible interfaces. Protects your domain from vendor-specific shapes. The author's rule: keep adapters focused on translation; when they start containing business rules, split those into separate components.

- **Facade** - Simplify complex subsystems with multiple steps that must be invoked correctly. Makes the safe path easy (example: video conversion workflow with probing, transcoding, metadata extraction, storage, cleanup).

- **Decorator** - Add optional features without subclass explosion. For combinations like logging + encryption + compression + caching. Each wrapper should be small and predictable.

- **Proxy** - Create stand-in objects for lazy loading, caching, access control, instrumentation, or remote calls behind local-looking interfaces.

- **Composite** - For tree structures where you want uniform treatment of leaf nodes and containers (file systems, UI components, nested content).

- **Flyweight** - Optimize memory when many objects share identical data and duplication is expensive (editors, renderers, large in-memory models).

- **Bridge** - Vary abstraction and implementation independently to avoid a matrix of subclasses (like "export format" vs "export destination").

### 3. **Handling Behaviour** (Behavioural Patterns)
**When to use:** The main issue is changing rules and workflow logic—methods accumulating `if` branches, algorithms changing per customer, or hard-to-extend pipelines.

**Key Patterns:**

- **Chain of Responsibility** - For middleware-style pipelines where each step may stop or pass requests along. Stays healthy when each handler has single responsibility and clear "stop vs continue" contracts.

- **Command** - Represent actions as objects for operational benefits like job queues, retries, audit logs, undo stacks, or "run later" workflows.

- **Strategy** - Swap algorithms without changing the caller. High-ROI for multiple implementations of the same behavior—common for payment providers, routing decisions, recommendation policies, rate limiting. The practical signal is recurring branching like "if plan is X do this, if plan is Y do that."

- **State** - For well-defined modes and transitions (connections, approval workflows, session lifecycles). Makes behavior explicit per state, reducing complex branching.

- **Observer** - One-to-many notifications for subscription-style updates. Can be clean with domain events but can hide control flow—keep observers visible.

- **Memento** - Snapshots and restores for undo features, rollbacks, and version restoration without exposing internal representation.

- **Mediator** - Coordinate so objects don't depend directly on each other. Risk is concentrating too much logic in one place; stays manageable with narrow, well-defined responsibilities.

- **Visitor** - Add new operations over stable object structures (like ASTs) without changing the structure itself.

## Practical Applications

The article provides three real-world scenarios:

1. **Notification delivery** (email, SMS, push) → **Strategy** lets you add channels without rewriting callers

2. **API request processing** (rate limiting → auth → handler) → **Chain of Responsibility** keeps steps small, testable, and reorderable

3. **Report generation** (many options, formats) → **Builder** for configuration + **Strategy** for format selection

## Key Takeaways

Patterns earn their place when they reduce recurring costs: changes requiring too many file touches, slow/brittle tests with no clean seams, external APIs leaking everywhere, growing constructors, or duplicated logic.

The mistake is treating patterns as upgrades in themselves. A pattern is a way of paying for flexibility in a controlled place instead of paying for it everywhere, repeatedly.

This approach transforms pattern selection from memorization to diagnosis: identify your pain point, follow the decision tree, and choose the pattern that addresses your specific friction.

------

I'll create a comprehensive guide with real-world scenarios for each design pattern. Let me break this down with practical examples you'd encounter in actual software development.

## **CREATIONAL PATTERNS: The "How Do I Create Objects?" Problems**

---

### **1. SINGLETON - "I Need Exactly One Instance"**

**Real Scenario: Application Configuration Manager**

Imagine you're building an e-commerce application. You have a configuration file that stores API keys, database connections, feature flags, etc.

**The Problem Without Singleton:**
```python
# Every time someone needs config, they create a new instance
config1 = Configuration()  # Reads file from disk
config2 = Configuration()  # Reads file from disk AGAIN
config3 = Configuration()  # Reads file from disk AGAIN

# This is wasteful and causes issues:
# - Multiple file I/O operations
# - Inconsistent state if file changes
# - Memory waste with duplicate objects
```

**Solution with Singleton:**
```python
class Configuration:
    _instance = None
    _config_data = None
    
    def __new__(cls):
        if cls._instance is None:
            cls._instance = super().__new__(cls)
            # Load config only once
            cls._config_data = cls._load_from_file()
        return cls._instance
    
    @staticmethod
    def _load_from_file():
        # Expensive operation - read from disk
        return {
            'api_key': 'xyz123',
            'db_host': 'localhost',
            'max_connections': 100
        }
    
    def get(self, key):
        return self._config_data.get(key)

# Usage:
config1 = Configuration()
config2 = Configuration()
config3 = Configuration()

print(config1 is config2 is config3)  # True - same instance!
# File is read only ONCE, no matter how many times we "create" it
```

**When NOT to Use:**
```python
# BAD: Using Singleton for shopping cart
class ShoppingCart:  # DON'T DO THIS
    _instance = None
    # This would be shared across ALL users!
    # User A's items would appear in User B's cart!
```

---

### **2. BUILDER - "Construction Has Too Many Options"**

**Real Scenario: Building HTTP Requests**

You're creating an API client that needs to make HTTP requests with various options.

**The Problem Without Builder:**
```python
# Confusing constructor with 10+ parameters
request = HTTPRequest(
    'https://api.example.com/users',
    'POST',
    {'Authorization': 'Bearer token'},
    {'name': 'John', 'email': 'john@example.com'},
    30,  # timeout
    3,   # max_retries
    True, # follow_redirects
    False, # verify_ssl
    'gzip', # encoding
    None  # proxy
)

# What does each parameter mean? Easy to make mistakes!
# What if I only want to set timeout and headers?
```

**Solution with Builder:**
```python
class HTTPRequest:
    def __init__(self, url):
        self.url = url
        self.method = 'GET'
        self.headers = {}
        self.body = None
        self.timeout = 30
        self.max_retries = 0
        self.follow_redirects = True
        self.verify_ssl = True
        self.encoding = None
        self.proxy = None

class HTTPRequestBuilder:
    def __init__(self, url):
        self._request = HTTPRequest(url)
    
    def method(self, method):
        self._request.method = method
        return self  # Return self for chaining
    
    def headers(self, headers):
        self._request.headers = headers
        return self
    
    def body(self, body):
        self._request.body = body
        return self
    
    def timeout(self, seconds):
        if seconds <= 0:
            raise ValueError("Timeout must be positive")
        self._request.timeout = seconds
        return self
    
    def retries(self, count):
        self._request.max_retries = count
        return self
    
    def build(self):
        # Validation happens here
        if self._request.method == 'POST' and not self._request.body:
            raise ValueError("POST requests need a body")
        return self._request

# Usage - Much clearer!
request = (HTTPRequestBuilder('https://api.example.com/users')
    .method('POST')
    .headers({'Authorization': 'Bearer token'})
    .body({'name': 'John', 'email': 'john@example.com'})
    .timeout(45)
    .retries(3)
    .build())

# Or a simple GET:
simple_request = (HTTPRequestBuilder('https://api.example.com/status')
    .timeout(10)
    .build())
```

**Step-by-Step Breakdown:**
1. **Create builder** with required parameter (URL)
2. **Chain optional settings** - only add what you need
3. **Validation happens in `build()`** - catches errors early
4. **Each method returns `self`** - enables chaining
5. **Final `build()` call** produces the validated object

---

### **3. FACTORY METHOD - "I Need Different Implementations"**

**Real Scenario: Payment Processing System**

Your e-commerce site accepts multiple payment methods: Credit Card, PayPal, Crypto.

**The Problem Without Factory:**
```python
# Scattered logic everywhere
def checkout(payment_type, amount):
    if payment_type == 'credit_card':
        processor = CreditCardProcessor()
        processor.validate_card()
        processor.charge(amount)
    elif payment_type == 'paypal':
        processor = PayPalProcessor()
        processor.authenticate()
        processor.charge(amount)
    elif payment_type == 'crypto':
        processor = CryptoProcessor()
        processor.verify_wallet()
        processor.charge(amount)
    # Every new payment type requires modifying this code!
```

**Solution with Factory Method:**
```python
from abc import ABC, abstractmethod

# Step 1: Define the interface
class PaymentProcessor(ABC):
    @abstractmethod
    def process_payment(self, amount):
        pass

# Step 2: Concrete implementations
class CreditCardProcessor(PaymentProcessor):
    def process_payment(self, amount):
        print(f"Processing ${amount} via Credit Card")
        # Validate card, check funds, charge
        return {'status': 'success', 'transaction_id': 'CC123'}

class PayPalProcessor(PaymentProcessor):
    def process_payment(self, amount):
        print(f"Processing ${amount} via PayPal")
        # Authenticate, redirect, confirm
        return {'status': 'success', 'transaction_id': 'PP456'}

class CryptoProcessor(PaymentProcessor):
    def process_payment(self, amount):
        print(f"Processing ${amount} via Cryptocurrency")
        # Verify wallet, create transaction, wait for confirmation
        return {'status': 'pending', 'transaction_id': 'BTC789'}

# Step 3: The Factory
class PaymentProcessorFactory:
    @staticmethod
    def create_processor(payment_type):
        processors = {
            'credit_card': CreditCardProcessor,
            'paypal': PayPalProcessor,
            'crypto': CryptoProcessor
        }
        
        processor_class = processors.get(payment_type)
        if not processor_class:
            raise ValueError(f"Unknown payment type: {payment_type}")
        
        return processor_class()

# Usage - Clean and extensible!
def checkout(payment_type, amount):
    processor = PaymentProcessorFactory.create_processor(payment_type)
    result = processor.process_payment(amount)
    return result

# In your code:
checkout('credit_card', 99.99)
checkout('paypal', 149.99)
checkout('crypto', 0.005)

# Adding a new payment type? Just add a new class!
class ApplePayProcessor(PaymentProcessor):
    def process_payment(self, amount):
        print(f"Processing ${amount} via Apple Pay")
        return {'status': 'success', 'transaction_id': 'AP999'}

# Register it in the factory - that's it!
```

**Why This Works:**
1. **Open/Closed Principle**: Open for extension (new processors), closed for modification (existing code unchanged)
2. **Single place** to add new payment types
3. **Consistent interface** - all processors have `process_payment()`
4. **Easy testing** - mock the factory in tests

---

### **4. ABSTRACT FACTORY - "I Need Related Objects That Match"**

**Real Scenario: Multi-Theme UI Components**

You're building an application that supports Light Mode and Dark Mode, and need buttons, text fields, and checkboxes that all match the theme.

**The Problem Without Abstract Factory:**
```python
# Inconsistent theming - easy to mix and match wrong components
button = DarkButton()  # Oops!
text_field = LightTextField()  # These don't match!
checkbox = DarkCheckbox()
```

**Solution with Abstract Factory:**
```python
from abc import ABC, abstractmethod

# Step 1: Define component interfaces
class Button(ABC):
    @abstractmethod
    def render(self):
        pass

class TextField(ABC):
    @abstractmethod
    def render(self):
        pass

class Checkbox(ABC):
    @abstractmethod
    def render(self):
        pass

# Step 2: Light theme family
class LightButton(Button):
    def render(self):
        return "<button style='background: white; color: black'>Click</button>"

class LightTextField(TextField):
    def render(self):
        return "<input style='background: white; border: 1px solid gray'>"

class LightCheckbox(Checkbox):
    def render(self):
        return "<input type='checkbox' style='border: 1px solid gray'>"

# Step 3: Dark theme family
class DarkButton(Button):
    def render(self):
        return "<button style='background: #333; color: white'>Click</button>"

class DarkTextField(TextField):
    def render(self):
        return "<input style='background: #222; border: 1px solid #444; color: white'>"

class DarkCheckbox(Checkbox):
    def render(self):
        return "<input type='checkbox' style='border: 1px solid #444'>"

# Step 4: The Abstract Factory
class UIFactory(ABC):
    @abstractmethod
    def create_button(self):
        pass
    
    @abstractmethod
    def create_text_field(self):
        pass
    
    @abstractmethod
    def create_checkbox(self):
        pass

class LightThemeFactory(UIFactory):
    def create_button(self):
        return LightButton()
    
    def create_text_field(self):
        return LightTextField()
    
    def create_checkbox(self):
        return LightCheckbox()

class DarkThemeFactory(UIFactory):
    def create_button(self):
        return DarkButton()
    
    def create_text_field(self):
        return DarkTextField()
    
    def create_checkbox(self):
        return DarkCheckbox()

# Step 5: Application code
class LoginForm:
    def __init__(self, factory: UIFactory):
        self.factory = factory
    
    def render(self):
        button = self.factory.create_button()
        text_field = self.factory.create_text_field()
        checkbox = self.factory.create_checkbox()
        
        return f"""
        Username: {text_field.render()}
        Password: {text_field.render()}
        Remember me: {checkbox.render()}
        {button.render()}
        """

# Usage - Components are GUARANTEED to match!
user_preference = 'dark'  # From settings

if user_preference == 'dark':
    factory = DarkThemeFactory()
else:
    factory = LightThemeFactory()

login_form = LoginForm(factory)
print(login_form.render())
# All components will be consistently themed!
```

**Real-World Benefit:**
- **Adding a theme?** Create new factory + new component classes
- **Impossible to mix themes** - factory ensures consistency
- **Components always compatible** - they're designed to work together

---

## **STRUCTURAL PATTERNS: The "How Do Objects Fit Together?" Problems**

---

### **5. ADAPTER - "I Need to Connect Incompatible Interfaces"**

**Real Scenario: Integrating Third-Party Payment APIs**

Your application uses your own `PaymentGateway` interface, but you need to integrate Stripe, which has a completely different API.

**The Problem:**
```python
# Your existing interface
class PaymentGateway:
    def charge(self, amount, card_token):
        pass

# Your code expects this interface
def process_order(gateway: PaymentGateway, amount, card):
    result = gateway.charge(amount, card)
    return result

# But Stripe's API is completely different:
import stripe
stripe.api_key = 'sk_test_xyz'

# Stripe uses: stripe.Charge.create(amount=, source=, currency=)
# This doesn't match your interface!
```

**Solution with Adapter:**
```python
import stripe

# Step 1: Your standard interface
class PaymentGateway:
    def charge(self, amount, card_token):
        pass

# Step 2: Stripe Adapter - translates your interface to Stripe's
class StripeAdapter(PaymentGateway):
    def __init__(self, api_key):
        stripe.api_key = api_key
    
    def charge(self, amount, card_token):
        # Translate your interface to Stripe's format
        try:
            charge = stripe.Charge.create(
                amount=int(amount * 100),  # Stripe uses cents
                source=card_token,
                currency='usd'
            )
            return {
                'success': True,
                'transaction_id': charge.id,
                'amount': amount
            }
        except stripe.error.CardError as e:
            return {
                'success': False,
                'error': str(e)
            }

# Step 3: Let's add PayPal too - different API again
class PayPalAdapter(PaymentGateway):
    def __init__(self, client_id, secret):
        self.client_id = client_id
        self.secret = secret
    
    def charge(self, amount, card_token):
        # PayPal's completely different API
        # paypal.Payment.create(...) with different parameters
        
        # Adapter translates it to your standard format
        return {
            'success': True,
            'transaction_id': 'PAYPAL_123',
            'amount': amount
        }

# Step 4: Your application code stays THE SAME!
def process_order(gateway: PaymentGateway, amount, card):
    result = gateway.charge(amount, card)
    
    if result['success']:
        print(f"Payment successful: {result['transaction_id']}")
    else:
        print(f"Payment failed: {result.get('error')}")
    
    return result

# Usage - switch providers without changing code!
stripe_gateway = StripeAdapter('sk_test_xyz')
process_order(stripe_gateway, 99.99, 'tok_visa')

paypal_gateway = PayPalAdapter('client_id', 'secret')
process_order(paypal_gateway, 99.99, 'pp_token')

# Switch providers via configuration!
config = {'provider': 'stripe'}
gateway = StripeAdapter('key') if config['provider'] == 'stripe' else PayPalAdapter('id', 'secret')
process_order(gateway, 99.99, 'token')
```

**Step-by-Step Benefits:**
1. **Your code** doesn't know about Stripe or PayPal specifics
2. **Easy to switch providers** - just swap the adapter
3. **Easy to test** - mock the adapter interface
4. **Future-proof** - new provider? Create new adapter, done!

---

### **6. FACADE - "The Subsystem Is Too Complex"**

**Real Scenario: Video Processing System**

You need to convert uploaded videos: extract metadata, transcode, generate thumbnail, upload to storage, send notification.

**The Problem Without Facade:**
```python
# Every developer needs to remember this complex sequence:
def handle_video_upload(video_file):
    # Step 1: Validate
    validator = VideoValidator()
    if not validator.check_format(video_file):
        raise Exception("Invalid format")
    if not validator.check_size(video_file):
        raise Exception("File too large")
    
    # Step 2: Extract metadata
    metadata_extractor = FFProbeMetadata()
    metadata = metadata_extractor.probe(video_file)
    metadata_extractor.parse_streams(metadata)
    duration = metadata_extractor.get_duration()
    
    # Step 3: Transcode
    transcoder = FFMpegTranscoder()
    transcoder.set_input(video_file)
    transcoder.set_codec('h264')
    transcoder.set_quality('high')
    output = transcoder.process()
    
    # Step 4: Thumbnail
    thumbnailer = ThumbnailGenerator()
    thumbnailer.seek_to(duration / 2)
    thumbnail = thumbnailer.extract_frame(output)
    thumbnail_file = thumbnailer.save(thumbnail)
    
    # Step 5: Upload
    s3_client = boto3.client('s3')
    s3_client.upload_file(output, 'bucket', 'videos/output.mp4')
    s3_client.upload_file(thumbnail_file, 'bucket', 'thumbnails/thumb.jpg')
    
    # Step 6: Cleanup
    os.remove(video_file)
    os.remove(output)
    os.remove(thumbnail_file)
    
    # Step 7: Notify
    notifier = NotificationService()
    notifier.send_email("Video processed!")
    
    # This is 40+ lines of complex orchestration
    # Easy to forget steps, make mistakes, hard to maintain
```

**Solution with Facade:**
```python
# The complex subsystems still exist, but...
class VideoValidator:
    def validate(self, file):
        # Complex validation logic
        return True

class VideoMetadata:
    def extract(self, file):
        # FFProbe operations
        return {'duration': 120, 'resolution': '1920x1080'}

class VideoTranscoder:
    def transcode(self, file, quality='high'):
        # FFMpeg operations
        return 'output.mp4'

class ThumbnailGenerator:
    def generate(self, video_file, at_second):
        # Frame extraction
        return 'thumbnail.jpg'

class StorageService:
    def upload(self, files):
        # S3 upload
        return ['url1', 'url2']

class NotificationService:
    def notify(self, message):
        # Email/webhook
        pass

# The Facade - Simple interface to complex system
class VideoProcessingFacade:
    def __init__(self):
        self.validator = VideoValidator()
        self.metadata = VideoMetadata()
        self.transcoder = VideoTranscoder()
        self.thumbnailer = ThumbnailGenerator()
        self.storage = StorageService()
        self.notifier = NotificationService()
    
    def process_video(self, video_file, user_email):
        """
        One simple method that does everything correctly!
        """
        try:
            # Step 1: Validate
            if not self.validator.validate(video_file):
                raise ValueError("Invalid video file")
            
            # Step 2: Extract metadata
            metadata = self.metadata.extract(video_file)
            
            # Step 3: Transcode
            output_file = self.transcoder.transcode(video_file)
            
            # Step 4: Generate thumbnail
            thumbnail_file = self.thumbnailer.generate(
                output_file, 
                at_second=metadata['duration'] / 2
            )
            
            # Step 5: Upload
            urls = self.storage.upload([output_file, thumbnail_file])
            
            # Step 6: Cleanup
            self._cleanup([video_file, output_file, thumbnail_file])
            
            # Step 7: Notify
            self.notifier.notify(f"Video processed: {urls[0]}")
            
            return {
                'video_url': urls[0],
                'thumbnail_url': urls[1],
                'metadata': metadata
            }
            
        except Exception as e:
            self._cleanup([video_file])
            raise
    
    def _cleanup(self, files):
        for file in files:
            if os.path.exists(file):
                os.remove(file)

# Usage - Incredibly simple!
facade = VideoProcessingFacade()

# Instead of 40+ lines of complex orchestration:
result = facade.process_video('upload.mp4', 'user@example.com')

# That's it! One line!
print(f"Video available at: {result['video_url']}")
```

**Why This Is Powerful:**
1. **Developers can't make mistakes** - the sequence is encapsulated
2. **Easy to maintain** - changes to the process happen in one place
3. **Testable** - facade can be mocked
4. **Reusable** - same logic everywhere
5. **Hides complexity** - new developers can use it immediately

---

### **7. DECORATOR - "I Need Optional Features Without Explosion"**

**Real Scenario: Coffee Shop Ordering System**

You sell coffee with optional add-ons: milk, whipped cream, caramel, extra shot, etc. Without decorator, you'd need `CoffeeWithMilk`, `CoffeeWithMilkAndCaramel`, `CoffeeWithMilkAndCaramelAndWhip`... endless combinations!

**The Problem:**
```python
# Combinatorial explosion!
class Coffee:
    def cost(self): return 2.0

class CoffeeWithMilk:
    def cost(self): return 2.5

class CoffeeWithMilkAndCaramel:
    def cost(self): return 3.0

class CoffeeWithMilkAndCaramelAndWhip:
    def cost(self): return 3.5

# With 5 add-ons, you need 2^5 = 32 classes!
```

**Solution with Decorator:**
```python
from abc import ABC, abstractmethod

# Step 1: Base component
class Beverage(ABC):
    @abstractmethod
    def cost(self):
        pass
    
    @abstractmethod
    def description(self):
        pass

class Coffee(Beverage):
    def cost(self):
        return 2.0
    
    def description(self):
        return "Coffee"

# Step 2: Base decorator
class BeverageDecorator(Beverage):
    def __init__(self, beverage: Beverage):
        self._beverage = beverage

# Step 3: Concrete decorators - each adds one feature
class Milk(BeverageDecorator):
    def cost(self):
        return self._beverage.cost() + 0.5
    
    def description(self):
        return self._beverage.description() + ", Milk"

class Caramel(BeverageDecorator):
    def cost(self):
        return self._beverage.cost() + 0.7
    
    def description(self):
        return self._beverage.description() + ", Caramel"

class WhippedCream(BeverageDecorator):
    def cost(self):
        return self._beverage.cost() + 0.6
    
    def description(self):
        return self._beverage.description() + ", Whipped Cream"

class ExtraShot(BeverageDecorator):
    def cost(self):
        return self._beverage.cost() + 1.0
    
    def description(self):
        return self._beverage.description() + ", Extra Shot"

# Usage - Build any combination dynamically!

# Simple coffee
order1 = Coffee()
print(f"{order1.description()}: ${order1.cost()}")
# Output: Coffee: $2.0

# Coffee with milk
order2 = Milk(Coffee())
print(f"{order2.description()}: ${order2.cost()}")
# Output: Coffee, Milk: $2.5

# Coffee with milk and caramel
order3 = Caramel(Milk(Coffee()))
print(f"{order3.description()}: ${order3.cost()}")
# Output: Coffee, Milk, Caramel: $3.2

# The works!
order4 = WhippedCream(ExtraShot(Caramel(Milk(Coffee()))))
print(f"{order4.description()}: ${order4.cost()}")
# Output: Coffee, Milk, Caramel, Extra Shot, Whipped Cream: $4.8

# Easy to add new add-ons - just create new decorator!
class VanillaSyrup(BeverageDecorator):
    def cost(self):
        return self._beverage.cost() + 0.4
    
    def description(self):
        return self._beverage.description() + ", Vanilla"

order5 = VanillaSyrup(Milk(Coffee()))
print(f"{order5.description()}: ${order5.cost()}")
# Output: Coffee, Milk, Vanilla: $2.9
```

**Step-by-Step Process:**
1. **Start with base** (Coffee: $2.00)
2. **Wrap in Milk** (Coffee + Milk: $2.50)
3. **Wrap in Caramel** (Coffee + Milk + Caramel: $3.20)
4. **Each decorator adds behavior** without modifying existing code

**Real-World Example: Text Formatting**
```python
class Text:
    def __init__(self, content):
        self.content = content
    
    def render(self):
        return self.content

class Bold(BeverageDecorator):
    def render(self):
        return f"<b>{self._beverage.render()}</b>"

class Italic(BeverageDecorator):
    def render(self):
        return f"<i>{self._beverage.render()}</i>"

class Underline(BeverageDecorator):
    def render(self):
        return f"<u>{self._beverage.render()}</u>"

# Usage:
text = Text("Hello World")
formatted = Underline(Italic(Bold(text)))
print(formatted.render())
# Output: <u><i><b>Hello World</b></i></u>
```

---

## **BEHAVIORAL PATTERNS: The "How Does Behavior Change?" Problems**

---

### **8. STRATEGY - "I Need to Swap Algorithms"**

**Real Scenario: Shipping Cost Calculator**

Your e-commerce site offers different shipping options: Standard, Express, Overnight, International.

**The Problem Without Strategy:**
```python
def calculate_shipping(items, destination, shipping_type):
    total_weight = sum(item.weight for item in items)
    
    if shipping_type == 'standard':
        if destination == 'domestic':
            cost = total_weight * 0.5
            days = 5
        else:
            cost = total_weight * 2.0
            days = 10
    elif shipping_type == 'express':
        if destination == 'domestic':
            cost = total_weight * 1.5
            days = 2
        else:
            cost = total_weight * 5.0
            days = 5
    elif shipping_type == 'overnight':
        if destination == 'domestic':
            cost = total_weight * 3.0
            days = 1
        else:
            return "Not available"
    # This gets worse with every new option!
    # What about insurance? Gift wrapping? Signature required?
```

**Solution with Strategy:**
```python
from abc import ABC, abstractmethod

# Step 1: Strategy interface
class ShippingStrategy(ABC):
    @abstractmethod
    def calculate(self, items, destination):
        pass

# Step 2: Concrete strategies
class StandardShipping(ShippingStrategy):
    def calculate(self, items, destination):
        total_weight = sum(item['weight'] for item in items)
        
        if destination == 'domestic':
            return {
                'cost': total_weight * 0.5,
                'days': 5,
                'method': 'Standard'
            }
        else:
            return {
                'cost': total_weight * 2.0,
                'days': 10,
                'method': 'Standard International'
            }

class ExpressShipping(ShippingStrategy):
    def calculate(self, items, destination):
        total_weight = sum(item['weight'] for item in items)
        
        if destination == 'domestic':
            return {
                'cost': total_weight * 1.5,
                'days': 2,
                'method': 'Express'
            }
        else:
            return {
                'cost': total_weight * 5.0,
                'days': 5,
                'method': 'Express International'
            }

class OvernightShipping(ShippingStrategy):
    def calculate(self, items, destination):
        if destination != 'domestic':
            raise ValueError("Overnight shipping only available domestically")
        
        total_weight = sum(item['weight'] for item in items)
        return {
            'cost': total_weight * 3.0,
            'days': 1,
            'method': 'Overnight'
        }

class FreeShipping(ShippingStrategy):
    """New strategy - easy to add!"""
    def calculate(self, items, destination):
        return {
            'cost': 0,
            'days': 7,
            'method': 'Free Standard (orders over $50)'
        }

# Step 3: Context - uses a strategy
class ShoppingCart:
    def __init__(self):
        self.items = []
        self.shipping_strategy = None
    
    def add_item(self, item):
        self.items.append(item)
    
    def set_shipping_strategy(self, strategy: ShippingStrategy):
        self.shipping_strategy = strategy
    
    def calculate_total(self, destination):
        subtotal = sum(item['price'] for item in self.items)
        
        if self.shipping_strategy is None:
            raise ValueError("Please select a shipping method")
        
        shipping = self.shipping_strategy.calculate(self.items, destination)
        
        return {
            'subtotal': subtotal,
            'shipping_cost': shipping['cost'],
            'shipping_method': shipping['method'],
            'estimated_days': shipping['days'],
            'total': subtotal + shipping['cost']
        }

# Usage - Clean and flexible!

# Create cart
cart = ShoppingCart()
cart.add_item({'name': 'Book', 'price': 15.99, 'weight': 1.0})
cart.add_item({'name': 'Laptop', 'price': 899.99, 'weight': 5.0})

# Customer chooses Standard shipping
cart.set_shipping_strategy(StandardShipping())
result = cart.calculate_total('domestic')
print(f"Total with {result['shipping_method']}: ${result['total']}")
# Output: Total with Standard: $918.98 (arrives in 5 days)

# Customer changes to Express
cart.set_shipping_strategy(ExpressShipping())
result = cart.calculate_total('domestic')
print(f"Total with {result['shipping_method']}: ${result['total']}")
# Output: Total with Express: $924.98 (arrives in 2 days)

# Apply free shipping for orders over $50
if cart.calculate_total('domestic')['subtotal'] > 50:
    cart.set_shipping_strategy(FreeShipping())
result = cart.calculate_total('domestic')
print(f"Total with {result['shipping_method']}: ${result['total']}")
# Output: Total with Free Standard: $915.98 (arrives in 7 days)
```

**Real-World A/B Testing Example:**
```python
class RecommendationStrategy(ABC):
    @abstractmethod
    def get_recommendations(self, user):
        pass

class CollaborativeFiltering(RecommendationStrategy):
    def get_recommendations(self, user):
        # Find similar users, recommend what they liked
        return ['Product A', 'Product B']

class ContentBased(RecommendationStrategy):
    def get_recommendations(self, user):
        # Recommend based on user's past purchases
        return ['Product C', 'Product D']

class HybridRecommendation(RecommendationStrategy):
    def get_recommendations(self, user):
        # Combine multiple approaches
        return ['Product A', 'Product C', 'Product E']

# A/B testing - easy to switch strategies!
if user.id % 3 == 0:
    strategy = CollaborativeFiltering()
elif user.id % 3 == 1:
    strategy = ContentBased()
else:
    strategy = HybridRecommendation()

recommendations = strategy.get_recommendations(user)
```

---

### **9. OBSERVER - "I Need to Notify Multiple Objects"**

**Real Scenario: Stock Price Monitoring System**

Multiple displays need to update when stock price changes: mobile app, web dashboard, email alerts, trading bot.

**The Problem Without Observer:**
```python
class StockMarket:
    def __init__(self):
        self.price = 100.0
    
    def update_price(self, new_price):
        self.price = new_price
        
        # Tightly coupled to all displays!
        mobile_app.update(new_price)
        web_dashboard.update(new_price)
        email_alerter.send_alert(new_price)
        trading_bot.check_triggers(new_price)
        
        # Adding new display? Modify this method!
        # What if one display crashes? Affects all others!
```

**Solution with Observer:**
```python
from abc import ABC, abstractmethod

# Step 1: Observer interface
class StockObserver(ABC):
    @abstractmethod
    def update(self, stock_symbol, price):
        pass

# Step 2: Subject (Observable)
class Stock:
    def __init__(self, symbol, price):
        self.symbol = symbol
        self.price = price
        self._observers = []
    
    def attach(self, observer: StockObserver):
        """Register an observer"""
        if observer not in self._observers:
            self._observers.append(observer)
            print(f"{observer.__class__.__name__} subscribed to {self.symbol}")
    
    def detach(self, observer: StockObserver):
        """Unregister an observer"""
        self._observers.remove(observer)
        print(f"{observer.__class__.__name__} unsubscribed from {self.symbol}")
    
    def notify(self):
        """Notify all observers"""
        for observer in self._observers:
            observer.update(self.symbol, self.price)
    
    def set_price(self, new_price):
        """When price changes, notify everyone"""
        print(f"\n{self.symbol} price changed: ${self.price} -> ${new_price}")
        self.price = new_price
        self.notify()

# Step 3: Concrete observers
class MobileApp(StockObserver):
    def update(self, stock_symbol, price):
        print(f"📱 Mobile App: {stock_symbol} is now ${price}")
        # Push notification to user's phone

class WebDashboard(StockObserver):
    def update(self, stock_symbol, price):
        print(f"🖥️  Web Dashboard: Updating {stock_symbol} chart to ${price}")
        # Update real-time chart

class EmailAlerter(StockObserver):
    def __init__(self, threshold):
        self.threshold = threshold
    
    def update(self, stock_symbol, price):
        if price > self.threshold:
            print(f"📧 Email Alert: {stock_symbol} exceeded ${self.threshold}! Now ${price}")
            # Send email

class TradingBot(StockObserver):
    def __init__(self, buy_below, sell_above):
        self.buy_below = buy_below
        self.sell_above = sell_above
    
    def update(self, stock_symbol, price):
        if price < self.buy_below:
            print(f"🤖 Trading Bot: BUY signal for {stock_symbol} at ${price}")
            # Execute buy order
        elif price > self.sell_above:
            print(f"🤖 Trading Bot: SELL signal for {stock_symbol} at ${price}")
            # Execute sell order

class SMSNotifier(StockObserver):
    """New observer - easy to add!"""
    def update(self, stock_symbol, price):
        print(f"📱 SMS: {stock_symbol} price update: ${price}")
        # Send SMS

# Usage - Flexible subscription system

# Create stock
apple_stock = Stock('AAPL', 150.0)

# Create observers
mobile = MobileApp()
dashboard = WebDashboard()
email = EmailAlerter(threshold=160.0)
bot = TradingBot(buy_below=145.0, sell_above=165.0)
sms = SMSNotifier()

# Subscribe observers
apple_stock.attach(mobile)
apple_stock.attach(dashboard)
apple_stock.attach(email)
apple_stock.attach(bot)

# Price changes - all observers notified automatically!
apple_stock.set_price(155.0)
# Output:
# 📱 Mobile App: AAPL is now $155.0
# 🖥️  Web Dashboard: Updating AAPL chart to $155.0
# (Email: no alert, below threshold)
# (Bot: no action, not at trigger price)

apple_stock.set_price(162.0)
# Output:
# 📱 Mobile App: AAPL is now $162.0
# 🖥️  Web Dashboard: Updating AAPL chart to $162.0
# 📧 Email Alert: AAPL exceeded $160.0! Now $162.0
# (Bot: no action)

apple_stock.set_price(167.0)
# Output:
# 📱 Mobile App: AAPL is now $167.0
# 🖥️  Web Dashboard: Updating AAPL chart to $167.0
# 📧 Email Alert: AAPL exceeded $160.0! Now $167.0
# 🤖 Trading Bot: SELL signal for AAPL at $167.0

# Add SMS notifications mid-stream
apple_stock.attach(sms)
apple_stock.set_price(170.0)
# SMS is now also notified!

# User unsubscribes from mobile
apple_stock.detach(mobile)
apple_stock.set_price(165.0)
# Mobile no longer notified, others still are
```

**Why This Works:**
1. **Loose coupling** - Stock doesn't know about specific observers
2. **Easy to add/remove** observers at runtime
3. **One observer failure** doesn't affect others
4. **Each observer** handles updates independently

---

### **10. CHAIN OF RESPONSIBILITY - "I Need Sequential Processing"**

**Real Scenario: Support Ticket System**

Support tickets go through levels: Basic Support → Technical Support → Manager → Engineering.

**The Problem Without Chain:**
```python
def handle_ticket(ticket):
    if ticket.complexity == 'basic':
        basic_support.handle(ticket)
    elif ticket.complexity == 'technical':
        tech_support.handle(ticket)
    elif ticket.complexity == 'escalated':
        manager.handle(ticket)
    elif ticket.complexity == 'critical':
        engineering.handle(ticket)
    
    # What if basic can't handle it? Need to escalate manually!
    # Hard to add new levels
    # Can't dynamically change the chain
```

**Solution with Chain of Responsibility:**
```python
from abc import ABC, abstractmethod

# Step 1: Handler interface
class SupportHandler(ABC):
    def __init__(self):
        self._next_handler = None
    
    def set_next(self, handler):
        """Set the next handler in the chain"""
        self._next_handler = handler
        return handler  # For chaining: h1.set_next(h2).set_next(h3)
    
    @abstractmethod
    def can_handle(self, ticket):
        """Can this handler deal with this ticket?"""
        pass
    
    def handle(self, ticket):
        """Handle or pass to next"""
        if self.can_handle(ticket):
            return self.process(ticket)
        elif self._next_handler:
            print(f"{self.__class__.__name__} passing to next handler...")
            return self._next_handler.handle(ticket)
        else:
            return f"No handler available for ticket #{ticket.id}"
    
    @abstractmethod
    def process(self, ticket):
        """Actually handle the ticket"""
        pass

# Step 2: Concrete handlers
class BasicSupport(SupportHandler):
    def can_handle(self, ticket):
        return ticket.complexity == 'basic'
    
    def process(self, ticket):
        return f"✅ BasicSupport resolved ticket #{ticket.id}: {ticket.issue}"

class TechnicalSupport(SupportHandler):
    def can_handle(self, ticket):
        return ticket.complexity in ['basic', 'technical']
    
    def process(self, ticket):
        if ticket.complexity == 'basic':
            # Technical can handle basic too, but prefers to pass down
            return f"⚙️  TechnicalSupport resolved ticket #{ticket.id}: {ticket.issue}"
        return f"⚙️  TechnicalSupport resolved ticket #{ticket.id}: {ticket.issue}"

class ManagerSupport(SupportHandler):
    def can_handle(self, ticket):
        return ticket.complexity in ['basic', 'technical', 'escalated']
    
    def process(self, ticket):
        return f"👔 Manager resolved ticket #{ticket.id}: {ticket.issue}"

class EngineeringSupport(SupportHandler):
    def can_handle(self, ticket):
        return True  # Engineering handles everything as last resort
    
    def process(self, ticket):
        return f"🔧 Engineering resolved critical ticket #{ticket.id}: {ticket.issue}"

# Step 3: Ticket class
class Ticket:
    def __init__(self, ticket_id, issue, complexity):
        self.id = ticket_id
        self.issue = issue
        self.complexity = complexity

# Usage - Build the chain
basic = BasicSupport()
technical = TechnicalSupport()
manager = ManagerSupport()
engineering = EngineeringSupport()

# Chain them: basic → technical → manager → engineering
basic.set_next(technical).set_next(manager).set_next(engineering)

# Test different tickets
tickets = [
    Ticket(1, "Password reset", "basic"),
    Ticket(2, "Server configuration", "technical"),
    Ticket(3, "Refund request", "escalated"),
    Ticket(4, "Database corruption", "critical")
]

for ticket in tickets:
    print(f"\n🎫 Processing ticket #{ticket.id} ({ticket.complexity}):")
    result = basic.handle(ticket)
    print(result)

# Output:
# 🎫 Processing ticket #1 (basic):
# ✅ BasicSupport resolved ticket #1: Password reset

# 🎫 Processing ticket #2 (technical):
# BasicSupport passing to next handler...
# ⚙️  TechnicalSupport resolved ticket #2: Server configuration

# 🎫 Processing ticket #3 (escalated):
# BasicSupport passing to next handler...
# TechnicalSupport passing to next handler...
# 👔 Manager resolved ticket #3: Refund request

# 🎫 Processing ticket #4 (critical):
# BasicSupport passing to next handler...
# TechnicalSupport passing to next handler...
# ManagerSupport passing to next handler...
# 🔧 Engineering resolved critical ticket #4: Database corruption
```

**Real-World Example: Authentication Middleware**
```python
class AuthenticationHandler(ABC):
    def __init__(self):
        self._next = None
    
    def set_next(self, handler):
        self._next = handler
        return handler
    
    @abstractmethod
    def handle(self, request):
        pass

class RateLimitHandler(AuthenticationHandler):
    def handle(self, request):
        print("Checking rate limit...")
        if request.get('requests_count', 0) > 100:
            return "Rate limit exceeded"
        if self._next:
            return self._next.handle(request)

class AuthenticationCheck(AuthenticationHandler):
    def handle(self, request):
        print("Checking authentication...")
        if not request.get('auth_token'):
            return "Authentication required"
        if self._next:
            return self._next.handle(request)

class AuthorizationCheck(AuthenticationHandler):
    def handle(self, request):
        print("Checking authorization...")
        if request.get('role') != 'admin':
            return "Insufficient permissions"
        if self._next:
            return self._next.handle(request)

class RequestProcessor(AuthenticationHandler):
    def handle(self, request):
        return f"✅ Request processed: {request.get('action')}"

# Build middleware chain
rate_limit = RateLimitHandler()
auth = AuthenticationCheck()
authz = AuthorizationCheck()
processor = RequestProcessor()

rate_limit.set_next(auth).set_next(authz).set_next(processor)

# Test
request = {
    'requests_count': 50,
    'auth_token': 'valid_token',
    'role': 'admin',
    'action': 'delete_user'
}

result = rate_limit.handle(request)
print(result)
# Output:
# Checking rate limit...
# Checking authentication...
# Checking authorization...
# ✅ Request processed: delete_user
```

---

### **11. STATE - "Behavior Changes Based on State"**

**Real Scenario: Document Workflow**

A document goes through states: Draft → Pending Review → Approved → Published. Actions available depend on the current state.

**The Problem Without State:**
```python
class Document:
    def __init__(self):
        self.state = 'draft'
        self.content = ""
    
    def publish(self):
        if self.state == 'draft':
            return "Can't publish draft! Submit for review first."
        elif self.state == 'pending_review':
            return "Can't publish! Needs approval."
        elif self.state == 'approved':
            self.state = 'published'
            return "Published!"
        elif self.state == 'published':
            return "Already published."
        # Complex conditionals everywhere!
```

**Solution with State Pattern:**
```python
from abc import ABC, abstractmethod

# Step 1: State interface
class DocumentState(ABC):
    @abstractmethod
    def edit(self, document):
        pass
    
    @abstractmethod
    def submit_for_review(self, document):
        pass
    
    @abstractmethod
    def approve(self, document):
        pass
    
    @abstractmethod
    def publish(self, document):
        pass
    
    @abstractmethod
    def reject(self, document):
        pass

# Step 2: Concrete states
class DraftState(DocumentState):
    def edit(self, document):
        return "✏️  Editing document..."
    
    def submit_for_review(self, document):
        document.set_state(PendingReviewState())
        return "📝 Document submitted for review"
    
    def approve(self, document):
        return "❌ Can't approve a draft! Submit for review first."
    
    def publish(self, document):
        return "❌ Can't publish a draft! Submit for review first."
    
    def reject(self, document):
        return "❌ Can't reject a draft."

class PendingReviewState(DocumentState):
    def edit(self, document):
        return "❌ Can't edit while under review."
    
    def submit_for_review(self, document):
        return "ℹ️  Already under review."
    
    def approve(self, document):
        document.set_state(ApprovedState())
        return "✅ Document approved!"
    
    def publish(self, document):
        return "❌ Must be approved first."
    
    def reject(self, document):
        document.set_state(DraftState())
        return "🔙 Document rejected, returned to draft."

class ApprovedState(DocumentState):
    def edit(self, document):
        document.set_state(DraftState())
        return "📝 Editing approved document, returning to draft..."
    
    def submit_for_review(self, document):
        return "ℹ️  Already approved!"
    
    def approve(self, document):
        return "ℹ️  Already approved!"
    
    def publish(self, document):
        document.set_state(PublishedState())
        return "🚀 Document published!"
    
    def reject(self, document):
        document.set_state(DraftState())
        return "🔙 Approval revoked, returned to draft."

class PublishedState(DocumentState):
    def edit(self, document):
        return "❌ Can't edit published document! Create new version."
    
    def submit_for_review(self, document):
        return "❌ Already published."
    
    def approve(self, document):
        return "ℹ️  Already published!"
    
    def publish(self, document):
        return "ℹ️  Already published!"
    
    def reject(self, document):
        return "❌ Can't reject published document."

# Step 3: Context (Document)
class Document:
    def __init__(self, title):
        self.title = title
        self.content = ""
        self._state = DraftState()  # Initial state
    
    def set_state(self, state: DocumentState):
        print(f"State changed to: {state.__class__.__name__}")
        self._state = state
    
    def get_state_name(self):
        return self._state.__class__.__name__
    
    # Delegate to current state
    def edit(self):
        return self._state.edit(self)
    
    def submit_for_review(self):
        return self._state.submit_for_review(self)
    
    def approve(self):
        return self._state.approve(self)
    
    def publish(self):
        return self._state.publish(self)
    
    def reject(self):
        return self._state.reject(self)

# Usage - State transitions are clean!

doc = Document("Design Patterns Guide")
print(f"Current state: {doc.get_state_name()}\n")

# In Draft state
print(doc.edit())
# ✏️  Editing document...

print(doc.publish())
# ❌ Can't publish a draft! Submit for review first.

# Submit for review
print(doc.submit_for_review())
# 📝 Document submitted for review
# State changed to: PendingReviewState

# Try to edit (not allowed)
print(doc.edit())
# ❌ Can't edit while under review.

# Approve it
print(doc.approve())
# ✅ Document approved!
# State changed to: ApprovedState

# Now publish
print(doc.publish())
# 🚀 Document published!
# State changed to: PublishedState

# Try to edit published (not allowed)
print(doc.edit())
# ❌ Can't edit published document! Create new version.

# Alternative flow: rejection
doc2 = Document("Another Doc")
doc2.submit_for_review()
print(doc2.reject())
# 🔙 Document rejected, returned to draft.
# State changed to: DraftState
```

**Real-World Example: TCP Connection**
```python
class TCPState(ABC):
    @abstractmethod
    def open(self, connection): pass
    
    @abstractmethod
    def close(self, connection): pass
    
    @abstractmethod
    def send(self, connection, data): pass

class ClosedState(TCPState):
    def open(self, connection):
        connection.set_state(ListenState())
        return "Opening connection..."
    
    def close(self, connection):
        return "Already closed"
    
    def send(self, connection, data):
        return "Can't send, connection closed"

class ListenState(TCPState):
    def open(self, connection):
        return "Already open"
    
    def close(self, connection):
        connection.set_state(ClosedState())
        return "Closing connection..."
    
    def send(self, connection, data):
        connection.set_state(EstablishedState())
        return f"Sending data: {data}"

class EstablishedState(TCPState):
    def open(self, connection):
        return "Already established"
    
    def close(self, connection):
        connection.set_state(ClosedState())
        return "Connection closed"
    
    def send(self, connection, data):
        return f"Data sent: {data}"

class TCPConnection:
    def __init__(self):
        self._state = ClosedState()
    
    def set_state(self, state):
        self._state = state
    
    def open(self):
        return self._state.open(self)
    
    def close(self):
        return self._state.close(self)
    
    def send(self, data):
        return self._state.send(self, data)
```

---

This covers the most commonly used design patterns with real scenarios. Each pattern solves a specific pain point:

- **Creational** = Object creation complexity
- **Structural** = How objects fit together
- **Behavioral** = How behavior changes

The key is to identify your specific problem first, then choose the pattern that addresses it. Would you like me to dive deeper into any specific pattern or scenario?
