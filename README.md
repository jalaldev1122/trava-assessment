# Trava Assessment Solution

## Step 1 - Response to CEO

Quick response to the CEO with these 3 points:

1. Gratitude for appreciation that CEO likes the feature and that it’s useful
2. Acknowledgement of the problem and identify one issue resulting from this problem (the one mentioned by the CEO can be used or I can identify another if need be)
3. I can already think of some solutions, I’ll research some options and get back with the one that’s best suited.

## Step 2 - Analyze Problem & Identify Constraints

Before jumping to any solution right away, I’d consider taking a step back and go through the user flow and all filters to identify:

- Which filters cause the URL to be long?
- What is the length of the URL with minimum and maximum filters?
- What is the type of information to be passed in the URL? (e.g., strings, numbers, booleans, stringified array, etc.)
- This can help identify the size of the information as well
- Note down any odd behavior during user flow that might be relevant to the problem at hand.

Next, I’d go through the codebase to identify:

- How is the information being passed and parsed?
- Are there any side-effects for the information being passed in the URL?
  - For example, some information could be related to localStorage or sessionStorage or cookies.
- Note down any constraints identified in the codebase that might be relevant to the problem at hand.

## Step 3 - Research

Before diving into potential solutions, it’s important to analyze how major platforms that rely on URL sharing handle long URLs and shortening strategies.

### LinkedIn

LinkedIn shortens links shared within posts to prevent long, distracting URLs.

- Uses the `lnkd.in` domain as a redirection service.
- Business accounts can track clicks for engagement analytics.

Example shortened URL:

```
https://lnkd.in/g23xXbA
```

Key Takeaways:

- Uses backend storage for link shortening instead of encoding in the URL.
- Provides click analytics to users.
- Uses a fixed-length hash (g23xXbA), improving readability.

## Step 4 - Possible Solution

Since the table supports many filters, sorts, and search options, we need a way to encode this information efficiently. We can use the following approach:

### Shorten URL Service on Backend

This approach works very similar to how most tools that shorten URL work. We have the similar expectation from our platform - creating an easy to share URL for chats/emails.

#### How It Works

When user wants to share a URL,

1. The URL is sent to a backend service through an endpoint. (e.g., `/shorten`)
2. A short hash is created against the URL
3. The original URL is stored in the database along with the short hash pointing to this URL.
4. The hash is then used to construct a URL that is relatively much shorter.
5. This short URL can be shared with other users.

When another user receives and accesses the URL,

1. The URL is sent to a backend service through an endpoint. (e.g., `/get-original-url`)
2. Using the hash from URL, the original URL can be retrieved from database.
3. We update the `last_used` attribute in db against the hash and original URL.
   - This can be helpful in cleanup.
4. The original URL can be sent back to the frontend.
5. The frontend receives the original URL with all the original filters defined.

Note: To improve performance, we can store the URL and hash in a cache service like Redis, etc.

#### Advantages

- The URL is very short and easy to share.
- This approach provides simplicity and offers scalability.
- Tracking for shared URLs can be implemented.
  - Tracking has a whole set of benefits. It can be used to view which types of filters are being used more often, that can help us create presets for filters later on.

#### Trade-Offs

- Requires backend storage (in the form of database and cache).
- Security measures must be taken to protect backend service from malicious content.
- Requires cleanup in db to remove URLs no longer in use.

#### Handling all Use Cases

Note: For the following use cases,

- `n` denotes `number of filters in a URL`

##### 1. When sharing a URL with no filters (n=0)

This case doesn't require much computation on the backend. We don't need to shorten this URL nor store it in db or cache.

##### 2. When sharing a URL with less filters (n<3)

Since we do not have any tracking requirements for the URL, we can also ignore shortening this URL and storing it in db or cache.

##### 3. When sharing a URL with more filters (n>=3)

Having more than 3 filters can cause the URL to be long and can be difficult to maintain. In this case, we can use this approach to create hash and then create a short URL to be shared with other users.

##### 4. Shared URL is not opened for more than 30 days.

Assuming we set 30 days as the cleanup time for old URLs, if a URL has not been used for 30 days or more, we can remove it from the database as well as cache. We can consume `last_used` attribute against a short URL to determine how long has the URL has remained unused.

## Step 5 - Writing Technical Specifications

In this step, we will define the technical details of implementing the URL shortening solution. This includes architecture, database schema, API design, security measures, and scalability considerations.

[Technical Specifications](technical-specifications.md)

## Step 6 - Planning, Prioritization & Scheduling

Now that we have a technical specification, the next step is to plan, prioritize and schedule for the development process efficiently.

[Planning, Prioritization & Scheduling](plan.md)

## Step 7 - Ongoing Maintenance & Monitoring

### Ongoing Maintenance Requirements

Once the solution is live, it’s important to ensure long-term stability, performance, and security. The following are areas that will require ongoing attention:

#### 1. Database and Cache Management

##### Cleanup of Old URLs:

- Periodically remove old, unused short URLs (e.g., URLs not accessed in the last 30 days) to avoid database bloat.
- Implement automatic cleanup jobs (e.g., nightly) using cron jobs or similar schedulers.
- Use TTL (Time-to-Live) for cache entries to ensure URLs that haven’t been used recently are cleared automatically.

