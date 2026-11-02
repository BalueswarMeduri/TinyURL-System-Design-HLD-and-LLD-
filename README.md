# TinyURL – System Design (HLD & LLD)

A system design implementation of **TinyURL**, a URL shortening service that converts long URLs into short, easy-to-share links and redirects users to the original URLs.

## Table of Contents

1. [Functional Requirements](#1-functional-requirements)
2. [Non-Functional Requirements](#2-non-functional-requirements)
3. [High-Level Design (HLD)](#3-high-level-design-hld)
4. [Conditional Write](#4-conditional-write)
5. [Atomic Transaction](#5-atomic-transaction)
6. [Database Design](#6-database-design)

---

## 1. Functional Requirements

These describe what our TinyURL system should do.

- **Generate Short URL:** Users should be able to generate a short URL from a long URL.
- **URL Redirection:** When users click a short URL, they should be redirected to the correct original URL.
- **Consistent URL Mapping:** The same long URL should always return the same short URL.
- **Custom Short URL:** Users should be able to create a custom short URL.
- **URL Expiration:** Users should be able to set an expiration time for their short URLs.
- **Delete Short URL:** Users should be able to delete their previously generated short URLs.

## 2. Non-Functional Requirements

These describe how well our system should work.

| Requirement | Description |
|---|---|
| **Low Latency** | Short URL redirection should happen very quickly. |
| **High Availability** | Our service should be available most of the time, with minimal downtime. |
| **Scalability** | Our system should handle millions of URLs and users as traffic increases. |

## 3. High-Level Design (HLD)

The following diagram represents the overall architecture of our TinyURL system.

![TinyURL High-Level Design](https://github.com/user-attachments/assets/e9f3a966-f660-41ce-b288-e6cbbb7feb8d)

### Key Components

- **API Gateway:** Handles incoming requests and routes them to the appropriate services.
- **URL Shortener Service:** Generates unique short codes for long URLs.
- **Snowflake ID:** Generates unique IDs for short URL creation.
- **Base62 Encoding:** Converts generated IDs into shorter alphanumeric codes.
- **Redis Cache:** Stores frequently accessed URL mappings to reduce database reads.
- **Redirection Service:** Retrieves the original URL and redirects the user.
- **Database:** Stores the long URL and short URL mappings.

## 4. Conditional Write

A conditional write means a database operation will be performed only if a specified condition is satisfied.

In TinyURL, before inserting a mapping, we check whether the URL or short code already exists.

**Example:**
- If `google.com` does not exist, allow the insertion.
- If `google.com` already exists, return its existing short URL instead of creating a duplicate.
- If a generated short code already exists, generate another code.

This helps prevent duplicate mappings and unwanted overwrites, especially when multiple users try to shorten the same URL simultaneously.

## 5. Atomic Transaction

Atomicity means either all database operations succeed together or none of them succeed.

In TinyURL, we need to save two mappings:

1. Long URL → Short URL
2. Short URL → Long URL

Suppose the first mapping is successfully saved, but the second mapping fails due to a server crash. This would leave our database with inconsistent data.

To prevent this, we use an atomic transaction:

- If both writes succeed, commit the transaction.
- If either write fails, roll back the entire transaction.

### Why Do We Need Both?

| Concept | Purpose |
|---|---|
| **Conditional Write** | Prevents duplicate records and unwanted overwrites. |
| **Atomic Transaction** | Ensures all related database writes succeed or fail together. |

Together, they help maintain data consistency and safely handle concurrent requests.

## 6. Database Design

We maintain two tables to support efficient URL lookups in both directions.

### Table 1: Long URL → Short URL (L → S)

This table is used when a user submits a long URL. It helps us check whether the URL has already been shortened and retrieve its corresponding short code.

![Long URL to Short URL Table](https://github.com/user-attachments/assets/b622dfe2-5af7-429d-9672-4e0cba8198c3)

### Table 2: Short URL → Long URL (S → L)

This table is used when someone clicks a short URL. It helps us retrieve the original long URL quickly and redirect the user.

![Short URL to Long URL Table](https://github.com/user-attachments/assets/aa532970-51b9-44b7-87d3-a76b853acdf8)

### Why Two Tables?

| Operation | Table Used |
|---|---|
| Check whether a long URL exists | Long URL → Short URL |
| Retrieve short URL | Long URL → Short URL |
| Retrieve original URL during redirection | Short URL → Long URL |

Maintaining two lookup directions avoids expensive database scans and allows efficient retrieval using primary keys.

---
