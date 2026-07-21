# HCM Project — Interview Revision Notes

Project: HCM Platform (similar to Darwinbox) — Employee Dashboard
Company: Document IT LLC

---

### Q1. Explain about this project brief and your role.

I worked on an HCM product, similar to Darwinbox, at Document IT LLC. It's a platform where employees log in and check things like attendance, leave balance, payroll, and their profile info — all shown on a dashboard.

My role was full-stack, but for this part of the project, I mainly worked on the backend. The dashboard was earlier using normal REST APIs, and it was slow because the frontend had to make many API calls to load one screen. My job was to fix this performance issue.

I did two main things — first, I moved the APIs from REST to GraphQL, so the frontend can get all the data in one single request instead of many. Second, I added indexing in MongoDB for the collections we use often, like employee and attendance data, so the queries run faster.

Because of these changes, the dashboard became faster and the API response time improved by around 20%.

---

### Q2. Tell me the collection names and API endpoints you have worked on.

The main collections I worked with were `employees`, `attendance`, `leaves`, and `notifications`.

Earlier, the frontend used separate REST endpoints — like `/api/employee/:id`, `/api/attendance/:id`, and `/api/leave-balance/:id` — to get this data. Each widget on the dashboard called its own endpoint, so it took multiple round trips.

After moving to GraphQL, I created a single query called `getDashboardData`, which takes the employee ID and returns employee profile, attendance, leave balance, and notifications together in one response. This reduced the number of calls the frontend needed to make.

For indexing, I added indexes on `employeeId` in the `attendance` and `leaves` collections, since those were the most frequently queried fields, and it helped avoid full collection scans.

---

### Q3. What technical issues did you face and how did you resolve them?

**Issue 1 — N+1 problem:**
When I combined employee, attendance, and leave data into one GraphQL query, the resolver was calling the database separately for each field — so instead of one call, it was making many small calls. This slowed things down.

I fixed this using **DataLoader**. It batches multiple requests into one single database call and also caches the result within the same request, so we don't hit the database again and again for the same data.

**Issue 2 — Slow MongoDB queries:**
Some queries on `attendance` and `leaves` were doing full collection scans because there was no index on `employeeId`. I checked this using MongoDB's `explain()` method, saw it was scanning the whole collection, then added an index on `employeeId`. After that, the query used the index instead of scanning everything, and response time improved a lot.

---

### Q4. What is DataLoader?

DataLoader is a small library (made by Facebook) mainly used with GraphQL to fix the N+1 query problem.

Without DataLoader — if you ask for 10 employees and each one needs their leave balance, the resolver hits the database 10 separate times, once per employee. That's slow.

With DataLoader — it collects all those requests happening in the same tick, batches them into **one single database call** (like `WHERE employeeId IN [1,2,3...10]`), and returns the right result to each one. It also **caches** results during that same request, so the same data isn't fetched twice.

**In short: batching + caching, per request.**

> "I used DataLoader to batch multiple database calls into one, instead of calling the database separately for each item in a list. This solved the N+1 problem and reduced the load on MongoDB."

---

### Q5. Tell me more techniques that help improve API response.

1. **Caching (Redis)** — Store frequently accessed data (employee profile, org hierarchy) in Redis instead of hitting DB every time.
2. **Pagination** — Return data in small chunks instead of all records at once.
3. **Field-level selection (GraphQL)** — Frontend asks only for fields it needs; backend doesn't send extra data.
4. **Database indexing** — Add indexes on fields filtered often (`employeeId`, `status`, `date`).
5. **Connection pooling** — Reuse a pool of open DB connections instead of opening new ones per request.
6. **Compression (gzip)** — Compress API response before sending, especially for large JSON payloads.
7. **Query projection** — Select only required fields from MongoDB (`.select('name email')`) instead of full documents.
8. **Avoiding N+1 in loops** — Batch DB calls instead of looping and calling DB one by one.
9. **Async/parallel calls** — Use `Promise.all` for independent data fetches instead of sequential calls.
10. **Load balancing / horizontal scaling** — Run multiple API server instances behind a load balancer for high traffic.

