**Module 4: Use Case 2: Real-Time Pub/Sub**

**Goal:** Understand the Publish/Subscribe messaging model, how Valkey facilitates efficient real-time communication, and its key characteristics and limitations.

**Content:**

1. **Understanding Pub/Sub**
   
   - **Concept:** The Publish/Subscribe (Pub/Sub) messaging paradigm is a powerful pattern that allows publishers to send messages to designated "channels" without needing to know which receivers (subscribers) exist. Conversely, subscribers express interest in one or more channels and only receive messages relevant to their interests, without needing to know who the publishers are.
   - **Key Benefits:**
     - **Decoupling:** Publishers and subscribers operate independently, simplifying application design and allowing different components or microservices to evolve separately.
     - **Real-time Capabilities & Low Latency:** Valkey's in-memory nature makes its Pub/Sub system exceptionally performant. Messages are processed and delivered rapidly, enabling real-time features like live feeds, instant notifications, and dynamic dashboards.
     - **Scalability:** The decoupled nature and Valkey's efficient handling of connections contribute to scalability. (For advanced scenarios in cluster mode, Valkey also offers **Sharded Pub/Sub**, allowing messages to be restricted to a specific shard, reducing cluster bus traffic.)
   - **Components:**
     - **Publishers:** Send messages to a channel.
     - **Subscribers:** Listen for messages on one or more channels.
     - **Channels:** Named message queues that publishers send to and subscribers listen on.
   - **Subscription Flexibility:**
     - **`SUBSCRIBE channel [channel2 ...]`:** Listen for messages on specific, named channels.
     - **`PSUBSCRIBE pattern [pattern2 ...]`:** Listen for messages on channels matching a glob-style pattern (e.g., `news:*` matches `news:sports`, `news:finance`).
   - **Important Characteristics & Limitations:**
     - **Fire-and-Forget / At-Most-Once Delivery:** This is critical! Valkey Pub/Sub messages are **not persisted**. If no subscriber is connected to a channel when a message is published, that message is lost forever. If a subscriber disconnects and reconnects, it will *not* receive messages published while it was offline.
     - **No Relation to Key Space/Databases:** Pub/Sub operates globally across the Valkey server. A message published on `DB0` can be heard by a subscriber connected to `DB1`. If you need to scope channels (e.g., for different environments), you should use channel name prefixes (like `dev:chat`, `prod:chat`).
     - **Client Behavior in Subscribed State:** When a client enters Pub/Sub mode (via `SUBSCRIBE` or `PSUBSCRIBE`), it dedicates that connection to receiving messages. Historically (in RESP2 protocol), this connection could only issue Pub/Sub related commands (`UNSUBSCRIBE`, `PING`, `QUIT`). With the newer **RESP3 protocol**, this restriction is generally lifted, allowing clients to send regular commands on the same connection.
     - **Message Format:** Messages received by a subscriber are typically an array with three elements: `[type, channel_name, message_payload]`. For pattern-matching subscriptions, it's `[type, pattern_matched, channel_name, message_payload]`.

