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
  - I already know it is being passed as query params but
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