**Most relevant for this project (mention these first):** Redis caching, indexing, DataLoader, GraphQL field selection, query projection.

---

### Q6. Why GraphQL over REST, when REST can also handle over-fetching?

You're right, over-fetching in REST can be handled — using query params or custom endpoints. But that's not the main reason GraphQL was chosen.

1. **Multiple round trips problem** — Even with over-fetching fixed, the dashboard still needs data from 4-5 resources (employee, attendance, leave, notifications). REST means 4-5 separate calls. GraphQL gets all of this in **one single request**.

2. **Every new UI need = new endpoint (in REST)** — New field or data combination means a new/modified REST endpoint with versioning. In GraphQL, frontend just asks for the extra field — no backend endpoint change needed.

3. **Frontend controls exact shape of data** — In REST, backend decides response shape. In GraphQL, frontend decides exactly what fields and nested data it wants, in one query — useful when different dashboard widgets need different data combinations.

4. **Simpler for evolving requirements** — Dashboard kept growing with more widgets. REST means constant endpoint changes; GraphQL means schema/resolver updates with less backend churn.

**Bottom line:** The real reason wasn't just fixing over-fetching — REST can patch that. The real reason was reducing multiple round trips into one, and making it easier to scale the dashboard as requirements grew.

---

### Q7. Can you write a small code sample of your implementation?

```javascript
// 1. MongoDB Index (run once, or in schema)
// Speeds up queries filtering by employeeId
db.attendance.createIndex({ employeeId: 1 });
db.leaves.createIndex({ employeeId: 1 });

// 2. DataLoader setup - batches DB calls
const DataLoader = require("dataloader");

const attendanceLoader = new DataLoader(async (employeeIds) => {
  const records = await Attendance.find({
    employeeId: { $in: employeeIds },
  });

  // Map results back in the same order as employeeIds
  return employeeIds.map((id) =>
    records.find((r) => r.employeeId.toString() === id.toString()),
  );
});

const leaveLoader = new DataLoader(async (employeeIds) => {
  const records = await Leave.find({
    employeeId: { $in: employeeIds },
  });
  return employeeIds.map((id) =>
    records.find((r) => r.employeeId.toString() === id.toString()),
  );
});

// 3. GraphQL Schema
const typeDefs = `
  type Employee {
    id: ID
    name: String
    attendance: Attendance
    leave: Leave
  }

  type Attendance {
    status: String
    checkIn: String
  }

  type Leave {
    balance: Int
  }

  type Query {
    getDashboardData(employeeId: ID!): Employee
  }
`;

// 4. Resolvers - use DataLoader instead of direct DB calls
const resolvers = {
  Query: {
    getDashboardData: async (_, { employeeId }) => {
      const employee = await Employee.findById(employeeId);
      return employee;
    },
  },
  Employee: {
    attendance: (employee) => attendanceLoader.load(employee.id),
    leave: (employee) => leaveLoader.load(employee.id),
  },
};
```

**How to explain in interview:**

> "Here, when frontend asks for employee data with attendance and leave, instead of calling database separately for each one, DataLoader collects all the IDs and makes **one batched call** using `$in` query. And I added index on `employeeId` in both collections so this batched query is fast, not scanning the whole collection."

---

## Quick Recap (30-second version)

- Built HCM employee dashboard, migrated REST → GraphQL to cut multiple API round trips into one
- Added MongoDB indexes on `employeeId` in `attendance` and `leaves` collections to avoid collection scans
- Used DataLoader to batch and cache DB calls, solving N+1 problem inside resolvers
- Result: ~20% improvement in API response time
- Other techniques known: Redis caching, pagination, query projection, connection pooling, parallel async calls
