**Module 2: Valkey Hands-On: Core Data Structures**

**Goal:** Master the fundamental Valkey data structures through practical examples using `valkey-py`, understanding their characteristics and primary use cases.

**Activity:** Interactive Python code exercises for each data structure.

**Setup**

- Ensure `pip install valkey` is run in your Python environment.

- Your Valkey Docker container (`valkey-dev`) from Module 1 should be running (`docker ps` to verify).

- Initial Python connection boilerplate (copy and paste into a new `data_structures.py` file):
  
  ```python
  import valkey
  import json # Useful for serializing/deserializing complex Python objects into JSON strings
  
  # Connect to the local Valkey server running in Docker
  # The decode_responses=True argument is crucial! It converts binary replies from Valkey to UTF-8 strings.
  v = valkey.Valkey(host='localhost', port=6379, db=0, decode_responses=True)
  
  # Basic connection check
  try:
      if v.ping():
          print("Successfully connected to Valkey!")
          # Optional: Clear all keys for a clean start in this module (use with caution in real apps!)
          v.flushdb()
          print("Valkey database flushed for fresh examples.")
      else:
          print("Failed to connect to Valkey! Please ensure your Docker container is running.")
          exit() # Stop the script if connection fails
  except valkey.exceptions.ConnectionError as e:
      print(f"Could not connect to Valkey: {e}. Is your Docker container running and port 6379 accessible?")
      exit()
  ```

**Content (Live Coding & Participant Exercises):**

Valkey provides a diverse set of native data structures, each highly optimized for different use cases. You'll find that the `valkey-py` client library maps directly to Valkey's commands, making interaction intuitive. While we'll focus on the core five here, keep in mind Valkey offers many more advanced types (which we'll touch on later!).

1. **Strings**
   
   - **Concept:** This is the **most fundamental data type** in Valkey. A String is simply a sequence of bytes. They are **binary-safe**, meaning they can hold any kind of data (text, integers, floats, images, JSON strings, etc.) up to 512 MB. Strings are very memory-efficient for simple key-value storage.
   
   - **Python Examples:**
     
     ```python
     print("\n--- Strings ---")
     # 1. Store and retrieve a simple text value
     v.set('greeting', 'Hello, Valkey! From Python')
     print(f"GET greeting: {v.get('greeting')}")
     
     # 2. Store a JSON string (Valkey doesn't interpret it, just stores as a string)
     user_data = {"name": "Alice", "role": "admin", "status": "active"}
     v.set('user:session:123', json.dumps(user_data)) # Convert Python dict to JSON string
     retrieved_user_json = v.get('user:session:123')
     print(f"User Session (raw JSON): {retrieved_user_json}")
     print(f"User Session (parsed): {json.loads(retrieved_user_json)}")
     
     # 3. Set with expiration (TTL - Time To Live)
     v.setex('temp_message', 10, 'This message will vanish!') # Sets key 'temp_message' with 10s expiry
     print(f"GET temp_message: {v.get('temp_message')}")
     print(f"TTL for temp_message: {v.ttl('temp_message')} seconds (should be ~10)")
     #Experiment: time.sleep(11) then print(v.get('temp_message')) -> None
     
     # 4. Atomic counters: INCR/INCRBY are atomic operations, crucial for concurrency!
     v.set('page_views', 0) # Initialize counter (can be set to any number)
     v.incr('page_views')    # Atomically increments by 1
     v.incrby('page_views', 5) # Atomically increments by 5
     print(f"Current page views: {v.get('page_views')}") # Note: Valkey stores numbers as strings, valkey-py decodes.
     v.decr('page_views')    # Atomically decrements by 1
     print(f"Page views after decrement: {v.get('page_views')}")
     ```
   
   - **Use Cases:** General-purpose caching (HTML fragments, JSON objects), session tokens, hit counters, unique ID generation (e.g., `INCR` on a key).

