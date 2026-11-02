# TinyURL-System-Design-HLD-and-LLD-

## Functional requirements:
  *  User should be able to generate the short url from long url
  *  User should able to redirect to correct long url on clicking short url
  *  same long url should always generates same short url
  *  Custom short URL
  *  URL expiration
  *  Delete short URL

## Non-Functional requirement:
  * Low Latency	- Short URL redirection should happen very quickly.
  * High Availability Our service should be available most of the time, with minimal downtime.
  * Scalability	Our system should handle millions of URLs and users as traffic increases.

<img width="1297" height="707" alt="image" src="https://github.com/user-attachments/assets/e9f3a966-f660-41ce-b288-e6cbbb7feb8d" />

1. Conditional Write

A conditional write means a database operation will be performed only if a specified condition is satisfied.

In TinyURL, before inserting a mapping, we check whether the URL or short code already exists.

For example:
- If google.com does not exist, allow the insertion.
- If google.com already exists, reject the insertion to prevent duplicate mappings or overwriting existing data.
This helps handle concurrent requests when multiple users try to shorten the same URL simultaneously.

2. Atomic Transaction

Atomicity means either all database operations succeed together or none of them succeed.

In TinyURL, we need to save two mappings:
- Long URL → Short URL
- Short URL → Long URL
Suppose the first mapping is successfully saved, but the second mapping fails due to a server crash. This would leave our database with inconsistent data.

To prevent this, we use an atomic transaction:
- If both writes succeed, commit the transaction.
- If either write fails, roll back the entire transaction.
Why do we need both?
- Conditional Write → Prevents duplicate records and unwanted overwrites.
- Atomic Transaction → Ensures all related database writes succeed or fail together.
