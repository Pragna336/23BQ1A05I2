# Stage 1:

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

---

## 4. REST API Design

---

### 4.1 Create Notification

**Endpoint:**
POST /api/notifications

**Headers:**
```json
{
  "Content-Type": "application/json",
  "Authorization": "Bearer <token>"
}
```

**Request Body:**
```json
{
  "userId": "12345",
  "title": "New Message",
  "message": "You have received a new message",
  "type": "info"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Notification created successfully",
  "data": {
    "notificationId": "n1",
    "userId": "12345"
  }
}
```

---

### 4.2 Get User Notifications

**Endpoint:**
GET /api/notifications/:userId

**Headers:**
```json
{
  "Authorization": "Bearer <token>"
}
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "notificationId": "n1",
      "title": "New Message",
      "message": "You have received a new message",
      "type": "info",
      "isRead": false,
      "createdAt": "2026-06-05T10:00:00Z"
    }
  ]
}
```

---

### 4.3 Mark Notification as Read

**Endpoint:**
PATCH /api/notifications/:notificationId/read

**Headers:**
```json
{
  "Authorization": "Bearer <token>"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Notification marked as read"
}
```

---

### 4.4 Delete Notification

**Endpoint:**
DELETE /api/notifications/:notificationId

**Headers:**
```json
{
  "Authorization": "Bearer <token>"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Notification deleted successfully"
}
```

---

## 5. Real-Time Notification Mechanism

The system uses **WebSockets (Socket.IO)** for real-time notifications.

### 5.1 Connection Setup
Client connects using:
ws://server-url?token=<JWT>

---

### 5.2 Server Event

```json
{
  "event": "new_notification",
  "data": {
    "notificationId": "n1",
    "title": "Alert",
    "message": "Your task is completed"
  }
}
```

---

### 5.3 Client Listener

```javascript
socket.on("new_notification", (data) => {
  console.log("New Notification Received:", data);
});
```

---

## 6. Architecture Flow

Frontend → REST API → Backend → Database  
                      ↓  
              WebSocket Server  
                      ↓  
        Real-time Notification Push

---

# Stage 2

## 1. Database Choice

We choose **MongoDB (NoSQL Database)** for the Notification System.

### Why MongoDB?
- Stores data in JSON-like format (perfect for notifications)
- Easy scalability for large number of users
- Fast read and write operations
- Flexible schema (easy to modify notification structure)
- Suitable for real-time applications

---

## 2. Database Name

```
notificationDB
```

---

## 3. Collection Design

### Collection: notifications

```json
{
  "_id": "ObjectId",
  "notificationId": "n1",
  "userId": "12345",
  "title": "New Message",
  "message": "You have received a new message",
  "type": "info",
  "isRead": false,
  "createdAt": "2026-06-05T10:00:00Z"
}
```

---

## 4. MongoDB Operations

---

### 4.1 Insert Notification

```js
db.notifications.insertOne({
  notificationId: "n1",
  userId: "12345",
  title: "New Message",
  message: "You have received a new message",
  type: "info",
  isRead: false,
  createdAt: new Date()
})
```

---

### 4.2 Get User Notifications

```js
db.notifications.find({ userId: "12345" }).sort({ createdAt: -1 })
```

---

### 4.3 Get Unread Notifications

```js
db.notifications.find({
  userId: "12345",
  isRead: false
})
```

---

### 4.4 Mark Notification as Read

```js
db.notifications.updateOne(
  { notificationId: "n1" },
  { $set: { isRead: true } }
)
```

---

### 4.5 Delete Notification

```js
db.notifications.deleteOne({ notificationId: "n1" })
```

---

## 5. Indexing (Performance Improvement)

```js
db.notifications.createIndex({ userId: 1 })
db.notifications.createIndex({ createdAt: -1 })
```

---

## 6. Scaling Considerations

- Use indexing for fast queries
- Use pagination for large datasets
- Archive old notifications
- Use caching (Redis) for frequently accessed data

---

# Stage 3

## 1. Given Query Analysis

