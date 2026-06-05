# Stage 1: Notification System Design (REST API Contract + Real-time Design)

---

## 1. Objective

Design a scalable notification system that allows backend services to send, store, and deliver notifications to logged-in users in real-time and also provide REST APIs for frontend consumption.

---

## 2. Core Actions Supported

The notification platform should support the following core actions:

1. Create Notification
2. Fetch User Notifications
3. Mark Notification as Read
4. Delete Notification
5. Real-time Notification Delivery

---

## 3. Data Model (Notification Schema)

```json
{
  "notificationId": "string",
  "userId": "string",
  "title": "string",
  "message": "string",
  "type": "info | warning | error | success",
  "isRead": false,
  "createdAt": "timestamp"
}
```
