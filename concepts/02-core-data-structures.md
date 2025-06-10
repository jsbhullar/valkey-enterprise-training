### **Module 2: Valkey Hands-On: Core Data Structures**

**Goal:** Master the fundamental Valkey data structures through practical, illustrative examples using `valkey-py`, understanding their core characteristics, optimal use cases, and key commands.

**Activity:** Interactive Python code exercises for each data structure. Participants will type and run code alongside the trainer, observing immediate results.

**Setup (Pre-check for participants):**

*   Ensure `pip install valkey` is run in your Python environment.
*   Your Valkey Docker container (`valkey-dev`) from Module 1 should be running (`docker ps` to verify).
*   Initial Python connection boilerplate (copy and paste this into a new file named `data_structures_lab.py` in your `part1/module2/` directory):

    ```python
    import valkey
    import json # Useful for serializing/deserializing complex Python objects into JSON strings
    import time # For demonstrating TTL effects
    import random # For simulating variable worker delays in List example

    # Connect to the local Valkey server running in Docker
    # The decode_responses=True argument is crucial! It converts binary replies from Valkey to UTF-8 strings.
    v = valkey.Valkey(host='localhost', port=6379, db=0, decode_responses=True)

    # Basic connection check
    try:
        if v.ping():
            print("Successfully connected to Valkey!")
            # Optional: Clear all keys for a clean start in this module (use with extreme caution in real apps!)
            v.flushdb()
            print("Valkey database flushed for fresh examples.")
        else:
            print("Failed to connect to Valkey! Please ensure your Docker container is running.")
            exit() # Stop the script if connection fails
    except valkey.exceptions.ConnectionError as e:
        print(f"Could not connect to Valkey: {e}. Is your Docker container running and port 6379 accessible?")
        exit()

    print("\n--- Starting Data Structures Lab ---")
    ```
    

**Content (Live Coding & Participant Exercises):**

Valkey provides a diverse set of native data structures, each highly optimized for different use cases. You'll find that the `valkey-py` client library maps directly to Valkey's commands, making interaction intuitive. While we'll focus on the five core, most frequently used types here, keep in mind Valkey offers many more advanced types (which we'll touch on later!).

---

### **1. Strings**

*   **Concept:** This is the **most fundamental data type** in Valkey. A String is simply a sequence of bytes. They are **binary-safe**, meaning they can hold any kind of data (text, integers, floats, images, JSON strings, etc.) up to 512 MB. Strings are very memory-efficient for simple key-value storage.


*   **Python Examples:**
    ```python
    print("\n--- Strings ---")
    # 1. Store and retrieve a simple text value
    v.set('greeting', 'Hello, Valkey! From Python')
    print(f"GET greeting: {v.get('greeting')}")

    # 2. Store a serialized JSON object (Valkey treats it as a plain string)
    user_session_data = {"user_id": "U12345", "last_activity": int(time.time()), "cart_items": 3}
    v.set('user:session:U12345', json.dumps(user_session_data)) # Convert Python dict to JSON string
    retrieved_session_json = v.get('user:session:U12345')
    print(f"User Session (raw JSON string): {retrieved_session_json}")
    print(f"User Session (parsed Python dict): {json.loads(retrieved_session_json)}")

    # 3. Set with expiration (TTL - Time To Live)
    # 'ex' argument for seconds, 'px' for milliseconds
    v.set('temp_message', 'This message will vanish!', ex=5) # Sets key 'temp_message' with 5s expiry
    print(f"GET temp_message: {v.get('temp_message')}")
    print(f"TTL for temp_message: {v.ttl('temp_message')} seconds (should be ~5)")
    time.sleep(6) # Wait for the key to expire
    print(f"GET temp_message after 6 seconds: {v.get('temp_message')} (Should be None)") # Expected: None

    # 4. Atomic counters: INCR/INCRBY are atomic operations, crucial for concurrency control!
    v.set('website:page_views', 0) # Initialize a counter for a website's page views
    v.incr('website:page_views')    # Atomically increments by 1
    v.incrby('website:page_views', 10) # Atomically increments by 10
    print(f"Current website page views: {v.get('website:page_views')}")
    v.decr('website:page_views')    # Atomically decrements by 1
    print(f"Page views after decrement: {v.get('website:page_views')}")
    ```


