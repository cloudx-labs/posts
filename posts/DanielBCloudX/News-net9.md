---
title: What's New in .NET 9
published: false
description: this article will showcase the latest updates in .NET 9.
tags: 'one-tag, another-tag'
cover_image: ./assets/upgrade-Net9.png
---

## Introduction

.NET 9 is a Short-Term Support (STS) release, setting the stage for the highly anticipated Long-Term Support (LTS) version 10. Despite its short-term nature, .NET 9 introduces several exciting new features and enhancements that provide a glimpse into the future direction of the .NET ecosystem. In this article, I will showcase the latest updates in .NET 9.

## Improved Performance and Scalability

### Key Performance Enhancements

1. **Adaptive Server GC (Garbage Collection):** .NET 9 introduces adaptive server garbage collection, which adjusts memory usage according to application demands. This optimizes performance in environments with fluctuating memory needs.
2. **Vectorization Improvements:** The runtime has been optimized to support new silicon architectures like Arm64 SVE and Intel AVX10. Loop vectorization and bounds check elimination accelerate workloads, while exceptions now run 50% faster.
3. **Optimized LINQ Operations:** Common LINQ operations, such as `Take` and `DefaultIfEmpty`, are now up to 10 times faster. This reduces the overhead of these operations, improving overall performance in data-intensive applications.
4. **Adaptive Server GC Enhancements:**
   - **Heap Utilization:**
     - **.NET 8.0:** 70%
     - **.NET 9.0:** 85%
     - **Improvement:** 15% increase in heap utilization.

   - **Pause Time Reduction:**
     - **.NET 8.0:** 200 ms
     - **.NET 9.0:** 150 ms
     - **Improvement:** 25% reduction in pause times.

   - **Memory Footprint:**
     - **.NET 8.0:** 1.5 GB
     - **.NET 9.0:** 1.2 GB
     - **Improvement:** 20% reduction in memory footprint.

## OpenAPI Support and Swagger

Starting with .NET 9, Swagger is no longer included by default. Instead, developers are encouraged to use `Microsoft.AspNetCore.OpenApi` for API documentation. This change aims to provide a more streamlined and integrated experience for documenting APIs.

## GUID Enhancements in .NET 9

### Guid V4

The Guid type has been present in .NET from the very beginning, and a new Guid can be created using the `NewGuid()` method. This method generates a version 4 GUID following the UUID Version 4 specification in RFC 9562. GUIDs are globally unique, making them ideal for distributed systems. However, version 4 GUIDs are not sequential, which can lead to index fragmentation in relational databases.

### Guid V7

.NET 9 introduces a new GUID implementation based on timestamp and randomness. A Guid can be created using the `CreateVersion7()` method, which produces sequential GUIDs. This sequentiality makes them more suitable for relational databases, alleviating fragmentation issues. Additionally, the Guid type now includes a new `Version` property, allowing you to check the version of the created GUID.

### Example of GUID Generation

```csharp
for (int i = 0; i < 5; i++)
{
    var guid = Guid.NewGuid();
    Console.WriteLine($"V{guid.Version}: {guid}");
}
**Output:**
V4: 0557b321-abcf-4390-abee-4b8fbf93ff34
V4: 21a98165-af1e-477e-9dee-7eb9c79e6c77
V4: 7dbbf973-c55a-4917-87a5-95c16f356262
V4: b13892f2-334f-409a-b9de-d90dea21eed4**Output:**

for (int i = 0; i < 5; i++)
{
    var guid = Guid.CreateVersion7();
    Console.WriteLine($"V{guid.Version}: {guid}");
}

**Output:**
V7: 01917bbe-d973-7beb-a813-106fcb4eff98
V7: 01917bbe-d973-703c-8365-b7596740ac82
V7: 01917bbe-d973-7234-a580-5f07730a3ad7
V7: 01917bbe-d973-7751-b8ba-bb73afab4a5d
V7: 01917bbe-d973-7d36-9be0-2e6317919153

```

Version 7 GUIDs are sequential, making them more suitable for use in relational databases. You can also pass a `DateTimeOffset` when creating a new Guid to control the timestamp.
V4: 52dc44f7-76e0-4689-a5e6-1a0f1c5f37a3

## Advanced Runtime Features

.NET 9 introduces new runtime features that enhance performance and flexibility:

**Feature Switches with Trimming:** Developers can now toggle features on or off at runtime, allowing for leaner builds tailored to specific deployment scenarios. This results in smaller application sizes and faster load times.

**Enhanced Arm64 Vectorization:** Applications targeting modern Arm64 hardware benefit from optimized vectorization and improved code generation. This ensures that .NET applications can leverage the full power of devices ranging from mobile phones to cloud-based servers.

These runtime improvements not only enhance performance but also provide developers with greater control over application behavior and deployment.

## Cloud-Native Development with .NET Aspire

.NET 9 takes cloud-native development to the next level with the introduction of .NET Aspire, an all-in-one toolkit designed to simplify the creation of production-ready, observable cloud-native applications.

