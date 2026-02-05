# 02 - Engine Infrastructure

> **Calculation Engine Integration**

The `engine` package acts as the bridge to our high-performance computational core (e.g., Python AI Model or Optimized Solver).

---

## 1. Schedule Engine Client
**File**: ``ScheduleEngineClient.java``

This Interface describes the contract for offloading complex scheduling tasks.

### Use Case
1.  User clicks "Auto-Schedule".
2.  Backend collects all Shifts, Employees, and Rules.
3.  Backend sends this huge payload to the **Engine Service**.
4.  Engine returns the optimized schedule.

{% hint style="danger" %}
**Critical:** The Engine is stateless. You must verify that the inputs provided to it (Employee IDs, Shift IDs) are valid before making the call. The Engine assumes garbage out, garbage in.
{% endhint %}
