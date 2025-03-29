+++
author = "Amir Candido"
title = "Implementing Attribute-Based Access Control (ABAC) In Modern Applications"
date = "2024-11-16"
description = "What is ABAC and How Does It Work?"
draft  = false
tags = [
    "permissions",
    "user-management",
    "react",
    "express",
    "javascript",
]
categories = [
    "APIs",
    "Security"
]
series = ["Themes Guide"]
aliases = ["migrate-from-jekyl"]
image = "code-snippet.png"
+++

### Introduction

In the evolving landscape of software development, ensuring secure and flexible access control is paramount. Traditional **Role-Based Access Control (RBAC)**, which grants permissions based solely on predefined roles, often falls short in applications requiring more nuanced control. This is where **Attribute-Based Access Control (ABAC)** steps in as a powerful alternative.

ABAC extends the capabilities of RBAC by introducing a more granular approach, considering various attributes such as the user's role, their relationship to the resource, the resource’s properties, and even the context of the request. This fine-grained control makes ABAC especially valuable in modern applications that require complex permission logic or dynamic policies.

In this tutorial, we’ll explore how to implement ABAC in a Express-API app. We’ll demonstrate how to enforce permissions dynamically using attributes and showcase how ABAC ensures secure and scalable access management for modern web applications.

By the end of this article, you'll have a comprehensive understanding of ABAC, its implementation, and how to integrate it seamlessly into your web applications for superior access control.

### What is Attribute-Based Access Control?

Attribute-Based Access Control (ABAC) is an advanced access control paradigm that determines permissions based on the evaluation of attributes. Unlike traditional access control mechanisms like Role-Based Access Control (RBAC), which rely on predefined roles, ABAC uses a more dynamic and fine-grained approach by evaluating metadata (attributes) associated with users, resources, and the environment.

ABAC’s flexibility makes it an ideal choice for applications requiring highly granular access controls or those needing to adapt to complex, context-dependent rules. Let’s delve into its key components:

#### **Key Components of ABAC**

1. **Subjects**  
   These are the entities (typically users) making the request. In ABAC, attributes such as user roles, department, or location define the subject. For example:  
   - A user’s role (`admin`, `editor`, `viewer`).  
   - The department they belong to (`finance`, `HR`).  
   - Their geographical location (`New York`, `Remote`).

2. **Objects**  
   Objects refer to the resources being accessed. Attributes describe the object’s properties, like ownership, sensitivity level, or creation date. For instance:  
   - A document’s sensitivity (`confidential`, `public`).  
   - The owner of a file (`user123`).  
   - A task’s priority (`high`, `low`).  

3. **Attributes**  
   Attributes are metadata that define the context of access. These can belong to the subject, object, or environment:  
   - **Subject attributes**: User's ID, roles, or clearance level.  
   - **Object attributes**: Resource type, ownership, or status.  
   - **Environmental attributes**: Time of request, device used, or IP address.  

4. **Policies**  
   Policies are the core of ABAC. They are logical rules that evaluate attributes to grant or deny access. Policies can take various forms, such as:  
   - "Allow access if the user is an `admin` and the document sensitivity is `public`."  
   - "Deny access if the user is accessing from an untrusted IP address."  

#### **ABAC vs. RBAC: When to Use Each**

| **Aspect**       | **Role-Based Access Control (RBAC)**                                      | **Attribute-Based Access Control (ABAC)**                                             |
| ---------------- | ------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Access Model** | Permissions based on predefined roles.                                    | Permissions determined by evaluating attributes.                                      |
| **Flexibility**  | Limited; requires new roles for additional access rules.                  | Highly flexible; dynamic policies adapt to various scenarios.                         |
| **Scalability**  | Becomes complex with increasing roles and permissions.                    | Scales well for large systems with diverse access requirements.                       |
| **Use Cases**    | Suitable for applications with simple, role-based hierarchies.            | Ideal for applications requiring fine-grained, context-dependent access control.      |
| **Example**      | A document management system where roles like `admin` and `user` suffice. | A system where access depends on attributes like time, location, and resource status. |

#### **When to Use ABAC Over RBAC**  
ABAC is particularly useful in the following scenarios:  
- **Dynamic Environments**: Where access rules change based on context, such as location or device type.  
- **Complex Systems**: Applications with diverse users and resource types requiring highly specific access rules.  
- **Regulated Industries**: Environments needing compliance with strict security and audit requirements, such as healthcare or finance.  

By combining these components, ABAC enables developers to implement highly adaptable and secure access control systems. In the next sections, we’ll explore how ABAC can be implemented in an ExpressJs application.

