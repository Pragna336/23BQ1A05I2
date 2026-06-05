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
