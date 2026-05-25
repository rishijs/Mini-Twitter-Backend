# Spring Boot Social Media API (Twitter-like)

## Overview
This project implements a RESTful API using **Spring Boot**, **JPA**, and **PostgreSQL**. The API models a simplified social media platform similar to Twitter, supporting users, tweets, hashtags, follows, likes, replies, reposts, and feeds.

The system is built from scratch based on a defined endpoint specification and requires designing an appropriate database schema, service layer, and REST controllers.

---

## Tech Stack
- Java 17+
- Spring Boot
- Spring Web
- Spring Data JPA
- PostgreSQL
- Maven

---

## Project Structure (Recommended)
```
src/main/java/com/example/app
 ├── controller
 ├── service
 ├── repository
 ├── model
 │    ├── entity
 │    ├── embedded
 │    └── dto
 ├── util
 └── exception
```

---

## Domain Model

### Entities
You must implement the following entities:
- User
- Tweet
- Hashtag

### Embedded Classes
- Credentials (embedded in User)
- Profile (embedded in User)

> Note: The User table must use a custom table name because `user` is a reserved keyword in PostgreSQL.

---

## Data Model Overview

### User
Represents a platform user.
- username (unique)
- credentials
- profile
- joined timestamp (immutable)

### Profile
- firstName (optional)
- lastName (optional)
- email (required)
- phone (optional)

### Credentials
- username
- password

### Tweet
Represents a post.
- id
- author (User)
- posted timestamp (immutable)
- content (optional depending on type)
- inReplyTo (optional)
- repostOf (optional)

Tweet types:
- Simple tweet: content only
- Reply: content + inReplyTo
- Repost: repostOf only

### Hashtag
- label (unique, case-insensitive)
- firstUsed timestamp (immutable)
- lastUsed timestamp (updated on reuse)

---

## DTOs

### UserResponseDto
```json
{
  "username": "string",
  "profile": "ProfileDto",
  "joined": "timestamp"
}
```

### ProfileDto
```json
{
  "firstName": "string?",
  "lastName": "string?",
  "email": "string",
  "phone": "string?"
}
```

### CredentialsDto
```json
{
  "username": "string",
  "password": "string"
}
```

### HashtagDto
```json
{
  "label": "string",
  "firstUsed": "timestamp",
  "lastUsed": "timestamp"
}
```

### TweetResponseDto
```json
{
  "id": "integer",
  "author": "UserResponseDto",
  "posted": "timestamp",
  "content": "string?",
  "inReplyTo": "TweetResponseDto?",
  "repostOf": "TweetResponseDto?"
}
```

### ContextDto
```json
{
  "target": "TweetResponseDto",
  "before": ["TweetResponseDto"],
  "after": ["TweetResponseDto"]
}
```

---

## API Endpoints

### Validation
- GET `/validate/tag/exists/{label}` → boolean
- GET `/validate/username/exists/@{username}` → boolean
- GET `/validate/username/available/@{username}` → boolean

---

## Users

- GET `/users` → list users
- POST `/users` → create user
- GET `/users/@{username}` → get user
- PATCH `/users/@{username}` → update profile
- DELETE `/users/@{username}` → delete (soft delete)

### Follow System
- POST `/users/@{username}/follow`
- POST `/users/@{username}/unfollow`

### User Social Data
- GET `/users/@{username}/feed`
- GET `/users/@{username}/tweets`
- GET `/users/@{username}/mentions`
- GET `/users/@{username}/followers`
- GET `/users/@{username}/following`

---

## Tags

- GET `/tags`
- GET `/tags/{label}`

---

## Tweets

- GET `/tweets`
- POST `/tweets`
- GET `/tweets/{id}`
- DELETE `/tweets/{id}`

### Tweet Actions
- POST `/tweets/{id}/like`
- POST `/tweets/{id}/reply`
- POST `/tweets/{id}/repost`

### Tweet Metadata
- GET `/tweets/{id}/tags`
- GET `/tweets/{id}/likes`
- GET `/tweets/{id}/context`
- GET `/tweets/{id}/replies`
- GET `/tweets/{id}/reposts`
- GET `/tweets/{id}/mentions`

---

## Business Rules

### Soft Deletion
- Users and tweets are never physically removed
- Deleted entities must remain in DB but be marked inactive
- Relationships must remain intact

### Mentions and Hashtags
- Parsed automatically from tweet content
- Format:
  - Mentions: `@username`
  - Hashtags: `#tag`

---

## Feed Logic
User feed includes:
- User’s own tweets
- Tweets from followed users
- Ordered reverse-chronologically

---

## Context Rules
- `before`: chain leading to target tweet
- `after`: all descendant replies (flattened, chronological)
- Deleted tweets excluded but do not break chains

---

## Testing

This project uses Postman/Newman tests.

### Install Newman
```bash
npm install -g newman
```

### Run Tests
```bash
newman run "Assessment 1 Test Suite.postman_collection.json" -e "Assessment 1.postman_environment.json"
```

Expected result: **330 passing assertions**

---

## Notes
- Ensure consistent REST conventions
- Ensure proper validation on all endpoints
- Ensure DTO separation from entities
- Ensure transactional integrity for follow/like/repost actions
- Ensure hashtag updates are handled automatically during tweet creation

---