### Overview of the Application Architecture  

This section provides a high-level overview of the technologies, use case, and data flow in our application.

#### **Tech Stack**  

- **Frontend**: We will not be implementing any frontend code in this tutorial. To test your endpoints I would recommend Postman or curl.  
- **Backend**: The server-side logic will be handled by **Express**. Express will evaluate attributes and enforce ABAC policies we will import and implement in `controllers/tasksController.js`.  
- **Database**: Any relational or NoSQL database (e.g., MySQL, MongoDB) can be used to store user, resource, and attribute data.  For the purpose of this tutorial, though, we'll use a mock JSON Server to serve as our databse.

#### **Use Case: Access Based on Attributes**  

Imagine an internal task and record management system used by employees across multiple departments. The app enforces access rules based on the following attributes:

### 1. **Admin**
- **Tasks**: Full access to create, read, and update tasks.
- **Records**: Full access to create, read, update, and delete any record.

### 2. **Moderator**
- **Tasks**: Full access to create, read, and update tasks. Can delete a task only if they are the task’s author.
- **Records**: Full access to create, read, and update records. Can delete a record only if they are the record's author.

### 3. **User**
- **Tasks**:
  - **Create**: Allowed.
  - **Read**: Can view tasks they authored or were invited to.
  - **Update**: Can update only tasks they authored.
  - **Delete**: Can delete a task only if they are the task's author.
- **Records**:
  - **Create**: Allowed.
  - **Read**: Can view records unless blocked by the record's owner.
  - **Update**: Can update records they authored or were invited to.
  - **Delete**: Can delete completed records they authored or were invited to.

Each role defines progressively stricter permissions, from admin (full control) to user (context-based and restricted).

#### **High-Level Data Flow**  

In a full-stack implementation, the interaction between the frontend and the Express backend would thus follow this flow:  

1. **Frontend Requests**:  
   - Users log in to the system, providing credentials.  
   - The frontend sends a request to the backend API to authenticate the user and retrieve user attributes such as roles and departments.  

2. **Backend Authentication**:  
   - The backend validates the credentials and generates a JWT (JSON Web Token) containing encoded user attributes.  
   - These attributes are returned to the frontend and stored securely (e.g., in memory or a secure HTTP-only cookie).  

3. **Frontend Access Control**:  
   - Frontend components use the user attributes in the JWT to dynamically display or hide UI elements, such as buttons or menu items.  
   - For example, a "Delete Task" button might only be visible to `admins`.  

4. **Backend Policy Enforcement**:  
   - When a user performs an action (e.g., creating a task), the frontend sends a request to the backend with relevant data.  
   - The backend uses middleware to extract attributes from the JWT, evaluate them against the defined ABAC policies, and grant or deny access.  

5. **Response to Frontend**:  
   - The backend responds to the frontend with the result of the action (success or error).  
   - If access is denied, the frontend can display an appropriate error message.

This architecture ensures that ABAC rules are enforced consistently across the application, providing both security and flexibility. In the next section, we’ll begin implementing the backend, where the core logic for ABAC resides.


### Backend Implementation: Express API

In this section, we will implement the **backend of the application** using Express, with ABAC logic to restrict access based on attributes like roles and metadata about users and resources. We'll use **json-server** as a mock database for simplicity.

---

#### Setting Up the Backend

To get started, initialize your project and install the necessary dependencies:

```bash
npm init -y
npm install express json-server cors
```

Update your project structure to match the following:

```bash
backend/
├── src/
│   ├── utils/
│   │   └── permissions.js      //ABAC logic and utility functions
│   ├── routes/
│   │   ├── task.js             //Routes for Task CRUD operations
│   ├── controllers/
│   │   ├── tasksController.js //Handles business logic for Tasks
│   ├── models/
│   │   ├── Task.js            // Database model for Tasks
│   ├── db.js                  // Mock database
│   ├── app.js                 //Main application logic
│   ├── server.js              //Entry point to start the server
└── package.json
```

---

#### 1. **Configuring `json-server` for the Mock Database**

Create a `db.json` file in the root of your project to simulate your database:

