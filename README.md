# Assignment 1: Serverless Computing - Critical Analysis

## Part 1: Paper Summary

According to the article “Serverless Computing: One Step Forward, Two Steps Back,” by Hellerstein et al., serverless computing enhances cloud usability by eliminating the need for infrastructure management and enabling automatic scalability. However, serverless computing has its drawbacks, as it cannot be used in situations involving large volumes of data. The title “One Step Forward, Two Steps Back” implies that although serverless computing makes it easier for the user, it takes one step backward in terms of performance.

One of the major limitations discussed is execution time constraints. The vast majority of FaaS systems impose execution time limits (seconds to minutes only), preventing developers from using serverless architectures to run long tasks. It includes data processing, iterations, and mathematical calculations. Developers are forced to split tasks into smaller units, which increases system complexity and introduces overhead from repeated function invocations.

Another significant issue is communication and network limitations. In serverless architectures, functions are intentionally isolated and do not communicate directly with each other. Instead, they must use external services such as object stores or databases to exchange data. This introduces significant latency and creates an I/O bottleneck because even basic communication between parts of the system has to go through storage. It also removes direct communication between compute nodes, which is normally available in traditional distributed systems.

A related issue is the data-shipping anti-pattern, in which large amounts of data are transferred to serverless functions for processing. This is less effective than traditional systems that move computation closer to where data is stored. The authors argue that this can increase network traffic, raise costs, and reduce performance.

In addition, the paper also highlights limited hardware access as a major challenge. First-generation serverless platforms typically do not support hardware such as GPUs and FPGAs. Consequently, serverless technologies cannot be used in areas where hardware is necessary, such as in high-performance computing and deep learning training.

Finally, the authors point out problems with running stateful and distributed workloads. Serverless functions do not keep data between runs, so they are stateless and short-lived. This means developers must use external storage to save state, which makes systems more complex and slower. It also makes it harder to build workflows that need coordination or repeated processing across multiple steps. 

To address these challenges, the authors suggest several improvements for serverless computing. They recommend better support for stateful and long-running computations, reducing data movement by processing data closer to where it is stored, and improving communication between functions. They also propose adding support for specialized hardware and creating programming models that are better suited for distributed applications.


In conclusion, although serverless computing appears very effective and scalable, there are some limitations in the way it is currently implemented that need to be improved before it can be used effectively for a wider range of applications.

## Part 2: Azure Durable Functions Deep Dive

### Orchestration Model

Azure Durable Functions introduces a three-tier model composed of client, orchestrator, and activity functions. The client function initiates the orchestration workflow, the orchestrator functions manage the workflow logic, such as the order of steps, branching, parallel tasks, and waiting for events, and the Activity functions perform the actual work, such as calling APIs, accessing databases, or processing data. This is different from basic serverless functions (FaaS), where each function runs independently, and none of them can retain any data between executions. Durable Functions keeps track of the whole workflow through the orchestrator, allowing serverless apps to support long-running and stateful processes. (Microsoft Learn – Durable Orchestrations, Keyhole Software) 


### State Management 

Rather than storing a snapshot of the current state, Azure Durable Functions records every step of a workflow in an event history stored in Azure Storage. When the workflow resumes after a restart or checkpoint, it replays these recorded events to rebuild its state and continue where it left off. This addresses one of the main issues faced by traditional serverless functions since these functions are stateless and cannot maintain any information between executions. With event sourcing and checkpointing, it becomes possible to execute durable workflows that can span several minutes or even several days without failing. To ensure the replay process works correctly, orchestrator functions must produce the same results every time they are replayed. (Microsoft Learn – Durable Orchestrations)


### Execution Timeouts

The orchestrator function in Azure Durable Functions is not constrained by the time-out of Azure functions. Rather than running indefinitely, they make checkpoints, pause, and resume after an event trigger, enabling workflows that may last for hours, days, or even weeks to run without timeouts. However, activity functions still have execution time limits.  Developers can also use durable timers to set custom timeouts for activities or workflows.

 
### Communication Between Functions

In standard FaaS, functions don’t communicate directly with each other. They usually communicate through external storage systems like S3 or DynamoDB, which adds latency. Durable Functions improve this by letting orchestrator functions work together to invoke activity functions through the Durable Task Framework. The framework automatically handles messaging, state tracking, and execution history, so developers do not have to create any storage-based message communication mechanism. However, communication still depends on some backend storage services like Azure Storage.  So, while Durable Functions makes communication much easier and more structured, it still relies on storage and does not allow for direct communication between functions.  (Microsoft Learn – Performance and Scale)


