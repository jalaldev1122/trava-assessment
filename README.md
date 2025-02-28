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

#### Encoding

Let’s use this URL for this example:

```
https://app.coolcompany.com/users?first_name=John&first_name_op=eq&last_name=Snow&last_name_op=neq&age=30&age_op=gte&sort=age&sort_dir=asc
```

Each filter can be mapped into a binary structure, based on:

- Field Names (first_name, last_name, age)
- Operators (eq, neq, gte)
- Values (John, Snow, 30)
- Sorting (sort: age asc)

We can define bit mapping against each attribute and operator:

| Filter Component | Bitwise Representation |
| ---------------- | ---------------------- |
| first_name       | 0001                   |
| last_name        | 0010                   |
| age              | 0011                   |
| eq               | 00                     |
| neq              | 01                     |
| gte              | 10                     |
| asc              | 0                      |
| desc             | 1                      |

We can convert data to binary:

| Filter Component | Operator | Value | Binary Representation |
| ---------------- | -------- | ----- | --------------------- |
| first_name       | eq       | John  | 0001 00 0100101011    |
| last_name        | neq      | Snow  | 0010 01 0101010101    |
| age              | gte      | 30    | 0011 10 00011110      |
| Sorting          | age asc  | -     | 0011 0                |

Combining all the data
`0001 00 0100101011 0010 01 0101010101 0011 10 00011110 0011 0`

To make the binary string URL safe, we can:

1. Convert it to large integer
2. Encode it using Base62 (using digits and letters).

Base62 for Binary String becomes: `F6X8G2`

Now the final URL becomes:

```
https://app.coolcompany.com/users?s=F6X8G2
```

This URL is much shorter than the original URL.

#### Decoding

1. Decode Base62 to Binary String,
2. Split binary segments into field_names, operators, and values.
3. Restore the original filter structure.
4. Apply filters and sort the table accordingly.

#### Pros & Cons

##### Advantages

- Converts long URLs into very short base62 strings.
- Encoding and decoding are efficient with bitwise operations.
- Base62 avoids special characters making it URL-safe.

##### Trade-Offs

- The base62 string is not human readable and might not be understandable by the user.
- Each new field requires a predefined bit assignment.
- Adding new filters requires modifying the binary string.
