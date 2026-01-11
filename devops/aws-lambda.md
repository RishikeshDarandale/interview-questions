# AWS Lambda

<details>
  <summary>What is AWS Lambda?</summary>
  AWS Lambda is an event-driven, serverless compute service that runs your code without requiring you to provision or manage servers. You only pay for the compute time consumed, not when the code is idle.
</details>
<details>
  <summary>Which programming languages does AWS Lambda support?<</summary>
  Lambda supports several runtimes including Node.js, Python, Java, C#, Go, and Ruby, along with a custom runtime interface for others.
</details>
<details>
  <summary>What are the key components of an AWS Lambda function?</summary>
  The main components are the function code itself, an event source (the AWS service or trigger that invokes the function), and an execution role (IAM role) that grants permissions to access other AWS resources.
</details>
<details>
  <summary>What is the maximum execution time (timeout) for a Lambda function?</summary>
  A Lambda function can run for a maximum of 15 minutes (900 seconds). The default timeout is 3 seconds.
</details>
<details>
  <summary>How is AWS Lambda priced?</summary>
  Pricing is based on the number of requests and the duration your code executes, measured in milliseconds and depending on the memory allocated. 
</details>
<details>
  <summary>Explain the "cold start" phenomenon in AWS Lambda and how to mitigate it.</summary>
  A "cold start" is the latency experienced when a function is invoked and AWS needs to initialize a new execution environment (download code, start the runtime). Mitigation strategies include using Provisioned Concurrency to keep functions pre-initialized, optimizing code for lower memory usage, and minimizing deployment package size.
</details>
<details>
  <summary>How do you monitor and troubleshoot the performance of a Lambda function?</summary>
  Monitoring is primarily done using Amazon CloudWatch Logs for function output and errors, and CloudWatch Metrics for performance metrics (invocations, errors, duration). AWS X-Ray can be used for detailed tracing and performance analysis of distributed requests.
</details>
<details>
  <summary>What are the limitations of AWS Lambda?</summary>
  Limitations include a maximum execution time of 15 minutes, a 512 MB ephemeral disk space (in /tmp/), default concurrency limits at the account level, and the inability to accept inbound network connections.
</details>
<details>
  <summary>How does Lambda handle failures and retries for asynchronous invocations?</summary>
  For asynchronous invocations, Lambda automatically retries failed events up to two times with exponential backoff. You can configure a Dead Letter Queue (DLQ) (either SQS or SNS) to capture failed events for inspection.
</details>
<details>
  <summary>When would you choose AWS Lambda over Amazon EC2?</summary>
  Choose Lambda for event-driven, stateless workloads where you want minimal operational overhead for server management (patching, scaling, etc.). EC2 offers full control over the operating system and infrastructure, making it suitable for long-running, stateful, or highly specific application requirements. 
</details>
<details>
  <summary>Describe a Lambda function's lifecycle from "Cold Start" to "Warm."</summary>
  A Lambda function's lifecycle moves from Cold Start (slow, setting up a new execution environment: download code, start microVM, load runtime, run init code) to Warm (fast, reusing that environment for subsequent requests, skipping setup and running only the handler), with a brief Freeze phase (pausing the environment for a few minutes) in between, allowing for rapid reuse before termination if inactive, forcing a new cold start next time. 
  
  ### Cold Start (Initialization Phase)
  - **Request Received:** An invocation request arrives when no environment is ready.
  - **Environment Provisioning:** Lambda creates a secure execution environment (a Firecracker microVM).
  - **Code & Runtime Download:** Your function's code is downloaded from S3, and the specified runtime (e.g., Node.js, Python) is prepared.
  - **Initialization Code Execution:** Code outside your main handler (global variables, database connections) runs.
  - **Handler Invocation:** The handler function executes with the event data.
  Result: Slower response due to setup overhead (Init Duration metric).

  ### Warm Start (Reuse Phase)
  - Environment Reused: After the first request, the environment stays "warm" (paused) for a short period (e.g., 5-15 mins).
  - Handler Execution: A new request reuses the existing environment; only the handler runs, skipping the Init phase.
  - Result: Much faster response (Duration metric, no Init Duration). 
  ### Shutdown Phase
  If the environment remains idle, it eventually gets terminated, leading to the next invocation requiring a new Cold Start. 
</details>