```json
{
  "tasks": [
    {"id": 1, "title": "Task 1", "authorId": 1, "department": "HR", "invitedUsers": []},
    {"id": 2, "title": "Task 2", "authorId": 2, "department": "Finance", "invitedUsers": [3]},
    {"id": 3, "title": "Task 3", "authorId": 3, "department": "HR", "invitedUsers": [1]}
  ],
  "records": [
    {"id": 1, "content": "Employee Record #1", "authorId": 1, "status": "completed", "invitedUsers": [3]},
    {"id": 2, "content": "Employee Record #2", "authorId": 3, "status": "in-progress", "invitedUsers": [1]},
    {"id": 3, "content": "Employee Record #3", "authorId": 2, "status": "completed", "invitedUsers": [3]}
  ],
  "users": [
    {"id": 1, "name": "Alice", "roles": ["admin"], "department": "HR", "blockedBy": []},
    {"id": 2, "name": "Bob", "roles": ["moderator"], "department": "Finance", "blockedBy": [3]},
    {"id": 3, "name": "Charlie", "roles": ["user"], "department": "HR", "blockedBy": []}
  ]
}
```

Run the mock database server:

```bash
npx json-server --watch db.json --port 3001
```

---

#### 2. **ABAC Implemetation (`utils/permissions.js`)**

Implement the ABAC logic to check permissions based on user and resource attributes.

```javascript
export const ROLES = {
  admin: {
    tasks: {
      create: true,
      read: true,
      update: true,
    },
    records: {
      create: true,
      read: true,
      update: true,
      delete: true,
    },
  },
  moderator: {
    tasks: {
      create: true,
      read: true,
      update: true,
      delete: (user, task) => task.authorId === user.id,
    },
    records: {
      create: true,
      read: true,
      update: true,
      delete: (user, record) => record.authorId === user.id,
    },
  },
  user: {
    tasks: {
      create: true,
      read: (user, task) =>
        task.authorId === user.id || task.invitedUsers.includes(user.id),
      update: (user, task) => task.authorId === user.id,
      delete: (user, record) => record.authorId === user.id,
    },
    records: {
      create: true,
      read: (user, record) => !user.blockedBy.includes(record.authorId),
      update: (user, record) =>
        record.authorId === user.id || record.invitedUsers.includes(user.id),
      delete: (user, record) =>
        (record.authorId === user.id ||
          record.invitedUsers.includes(user.id)) &&
        record.completed,
    },
  },
};

// Utility function to check permissions
export const hasPermission = (user, resource, action, data) => {
  return user.roles.some((role) => {
    const permissions = ROLES[role]?.[resource]?.[action];
    if (permissions == null) {
      return false;
    }

    // If permission is a boolean, return it directly
    if (typeof permissions === "boolean") {
      return permissions;
    }
    // If permission is a function, validate data and evaluate it
    return data != null && permissions(user, data);
  });
};


```

This file is the core of our ABAC system. Here's a detailed discussion:


### **Contents Breakdown**

1. **Roles and Permissions (`ROLES` Object)**:
   - The `ROLES` object maps user roles (`admin`, `moderator`, `user`) to resources (`tasks`, `records`) and actions (`create`, `read`, `update`, `delete`).
   - Permissions can be either:
     - **Boolean values** (`true` or `false`): Indicating unconditional access for that role.
     - **Functions**: Implementing dynamic, context-sensitive rules for access decisions based on attributes of the user, the resource, or both (ABAC).

   **Examples**:
   - `admin` has full access to all resources.
   - `moderator` can delete a task only if `task.authorId === user.id`.
   - `user` has conditional read/write access, e.g., they can read a task if they are the author or in the `invitedUsers` list.

2. **Utility Function (`hasPermission`)**:
   - The `hasPermission` function is a central mechanism to determine if a user has access to perform an action on a resource.
   - It evaluates the user's roles against the defined permissions, returning `true` or `false` based on the following:

---

### **Detailed Walkthrough of `hasPermission`**

1. **Roles Iteration**:
   - The function iterates over all the roles assigned to the user (`user.roles`).
   - For each role, it retrieves the associated permissions for the given `resource` and `action`:
     ```javascript
     const permissions = ROLES[role]?.[resource]?.[action];
     ```

2. **Null or Undefined Permissions**:
   - If permissions for the role, resource, or action are `null` or `undefined`, it returns `false` for that role, skipping further checks.

3. **Boolean Permissions**:
   - If permissions are a boolean (`true` or `false`), the value is returned directly:
     ```javascript
     if (typeof permissions === "boolean") {
       return permissions;
     }
     ```

4. **Function Permissions**:
   - If permissions are a function, it assumes an **ABAC rule** is defined.
   - The function is invoked with `user` and `data`:
     ```javascript
     return data != null && permissions(user, data);
     ```
   - If `data` is `null` or `undefined`, the function denies access by default.

5. **Short-Circuiting**:
   - If **any role** grants permission (`true`), the iteration stops early and the user is authorized for the action.

---

### **Key Strengths**