*   **Illustrative Use Cases:**
    *   **Caching Dynamic Content:** Storing rendered HTML fragments, JSON API responses, or serialized objects (like `user:session:U12345`) to reduce database load.
    *   **Session Management:** Storing user session data (often with a TTL).
    *   **Atomic Counters:** Tracking real-time metrics like website page views, unique downloads, or generating unique IDs for new records safely in a concurrent environment.

---

### **2. Hashes**

*   **Concept:** Hashes are collections of field-value pairs within a single key, acting like dictionaries or objects. They are notably **memory efficient**, especially when storing a small number of elements with small field/value sizes, potentially using significantly less memory than storing each field as a separate String key. They are ideal for representing structured objects or grouping related fields together.


*   **Python Examples:**
    ```python
    print("\n--- Hashes ---")
    product_id = 'product:details:SKU456'
    # 1. HSET: Store a product's details as a hash
    v.hset(product_id, mapping={
        'name': 'Ergonomic Keyboard Pro',
        'category': 'Peripherals',
        'price': '129.99',
        'stock': 150,
        'last_updated': '2024-07-26T14:00:00Z'
    })
    print(f"Product Name: {v.hget(product_id, 'name')}") # Get a specific field
    print(f"Product Price: {v.hget(product_id, 'price')}")

    # 2. HINCRBY: Atomically increment a numeric field within the hash (e.g., stock quantity)
    v.hincrby(product_id, 'stock', -5) # 5 units sold
    print(f"Remaining Stock: {v.hget(product_id, 'stock')}")

    # 3. HGETALL: Get all fields and values of a hash
    print(f"All product data: {v.hgetall(product_id)}")

    # 4. HEXISTS: Check if a field exists
    print(f"Does 'description' field exist? {v.hexists(product_id, 'description')}")

    # 5. HDEL: Delete specific fields from a hash
    v.hset(product_id, 'deprecated_field', 'value_to_remove')
    print(f"Before HDEL: {v.hgetall(product_id)}")
    v.hdel(product_id, 'deprecated_field')
    print(f"After HDEL: {v.hgetall(product_id)}")
    ```


*   **Illustrative Use Cases:**
    *   **User Profiles:** Storing comprehensive user data (name, email, preferences, settings) where individual fields might be frequently updated or retrieved without fetching the entire object.
    *   **Product Catalogs/Inventory:** Storing detailed product attributes (name, price, stock, description) that can be partially retrieved or updated, making inventory management efficient.
    *   **Configuration Management:** Managing dynamic configurations for microservices or applications, allowing atomic updates of individual settings.

---

### **3. Lists**

*   **Concept:** Lists are **ordered collections of strings**, sorted by insertion order. They are implemented as a linked list, making additions and removals from both ends very fast (O(1)). This makes them perfect for building **queues (FIFO - First-In, First-Out)** or **stacks (LIFO - Last-In, First-Out)**. The atomic blocking capabilities (`BLPOP`/`BRPOP`) are crucial, enabling Valkey to function as a reliable message broker or lightweight task queue.


*   **Python Examples:**
    ```python
    print("\n--- Lists (as Queues/Stacks) ---")
    message_queue = 'notification:email_queue'
    # 1. LPUSH: Add elements to the HEAD (left) of the list (stack-like or for FIFO when consumed from right)
    v.lpush(message_queue, "Email: User registration complete")
    v.lpush(message_queue, "Email: Password reset initiated")

    # 2. RPUSH: Add elements to the TAIL (right) of the list (queue-like)
    v.rpush(message_queue, "Email: Order confirmation #123")
    v.rpush(message_queue, "Email: Newsletter subscription")

    # 3. LRANGE: Get a range of elements (0 to -1 means all elements)
    print(f"Current email queue: {v.lrange(message_queue, 0, -1)}")
    print(f"Email queue length: {v.llen(message_queue)}")

    # 4. LPOP: Remove and return an element from the HEAD (left)
    next_stack_item = v.lpop(message_queue)
    print(f"Processed (LPOP - Stack-like): {next_stack_item}")
    print(f"Remaining queue: {v.lrange(message_queue, 0, -1)}")

    # 5. RPOP: Remove and return an element from the TAIL (right)
    next_queue_item = v.rpop(message_queue)
    print(f"Processed (RPOP - Queue-like): {next_queue_item}")
    print(f"Remaining queue: {v.lrange(message_queue, 0, -1)}")

    # 6. BLPOP/BRPOP: Blocking Pop - Crucial for efficient background workers!
    # This will block if the list is empty until an item is added or timeout occurs.
    print("\n--- Demonstrating BLPOP (Blocking List Pop) ---")
    print("This will wait for 2 seconds for an item to appear in 'temp_queue_for_blpop'.")
    print("Try running `v.lpush('temp_queue_for_blpop', 'Hello Blocking!')` in another Python script or `docker exec -it valkey-dev valkey-cli LPUSH temp_queue_for_blpop 'Hello Blocking!'` in another terminal while this is blocking.")
    start_blpop = time.time()
    # BLPOP returns a tuple: (queue_name, item_data)
    blocking_item = v.blpop('temp_queue_for_blpop', timeout=2) # Wait for max 2 seconds
    if blocking_item:
        print(f"BLPOP received '{blocking_item[1]}' from '{blocking_item[0]}' after {time.time() - start_blpop:.2f}s")
    else:
        print(f"BLPOP timed out after {time.time() - start_blpop:.2f}s. No item received.")
    ```


