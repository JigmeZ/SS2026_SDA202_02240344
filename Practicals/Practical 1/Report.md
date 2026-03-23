# CHAPTER 8 : DESIGN a URL Shortener 

In this report we were presented with a candidate and an interviewer discussing how to solve the LONG URL format through an interview.
It walks us through four different steps namely: 
1. Understanding the problem and establishing design scope
2. Proposing high-level design and get buy-in
3. Designing deep dive
4. Wrapping up

---

## STEP 1 - Understanding the problem and Establishing design Scope 

In these steps they talk about how long the usual URL is given the example 
https://www.systeminterview.com/q=chatsystem&c=loggedin&v=v3&l=long 

Through aliases they make shorter URLs like https://tinyurl.com/y7keocwj. If you click the alias, it redirects you back to the original URL.

The main objective in these steps was to find the problems and gather the requirements of the problems. The candidate asks the interviewer key questions such as:

- How much traffic will the system handle?  
  100 millions URL per day  

- How short should the URL be?  
  As short as possible  

- What characters are allowed?  
  Combinations of number and alphabets  

- Can URLs be deleted or updated?  
  Assuming "No" in this case  

### Basic Use Cases

1. URL shortening: given a long URL => return a much shorter URL  
2. URL redirecting: given a shorter URL => redirect to the original URL  
3. High availability, scalability, and fault tolerance considerations  

### Back of the envelope estimation

- Write operation: 100 million URLs are generated per day  
- Write operation per second: 100 million / 24 / 3600 = 1160  
- Read operation: Assuming ratio of read operation to write operation is 10:1, read operation per second: 1160 * 10 = 11,600  
- Assuming the URL shortener service will run for 10 years, this means we must support 100 million * 365 * 10 = 365 billion records  
- Assume average URL length is 100  
- Storage requirement over 10 years: 365 billion * 100 bytes * 10 years = 365 TB  

---

## STEP 2 - Proposing high-level design and get buy-in

In this Step it talks about How api endpoint helps URL shortening with the help of REST API and also how redirect works and also the flow of URL.

### API ENDPOINTS

API Endpoints - it facilitates the communication between the clients and the server.

And for this A URL shortening only need two API endpoints: 

### URL shortening
to create new short URL the client need to send a POST request containing one parameter  

POST api/v1/data/shorten  

• request parameter: {longUrl: longURLString}  
• return shortURL  

### URL redirecting

To redirect a short URL to the corresponding long URL, a client sends a GET request. The API looks like this: 

GET api/v1/shortUrl  

• Return longURL for HTTP redirection  

### URL redirecting

When a user clicks a short URL, the server looks up the original long URL and redirects the user. There are two types of redirects:

- 301 redirect — permanent. The browser remembers it and goes directly next time. Saves server load but cannot track clicks.  
- 302 redirect — temporary. The browser always asks the server first. Slightly slower but allows click tracking and analytics.  

Each redirection method has its pros and cons. If the priority is to reduce the server load, using 301 redirect makes sense as only the first request of the same URL is sent to URL shortening servers. However if analytics is important, 302 redirect is a better choice as it can track click rate and source of the click more easily.

### URL Shortening

The server uses a hash function to convert a long URL into a short 7-character code. This code is stored in a database alongside the original URL as a simple mapping of short URL to long URL. The hash function must satisfy the following requirements:

- Each longURL must be hashed to one hashValue  
- Each hashValue can be mapped back to the longURL  

---

## STEP 3 - Designing Deep Dive

In this section we go on a deep dive into the following:

- Data model  
- Hash function  
- URL shortening  
- URL redirecting  

### Data model 

In the high-level design, we assumed that all data is stored in a hash table. However, in real-world systems, this is not practical because memory is limited and expensive.  
So instead, we use a relational database to store the mapping between short and long URLs. The simplified version of the table contains 3 columns: id, shortURL, longURL.

### Hash Function

A hash function is used to convert a long URL into a short code (shortURL).

### Character Set

The short URL uses:

- Numbers (0–9) → 10  
- Lowercase letters (a–z) → 26  
- Uppercase letters (A–Z) → 26  

Total = 62 characters  

We need to support 365 billion URLs, so we calculate:

62⁷ = 3.5 trillion  

This is enough, so we use a 7-character short URL.

### Hash + Collision Resolution

We can use hash functions like:

- CRC32  
- MD5  
- SHA-1  

But these produce long outputs. So we:

- Take the first 7 characters  
- Check if it already exists  
- If collision occurs → add extra string and retry  

However this method is expensive because it has to query the database to check if a shortURL exists for every request. A technique called Bloom Filter can improve performance. A bloom filter is a space-efficient probabilistic technique to test if an element is a member of a set.

### Base 62 conversion

This is a better approach. Instead of hashing, we first generate a unique number and then we convert it into Base 62. Base conversion helps to convert the same number between its different number representation systems. It is used as there are 62 possible characters for hashValue. It is one of the best approaches.

For example to convert 11157 in base 10 to base 62:  
11157 = 2 x 62² + 55 x 62¹ + 59 x 62⁰ = [2, 55, 59] → [2, T, X] in base 62  
So the short URL becomes: https://tinyurl.com/2TX

### Advantage if we use Base 62:

- There will be no collision  
- It's Faster  
- Simple and efficient  

### URL shortening Flow

This is the process when a user creates a short URL using Base 62:

- User inputs a long URL  
- System checks database:  
  - If already exists → return existing short URL  
- If new URL:  
  - Generate a unique ID using a unique ID generator  
  - Convert ID → short URL using Base 62  
  - Store (id, shortURL, longURL) in database  
- Return short URL to user  

For example:  
- Input longURL: https://en.wikipedia.org/wiki/Systems_design  
- Unique ID generator returns ID: 2009215674938  
- ID is converted to shortURL: "zn9edcu" using Base 62 conversion  
- ID, shortURL and longURL are saved to the database  

### URL redirecting deep dive 

The redirect process starts when the user presses the shortened Url. The system processes the request through several components to retrieve the original long URL efficiently.

- The user clicks a short URL  
- The request is first handled by a load balancer and then forwarded to the server  
- The system then checks the cache  
  - If found the Url is returned Immediately  
  - If not the database is queried  
- And then if the URL exist in the database it is returned to the user and also stores in cache for future requests  
- And if there is no Short URL, an error message will be displayed  
- Finally the user will be redirected to the original URL  

---

## STEP 4 - Wrap Up

In this chapter, the design of a URL shortener system was discussed in detail, covering both high-level and low-level design aspects. The system was structured to efficiently convert long URLs into shorter, manageable links and redirect users back to the original URLs when required.

The design included key components such as API endpoints, database structure, hash functions, and caching mechanisms. A relational database was used to store mappings between short and long URLs, while Base 62 conversion was applied to generate compact and unique short URLs. Additionally, caching was introduced to improve performance during URL redirection, as read operations are significantly higher than write operations.

Beyond the core design, several important considerations can further enhance the system:

### Rate Limiting:
To prevent abuse, the system can limit the number of requests from a single user or IP address. This helps protect against malicious attacks and excessive usage.

### Web Server Scaling:
Since the web servers are stateless, the system can easily scale horizontally by adding more servers to handle increased traffic.

### Database Scaling:
Techniques such as replication and sharding can be used to manage large volumes of data and improve system performance and reliability.

### Analytics Integration:
Adding analytics allows the system to track user interactions, such as the number of clicks, user location, and access time. This data can be valuable for business insights.

### Availability and Reliability:
Ensuring high availability and fault tolerance is critical. The system should remain operational even in the case of failures by using redundancy and distributed architecture.

---

## Conclusion

In conclusion, designing a URL shortener involves careful consideration of scalability, performance, and reliability. By applying efficient algorithms and system design principles, a robust and scalable solution can be achieved.

---

## STEP 5 - Critical Analysis

In this step we go through a detailed critical analysis of the full chapter covering the following:

1. Analysis of the Problem Statement
2. Analysis of the Solutions given by the Author
3. Review of the Analysis — understanding, confusions and further topics to explore
4. Brief Summary of Critical Analysis

---

### 1. Analysis of the Problem Statement

In this chapter the problem statement talks about how long URLs are hard to share, store and display. The main objective was to convert a long URL like  
https://www.systeminterview.com/q=chatsystem&c=loggedin&v=v3&l=long  
into a short and manageable alias like https://tinyurl.com/y7keocwj that can be used easily.

The requirements gathering in Step 1 was the right starting point and the questions asked by the candidate were also reasonable. However the scope was kept very narrow. The report assumes that URLs cannot be deleted or updated which simplifies the design but in a real world product this would be a problem for things like expired marketing links or security takedowns.