### Parallel Execution (Fan-Out/Fan-In)

Durable Functions supports the fan-out/fan-in pattern, which lets an orchestrator run multiple functions in parallel and then aggregate the results. This approach helps process large amounts of data and enhance performance through parallel processing. It helps address some of the issues in the paper concerning distributed computing by providing a built-in way to coordinate parallel tasks without requiring developers to manually handle messaging, synchronization, or state management. Although it does not completely resolve all challenges of distributed systems, it makes serverless workflow development much easier. (Microsoft Learn – Fan-Out/Fan-In)


## Part 3: Critical Evaluation

Although Azure Durable Functions solves some of the limitations of first-generation serverless computing, several of the issues identified by the paper remain. Durable Functions improves workflow coordination, state management, and parallel processing, but it does not fundamentally change how serverless computing works underneath.

One limitation that remains largely unresolved is the data shipping anti-pattern. The paper argues that serverless platforms often move data to the compute resources instead of moving the code closer to the data. Azure Durable Functions does not fully resolve this issue because activity functions still need to retrieve data from services such as Azure Blob Storage, Azure SQL Database, or Cosmos DB before processing it. While Durable Functions makes workflows easier to manage, it does not reduce the amount of data that must be transferred. Therefore, data-intensive applications can still experience higher network usage, increased latency, and additional costs when working with large datasets.

A second limitation that remains unresolved is limited access to specialized hardware. The paper notes that serverless platforms do not provide direct access to the hardware associated with machine learning and high-performance computing, including GPUs and FPGAs. Azure Durable Functions does not solve this issue because it is designed to coordinate workflows, not provide computing resources. While activity functions can call external services that use GPUs, Durable Functions itself does not offer native access to specialized hardware. As a result, applications that require GPU acceleration must still rely on other services, such as Azure Machine Learning or Azure Kubernetes Service, making the solution more complex.

Another limitation, only partially addressed, is communication between functions. Durable Functions makes it easier for orchestrator functions to coordinate activity functions. However, communication still relies on storage and messaging services behind the scenes. While this simplifies development, it does not completely solve the paper’s concern that storage-based communication can cause delays and reduce performance. 
 
 Overall, Azure Durable Functions is a major improvement over traditional FaaS platforms. It adds support for long-running workflows, stateful execution, orchestration, and parallel processing, which helps address many of the limitations discussed in the paper. However, it does not completely solve all of the problems with serverless computing. Challenges such as moving large amounts of data, relying on storage for communication,  and limited access to specialized hardware still exist. As a result, Durable Functions should be seen as an improvement to serverless computing rather than a complete solution. It shows that serverless technology has advanced, but some of the core limitations identified in the paper remain. 

 ## References

1. Hellerstein, J. M., Faleiro, J., Gonzalez, J. E., Schleier-Smith, J., Sreekanti, V., Tumanov, A., & Wu, C. (2019). Serverless Computing: One Step Forward, Two Steps Back. https://www.cidrdb.org/cidr2019/papers/p119-hellerstein-cidr19.pdf
2. Microsoft Learn. (2026). Azure Durable Functions Overview. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview
3. Microsoft Learn. Durable Task Timers. (2026). https://learn.microsoft.com/en-us/azure/durable-task/common/durable-task-timers
4. Microsoft Learn. (2026). Durable orchestrations. https://docs.azure.cn/en-us/azure-functions/durable/durable-functions-orchestrations
5. Microsoft Learn. (2026). Fan-out/fan-in scenarios in Durable Functions. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-fan-in-fan-out
6. Microsoft Learn. (2026). Performance and scale in Durable Functions. https://learn.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-perf-and-scale 
7. Keyhole Software. (2025). Long-Running Workflows Made Simple with C# + Azure Durable Functions. https://keyholesoftware.com/long-running-workflows-made-simple-with-c-azure-durable-functions/
8. SystemsArchitect.io. (2026). Need for GPU or specialized hardware acceleration. https://www.systemsarchitect.io/services/azure-functions/seek-alternatives-if-you-need/pt/azure-functions-seek-alternatives-if-you-need-need-for-gpu-or-specialized-hardware-acceleration