*   **Illustrative Use Cases:**
    *   **Message Queues / Task Queues:** Implementing robust task queues for background jobs (e.g., sending emails, processing images, generating reports) where tasks are processed by a single worker.
    *   **Activity Feeds:** Storing recent user activities or notifications in chronological order (e.g., "User X commented on Post Y").
    *   **"Last N" Features:** Maintaining lists of the most recent items, such as the last 10 viewed products or latest news headlines.

---

### **4. Sets**

*   **Concept:** Sets are **unordered collections of unique strings**. They automatically handle duplicates. Valkey provides **fast (O(1)) operations** for adding members (`SADD`), removing members (`SREM`), and checking for the existence of a member (`SISMEMBER`). They are excellent for managing distinct items and performing mathematical set operations directly on the server.


*   **Python Examples:**
    ```python
    print("\n--- Sets ---")
    daily_unique_logins = 'app:unique_logins:2024-07-26'
    # 1. SADD: Add members to a set. Duplicates are automatically ignored.
    v.sadd(daily_unique_logins, 'user_alice', 'user_bob', 'user_alice', 'user_charlie', 'user_david')
    print(f"All unique logins today: {v.smembers(daily_unique_logins)}") # Returns a Python set of decoded strings

    # 2. SISMEMBER: Check for presence of a member (very fast and efficient)
    print(f"Is 'user_bob' logged in today? {v.sismember(daily_unique_logins, 'user_bob')}")
    print(f"Is 'user_eve' logged in today? {v.sismember(daily_unique_logins, 'user_eve')}")

    # 3. SREM: Remove a member (e.g., if a user logs out, or for cleanup)
    v.srem(daily_unique_logins, 'user_charlie')
    print(f"Logins after 'user_charlie' removal: {v.smembers(daily_unique_logins)}")

    # 4. SCARD: Get the number of members in a set (cardinality)
    print(f"Total unique logins today: {v.scard(daily_unique_logins)}")

    # 5. Set operations: Demonstrate conceptual power for commonalities/differences
    # Valkey supports powerful set operations directly on the server side:
    v.sadd('team_alpha_members', 'dev_1', 'dev_2', 'dev_3')
    v.sadd('project_beta_contributors', 'dev_2', 'dev_3', 'dev_4')
    print(f"Common members (Intersection): {v.sinter('team_alpha_members', 'project_beta_contributors')}")
    print(f"All distinct members (Union): {v.sunion('team_alpha_members', 'project_beta_contributors')}")
    print(f"Members in Alpha but not Beta (Difference): {v.sdiff('team_alpha_members', 'project_beta_contributors')}")
    ```


*   **Illustrative Use Cases:**
    *   **Tracking Unique Items:** Accurately counting unique visitors to a website, unique elements in a large dataset (e.g., unique IP addresses accessing a service).
    *   **Access Control / Permissions:** Storing users assigned to a specific role or feature flag.
    *   **Social Networking:** Efficiently managing followers/following lists, determining mutual friends, or storing user-defined tags on content.

---

### **5. Sorted Sets (ZSets)**

*   **Concept:** Sorted Sets are like Sets (collections of unique strings), but each member is associated with a **numerical `score` (a float)**. This score is used to maintain a global order. Members are unique, but scores can be duplicated. This data type is exceptionally useful for implementing **leaderboard-style functionalities**, real-time ranking, and time-series data where elements need to be retrieved by value range or rank.


