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

## Step 4 - Possible Approaches to Solution

Since the table supports many filters, sorts, and search options, we need a way to encode this information efficiently. A few approaches can be used here:

### Bitwise Encoding

Bitwise encoding is an advanced technique for reducing URL length by converting filter parameters into compact binary data. This is useful when URLs contain repetitive patterns, predictable structures, and a limited set of possible values.

#### How It works

Instead of storing filters as plaintext key-value pairs,

- Assign a binary representation to each filter type and operator.
- Convert numerical values into efficient binary storage.
- Use Base62 (or similar) encoding to convert binary data into a compact string.

#### Advantages

- Converts long URLs into very short base62 strings.
- Encoding and decoding are efficient with bitwise operations.
- Base62 avoids special characters making it URL-safe.

#### Trade-Offs

- The base62 string is not human readable and might not be understandable by the user.
- Each new field requires a predefined bit assignment.
- Adding new filters requires modifying the binary string.