2. **Practical Lab: Live Notification System**
   
   - We'll use two separate Python scripts to demonstrate this interaction in real-time.
   
   **Script 1: `subscriber.py` (Run this in *Terminal 1*)**
   
   ```python
   # subscriber.py
   import valkey
   import time
   
   # Assumes 'v' connection object is available from a shared setup or recreated:
   # v = valkey.Valkey(host='localhost', port=6379, decode_responses=True)
   # Basic connection check for standalone script:
   try:
   v_pubsub_conn = valkey.Valkey(host='localhost', port=6379, decode_responses=True)
   if not v_pubsub_conn.ping():
       print("Failed to connect to Valkey. Exiting subscriber.")
       exit()
   except valkey.exceptions.ConnectionError as e:
   print(f"Could not connect to Valkey: {e}. Is your Docker container running?")
   exit()
   
   pubsub = v_pubsub_conn.pubsub()
   
   # Subscribe to specific channels
   pubsub.subscribe('system_notifications', 'chat:general')
   
   # You can also subscribe using glob-style patterns.
   # A single client can subscribe to both specific channels and patterns.
   pubsub.psubscribe('alerts:*') # Matches 'alerts:disk_space', 'alerts:cpu_load', etc.
   
   print("Listening for messages on channels: 'system_notifications', 'chat:general'")
   print("And on pattern: 'alerts:*'")
   print("Press Ctrl+C to stop.")
   
   try:
   for message in pubsub.listen():
       # The first messages received are confirmations of subscription.
       # 'type' can be 'subscribe', 'psubscribe', 'message', 'pmessage', 'unsubscribe', 'punsubscribe'.
       if message['type'] == 'subscribe' or message['type'] == 'psubscribe':
           channel_or_pattern = message.get('channel') or message.get('pattern')
           print(f"CONFIRMED SUBSCRIPTION: Type='{message['type']}', Target='{channel_or_pattern}', Active Subscriptions={message['data']}")
           continue # Skip processing confirmation messages as data
   
       # Process actual messages
       message_type = message['type']
       data = message['data']
   
       if message_type == 'message':
           # Message received from a direct channel subscription
           channel_name = message['channel']
           print(f"--> DIRECT MESSAGE: [Channel: {channel_name}] Data: '{data}'")
       elif message_type == 'pmessage':
           # Message received from a pattern subscription
           pattern_matched = message['pattern']
           originating_channel = message['channel']
           print(f"--> PATTERN MESSAGE: [Pattern: {pattern_matched}] [Channel: {originating_channel}] Data: '{data}'")
       else:
           # Handle unsubscribe or other message types if needed
           print(f"OTHER MESSAGE TYPE: {message}")
   
   except KeyboardInterrupt:
   print("\nStopping subscriber.")
   pubsub.unsubscribe() # Unsubscribe from all subscribed channels
   pubsub.punsubscribe() # Unsubscribe from all subscribed patterns
   pubsub.close() # Close the Pub/Sub connection
   v_pubsub_conn.close() # Close the underlying Valkey client connection
   except Exception as e:
   print(f"An unexpected error occurred in subscriber: {e}")
   pubsub.close()
   v_pubsub_conn.close()
   ```
   
   **Script 2: `publisher.py` (Run this in a *second* terminal)**
   
   ```python
   # publisher.py
   import valkey
   import time
   import random
   import json # For structured messages
   
   # Assumes 'v' connection object is available or recreated:
   # v = valkey.Valkey(host='localhost', port=6379, decode_responses=True)
   # Basic connection check for standalone script:
   try:
   v_publish_conn = valkey.Valkey(host='localhost', port=6379, decode_responses=True)
   if not v_publish_conn.ping():
       print("Failed to connect to Valkey. Exiting publisher.")
       exit()
   except valkey.exceptions.ConnectionError as e:
   print(f"Could not connect to Valkey: {e}. Is your Docker container running?")
   exit()
   
   print("Starting publisher...")
   messages_to_send = [
   {"channel": "system_notifications", "data": "ALERT: Disk usage is 90%!"},
   {"channel": "chat:general", "data": "Hey team, project update at 3 PM!"},
   {"channel": "system_notifications", "data": "INFO: Database backup completed."},
   {"channel": "alerts:cpu_load", "data": json.dumps({"level": "WARN", "value": "85%", "timestamp": time.time()})},
   {"channel": "chat:general", "data": "Anyone free for a quick sync?"},
   {"channel": "alerts:disk_space", "data": "CRITICAL: Service X storage is full!"}
   ]
   
   for msg_info in messages_to_send:
   channel = msg_info["channel"]
   data_payload = msg_info["data"]
   print(f"Publishing to '{channel}': '{data_payload}'")
   # PUBLISH returns the number of clients that received the message
   num_receivers = v_publish_conn.publish(channel, data_payload)
   print(f"Message published to '{channel}' and received by {num_receivers} subscriber(s).")
   time.sleep(random.uniform(0.5, 1.5)) # Simulate irregular publishing
   
   print("Finished publishing messages.")
   v_publish_conn.close() # Close the Valkey client connection
   ```
   
   - **Instructions (Live Demo):**
     1. Open **two separate terminal windows**.
     2. In Terminal 1, run `python subscriber.py`. Observe the initial subscription confirmation messages.
     3. In Terminal 2, run `python publisher.py`.
     4. Observe the real-time message flow appearing in the subscriber's terminal. Note how `alerts:*` pattern catches messages on specific alert channels.
     5. (Optional) Stop the subscriber (`Ctrl+C`), publish a few more messages from the publisher, then restart the subscriber. Point out that the messages published while the subscriber was offline are *not* received.

3. **Enterprise Use Cases for Pub/Sub**
   
   - **Real-time Dashboards:** Pushing instant updates to monitoring or analytics dashboards.
   - **Chat Applications & Social Media Feeds:** Enabling live message streams and content updates (e.g., live comments, social feeds).
   - **Event Notifications (Microservices):** Notifying other microservices about events (e.g., `order:created`, `user:registered`) for reactive architectures.
   - **IoT Data Streaming:** Broadcasting sensor data from connected devices for immediate processing or display.
   - **Live Game Updates:** Notifying players about game state changes, scores, or in-game events.
   - **Inter-Server Communication:** Facilitating low-latency communication between different application servers or background processes.
   - **Live Updates:** Generic applications requiring real-time updates like live inventory updates, personalized recommendations, or pricing adjustments.