*   **Python Examples:**
    ```python
    print("\n--- Sorted Sets (ZSets) ---")
    game_leaderboard = 'game:high_scores'
    # 1. ZADD: Add members with scores. If a member exists, its score is updated.
    v.zadd(game_leaderboard, mapping={'Alice': 1500, 'Bob': 1200, 'Charlie': 1800, 'David': 1500, 'Eve': 1650})

    # 2. ZREVRANGE: Get members by rank (descending score, highest first). '0' to '2' means top 3.
    # 'withscores=True' returns tuples of (member, score)
    print(f"Top 3 players (highest scores first): {v.zrevrange(game_leaderboard, 0, 2, withscores=True)}")

    # 3. ZRANGE: Get members by rank (ascending score, lowest first).
    print(f"Bottom 2 players (lowest scores first): {v.zrange(game_leaderboard, 0, 1, withscores=True)}")

    # 4. ZSCORE: Get the score of a specific member
    print(f"Alice's score: {v.zscore(game_leaderboard, 'Alice')}")

    # 5. ZINCRBY: Atomically increment a member's score (e.g., player gains points)
    v.zincrby(game_leaderboard, 250, 'Bob') # Bob's score becomes 1200 + 250 = 1450
    print(f"Bob's new score after playing: {v.zscore(game_leaderboard, 'Bob')}")

    # 6. ZRANGEBYSCORE: Get members within a specific score range
    print(f"Players with scores between 1400 and 1600: {v.zrangebyscore(game_leaderboard, 1400, 1600, withscores=True)}")

    # 7. ZRANK / ZREVRANK: Get the rank (position) of a member
    # ZRANK is 0-indexed, ascending order. ZREVRANK for descending.
    print(f"Charlie's rank (ascending, 0-indexed): {v.zrank(game_leaderboard, 'Charlie')}")
    print(f"Charlie's rank (descending, 0-indexed): {v.zrevrank(game_leaderboard, 'Charlie')}")
    ```


*   **Illustrative Use Cases:**
    *   **Leaderboards & Gaming:** Real-time ranking of players by score, retrieving top N players, or players within a certain score range.
    *   **Real-time Ranking:** Ranking of trending topics, popular products, or live statistics that need continuous updates and sorted retrieval.
    *   **Time-Series Data:** Storing events with timestamps as scores to query events within a specific time window (e.g., monitoring system logs, recent sensor readings).
    *   **Rate Limiting:** Implementing advanced sliding window rate limiters by storing timestamps of requests and counting/removing based on time.

---

**Module Conclusion**

You've just had comprehensive hands-on experience with Valkey's five core data structures. Notice how each is optimized for specific types of data and operations. This is Valkey's strength: it's not just a flat key-value store, but a powerful **data structure server** that makes complex data interactions simple and lightning-fast.

The atomic operations (like `INCR` for Strings, `HINCRBY` for Hashes, and `ZINCRBY` for Sorted Sets) are particularly important. They guarantee that your operations are safe even with multiple clients trying to modify the same data concurrently, which is vital in distributed enterprise environments.

While these five are crucial, Valkey also offers a wide array of more specialized data structures and functionalities, some natively, and some through powerful **Valkey Modules**. These include:

*   **Streams:** For persistent, high-throughput event logs and complex message queues (we'll cover this in Part 2!).
*   **Bitmaps & HyperLogLogs:** For highly memory-efficient tracking of unique items or boolean flags (also in Part 2!).
*   **Geospatial Indexes:** For location-based services (finding points within a radius, useful for delivery apps, social networks).
*   **Bitfields:** For managing multiple small counters within a single string key very efficiently.
*   **JSON:** Direct support for storing and querying JSON documents within Valkey.
*   **Probabilistic Data Structures via Modules:** Like **Bloom Filters** (for quickly checking if an item is *likely* present in a large set, avoiding database lookups) or **Count-Min Sketch** (for approximate frequency counts).
*   **Vector Search:** A cutting-edge capability for AI/ML workloads (semantic search, recommendation engines, image recognition) by storing and querying vector embeddings.

The key takeaway is that Valkey likely has a native solution or a module for almost any common data problem you'll encounter in enterprise applications. Next, we'll dive into how these core data structures are applied in real-world scenarios, starting with the ubiquitous use case of caching.

---