2. **Hashes**
   
   - **Concept:** Hashes are collections of field-value pairs within a single key, similar to Python dictionaries, Java HashMaps, or objects in JSON. They are **notably memory efficient**, especially when they contain a small number of elements and small field/value sizes, potentially using up to 10 times less memory than other encodings for specific use cases. They are ideal for representing objects or grouping related fields together.
   
   - **Python Examples:**
     
     ```python
     print("\n--- Hashes ---")
     user_id = 'user:profile:100'
     # 1. Store a user profile (mapping of fields to values)
     v.hset(user_id, mapping={
     'username': 'bob_the_builder',
     'email': 'bob@example.com',
     'posts_count': 0,
     'last_login': '2024-07-26T10:30:00Z'
     })
     print(f"Username: {v.hget(user_id, 'username')}") # Get a specific field
     print(f"User's email: {v.hget(user_id, 'email')}")
     
     # 2. Atomically increment a numeric field within the hash
     v.hincrby(user_id, 'posts_count', 1)
     v.hincrby(user_id, 'posts_count', 3)
     print(f"Posts count for {user_id}: {v.hget(user_id, 'posts_count')}")
     
     # 3. Get all fields and values of a hash
     print(f"All user data: {v.hgetall(user_id)}")
     
     # 4. Check if a field exists in a hash
     print(f"Does 'phone' field exist? {v.hexists(user_id, 'phone')}")
     ```
   
   - **Use Cases:** Storing user profiles, product details, configuration settings, or any object where you want to access/update individual fields efficiently.

3. **Lists**
   
   - **Concept:** Lists are **ordered collections of strings**, sorted by insertion order. They are implemented as a linked list, making additions and removals from the ends very fast (O(1)). This makes them perfect for building **queues (FIFO - First-In, First-Out)** or **stacks (LIFO - Last-In, First-Out)**. The atomic blocking capabilities (like `BLPOP`) enable Valkey to function as a reliable message broker or a lightweight task queue.
   
   - **Python Examples:**
     
     ```python
     print("\n--- Lists (as a Queue/Stack) ---")
     log_queue = 'application:error_logs'
     # 1. LPUSH: Add elements to the HEAD (left) of the list (stack-like)
     v.lpush(log_queue, "Error: DB Connection Timeout")
     v.lpush(log_queue, "Critical: Service Crashed")
     
     # 2. RPUSH: Add elements to the TAIL (right) of the list (queue-like)
     v.rpush(log_queue, "Warning: Low Disk Space")
     v.rpush(log_queue, "Info: User Login Success")
     
     # 3. LRANGE: Get a range of elements (0 to -1 means all)
     print(f"Current log queue: {v.lrange(log_queue, 0, -1)}")
     print(f"Log queue length: {v.llen(log_queue)}")
     
     # 4. LPOP: Remove and return an element from the HEAD (left)
     next_critical_log = v.lpop(log_queue)
     print(f"Processed (LPOP - Stack): {next_critical_log}")
     print(f"Remaining queue: {v.lrange(log_queue, 0, -1)}")
     
     # 5. RPOP: Remove and return an element from the TAIL (right)
     next_info_log = v.rpop(log_queue)
     print(f"Processed (RPOP - Queue): {next_info_log}")
     print(f"Remaining queue: {v.lrange(log_queue, 0, -1)}")
     ```
   
   - **Use Cases:** Message queues, task queues for background workers, activity feeds, recent item lists, historical data.

4. **Sets**
   
   - **Concept:** Sets are **unordered collections of unique strings**. They function similarly to sets in programming languages, automatically handling duplicates. Valkey provides **fast (O(1)) operations** for adding members (`SADD`), removing members (`SREM`), and checking for the existence of a member (`SISMEMBER`).
   
   - **Python Examples:**
     
     ```python
     print("\n--- Sets ---")
     unique_visitors = 'website:unique_visitors:2024-07-26'
     # 1. SADD: Add members to a set. Duplicates are ignored.
     v.sadd(unique_visitors, 'user_a', 'user_b', 'user_a', 'user_c', 'user_d')
     print(f"All unique visitors: {v.smembers(unique_visitors)}") # Returns a Python set
     
     # 2. SISMEMBER: Check for presence of a member
     print(f"Is 'user_b' a member? {v.sismember(unique_visitors, 'user_b')}")
     print(f"Is 'user_e' a member? {v.sismember(unique_visitors, 'user_e')}")
     
     # 3. SREM: Remove a member
     v.srem(unique_visitors, 'user_c')
     print(f"Visitors after removal: {v.smembers(unique_visitors)}")
     
     # 4. SCARD: Get the number of members in a set (cardinality)
     print(f"Number of unique visitors: {v.scard(unique_visitors)}")
     
     # 5. Set operations (conceptual, can be coded in Python/Valkey)
     v.sadd('sport_fans', 'alice', 'bob', 'charlie')
     v.sadd('music_fans', 'bob', 'david', 'alice')
     print(f"Fans of both (Intersection): {v.sinter('sport_fans', 'music_fans')}")
     print(f"All distinct fans (Union): {v.sunion('sport_fans', 'music_fans')}")
     ```
   
   - **Use Cases:** Tracking unique visitors, tags on articles/products, friends lists, common interests, access control lists (users in a role).

