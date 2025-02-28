# Technical Specifications

## Architecture Overview

The URL Shortening Service will be a backend service responsible for:

1. Generating a short hash for long URLs containing filters.
2. Storing the mapping of short hash -> original URL in a database (PostgreSQL) and a cache (Redis) for fast retrieval.
3. Retrieving the original URL when a user accesses the short link.
4. Implementing cleanup logic for unused short URLs.

## System Flow

1. User applies filters and generates a URL in the frontend.
2. Frontend sends the long URL to the backend (POST /shorten).
3. Backend checks if the URL is already stored:
4. If yes -> return existing short URL.
5. If no -> create a new hash, store in DB, return short URL.
6. User shares the short URL with others.
7. Another user accesses the short URL, triggering GET /get-original-url/{hash}.
8. Backend retrieves the original URL from cache or DB and redirects the user.
9. If the short URL is not accessed for 30+ days, it is deleted from the database.

## Database Schema

Using PostgreSQL as the primary database and Redis for caching frequently accessed short URLs.

### Table: `shortened_urls`

| Column Name  | Type                        | Description                          |
| ------------ | --------------------------- | ------------------------------------ |
| id           | SERIAL PRIMARY KEY          | Unique ID                            |
| original_url | TEXT NOT NULL               | The full, long URL                   |
| short_hash   | VARCHAR(10) UNIQUE NOT NULL | Shortened hash                       |
| created_at   | TIMESTAMP DEFAULT NOW()     | When the short URL was created       |
| last_used    | TIMESTAMP DEFAULT NOW()     | Last time the short URL was accessed |

### Redis Cache

- Key: short_url:{hash}
- Value: original_url
- TTL (Time-to-Live): 30 days

## API Endpoints

| Method | Endpoint                 | Description                               |
| ------ | ------------------------ | ----------------------------------------- |
| POST   | /shorten                 | Receives long URL and returns a short URL |
| GET    | /get-original-url/{hash} | Returns the original URL for a given hash |
| DELETE | /cleanup                 | Deletes old/unused URLs from DB & cache   |

## Security Considerations

1. Prevent Malicious URLs:

   - Validate URLs before shortening (e.g., check for phishing attempts).
   - Rate limit requests to avoid abuse.

2. Protect Against Hash Collisions:

   - Use UUID-based hashing instead of MD5 if needed.

3. Prevent Expired URL Access:
   - Return a 404 if a URL has been deleted after 30 days.

## Scalability Consideration

- Use Redis for Faster Retrieval
  - Storing frequently accessed short URLs in Redis improves performance.
- Optimize Hashing Algorithm
  - If MD5 collisions occur, use UUIDs or SHA-256.
- Horizontally Scale the Database
  - If DB queries get slow, use sharding or NoSQL (e.g., DynamoDB).
- Implement URL Expiration Policy
  - Short URLs that haven’t been accessed in 30+ days should be deleted automatically.
