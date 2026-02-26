# 📐 ScribeBoard API - Architecture Guide

A comprehensive guide explaining the concepts, layers, and architecture of the Blog CMS Backend.

---

## 📚 Table of Contents

1. [Project Overview](#project-overview)
2. [System Architecture](#system-architecture)
3. [Layered Architecture](#layered-architecture)
4. [Data Flow](#data-flow)
5. [Database Design](#database-design)
6. [Authentication Flow](#authentication-flow)
7. [Publishing Workflow](#publishing-workflow)
8. [Design Patterns](#design-patterns)
9. [Real-World Analogies](#real-world-analogies)

---

## 🎯 Project Overview

### What is ScribeBoard?

ScribeBoard is a **Content Management System (CMS) Backend** designed for blogging platforms. Think of it as the "engine" that powers sites like Medium, Dev.to, or WordPress.

### Core Concept

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SCRIBEBOARD CMS                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   "A platform where AUTHORS write content,                                  │
│    EDITORS review and manage it,                                            │
│    ADMINS control the system,                                               │
│    and READERS consume it."                                                 │
│                                                                              │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐ │
│   │   AUTHOR    │    │   EDITOR    │    │    ADMIN    │    │   READER    │ │
│   │             │    │             │    │             │    │             │ │
│   │  ✍️ Write   │    │  📝 Review  │    │  ⚙️ Manage  │    │  👁️ Read    │ │
│   │  📄 Draft   │    │  ✅ Approve │    │  👥 Users   │    │  💬 Comment │ │
│   │  📤 Submit  │    │  📊 Moderate│    │  🔒 Security│    │  ❤️ Like    │ │
│   └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Problems It Solves

| Problem | Solution |
|---------|----------|
| Multiple authors need to create content | Role-based user system |
| Content needs review before publishing | Publishing workflow (Draft → Review → Publish) |
| Comments can contain spam | Comment moderation system |
| Content needs organization | Categories and Tags |
| Users need to find content | Search, filter, and pagination |
| Track content changes | Revision history |

---

## 🏗️ System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              CLIENT LAYER                                    │
│        (Web App / Mobile App / Third-Party Integrations)                    │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     │ HTTP/HTTPS
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           API GATEWAY LAYER                                  │
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐           │
│  │   Helmet    │ │    CORS     │ │   Morgan    │ │   Express   │           │
│  │  (Security) │ │  (Access)   │ │  (Logging)  │ │  (Routing)  │           │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘           │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          MIDDLEWARE LAYER                                    │
│  ┌───────────────────┐ ┌───────────────────┐ ┌───────────────────┐         │
│  │   Authentication  │ │    Validation     │ │  Error Handling   │         │
│  │   (JWT Verify)    │ │   (Joi Schema)    │ │   (Catch Errors)  │         │
│  └───────────────────┘ └───────────────────┘ └───────────────────┘         │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            ROUTE LAYER                                       │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐          │
│  │   Auth   │ │   Post   │ │ Category │ │ Comment  │ │   User   │          │
│  │  Routes  │ │  Routes  │ │  Routes  │ │  Routes  │ │  Routes  │          │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘ └──────────┘          │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CONTROLLER LAYER                                    │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  • Extract data from request (req.body, req.params, req.query)      │   │
│  │  • Call appropriate service method                                   │   │
│  │  • Format and send response                                          │   │
│  │  • Handle errors and pass to error middleware                        │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                           SERVICE LAYER                                      │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │  BUSINESS LOGIC LIVES HERE                                           │   │
│  │  • AuthService: Login, Register, Token Management                    │   │
│  │  • PostService: CRUD, Publishing Workflow, Revisions                 │   │
│  │  • CategoryService: CRUD with slug generation                        │   │
│  │  • CommentService: CRUD, Moderation, Nested Replies                  │   │
│  │  • TokenService: JWT generation and verification                     │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                            DATA LAYER                                        │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                         PRISMA ORM                                   │   │
│  │  • Type-safe database queries                                        │   │
│  │  • Automatic migrations                                              │   │
│  │  • Relation management                                               │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│  ┌─────────────────────────────────────────────────────────────────────┐   │
│  │                        PostgreSQL                                    │   │
│  │  Tables: users, posts, categories, tags, comments, tokens            │   │
│  └─────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🧅 Layered Architecture

### Why Layers?

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    WHY SEPARATE INTO LAYERS?                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. SEPARATION OF CONCERNS                                                 │
│      Each layer has ONE job and does it well                                │
│                                                                              │
│   2. MAINTAINABILITY                                                        │
│      Change database? Only modify data layer                                │
│      Change validation? Only modify middleware                              │
│                                                                              │
│   3. TESTABILITY                                                            │
│      Test services without HTTP requests                                    │
│      Test controllers by mocking services                                   │
│                                                                              │
│   4. REUSABILITY                                                            │
│      Use the same service from different routes                             │
│      Share middleware across multiple endpoints                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Layer Responsibilities

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         LAYER RESPONSIBILITIES                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ROUTES (routes/*.js)                                                      │
│   ├── Define API endpoints (GET, POST, PUT, DELETE)                         │
│   ├── Apply middleware to routes                                            │
│   └── Connect URLs to controllers                                           │
│   │                                                                          │
│   │   Example:                                                              │
│   │   router.post('/posts', authenticate, validate, controller.create)     │
│   │                                                                          │
│   ▼                                                                          │
│   CONTROLLERS (controllers/*.js)                                            │
│   ├── Extract request data                                                  │
│   ├── Call services                                                         │
│   └── Send responses                                                        │
│   │                                                                          │
│   │   Example:                                                              │
│   │   const post = await postService.createPost(req.user.id, req.body);    │
│   │   sendCreated(res, post, 'Post created');                              │
│   │                                                                          │
│   ▼                                                                          │
│   SERVICES (services/*.js)                                                  │
│   ├── Business logic                                                        │
│   ├── Data validation                                                       │
│   ├── Database operations                                                   │
│   └── Error handling                                                        │
│   │                                                                          │
│   │   Example:                                                              │
│   │   const slug = await generateUniqueSlug(title, 'post', prisma);        │
│   │   const post = await prisma.post.create({ data: {...} });              │
│   │                                                                          │
│   ▼                                                                          │
│   DATA ACCESS (Prisma + config/database.js)                                 │
│   ├── Database connections                                                  │
│   ├── Query execution                                                       │
│   └── Transaction management                                                │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Code Flow Example

```
┌─────────────────────────────────────────────────────────────────────────────┐
│              CREATE POST - COMPLETE CODE FLOW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. CLIENT REQUEST                                                         │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  POST /api/v1/posts                                                 │   │
│   │  Headers: Authorization: Bearer <token>                             │   │
│   │  Body: { "title": "My Post", "content": "..." }                    │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   2. ROUTE (post.routes.js)                                                 │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  router.post('/',                                                   │   │
│   │    authenticate,        // Check JWT token                          │   │
│   │    authorOrAbove,       // Check role >= AUTHOR                     │   │
│   │    validate(schema),    // Validate request body                    │   │
│   │    postController.createPost                                        │   │
│   │  );                                                                 │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   3. MIDDLEWARE (auth.middleware.js)                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  // authenticate                                                    │   │
│   │  const token = req.headers.authorization.split(' ')[1];            │   │
│   │  const decoded = tokenService.verifyAccessToken(token);            │   │
│   │  req.user = await prisma.user.findUnique({ id: decoded.userId });  │   │
│   │  next();                                                            │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   4. CONTROLLER (post.controller.js)                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  const createPost = async (req, res, next) => {                    │   │
│   │    try {                                                            │   │
│   │      const post = await postService.createPost(                    │   │
│   │        req.user.id,    // From auth middleware                     │   │
│   │        req.body        // Validated by validate middleware         │   │
│   │      );                                                             │   │
│   │      sendCreated(res, post, 'Post created successfully');          │   │
│   │    } catch (error) {                                                │   │
│   │      next(error);      // Pass to error handler                    │   │
│   │    }                                                                │   │
│   │  };                                                                 │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   5. SERVICE (post.service.js)                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  const createPost = async (authorId, data) => {                    │   │
│   │    // Generate URL-friendly slug                                    │   │
│   │    const slug = await generateUniqueSlug(data.title, 'post');      │   │
│   │                                                                     │   │
│   │    // Calculate reading time                                        │   │
│   │    const readingTime = calculateReadingTime(data.content);         │   │
│   │                                                                     │   │
│   │    // Create post in database                                       │   │
│   │    const post = await prisma.post.create({                         │   │
│   │      data: {                                                        │   │
│   │        ...data,                                                     │   │
│   │        slug,                                                        │   │
│   │        readingTime,                                                 │   │
│   │        authorId,                                                    │   │
│   │        status: 'DRAFT'                                              │   │
│   │      }                                                              │   │
│   │    });                                                              │   │
│   │                                                                     │   │
│   │    return post;                                                     │   │
│   │  };                                                                 │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   6. RESPONSE                                                               │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  {                                                                  │   │
│   │    "success": true,                                                 │   │
│   │    "message": "Post created successfully",                          │   │
│   │    "data": {                                                        │   │
│   │      "id": "abc123",                                                │   │
│   │      "title": "My Post",                                            │   │
│   │      "slug": "my-post",                                             │   │
│   │      "status": "DRAFT",                                             │   │
│   │      ...                                                            │   │
│   │    }                                                                │   │
│   │  }                                                                  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Data Flow

### Request-Response Cycle

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                     REQUEST-RESPONSE CYCLE                                   │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   CLIENT                                                                    │
│     │                                                                        │
│     │  POST /api/v1/posts                                                   │
│     │  { title: "Hello", content: "World" }                                 │
│     ▼                                                                        │
│   ┌─────────────────────────────────────────────┐                           │
│   │              EXPRESS APP                     │                           │
│   │  ┌─────────────────────────────────────────┐│                           │
│   │  │         MIDDLEWARE CHAIN                ││                           │
│   │  │                                         ││                           │
│   │  │  helmet() ──▶ cors() ──▶ express.json() ││                           │
│   │  │      │                                  ││                           │
│   │  │      ▼                                  ││                           │
│   │  │  morgan() ──▶ authenticate() ──▶ validate()│                         │
│   │  │                                         ││                           │
│   │  └──────────────────┬──────────────────────┘│                           │
│   │                     │                       │                           │
│   │                     ▼                       │                           │
│   │  ┌─────────────────────────────────────────┐│                           │
│   │  │           ROUTE HANDLER                 ││                           │
│   │  │                                         ││                           │
│   │  │  postController.createPost(req, res)    ││                           │
│   │  │           │                             ││                           │
│   │  │           ▼                             ││                           │
│   │  │  postService.createPost(userId, data)   ││                           │
│   │  │           │                             ││                           │
│   │  │           ▼                             ││                           │
│   │  │  prisma.post.create({ data })           ││                           │
│   │  │           │                             ││                           │
│   │  │           ▼                             ││                           │
│   │  │  res.json({ success: true, data })      ││                           │
│   │  │                                         ││                           │
│   │  └─────────────────────────────────────────┘│                           │
│   └─────────────────────────────────────────────┘                           │
│     │                                                                        │
│     ▼                                                                        │
│   CLIENT receives response                                                  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Error Flow

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           ERROR HANDLING FLOW                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   REQUEST with invalid data                                                 │
│     │                                                                        │
│     ▼                                                                        │
│   ┌──────────────────────┐                                                  │
│   │  validate middleware │                                                  │
│   │  schema.validate()   │                                                  │
│   └──────────┬───────────┘                                                  │
│              │                                                               │
│              │  Validation fails                                            │
│              ▼                                                               │
│   ┌──────────────────────┐                                                  │
│   │  next(error)         │  ──────────────────────────────────┐             │
│   │  Pass to next        │                                    │             │
│   └──────────────────────┘                                    │             │
│                                                               │             │
│   REQUEST with valid data                                     │             │
│     │                                                         │             │
│     ▼                                                         │             │
│   ┌──────────────────────┐                                    │             │
│   │  controller          │                                    │             │
│   │  service.method()    │                                    │             │
│   └──────────┬───────────┘                                    │             │
│              │                                                │             │
│              │  Database error / Business rule violation      │             │
│              ▼                                                │             │
│   ┌──────────────────────┐                                    │             │
│   │  try { ... }         │                                    │             │
│   │  catch (error) {     │                                    │             │
│   │    next(error);  ─────────────────────────────────────────┤             │
│   │  }                   │                                    │             │
│   └──────────────────────┘                                    │             │
│                                                               │             │
│                                                               ▼             │
│                                              ┌──────────────────────────┐   │
│                                              │    ERROR MIDDLEWARE      │   │
│                                              │                          │   │
│                                              │  errorHandler(err,       │   │
│                                              │    req, res, next)       │   │
│                                              │                          │   │
│                                              │  • Log error             │   │
│                                              │  • Format response       │   │
│                                              │  • Send to client        │   │
│                                              └──────────┬───────────────┘   │
│                                                         │                   │
│                                                         ▼                   │
│                                              ┌──────────────────────────┐   │
│                                              │  {                       │   │
│                                              │    "success": false,     │   │
│                                              │    "message": "Error"    │   │
│                                              │  }                       │   │
│                                              └──────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🗄️ Database Design

### Entity Relationship Diagram

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ENTITY RELATIONSHIP DIAGRAM                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ┌──────────────┐                           ┌──────────────┐               │
│   │     USER     │                           │   CATEGORY   │               │
│   ├──────────────┤                           ├──────────────┤               │
│   │ id           │                           │ id           │               │
│   │ email        │                           │ name         │               │
│   │ password     │                           │ slug         │               │
│   │ firstName    │                           │ description  │               │
│   │ lastName     │                           │ color        │               │
│   │ role         │                           └───────┬──────┘               │
│   │ bio          │                                   │                      │
│   └───────┬──────┘                                   │ 1                    │
│           │                                          │                      │
│           │ 1                                        │                      │
│           │                                          │                      │
│           │         ┌────────────────────────────────┤                      │
│           │         │                                │                      │
│           │         │ *                              │ *                    │
│           ▼         ▼                                ▼                      │
│   ┌───────────────────────────────────────────────────────┐                │
│   │                        POST                            │                │
│   ├───────────────────────────────────────────────────────┤                │
│   │ id           │ title        │ slug          │ content │                │
│   │ status       │ publishedAt  │ authorId      │ categoryId              │
│   │ excerpt      │ coverImage   │ viewCount     │ readingTime             │
│   │ isFeatured   │ isPinned     │ allowComments │                         │
│   └───────────────────────────────────────────────────────┘                │
│           │                     │                                           │
│           │ 1                   │ 1                                         │
│           │                     │                                           │
│           │                     │                                           │
│           │ *                   │ *                                         │
│           ▼                     ▼                                           │
│   ┌──────────────┐      ┌──────────────┐                                   │
│   │   COMMENT    │      │   POST_TAG   │◀─────┐                            │
│   ├──────────────┤      ├──────────────┤      │                            │
│   │ id           │      │ postId       │      │ *                          │
│   │ content      │      │ tagId        │      │                            │
│   │ status       │      └──────────────┘      │                            │
│   │ postId       │             │              │                            │
│   │ authorId     │             │ *            │                            │
│   │ parentId     │◀────────────┼──────────────┤                            │
│   │ (self-ref)   │             ▼              │                            │
│   └──────────────┘      ┌──────────────┐      │                            │
│                         │     TAG      │──────┘                            │
│                         ├──────────────┤                                   │
│                         │ id           │                                   │
│                         │ name         │                                   │
│                         │ slug         │                                   │
│                         └──────────────┘                                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Relationship Types

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       DATABASE RELATIONSHIPS                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ONE-TO-MANY (1:N)                                                         │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                                                                     │   │
│   │   USER (1) ──────────────▶ POSTS (N)                               │   │
│   │   "One user can write many posts"                                   │   │
│   │                                                                     │   │
│   │   CATEGORY (1) ──────────▶ POSTS (N)                               │   │
│   │   "One category can have many posts"                                │   │
│   │                                                                     │   │
│   │   POST (1) ──────────────▶ COMMENTS (N)                            │   │
│   │   "One post can have many comments"                                 │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   MANY-TO-MANY (N:N)                                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                                                                     │   │
│   │   POST (N) ◀────POST_TAG────▶ TAG (N)                              │   │
│   │   "Posts can have many tags, tags can be on many posts"             │   │
│   │                                                                     │   │
│   │   Junction table: post_tags (postId, tagId)                        │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   SELF-REFERENTIAL (Nested Comments)                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                                                                     │   │
│   │   COMMENT ◀──────────────── COMMENT (replies)                      │   │
│   │   "A comment can have replies (child comments)"                     │   │
│   │                                                                     │   │
│   │   parentId references id in same table                             │   │
│   │                                                                     │   │
│   │   Comment 1 (parentId: null)                                        │   │
│   │     └── Comment 2 (parentId: 1)                                    │   │
│   │           └── Comment 3 (parentId: 2)                              │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🔐 Authentication Flow

### JWT Authentication

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      JWT AUTHENTICATION FLOW                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. LOGIN REQUEST                                                          │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  POST /api/v1/auth/login                                            │   │
│   │  { "email": "author@test.com", "password": "Admin@123" }           │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   2. SERVER VALIDATES                                                       │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  • Find user by email                                               │   │
│   │  • Compare password with bcrypt                                     │   │
│   │  • Check if account is active                                       │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   3. GENERATE TOKENS                                                        │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  Access Token (15 min)                                              │   │
│   │  ┌───────────────────────────────────────────────────────────────┐ │   │
│   │  │ { userId, email, role }  +  SECRET  →  JWT                    │ │   │
│   │  └───────────────────────────────────────────────────────────────┘ │   │
│   │                                                                     │   │
│   │  Refresh Token (7 days)                                             │   │
│   │  ┌───────────────────────────────────────────────────────────────┐ │   │
│   │  │ { userId, type: 'refresh' }  +  SECRET  →  JWT                │ │   │
│   │  │ Stored in database (tokens table)                             │ │   │
│   │  └───────────────────────────────────────────────────────────────┘ │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   4. RESPONSE                                                               │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  {                                                                  │   │
│   │    "accessToken": "eyJhbGciOiJIUzI1NiIs...",                       │   │
│   │    "refreshToken": "eyJhbGciOiJIUzI1NiIs...",                      │   │
│   │    "user": { "id": "...", "email": "...", "role": "AUTHOR" }       │   │
│   │  }                                                                  │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   5. SUBSEQUENT REQUESTS                                                    │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  GET /api/v1/posts                                                  │   │
│   │  Headers: Authorization: Bearer eyJhbGciOiJIUzI1NiIs...            │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                     │                                       │
│                                     ▼                                       │
│   6. TOKEN VERIFICATION (auth.middleware.js)                                │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │  • Extract token from header                                        │   │
│   │  • Verify with jwt.verify(token, SECRET)                           │   │
│   │  • Decode payload { userId, email, role }                          │   │
│   │  • Attach user to req.user                                         │   │
│   │  • Call next()                                                      │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Role-Based Access Control

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    ROLE-BASED ACCESS CONTROL (RBAC)                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   ROLE HIERARCHY                                                            │
│                                                                              │
│   ┌───────────┐                                                             │
│   │   ADMIN   │  ──▶  Full system access                                   │
│   └─────┬─────┘                                                             │
│         │                                                                    │
│         ▼                                                                    │
│   ┌───────────┐                                                             │
│   │   EDITOR  │  ──▶  Manage all content, moderate comments                │
│   └─────┬─────┘                                                             │
│         │                                                                    │
│         ▼                                                                    │
│   ┌───────────┐                                                             │
│   │   AUTHOR  │  ──▶  Create/edit own content                              │
│   └─────┬─────┘                                                             │
│         │                                                                    │
│         ▼                                                                    │
│   ┌───────────┐                                                             │
│   │   READER  │  ──▶  View content, comment                                │
│   └───────────┘                                                             │
│                                                                              │
│   PERMISSION MATRIX                                                         │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │ Action              │ READER │ AUTHOR │ EDITOR │ ADMIN │             │  │
│   │─────────────────────┼────────┼────────┼────────┼───────┤             │  │
│   │ Read published      │   ✅   │   ✅   │   ✅   │  ✅   │             │  │
│   │ Comment on posts    │   ✅   │   ✅   │   ✅   │  ✅   │             │  │
│   │ Create posts        │   ❌   │   ✅   │   ✅   │  ✅   │             │  │
│   │ Edit own posts      │   ❌   │   ✅   │   ✅   │  ✅   │             │  │
│   │ Edit any post       │   ❌   │   ❌   │   ✅   │  ✅   │             │  │
│   │ Publish posts       │   ❌   │   ✅   │   ✅   │  ✅   │             │  │
│   │ Moderate comments   │   ❌   │   ❌   │   ✅   │  ✅   │             │  │
│   │ Manage categories   │   ❌   │   ❌   │   ✅   │  ✅   │             │  │
│   │ Manage users        │   ❌   │   ❌   │   ❌   │  ✅   │             │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   MIDDLEWARE IMPLEMENTATION                                                 │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  // Require specific roles                                           │  │
│   │  const requireRole = (...roles) => {                                │  │
│   │    return (req, res, next) => {                                     │  │
│   │      if (!roles.includes(req.user.role)) {                          │  │
│   │        return next(AppError.forbidden('Access denied'));            │  │
│   │      }                                                               │  │
│   │      next();                                                         │  │
│   │    };                                                                │  │
│   │  };                                                                  │  │
│   │                                                                      │  │
│   │  // Usage in routes                                                  │  │
│   │  router.post('/categories', authenticate, editorOrAdmin, ...)       │  │
│   │  router.post('/posts', authenticate, authorOrAbove, ...)            │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📤 Publishing Workflow

### State Machine

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       PUBLISHING STATE MACHINE                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                              ┌─────────────┐                                │
│                              │   CREATE    │                                │
│                              └──────┬──────┘                                │
│                                     │                                       │
│                                     ▼                                       │
│                              ┌─────────────┐                                │
│              ┌───────────────│    DRAFT    │───────────────┐                │
│              │               └──────┬──────┘               │                │
│              │                      │                      │                │
│              │ edit                 │ submitForReview      │ publish        │
│              │                      ▼                      │ (by EDITOR+)   │
│              │               ┌─────────────────┐           │                │
│              │               │ PENDING_REVIEW  │           │                │
│              │               └────────┬────────┘           │                │
│              │                        │                    │                │
│              │        ┌───────────────┼───────────────┐    │                │
│              │        │ reject        │ approve       │    │                │
│              │        ▼               ▼               │    │                │
│              │  ┌─────────────┐ ┌─────────────┐      │    │                │
│              └──│    DRAFT    │ │  PUBLISHED  │◀─────┴────┘                │
│                 └─────────────┘ └──────┬──────┘                             │
│                                        │                                    │
│                                        │ unpublish                          │
│                                        ▼                                    │
│                                 ┌─────────────┐                             │
│                                 │ UNPUBLISHED │                             │
│                                 └──────┬──────┘                             │
│                                        │                                    │
│                         ┌──────────────┼──────────────┐                     │
│                         │ republish    │ archive      │                     │
│                         ▼              ▼              │                     │
│                  ┌─────────────┐ ┌─────────────┐      │                     │
│                  │  PUBLISHED  │ │  ARCHIVED   │      │                     │
│                  └─────────────┘ └─────────────┘      │                     │
│                                                                              │
│                                                                              │
│   STATE DESCRIPTIONS                                                        │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │ DRAFT          │ Work in progress. Not visible to public.            │  │
│   │ PENDING_REVIEW │ Submitted for editor review.                        │  │
│   │ PUBLISHED      │ Live and visible to everyone.                       │  │
│   │ UNPUBLISHED    │ Removed from public. Can be republished.            │  │
│   │ ARCHIVED       │ Long-term storage. Rarely accessed.                 │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🎨 Design Patterns

### Patterns Used in ScribeBoard

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         DESIGN PATTERNS USED                                 │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   1. REPOSITORY PATTERN (via Prisma)                                        │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  Services don't know about SQL/database specifics                    │  │
│   │  They use Prisma's abstraction layer                                 │  │
│   │                                                                      │  │
│   │  // Service doesn't write SQL                                        │  │
│   │  const posts = await prisma.post.findMany({                         │  │
│   │    where: { status: 'PUBLISHED' }                                   │  │
│   │  });                                                                 │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   2. MIDDLEWARE PATTERN                                                     │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  Chain of handlers that process request sequentially                 │  │
│   │                                                                      │  │
│   │  request ──▶ helmet ──▶ cors ──▶ auth ──▶ validate ──▶ controller   │  │
│   │                                                                      │  │
│   │  Each middleware can:                                                │  │
│   │  • Modify request (req.user = decoded)                              │  │
│   │  • End request (res.json())                                         │  │
│   │  • Pass to next (next())                                            │  │
│   │  • Pass error (next(error))                                         │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   3. SERVICE LAYER PATTERN                                                  │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  Business logic isolated from HTTP concerns                          │  │
│   │                                                                      │  │
│   │  Controller: Handles HTTP, calls service                            │  │
│   │  Service: Contains business rules, calls database                   │  │
│   │                                                                      │  │
│   │  Benefits:                                                           │  │
│   │  • Services can be tested without HTTP                              │  │
│   │  • Logic reusable across different routes                           │  │
│   │  • Clear separation of concerns                                     │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   4. FACTORY PATTERN (AppError)                                             │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  Create different error types with static methods                    │  │
│   │                                                                      │  │
│   │  throw AppError.notFound('Post not found');    // 404               │  │
│   │  throw AppError.forbidden('Access denied');    // 403               │  │
│   │  throw AppError.badRequest('Invalid input');   // 400               │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
│   5. SINGLETON PATTERN (Prisma Client)                                      │
│   ┌──────────────────────────────────────────────────────────────────────┐  │
│   │  Single database connection shared across application                │  │
│   │                                                                      │  │
│   │  // config/database.js                                               │  │
│   │  const prisma = new PrismaClient(); // Created once                 │  │
│   │  module.exports = { prisma };       // Exported everywhere          │  │
│   └──────────────────────────────────────────────────────────────────────┘  │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 🌍 Real-World Analogies

### The Restaurant Analogy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                       RESTAURANT ANALOGY                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SCRIBEBOARD                          RESTAURANT                           │
│   ─────────────                        ──────────                           │
│                                                                              │
│   Client App         ◀───────────────▶  Customer                           │
│   API Routes         ◀───────────────▶  Menu / Waiter                      │
│   Controllers        ◀───────────────▶  Waiter (takes order, serves)       │
│   Services           ◀───────────────▶  Kitchen (cooks the food)           │
│   Database           ◀───────────────▶  Pantry (stores ingredients)        │
│   Middleware         ◀───────────────▶  Security / Host (checks reservations)│
│                                                                              │
│   FLOW:                                                                     │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                                                                     │   │
│   │   Customer         Waiter           Kitchen          Pantry         │   │
│   │      │               │                │                │            │   │
│   │      │  "I want     │                │                │            │   │
│   │      │   pizza"     │                │                │            │   │
│   │      │─────────────▶│                │                │            │   │
│   │      │              │  "Make pizza"  │                │            │   │
│   │      │              │───────────────▶│                │            │   │
│   │      │              │                │  "Get flour,   │            │   │
│   │      │              │                │   cheese"      │            │   │
│   │      │              │                │───────────────▶│            │   │
│   │      │              │                │◀───────────────│            │   │
│   │      │              │                │  (ingredients) │            │   │
│   │      │              │◀───────────────│                │            │   │
│   │      │              │   (pizza)      │                │            │   │
│   │      │◀─────────────│                │                │            │   │
│   │      │   (pizza)    │                │                │            │   │
│   │                                                                     │   │
│   │   The waiter doesn't cook. The kitchen doesn't serve.              │   │
│   │   Each has a specific job!                                         │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Publishing House Analogy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    PUBLISHING HOUSE ANALOGY                                  │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   SCRIBEBOARD ROLES                    PUBLISHING HOUSE                     │
│   ─────────────────                    ────────────────                     │
│                                                                              │
│   AUTHOR             ◀───────────────▶  Writer                             │
│   EDITOR             ◀───────────────▶  Editor / Proofreader               │
│   ADMIN              ◀───────────────▶  Publisher / CEO                    │
│   READER             ◀───────────────▶  Book Reader                        │
│                                                                              │
│   PUBLISHING WORKFLOW:                                                      │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                                                                     │   │
│   │   ┌───────────────┐      ┌───────────────┐     ┌───────────────┐   │   │
│   │   │    AUTHOR     │      │    EDITOR     │     │    READERS    │   │   │
│   │   │               │      │               │     │               │   │   │
│   │   │  ✍️ Writes    │      │  📝 Reviews   │     │  📖 Reads     │   │   │
│   │   │   manuscript  │      │   edits       │     │   published   │   │   │
│   │   │               │      │   approves    │     │   book        │   │   │
│   │   └───────┬───────┘      └───────┬───────┘     └───────────────┘   │   │
│   │           │                      │                      ▲          │   │
│   │           │  Submit              │  Approve             │          │   │
│   │           ▼                      ▼                      │          │   │
│   │   ┌─────────────────────────────────────────────────────┤          │   │
│   │   │                   PUBLISHER                         │          │   │
│   │   │                                                     │          │   │
│   │   │    Manuscript ──▶ Review ──▶ Print ──▶ Distribute ──┘          │   │
│   │   │       (DRAFT)    (REVIEW)  (PUBLISH)                           │   │
│   │   │                                                                │   │
│   │   └────────────────────────────────────────────────────────────────┘   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### The Comment Moderation Analogy

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    SECURITY CHECKPOINT ANALOGY                               │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│   Comment System = Event Venue with Security                                │
│                                                                              │
│   ┌─────────────────────────────────────────────────────────────────────┐   │
│   │                                                                     │   │
│   │   Visitor                  Security Check              Venue        │   │
│   │   (Comment)                (Moderation)                (Post)       │   │
│   │      │                          │                         │         │   │
│   │      │  "I want to enter"       │                         │         │   │
│   │      │─────────────────────────▶│                         │         │   │
│   │      │                          │                         │         │   │
│   │      │                    ┌─────┴─────┐                   │         │   │
│   │      │                    │  CHECK:   │                   │         │   │
│   │      │                    │ • Valid?  │                   │         │   │
│   │      │                    │ • Spam?   │                   │         │   │
│   │      │                    │ • Threat? │                   │         │   │
│   │      │                    └─────┬─────┘                   │         │   │
│   │      │                          │                         │         │   │
│   │      │                    ┌─────┴─────┐                   │         │   │
│   │      │                    │           │                   │         │   │
│   │      │              APPROVED      REJECTED                │         │   │
│   │      │                 │              │                   │         │   │
│   │      │                 ▼              ▼                   │         │   │
│   │      │          ┌──────────┐   ┌──────────┐              │         │   │
│   │      │          │ Enter ✅  │   │ Denied ❌ │              │         │   │
│   │      │          │ (Shows   │   │ (Hidden) │              │         │   │
│   │      │          │  on post)│   │          │              │         │   │
│   │      │          └──────────┘   └──────────┘              │         │   │
│   │                                                                     │   │
│   └─────────────────────────────────────────────────────────────────────┘   │
│                                                                              │
│   COMMENT STATES:                                                           │
│   • PENDING  = Waiting in line for security check                          │
│   • APPROVED = Allowed to enter and mingle                                 │
│   • REJECTED = Turned away at the door                                     │
│   • SPAM     = Banned from the venue                                       │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## 📝 Summary

### Key Architectural Decisions

| Decision | Reason |
|----------|--------|
| **Layered Architecture** | Separation of concerns, maintainability |
| **Service Layer** | Reusable business logic, testability |
| **JWT Authentication** | Stateless, scalable auth |
| **Prisma ORM** | Type-safe queries, easy migrations |
| **Role-Based Access** | Flexible permission management |
| **State Machine for Posts** | Clear publishing workflow |
| **Nested Comments** | Rich discussion threads |

### Architecture Benefits

1. **Scalability** - Each layer can be scaled independently
2. **Maintainability** - Changes in one layer don't affect others
3. **Testability** - Services can be unit tested without HTTP
4. **Flexibility** - Easy to swap database or add new features
5. **Security** - Middleware chain enforces authentication/authorization

---

*This architecture guide serves as a reference for understanding how ScribeBoard is built and why certain design decisions were made.*