With .NET Aspire, developers gain access to pre-configured templates and tools that accelerate the setup of cloud-native architectures. Whether deploying to Kubernetes, leveraging containerized services, or setting up observability pipelines, .NET Aspire ensures a smooth and efficient workflow.

The platform supports microservices, seamlessly integrating with Azure Functions to create lightweight, modular components that are easy to deploy, maintain, and scale.

Additionally, .NET Aspire simplifies serverless computing, allowing developers to focus on writing business logic while the platform handles scalability and availability automatically.

As cloud-native solutions continue to dominate the software development landscape in 2025, .NET Aspire positions .NET 9 as a powerful choice for developers looking to deliver scalable, resilient, and efficient applications.

## AI Integration and Developer Productivity in .NET 9

In .NET 9, Microsoft has introduced a suite of tools and enhancements aimed at seamlessly integrating AI capabilities into applications while boosting developer productivity.

### AI Building Blocks

- The introduction of `Microsoft.Extensions.AI` and `Microsoft.Extensions.VectorData` provides a unified layer of C# abstractions for interacting with AI services, including large language models and vector stores.
- These libraries facilitate the integration of AI functionalities, enabling developers to perform CRUD operations on vector stores and utilize vector and text search capabilities.
- Additionally, .NET 9 includes new tensor types, such as `TensorPrimitives` and `Tensor<T>`, which expand AI capabilities by enabling efficient encoding, manipulation, and computation of multi-dimensional data.

### Boosting Productivity

- To enhance developer productivity, .NET 9 introduces new LINQ methods like `CountBy` and `AggregateBy`.
  - The `CountBy` method groups elements by a specified key and returns an enumeration of key-value pairs, simplifying the process of counting occurrences without the need for intermediate groupings via `GroupBy`.
  - The `AggregateBy` method allows for aggregating data by a key selector, providing a more streamlined approach to data aggregation tasks.
  
- These enhancements make data querying more intuitive and concise, reducing the complexity of common operations.
- Furthermore, improvements to the `TimeSpan` structure offer more intuitive methods for time calculations, allowing developers to work with time intervals more efficiently.

By integrating these AI building blocks and productivity enhancements, .NET 9 empowers developers to create intelligent, efficient, and maintainable applications, positioning them to meet the evolving demands of software development in 2025 and beyond.

## C# 13: Modernizing Code

C# 13 introduces features that make the language more expressive and reduce boilerplate code, streamlining the development process.

**Object Initializers with Implicit Indexers:** This feature allows developers to initialize collections or dictionary-like objects with more concise syntax. Instead of explicitly specifying keys or indices, implicit indexers simplify the creation of objects with complex structures, making your code cleaner and easier to maintain.

**Improved Method Group Handling:** C# 13 refines how method groups are treated, improving type inference and reducing ambiguity. This enhancement simplifies scenarios like passing methods as delegates or using them in LINQ expressions.

**params Collections:** Writing methods that accept a variable number of arguments becomes more elegant with params collections. This improvement allows developers to pass arrays or lists directly to variadic methods, enhancing readability and reducing intermediate steps when working with collections.

These updates help C# developers focus on implementing functionality without getting bogged down by verbose syntax or unnecessary complexity.

## .NET Libraries and Framework Updates

.NET 9 introduces significant updates to its libraries and frameworks, focusing on enhancing data handling, security, and database integration. These improvements help developers create efficient and secure applications with minimal overhead, positioning .NET 9 as a powerful tool for modern development in 2025.

### System.Text.Json Improvements

System.Text.Json, the lightweight JSON library in .NET, receives key updates:

- **JSON Schema Export:** Developers can now export JSON schemas directly from C# classes, simplifying validation and interoperability with APIs.
- **Customizable Indentation:** Improved control over serialization allows for tailored formatting of JSON output.
- **Support for Multiple Root-Level JSON Values:** System.Text.Json now handles multiple root-level JSON values in a single operation, enabling smoother integration with non-standard JSON streams.

These enhancements make System.Text.Json an even more robust tool for managing structured data in web applications and APIs.

### LINQ and Collections Additions

.NET 9 brings updates to LINQ and collections, making data handling simpler and more powerful:

- **PriorityQueue Updates:** Enhanced functionality for `PriorityQueue<TElement, TPriority>` streamlines the management of prioritized tasks or items.
- **New LINQ Methods:** Features like `CountBy` and `AggregateBy` reduce the complexity of common data aggregation tasks, allowing developers to write cleaner, more concise queries.

These additions improve productivity when working with complex data sets, making LINQ and collections more versatile and intuitive.

## Conclusion

.NET 9 focuses on reducing developers' workload by making applications faster and more efficient with minimal effort. It includes smarter memory management, enhanced performance, streamlined AI integration, and improved cross-platform compatibility. Key features like adaptive garbage collection, loop optimizations, and robust AI support with Microsoft.Extensions.AI and VectorData ensure that .NET 9 handles complex and frustrating tasks, allowing developers to concentrate on writing impactful code.