1. **Flexibility**:
   - Combines RBAC (static permissions via `true`/`false`) with ABAC (dynamic rules using functions).
   - Facilitates granular access control (e.g., task deletion based on `authorId`).

2. **Extensibility**:
   - Adding new roles, resources, or actions is straightforward by extending the `ROLES` object.
   - The `hasPermission` function can evaluate additional attributes or use external data sources (e.g., databases).

3. **Separation of Concerns**:
   - Keeps permission logic centralized, making controllers and middleware simpler.

4. **Multi-Role Support**:
   - Supports users with multiple roles, allowing any qualifying role to grant permission.

---

---

#### 3. **Controller Logic (`controllers/tasksController.js`)**

Implement controllers to handle business logic.

```javascript
import Task from "../models/Task.js"; // ORM model
import { hasPermission } from "../utils/permissions.js";

export const getTasks = async (req, res) => {
  try {
    const user = req.user;
    if (!user || !hasPermission(user, "tasks", "read")) {
      return res.status(403).json({ error: "Access denied" });
    }
    const tasks = await Task.findAll();
    res.json(tasks);
  } catch (err) {
    res
      .status(500)
      .json({ error: "Failed to fetch tasks", details: err.message });
  }
};

export const getTask = async (req, res) => {
  try {
    const { id } = req.params;
    const task = await Task.findById(id);
    const user = req.user;
    if (!user || !hasPermission(user, "tasks", "read", task)) {
      return res.status(403).json({ error: "Access denied" });
    }
    res.json(task);
  } catch (err) {
    res
      .status(500)
      .json({ error: "Failed to fetch tasks", details: err.message });
  }
};

export const createTask = async (req, res) => {
  try {
    const user = req.user;
    if (!user || !hasPermission(user, "tasks", "create")) {
      return res.status(403).json({ error: "Access denied" });
    }
    const newTask = await Task.create(req.body);
    res.status(201).json(newTask);
  } catch (err) {
    res
      .status(500)
      .json({ error: "Failed to create task", details: err.message });
  }
};

export const deleteTask = async (req, res) => {
  try {
    const { id } = req.params;
    const task = await Task.findById(id);
    if (!task) {
      return res.status(404).json({ message: "Task not found" });
    }
    const user = req.user;
    if (!user || !hasPermission(user, "tasks", "delete", task)) {
      return res.status(403).json({ error: "Access denied" });
    }
    const deleted = await Task.deleteOne(id);
    res
      .status(200)
      .json({ message: "Task deleted successfully", deletedData: deleted });
  } catch (error) {
    res
      .status(500)
      .json({ message: "Error deleting task", error: error.message });
  }
};


```

---

#### 4. **Routes (`routes/task.js`)**

Define the API routes for posts.

```javascript
import express from "express";
import {
  getTasks,
  getTask,
  createTask,
  deleteTask,
} from "../controllers/tasksController.js";

const router = express.Router();

router.post("/", createTask);
router.get("/", getTasks);
router.get("/:id", getTask);
router.delete("/:id", deleteTask);

export default router;

```

---

#### 5. **Application Entry Points**

**`app.js`:** Configure routes.

```javascript
import express from "express";
import cors from "cors";
import dotenv from "dotenv";
import morgan from "morgan";

import taskRoutes from "./src/routes/task.js";

const CLIENT_URL = process.env.CLIENT_URL || "http://localhost:3001";

dotenv.config();
const app = express();

app.use(cors({ origin: CLIENT_URL }));
app.use(morgan("combined"));
app.use(express.json());

// Mock user injection middleware
app.use((req, res, next) => {
  req.user = { id: 2, name: "Bob", roles: ["moderator"], department: "Finance" }; // Example user
  console.log("res:", res.data);
  next();
});

// Routes
app.use("/api/tasks", taskRoutes);

export default app;
```

---

### Conclusion

By implementing ABAC, we achieve a highly flexible and scalable access control mechanism, ideal for modern applications with complex permission requirements. This approach can adapt to a wide variety of scenarios and ensures robust security through the granular evaluation of user, resource, and contextual attributes.

ABAC transforms how permissions are managed in web applications, paving the way for secure, dynamic, and scalable solutions.

Feel free to extend this implementation to include additional attributes or integrate it with external identity providers for even more sophisticated access control capabilities. The flexibility of ABAC allows you to tailor it to fit the specific needs of your application, whether it’s managing permissions for diverse user groups or accommodating dynamic business rules.

To explore the full code implementation of this tutorial and experiment with the concepts discussed, visit the [GitHub repository](https://github.com/amir-canteetu/abac-express-tutorial). 