The back of the envelope estimation was also a strong part of this step. The traffic estimate of 100 million URLs per day was properly used to calculate write operations per second, read operations per second, total records over 10 years and also the total storage requirement of 365 TB. This kind of calculation is very important in a system design interview to show the scale of the problem.

---

### 2. Analysis of the Solutions given by the Author

### API Design

The API design in Step 2 was solid and it correctly identifies the two main endpoints which are the POST for creating short URLs and the GET for redirecting. The explanation of 301 and 302 redirects was one of the stronger parts of the report. The trade-off is real:

- 301 is more performant but it kills analytics  
- 302 preserves observability but adds slightly more server load  

This is a nuanced point and the report handled it well.

### Hash Function and Base 62

In Step 3 the Hash Function section is where things get a bit confusing. The report introduces CRC32, MD5 and SHA-1 as hash options and correctly notes that they produce outputs that are too long. It then moves to Base 62 conversion as a better approach which it is. However the contrast between the two approaches was not explained clearly enough. The reader is left uncertain about why exactly Base 62 is better beyond just "no collision."

The original passage also mentions Bloom Filters as a technique to improve the performance of the Hash + Collision Resolution approach but this was missing from the notes. A bloom filter is a space-efficient probabilistic technique to test if an element is a member of a set and it is worth mentioning.

There is also a notable error in the original notes. The notes mentioned "Base 64" in some places when describing the conversion method but the correct term is Base 62. Base 62 uses 62 characters which are 0–9, a–z and A–Z. Base 64 is a different thing and includes + and / which would cause problems in URLs. This inconsistency shows the concept was understood but the terminology was not precise.

### URL Redirecting Flow

The URL redirecting flow using a cache layer in Step 3 was well reasoned. Checking cache before the database is standard and sensible. The acknowledgment that reads vastly outnumber writes is a key insight that justifies the caching strategy used in the design.

---

### 3. Review of the Analysis

In this section we talk about what we understood clearly, where we found confusions and what further topics we want to explore from this chapter.

### What we understood clearly

The overall architecture is a classic read-heavy system. A unique ID is generated using a unique ID generator and then converted to Base 62 and then the mapping is stored and served through a cache-first lookup. This is clean and makes sense for this kind of system. The passage also mentions that the distributed unique ID generator was already discussed in Chapter 7 which shows that the system design book builds concepts on top of each other step by step.

### Where we found confusions

- The notes did not explain how the unique ID generator works in a distributed environment. The original passage mentions that implementing a unique ID generator is challenging in a highly distributed environment and refers to Chapter 7 for the solution. It would have been good to include a brief mention of this in the notes.  

- The Data Model section says a relational database is used with 3 columns which are id, shortURL and longURL but it does not talk about what indexes are used or how the table is queried efficiently at scale.  

- The Wrap Up section mentions database sharding as a scaling technique but does not connect it back to the actual design. How do you shard a URL table? By short URL hash? By creation time? It feels more like a checklist item rather than a thought through decision.  

### Further Topics we want to Explore

- **Distributed ID Generation** — How the unique ID generator from Chapter 7 works at scale and how it connects to Base 62 conversion  
- **Bloom Filters** — How they work and how they improve the performance of the Hash + Collision Resolution approach  
- **Cache Invalidation** — What happens when a URL needs to be deleted or updated even if it is out of scope for this chapter  
- **Consistent Hashing** — This is relevant if the cache layer is distributed across multiple nodes  

---

### 4. Brief Summary of Critical Analysis

In this brief summary the report provides a good beginner to intermediate overview of URL shortener design. It correctly identifies the core components such as REST endpoints, hashing vs Base 62 conversion, redirect types, caching and database storage. The 301 vs 302 redirect trade-off and the back of the envelope estimation stand out as genuinely strong points in the chapter.

However the notes have a few weaknesses. First there is a Base 62 and Base 64 terminology error that reduces technical accuracy. Second the Bloom Filter technique which was mentioned in the original passage was missing from the notes entirely. Third the unique ID generation problem which is arguably the hardest part of the design was not explained in the notes even though the original passage references Chapter 7 for this.

Overall the notes reflect a solid understanding of the what — meaning the components that are needed — but can be improved on the how — meaning the reasoning behind specific choices and the challenges of implementing them at real scale. Adding the Bloom Filter explanation, fixing the Base 62 terminology and briefly mentioning the distributed ID generator would make the notes much more complete and accurate.
