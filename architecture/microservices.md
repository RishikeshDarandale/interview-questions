# Questions
## Resilience and Fault Tolerance

<details>
  <summary>Explain the Circuit Breaker pattern. How does it prevent cascading failures in microservices?</summary>
The Circuit Breaker pattern prevents cascading failures by acting as a proxy that monitors calls to a service and stops sending requests if the service is failing, allowing it to recover. It achieves this by cycling through three states: Closed, Open, and Half-Open. In the Closed state, requests pass through normally; if too many failures occur, the breaker "trips" and moves to the Open state, where it immediately returns an error or a fallback response without trying the failing service. After a timeout, it enters the Half-Open state to test the service with a limited number of requests; if they succeed, the breaker resets to Closed. 

#### How the pattern prevents cascading failures

- **Isolates failing services**: When a service is struggling, the circuit breaker stops requests to it. This prevents the calling service from being overwhelmed by waiting for slow or failed responses, which would tie up its own resources (like threads and connections).
- **Prevents resource burnout**: By quickly failing requests to a down service, the circuit breaker allows the calling service to continue processing other requests, conserve resources, and avoid a complete system meltdown.
- **Enables graceful degradation**: Instead of an unhandled error, the circuit breaker can provide a fallback mechanism. This allows the application to offer a degraded but functional user experience, such as showing a cached response or a "service temporarily unavailable" message, instead of a hard crash.
- **Allows for automatic recovery**: The transition to the Half-Open state is crucial for recovery. By allowing a few test requests after a period of inactivity, the breaker can automatically determine when the failing service is back online and restore normal operation without manual intervention. 

```javascript
// Define the possible states of the circuit breaker
const CircuitBreakerState = {
  CLOSED: 'CLOSED', // Normal operation, failures are monitored
  OPEN: 'OPEN',     // Circuit tripped, requests blocked immediately
  HALF_OPEN: 'HALF_OPEN', // Test state, one request allowed to check service health
};

class CircuitBreaker {
  constructor(serviceCall, options = {}) {
    this.serviceCall = serviceCall;
    this.failureThreshold = options.failureThreshold || 3; // Max failures before opening
    this.resetTimeout = options.resetTimeout || 5000;     // Time to stay open (ms)
    this.state = CircuitBreakerState.CLOSED;
    this.failureCount = 0;
    this.nextAttemptTime = 0;
  }

  // The primary method to execute the protected service call
  async fire() {
    if (this.state === CircuitBreakerState.OPEN) {
      // Check if enough time has passed to transition to HALF_OPEN
      if (this.nextAttemptTime <= Date.now()) {
        this.state = CircuitBreakerState.HALF_OPEN;
      } else {
        // Circuit is OPEN and timeout hasn't passed, fail fast
        throw new Error('Circuit is OPEN. Try again later.');
      }
    }

    try {
      // Attempt the service call
      const response = await this.serviceCall();
      // If successful, call the success handler
      return this.success(response);
    } catch (error) {
      // If failed, call the failure handler
      return this.failure(error.message);
    }
  }

  // Handler for successful calls
  success(data) {
    this.failureCount = 0;
    if (this.state === CircuitBreakerState.HALF_OPEN) {
      this.state = CircuitBreakerState.CLOSED; // Service is healthy, close the circuit
      console.log('Circuit closed: Service recovered.');
    }
    return data;
  }

  // Handler for failed calls
  failure(error) {
    this.failureCount += 1;
    if (this.failureCount >= this.failureThreshold) {
      this.state = CircuitBreakerState.OPEN; // Failure threshold reached, open the circuit
      this.nextAttemptTime = Date.now() + this.resetTimeout; // Set the next attempt time
      console.log(`Circuit opened: Threshold reached. Next attempt at ${new Date(this.nextAttemptTime)}`);
    } else {
        console.log(`Failure count: ${this.failureCount}/${this.failureThreshold}`);
    }
    // In OPEN state, throw an error or return a fallback
    if (this.state === CircuitBreakerState.OPEN) {
        throw new Error(`Circuit is OPEN: ${error}`);
    }
    // In CLOSED/HALF_OPEN, still propagate the original error
    throw new Error(error);
  }
}
```

```javascript
// 1. Define the risky operation (simulated API call that fails 3 times, then succeeds)
let attemptCount = 0;
const riskyApiCall = async () => {
  attemptCount++;
  if (attemptCount <= 3) {
    console.log(`API Call failed (attempt ${attemptCount})`);
    throw new Error('Service Unavailable');
  }
  console.log('API Call successful!');
  return { status: 200, data: 'Success Data' };
};

// 2. Wrap the operation in a circuit breaker instance
const breaker = new CircuitBreaker(riskyApiCall, {
  failureThreshold: 2,
  resetTimeout: 2000, // Wait 2 seconds before trying again
});

// 3. Fire requests through the circuit breaker
async function makeRequest() {
  try {
    const result = await breaker.fire();
    console.log('Request succeeded:', result);
  } catch (err) {
    console.error('Request failed:', err.message);
  }
}

// Example sequence:
// Call 1: Fails (count 1)
// Call 2: Fails (count 2, circuit opens)
// Call 3: Fails fast (circuit open)
// Wait 2 seconds (half-open)
// Call 4: Succeeds (circuit closes)
```
</details>

