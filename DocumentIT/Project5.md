# Event-Driven Notification System (SyntraOne) — Interview Revision Notes

Project: SyntraOne — Enterprise Business Management Platform (ERP)
Feature: Event-Driven Notification System
My Role: Backend — event publishing, async consumers, queue setup, notification storage, real-time delivery

---

### Q1. Explain about the project you worked on.

I worked on **SyntraOne**, an enterprise business management platform — like an ERP — that helps companies manage day-to-day operations from one single system. It combines modules like finance, sales, procurement, inventory, HR, and project management, so businesses don't need to switch between multiple disconnected tools.

One of its key strengths is e-invoicing and business process automation — generating, validating, and managing invoices while integrating with existing systems and following government tax regulations.

My specific contribution was building the **Event-Driven Notification System**. Whenever an important business event happens in the ERP — like an invoice getting approved, a purchase order being created, or a payment failing — the system automatically notifies the right users, instead of someone having to manually check the ERP.

**Example:** When a manager approves an invoice, the application publishes an event as soon as that approval completes. Different services listen to that event and do their own job — one sends an email to the finance team, another shows an in-app notification, another updates the notification history, and if needed, it can trigger another workflow. Each service works independently without being tightly coupled.

I built the backend APIs, published events after business operations, wrote consumers to process events asynchronously using Redis and BullMQ, built notification templates, stored notification history in PostgreSQL, and exposed APIs for the React frontend to display notifications.

**Impact:** Before this, users had to manually check different modules to know if something happened. After, notifications went out automatically and in real time — reducing manual follow-ups, and keeping the system scalable since notifications are processed in the background, not blocking the main request.

---

### Q2. What was your role, and what technical issues did you face?

**My role:** Backend — building APIs that publish events after business actions (like invoice approval), writing consumers/workers to process events asynchronously, setting up Redis + BullMQ queue, designing notification templates, storing notification history in PostgreSQL, and exposing APIs for the React frontend.

**Issue 1 — Notifications slowing down the main request**

> **Q: What happened when notifications were sent directly inside the main request?**
> Sending emails or push notifications directly inside the same request (e.g., right when a manager approves an invoice) made the API slow, since the user had to wait for all of it to finish before getting a response.
>
> **Fix:** Moved to an event-driven approach — as soon as the main action completes, we publish an event to a Redis/BullMQ queue and return the response immediately. Email sending, in-app notification, and logging happen in the background through separate consumers.

**Issue 2 — Multiple services needed to react to the same event**

> **Q: How did you handle multiple different actions needing to happen for one event, without tightly coupling everything?**
> When an invoice was approved, several things needed to happen — send email, show in-app notification, update history, maybe trigger another workflow. Writing all this in one place made the code tightly coupled and hard to maintain.
>
> **Fix:** Used a publish-subscribe pattern — the main service publishes one event, like `invoice.approved`, and each consumer (email service, in-app notification service, audit log service) independently listens and does its own job. Adding a new notification channel later doesn't require touching existing code.

**Issue 3 — Making sure a failed job doesn't get lost**

> **Q: What if a notification job fails, like the email service being down?**
> Without proper handling, a failed notification job would just be lost, and the user would never know an action happened.
>
> **Fix:** BullMQ has built-in retry logic — configured jobs to retry a few times with a delay if they failed, instead of failing silently. Made the system more reliable without building retry logic manually.

**Issue 4 — Real-time delivery to the frontend**

> **Q: How did you make sure users saw notifications instantly, not just after a refresh?**
> Even after a notification was processed in the backend/queue, the user needed to see it immediately on the UI.
>
> **Fix:** Used Socket.IO to push real-time notifications to the connected user's browser as soon as the backend processed the event, so in-app notifications appeared instantly without needing a refresh.

---

### Q3. What tech stack did you use?

| Layer                    | Technology                      | Where used                                                                                                              |
| ------------------------ | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------- |
| Backend                  | TypeScript, Node.js, Express.js | APIs that publish events after business actions; core notification service logic                                        |
| Queue / Event Processing | Redis + BullMQ                  | Message queue — events like `invoice.approved` go into the queue and get processed asynchronously by background workers |
| Database                 | PostgreSQL                      | Stores notification history and templates, so users can see past notifications                                          |
| Frontend                 | React                           | Displays notifications to the user (built by frontend team; I exposed the APIs it consumes)                             |
| Real-time                | Socket.IO                       | Pushes real-time in-app notifications to the browser instantly, without page refresh                                    |
| Email                    | SMTP/Nodemailer                 | Sends email notifications (e.g., notifying finance team when invoice is approved)                                       |
| API                      | REST APIs                       | Standard request/response endpoints — fetching notification history, marking as read                                    |
| Version Control          | Git                             | Source control and collaboration with the team                                                                          |

**My main hands-on areas:** Node.js/Express backend, Redis + BullMQ for the event queue, PostgreSQL for notification storage, and Socket.IO for real-time delivery.

---

### Q4. Why was this feature needed in the project?

Before this feature, SyntraOne had many modules — finance, sales, procurement, inventory, HR — with things happening across them every day, like an invoice getting approved or a payment failing. But there was no way for users to know unless they manually opened the ERP and checked each module.

This caused real problems — the finance team not knowing an invoice was approved until they happened to check, or a manager finding out about a failed payment much later. This led to delays, missed follow-ups, and wasted time manually checking things.

This feature solved that — whenever an important business event happens, the right people get notified automatically and in real time, through email or in-app notification, without manual checking.

From a technical side, not every action needs to hold up the user's request — sending an email or writing an audit log can take time. Doing this directly inside the main request would make the API feel slow. This feature also kept the system fast and scalable, since background tasks are processed separately using a queue instead of blocking the main business operation.

**In short:** Needed to keep users informed automatically and in real time, reduce manual checking and delays, and keep the main application fast and scalable through asynchronous processing.

---

## Quick Recap (30-second version)

> "I built the Event-Driven Notification System for SyntraOne, an ERP platform covering finance, sales, procurement, inventory, and HR. Whenever a business event happens — like an invoice approval — the backend publishes an event to a Redis/BullMQ queue instead of handling notifications inline. Independent consumers then send emails, push in-app notifications via Socket.IO, and log history in PostgreSQL — all asynchronously, without slowing down the main request. This replaced manual checking across modules with automatic, real-time notifications, while keeping the system scalable through background processing and reliable through BullMQ's built-in retry logic."

![alt text](Event-Driven-Notification.png)