##### Cache Invalidation:

- If any data is updated or changed, we’ll need to ensure that the cache is invalidated properly to avoid serving stale information.
- Use cache versioning or triggers that force cache refresh when certain filters or fields are updated.

#### 2. Security Management

##### API Security:

- Implement rate limiting to avoid abuse of the URL shortening service. Tools like API Gateway or nginx can help enforce this.
- Regularly update dependencies to mitigate security vulnerabilities (e.g., SQL injection, XSS, CSRF).
- Monitor for abnormal usage patterns (e.g., excessive URL shortening requests or unusual access to the short URL service), which might indicate potential malicious activity.

##### Link Abuse Prevention:

- Implement URL sanitization (e.g., validating content before shortening URLs) to prevent the creation of potentially dangerous or misleading links.
- Review user behavior and add security checks, such as verifying URLs against known bad IPs or blocking domains that could be flagged for spam or malicious content.

#### 3. Monitoring and Alerting

- Set up comprehensive monitoring tools like Prometheus, Datadog, or Grafana to track system health and performance metrics.
- Alerting systems can notify the team of critical issues such as: - API downtime - High failure rates (e.g., shortening URL failures or database errors) - Slow response times (i.e., API response times exceeding a threshold)

#### 4. Backup and Recovery

- Regular backups of the database and cache are crucial to prevent data loss in case of failures.
  - Use tools like pg_dump for PostgreSQL backups, and Redis persistence options for backing up cache.
  - Set up automated backups with regular testing of recovery procedures to ensure data integrity.

### Metrics and KPIs to Track

#### 1. Engineering/Operational Metrics

Tracking technical performance is critical to ensure the URL shortening system remains reliable and responsive. Here are some metrics that should be continuously tracked:

##### API Performance

- API Response Time:
  - Measure the average response time for shortening a URL (<200ms target) and retrieving the original URL. This can be monitored using tools like New Relic or Datadog.
- API Failure Rate: - Monitor the percentage of failed requests (e.g., 4xx or 5xx errors). This helps in identifying service disruptions or broken endpoints. - Target: Failure rate should be kept below 0.1%.

##### System Resource Usage

- CPU and Memory Usage:
  - Track the server’s CPU and memory utilization to ensure the backend is scaling as needed.
    - Target: Ensure CPU usage stays below 80% and memory usage below 75%.

##### Database and Cache Efficiency

- Query Execution Time:

  - Track how long it takes for the system to query the database, especially for high-traffic endpoints like /get-original-url.
    - Target: Average query execution time should be under 50ms.

- Cache Hit Rate:
  - Monitor the hit rate of Redis cache (the ratio of cache hits to cache misses).
    - Target: Aim for at least 90% cache hit rate, ensuring most requests are served directly from cache for better performance.

#### 2. Business-Facing Metrics

Tracking user interactions with the service can help guide business decisions and enhance user experience.

##### Shortened URL Usage

- Number of Shortened URLs:

  - Track the total number of shortened URLs generated. This will indicate the popularity of the service and help plan for scalability.
    - Target: Grow the number of generated URLs over time, aiming for a steady increase in usage.

- Click-Through Rate (CTR):
  - Measure the clicks on shared URLs to understand how often shortened links are accessed.
    - Target: Improve CTR over time by optimizing filters and user engagement.

##### Filter Usage Insights

- Most Common Filters Applied:
  - Track the filters most frequently used in the shared URLs. This information can help optimize the system and create useful presets for users to quickly access common queries.
    - Target: Identify the top 3-5 most common filter combinations and consider adding them as one-click presets for users.

##### User Engagement

- Sharing Frequency:
  - Measure how often users share URLs with filters applied. This can guide decisions on how to streamline or improve sharing.
    - Target: Monitor if the feature is being used regularly and if there is any spike in usage during specific times or events.

### Continuous Improvement and Feature Enhancements

In addition to regular maintenance, there will be future improvements that can be added based on user feedback, business needs, and performance monitoring:

#### 1. Feature Enhancements

##### Custom Short URLs:

Allow users to create custom short URLs (e.g., coolcompany.com/age30-users). This can improve user experience and brand presence.

##### Presets for Filters:

Based on insights gathered from filter usage, we can pre-configure common filter combinations for users to apply instantly, thus reducing the need to manually re-enter the same filters.

#### 2. Performance Optimization

##### Database Sharding:

As the number of URLs and user interactions grows, we may need to shard the database to handle large amounts of data more efficiently.

##### Horizontal Scaling:

Increase system capacity by adding more instances for the backend services and caches.

#### 3. Security Improvements

##### Encryption:

Implement end-to-end encryption for URLs, ensuring that sensitive data passed through the short URLs is protected.

##### Advanced Malicious Link Detection:

Implement machine learning-based URL analysis to detect and block potentially harmful links or spammy URLs.

### Long-Term Strategy

Finally, I’d keep track of the overall user satisfaction and business impact to measure the success of the URL shortening feature:

#### 1. User Feedback:

Regular surveys and customer feedback sessions to assess if users are finding the feature valuable.

#### 2. Business Impact:

Measure how often shared URLs lead to successful actions (e.g., sign-ups, purchases, or engagement with filters).

## Step 8 -
