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

Through aliases they make shorter URLs.

The main objective in these steps was to find the problems and gather the requirements of the problems.The candidate asks the interviewer key questions such as:

- How much traffic will the system handle?  
  100 millions URL per day  

- How short should the URL be?  
  As short as possible  

- What characters are allowed?  
  Combinations of number and alphabets  

- Can URLs be deleted or updated?  
  Assuming “No” in this case  

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

### URL Shortening

The server uses a hash function to convert a long URL into a short 7-character code.This code is stored in a database alongside the original URL as a simple mapping of short URL to long URL.

---

## STEP 3 - Designing Deep Dive

In this section we go on a deep dive into the following:

- Data model  
- Hash function  
- URL shotening  
- URL redirecting  

### Data model 

In the high-level design, we assumed that all data is stored in a hash table. However, in real-world systems, this is not practical because memory is limited and expensive.  
So instead, we use a relational database to store the mapping between short and long URLs.

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

### Base 62 conversion

This is a better approach. Instead of hashing, we first generate a unique number and then we convert it into Base 64. Base conversion helps to convert the same number between its different number represent systems. It is used as there are 62 possible characters for Hashvalues. It is one of the best approaches.

### Advantage if we use Base 64:

- There will be no collision  
- It’s Faster  
- Simple and efficient  

### URL shortening Flow

This is the process when a user creates a short URL using Base 64:

- User inputs a long URL  
- System checks database:  
  - If already exists → return existing short URL  
- If new URL:  
  - Generate a unique ID  
  - Convert ID  → short URL using Base 62  
  - Store (id, shortURL, longURL) in database  
- Return short URL to user  

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