### Original Query:
```sql
select * 
from notifications 
where studentId = 1042 
and isRead = false 
order by createdAt desc;
```

---

## 2. Is the query accurate?

Yes, the query is logically correct.  
It fetches unread notifications for a specific student and sorts them by latest first.

---

## 3. Why is the query slow?

The query is slow due to the following reasons:

- No proper indexing on `studentId`, `isRead`, and `createdAt`
- Full table scan is performed
- Large dataset (millions of records)
- Sorting operation on unindexed column

---

## 4. Improved Solution

### Recommended Index:
```sql
CREATE INDEX idx_student_unread_created 
ON notifications (studentId, isRead, createdAt DESC);
```

---

### Optimized Query:
```sql
select *
from notifications
where studentId = 1042
and isRead = false
order by createdAt desc;
```

---

## 5. Computation Cost

### Before Index:
- Time Complexity: O(n)
- Full table scan required
- Expensive sorting operation

### After Index:
- Time Complexity: O(log n)
- Faster lookup using index tree
- Reduced sorting cost

---

## 6. Is indexing every column effective?

No, it is not effective.

Reasons:
- Increases storage cost
- Slows down insert, update, and delete operations
- Maintenance overhead for database
- Only useful indexes should be created based on query patterns

---

## 7. Query: Placement Notifications in Last 7 Days

```sql
SELECT *
FROM notifications
WHERE notificationType = 'Placement'
AND createdAt >= NOW() - INTERVAL 7 DAY;
```

---

## 8. Recommended Index

```sql
CREATE INDEX idx_type_created 
ON notifications (notificationType, createdAt);
```

---

# Stage 4

## 1. Problem Statement

Notifications are being fetched on every page load for every student.  
This is causing high database load, slow response times, and poor user experience.

---

## 2. Problems Identified

- Excessive database read operations
- Repeated fetching of same data
- High latency during page load
- DB overload under high concurrent users
- Inefficient use of resources

---

## 3. Suggested Solutions

### 3.1 Caching Layer (Recommended Solution)

Use Redis or in-memory caching to store frequently accessed notifications.

#### Approach:
- Store user notifications in cache after first DB fetch
- Serve subsequent requests from cache
- Update cache when new notification is created

#### Benefits:
- Very fast response time (O(1) access)
- Reduces database load significantly
- Improves scalability

#### Trade-offs:
- Cache invalidation complexity
- Possible stale data if not updated properly
- Additional infrastructure (Redis setup)

---

### 3.2 Pagination

Fetch notifications in chunks instead of loading all at once.

#### Example:
- Limit 20–50 notifications per request
- Use page and limit parameters

#### Benefits:
- Reduces memory usage
- Faster response time
- Scales well with large datasets

#### Trade-offs:
- More API calls needed for full history
- Slight complexity in frontend handling

---

### 3.3 Lazy Loading

Load notifications only when user scrolls or opens notification panel.

#### Benefits:
- Reduces initial page load time
- Improves perceived performance

#### Trade-offs:
- Slight delay in accessing older notifications
- Requires frontend implementation complexity

---

### 3.4 WebSocket Based Updates

Instead of fetching repeatedly:
- Maintain persistent connection
- Push notifications in real time

#### Benefits:
- Eliminates repeated polling
- Instant delivery of updates
- Reduces API calls

#### Trade-offs:
- Complex implementation
- Requires connection management
- Not ideal for unreliable networks

---

### 3.5 Database Optimization

- Proper indexing (userId, createdAt)
- Query optimization
- Avoid fetching unnecessary fields

#### Benefits:
- Improves query performance
- Reduces DB execution time

#### Trade-offs:
- Limited improvement compared to caching
- Does not solve repeated request issue fully

---

## 4. Recommended Combined Strategy

Best production approach:

1. Use Redis caching for frequent reads
2. Use pagination for large datasets
3. Use WebSockets for real-time updates
4. Optimize DB with proper indexing

---

## 5. Conclusion

The best scalable solution is a hybrid approach combining caching, pagination, and real-time WebSocket updates to reduce database load and improve user experience.
