## Improved API performance by 20% by optimizing GraphQL resolvers, MongoDB queries, indexing strategies, and Efficient data access patterns for high-scale distributed systems

# HRMS API Performance Optimization (Interview Revision)

# Project Context

Worked on an HRMS platform similar to Darwinbox/Workday.

**Contribution** - Improved API performance by \~20%. - Optimized
GraphQL resolvers. - Optimized MongoDB queries. - Added proper
indexes. - Reduced unnecessary database calls. - Improved data access
patterns for high-scale systems.

---

# 1. Employee Dashboard API

## Why is it expensive?

Every employee opens the dashboard after login.

Dashboard loads: - Profile - Attendance - Leave balance -
Notifications - Holidays - Pending approvals

### Problems

- Multiple database queries
- Sequential API calls
- Large payload

### Optimizations

### GraphQL

- Execute independent resolvers in parallel (`Promise.all`)
- Return only requested fields

### MongoDB

- Use projection
- Use aggregation for summaries

### Indexing

Compound indexes: - employeeId + date - employeeId + status

### Result

- Response time reduced significantly (e.g. 800ms → \~250ms)

---

# 2. Employee Directory API

## Why is it expensive?

Frequently used by: - HR - Managers - Employee Search - Organization
Chart

### Biggest Problem

GraphQL **N+1 Query Problem**

Instead of: - 1 employee query - 100 manager queries - 100 department
queries

Use DataLoader to batch requests.

### Optimizations

### GraphQL

- DataLoader
- Pagination
- Fetch only requested fields

### MongoDB

- Projection
- Efficient filtering

### Indexing

- email
- employeeId
- department
- status
- Compound: (department, status)

### Result

201 DB queries → \~3 queries

---

# 3. Attendance & Leave API

## Why is it expensive?

Large organizations generate millions of attendance records.

### Problems

Without indexes: - Collection scans - Slow queries

### Optimizations

### GraphQL

- Cursor pagination
- Fetch only required fields

### MongoDB

- Aggregation pipeline
- Projection
- Limit + Sort

### Indexing

Compound: - employeeId + date

### Result

350ms → \~40ms

---

# Key Interview Points

## GraphQL

- Solved N+1 problem using DataLoader.
- Parallel resolver execution.
- Returned only requested fields.
- Implemented pagination.

## MongoDB

- Projection
- Aggregation
- Efficient filters
- Limit + Sort

## Indexing

- Compound indexes
- Index frequently filtered columns
- Avoid collection scans

---

# Interview Answer (60 Seconds)

"I optimized high-traffic APIs such as the Employee Dashboard, Employee
Directory, and Attendance APIs. For GraphQL, I eliminated the N+1 query
problem using DataLoader, executed independent resolvers in parallel,
and returned only the requested fields. On MongoDB, I optimized queries
using projections, aggregation pipelines, pagination, sorting, and
compound indexes like employeeId + date. These optimizations reduced
unnecessary database calls, minimized collection scans, and improved API
response time by around 20%."

---

# Common Follow-up Questions

## Q1. Why GraphQL instead of REST?

**Answer:** GraphQL allows clients to request only the fields they need,
reducing over-fetching and under-fetching. It is ideal for dashboards
that combine data from multiple sources.

---

## Q2. What is the N+1 problem?

**Answer:** A parent query triggers additional queries for each child
record. DataLoader batches these into a single query, greatly reducing
database calls.

---

## Q3. What is DataLoader?

**Answer:** A GraphQL utility that batches and caches database requests
during a request lifecycle, preventing duplicate queries.

---

## Q4. Why use compound indexes?

**Answer:** If queries filter on multiple fields (e.g. employeeId and
date), a compound index allows MongoDB to find matching documents
quickly without scanning the collection.

---

## Q5. When should you use MongoDB Aggregation?

**Answer:** When calculations like totals, counts, grouping, or
summaries can be done in the database instead of application code.

---

## Q6. What is projection?

**Answer:** Fetching only required fields from MongoDB to reduce payload
size and improve performance.

---

## Q7. How did you measure the improvement?

**Answer:** Compared API response times before and after optimization
using logs/APM tools, monitored database query execution, and verified
reduced response latency and fewer database operations.

---

# Keywords to Remember

- GraphQL
- DataLoader
- N+1 Problem
- Promise.all
- Projection
- Aggregation
- Pagination
- Compound Index
- MongoDB Explain
- Query Optimization
- Collection Scan
- High Traffic APIs