<details>
  <summary>Describe the purpose of the Retry pattern and when it should be applied.</summary>
  The Retry pattern is a design strategy used to handle expected, short-lived (transient) failures in a system by automatically repeating a failed operation a limited number of times. Its main purpose is to increase an application's resilience and reliability by recovering from temporary issues without requiring manual intervention or immediately aborting the operation.

  #### Purpose of the Retry Pattern
  
  The primary goal is to minimize the impact of intermittent faults on the user experience and overall system stability. Transient failures are common in distributed systems and cloud environments due to factors like: 
  - Momentary loss of network connectivity or high network latency.
  - Temporary service unavailability or timeouts when a service is busy.
  - Database connection issues during a peak in concurrent requests.
  - Service throttling, where a service temporarily rejects requests to prevent overload (e.g., HTTP 429 Too Many Requests status code). 
  By automatically retrying, the pattern allows the system to overcome these self-correcting issues, ensuring that a brief disruption doesn't lead to a total process failure.

  #### When to Apply the Retry Pattern

  The Retry pattern should be applied in specific scenarios where the failure is likely to be transient and a subsequent attempt has a high probability of success: 
  - **For operations with transient failure potential**: Use it for network requests, external API calls, and database operations.
  - **When operations are idempotent**: An operation is idempotent if it can be performed multiple times without causing unintended side effects (e.g., performing a GET request or updating a resource with a unique key). Retrying non-idempotent operations (like a standard POST request that might charge a credit card multiple times) can lead to data inconsistencies.
  - **With a defined backoff strategy**: To prevent overwhelming the failing service, retries should use an exponential backoff strategy, increasing the delay between attempts. Adding random "jitter" to this delay helps prevent multiple clients from retrying simultaneously (a "retry storm").
  - **For a limited number of attempts**: Define a maximum number of retries (e.g., 3-5 attempts) to prevent endless loops and resource exhaustion.

  #### When Not to Apply the Retry Pattern

  - **For non-transient (persistent) errors**: Errors like authentication failures (HTTP 401/403) or business logic errors will not resolve themselves with retries.
  - **For long-lasting faults**: If a service is completely offline, continuous retries waste resources and affect application performance. This is where the Circuit Breaker pattern is more appropriate.
  - **As an alternative to scaling**: If failures are frequent due to a consistently overloaded service, retries will only exacerbate the issue; the underlying service needs to be scaled up.
</details>

<details>
  <summary>Retry pattern vs. the Circuit Breaker pattern</summary>
  The Retry and Circuit Breaker patterns often work in tandem within resilient systems, but they serve different purposes and handle different categories of faults.

  | Feature	| Retry Pattern	| Circuit Breaker Pattern |
  | :------- | :------ | :------- |
  | Primary Purpose	| Recover from transient (short-lived, self-correcting) failures.	| Prevent cascading failures caused by persistent (long-term, systemic) failures. |
  | When Applied	  | Applied immediately when a simple network glitch or momentary timeout occurs.	| Applied after repeated failures demonstrate a sustained problem with the dependency. |
  | Mechanism	      | Retries the operation immediately or after a short, increasing delay (exponential backoff).	| Stops attempting the operation entirely after a threshold is reached, and fails fast. |
  | Goal of Delay	  | To give the network/system a moment to clear the momentary bottleneck.	| To give the failing service time to recover without added load from the caller. |
  | Typical Use Cases	| Intermittent database connection drops, a single missed API request, brief server throttling (HTTP 429).	| A dependency service is offline, a major infrastructure failure, high latency across an entire service.|
  | Impact on System	| Adds minor load in the short term to ensure task completion.	| Reduces load on the target service and frees up resources in the calling service.|

  #### How They Complement Each Other
  A robust system often uses both patterns in combination:
  - **Initial Attempt & Retry Logic**: When a call is made to a service, a retry mechanism is typically the first line of defense. It attempts the call a few times (e.g., 3 retries with exponential backoff).
  - **Circuit Breaker Logic**: The circuit breaker monitors these attempts. If the retries consistently fail and the total number of failures reaches a certain threshold, the circuit breaker trips to the Open state.
  - **Fallback**: Once the circuit is open, subsequent requests immediately hit the circuit breaker, which instantly returns an error or a default fallback response without engaging the retry logic, saving valuable system resources.
  
  In summary: Retry handles the hiccup; Circuit Breaker handles the outage.
</details>
