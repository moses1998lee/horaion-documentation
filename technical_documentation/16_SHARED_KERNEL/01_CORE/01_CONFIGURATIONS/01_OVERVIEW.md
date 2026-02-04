# 01 - Configurations Overview

> **Global Spring Boot Configuration Beans**

---

## Introduction

This directory contains the central configuration classes that define how the Spring Boot application behaves at a low level. By centralizing these settings in the Shared Kernel, we ensure consistent behavior across all modules.

## Configuration Catalog

| Class | Purpose | Key Feature |
| :--- | :--- | :--- |
| **[AsyncConfiguration](02_ASYNC.md)** | Controls background processing. | Enables **Java 21 Virtual Threads** for high-throughput concurrency. |
| **[JacksonConfiguration](03_JACKSON.md)** | Controls JSON serialization. | Enforces **ISO-8601** date standards. |

---

{% hint style="info" %}
**Note:** These configurations are automatically loaded by the main application class. You typically do not need to modify them unless you are changing global system behavior.
{% endhint %}