5. **Sorted Sets (ZSets)**
   
   - **Concept:** Sorted Sets are like Sets (collections of unique strings), but **each member is associated with a numerical `score` (a float)**. This score is used to maintain an order, allowing for efficient retrieval by rank or score range. Members are unique, but scores can be duplicated. This data type is particularly useful for implementing **leaderboard-style functionalities** and time-series data.
   
   - **Python Examples:**
     
     ```python
     print("\n--- Sorted Sets (ZSets) ---")
     leaderboard = 'game:high_scores'
     # 1. ZADD: Add members with scores. If a member exists, its score is updated.
     v.zadd(leaderboard, mapping={'Alice': 1500, 'Bob': 1200, 'Charlie': 1800, 'David': 1500, 'Eve': 1650})
     
     # 2. ZREVRANGE: Get members by rank (descending score, highest first)
     # `withscores=True` returns tuples of (member, score)
     print(f"Top 3 players: {v.zrevrange(leaderboard, 0, 2, withscores=True)}") # 0 to 2 means first 3 (0-indexed)
     
     # 3. ZSCORE: Get the score of a specific member
     print(f"Alice's score: {v.zscore(leaderboard, 'Alice')}")
     
     # 4. ZINCRBY: Atomically increment a member's score
     v.zincrby(leaderboard, 200, 'Bob') # Bob's score becomes 1400
     print(f"Bob's new score: {v.zscore(leaderboard, 'Bob')}")
     
     # 5. ZRANGEBYSCORE: Get members within a score range
     print(f"Players with scores between 1400-1600: {v.zrangebyscore(leaderboard, 1400, 1600, withscores=True)}")
     
     # 6. ZRANK: Get the rank of a member (0-indexed, ascending order)
     print(f"Charlie's rank (ascending): {v.zrank(leaderboard, 'Charlie')}")
     # ZREVRANK for descending rank
     print(f"Charlie's rank (descending): {v.zrevrank(leaderboard, 'Charlie')}")
     ```
   
   - **Use Cases:** Leaderboards, real-time ranking, time-series data (e.g., events by timestamp), rate limiting (based on timestamps), maintaining ordered lists of items.

**Module Conclusion**

"You've just had hands-on experience with Valkey's five core data structures. Notice how each is optimized for specific types of data and operations. This is Valkey's strength: it's not just a flat key-value store, but a data structure server that makes complex data interactions simple and lightning-fast.

While these five are crucial, Valkey also offers a wide array of more specialized data structures and functionalities, some natively, and some through powerful **Valkey Modules**. These include:

- **Streams:** For persistent, high-throughput event logs (we'll cover this in Part 2!).
- **Bitmaps & HyperLogLogs:** For highly memory-efficient tracking of unique items or boolean flags (also in Part 2!).
- **Geospatial Indexes:** For location-based services (finding points within a radius).
- **Bitfields:** For managing multiple counters within a single string.
- **JSON:** Direct support for storing and querying JSON documents.
- **Probabilistic Data Structures via Modules:** Like **Bloom Filters** (for checking likely presence of items) or **Count-Min Sketch**.
- **Vector Search:** A cutting-edge capability for AI/ML workloads (semantic search, recommendation engines).

The key takeaway is that Valkey likely has a native solution or a module for almost any common data problem you'll encounter in enterprise applications. Next, we'll dive into how these core data structures are applied in real-world scenarios, starting with the ubiquitous use case of caching.
