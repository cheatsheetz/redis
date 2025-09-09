# Redis Cheat Sheet

Comprehensive reference for Redis in-memory data structure store - covering data structures, commands, persistence, clustering, and administration for production environments.

---

## Table of Contents
- [Installation & Connection](#installation--connection)
- [Data Structures & Commands](#data-structures--commands)
- [String Operations](#string-operations)
- [Hash Operations](#hash-operations)
- [List Operations](#list-operations)
- [Set Operations](#set-operations)
- [Sorted Set Operations](#sorted-set-operations)
- [Advanced Data Types](#advanced-data-types)
- [Key Management](#key-management)
- [Persistence](#persistence)
- [Replication](#replication)
- [Clustering](#clustering)
- [Pub/Sub](#pubsub)
- [Transactions](#transactions)
- [Lua Scripting](#lua-scripting)
- [Administration](#administration)
- [Security](#security)
- [Monitoring & Performance](#monitoring--performance)
- [Integration Patterns](#integration-patterns)

---

## Installation & Connection

### Installation
```bash
# Ubuntu/Debian
sudo apt update && sudo apt install redis-server

# CentOS/RHEL
sudo yum install redis

# macOS (Homebrew)
brew install redis

# Docker
docker run --name redis-container -d -p 6379:6379 redis:7-alpine

# Docker with persistence
docker run --name redis-container -d -p 6379:6379 -v redis-data:/data redis:7-alpine redis-server --appendonly yes
```

### Connection
```bash
# Redis CLI
redis-cli

# Connect to specific host/port
redis-cli -h hostname -p 6379

# Connect with authentication
redis-cli -h hostname -p 6379 -a password

# Connect to specific database
redis-cli -h hostname -p 6379 -n 2

# Execute single command
redis-cli SET mykey "Hello World"

# Batch mode
redis-cli --pipe < commands.txt

# Monitor mode
redis-cli MONITOR
```

### Connection Parameters
| Parameter | Description | Example |
|-----------|-------------|---------|
| `-h` | Hostname | `redis-cli -h localhost` |
| `-p` | Port | `redis-cli -p 6379` |
| `-a` | Password | `redis-cli -a mypassword` |
| `-n` | Database number | `redis-cli -n 1` |
| `--raw` | Raw output format | `redis-cli --raw` |
| `--latency` | Latency monitoring | `redis-cli --latency` |

---

## Data Structures & Commands

### Overview of Redis Data Types
| Data Type | Description | Use Cases |
|-----------|-------------|-----------|
| String | Binary-safe strings | Caching, counters, sessions |
| Hash | Field-value pairs | Objects, user profiles |
| List | Ordered collections | Queues, activity feeds, chat |
| Set | Unique unordered collections | Tags, unique visitors |
| Sorted Set | Ordered unique collections | Leaderboards, time series |
| Stream | Log-like data structure | Event sourcing, messaging |
| Bitmap | Bit arrays | Real-time analytics, bloom filters |
| HyperLogLog | Probabilistic data structure | Unique count estimation |

### Basic Commands
```bash
# Key operations
SET key value                    # Set string value
GET key                         # Get string value
DEL key                         # Delete key
EXISTS key                      # Check if key exists
TYPE key                        # Get data type
EXPIRE key seconds              # Set expiration
TTL key                         # Get time to live
KEYS pattern                    # Find keys (avoid in production)
SCAN cursor [MATCH pattern]     # Iterate keys efficiently

# Database operations
SELECT index                    # Switch database (0-15)
FLUSHDB                        # Clear current database
FLUSHALL                       # Clear all databases
DBSIZE                         # Number of keys in database

# Server operations
PING                           # Test connection
INFO                           # Server information
CONFIG GET parameter           # Get configuration
CONFIG SET parameter value     # Set configuration
```

---

## String Operations

### Basic String Commands
```bash
# Set and get
SET mykey "Hello World"
GET mykey

# Set with expiration
SETEX mykey 60 "expires in 60 seconds"
SET mykey "value" EX 60        # Same as above

# Set if not exists
SETNX mykey "value"
SET mykey "value" NX           # Same as above

# Set if exists
SET mykey "value" XX

# Get and set atomically
GETSET mykey "new value"

# Multiple operations
MSET key1 "value1" key2 "value2" key3 "value3"
MGET key1 key2 key3

# String length
STRLEN mykey

# Append to string
APPEND mykey " - appended"

# Get substring
GETRANGE mykey 0 4             # Get characters 0-4
SETRANGE mykey 6 "Redis"       # Replace from position 6
```

### Numeric Operations
```bash
# Increment/decrement
SET counter 10
INCR counter                   # Increment by 1 -> 11
INCRBY counter 5               # Increment by 5 -> 16
DECR counter                   # Decrement by 1 -> 15
DECRBY counter 3               # Decrement by 3 -> 12

# Float increment
SET price 10.50
INCRBYFLOAT price 2.25         # -> 12.75

# Bit operations
SETBIT bitmap 0 1              # Set bit 0 to 1
GETBIT bitmap 0                # Get bit 0
BITCOUNT bitmap                # Count set bits
BITOP AND result key1 key2     # Bitwise AND
```

### String Use Cases
```bash
# Session storage
SET "session:user123" '{"user_id": 123, "name": "John"}'
EXPIRE "session:user123" 3600

# Caching
SET "cache:user:123" '{"name": "John", "email": "john@example.com"}'
GET "cache:user:123"

# Rate limiting
SET "rate_limit:user123" 1
EXPIRE "rate_limit:user123" 60
INCR "rate_limit:user123"      # Will fail if > limit

# Counters
INCR "page_views:homepage"
INCRBY "downloads:file123" 1
```

---

## Hash Operations

### Hash Commands
```bash
# Set and get hash fields
HSET user:123 name "John Doe"
HSET user:123 email "john@example.com" age 30
HGET user:123 name

# Multiple field operations
HMSET user:123 name "John" email "john@example.com" age 30
HMGET user:123 name email age

# Get all fields and values
HGETALL user:123

# Check if field exists
HEXISTS user:123 name

# Delete field
HDEL user:123 age

# Get all fields or values
HKEYS user:123                 # Get all field names
HVALS user:123                 # Get all values

# Number of fields
HLEN user:123

# Increment hash field
HINCRBY user:123 login_count 1
HINCRBYFLOAT user:123 balance 10.50

# Set if field doesn't exist
HSETNX user:123 created_at "2023-01-01"

# Scan hash fields
HSCAN user:123 0 MATCH "email*"
```

### Hash Use Cases
```bash
# User profiles
HMSET user:123 name "John Doe" email "john@example.com" age 30 city "NYC"
HGET user:123 name

# Shopping cart
HSET cart:user123 product:1 "2"     # product_id: quantity
HSET cart:user123 product:5 "1"
HGETALL cart:user123

# Configuration settings
HMSET config:app debug "true" timeout "30" max_connections "100"
HGET config:app timeout

# Real-time analytics
HINCRBY stats:2023-01-01 page_views 1
HINCRBY stats:2023-01-01 unique_visitors 1
HGETALL stats:2023-01-01
```

---

## List Operations

### List Commands
```bash
# Add elements
LPUSH mylist "first"           # Add to beginning
RPUSH mylist "last"            # Add to end
LPUSH mylist "a" "b" "c"       # Add multiple

# Remove and get elements
LPOP mylist                    # Remove from beginning
RPOP mylist                    # Remove from end
BLPOP mylist 30                # Blocking pop (30s timeout)
BRPOP mylist 30                # Blocking pop from end

# Get elements
LINDEX mylist 0                # Get element at index
LRANGE mylist 0 -1             # Get range (0 to end)
LRANGE mylist 0 4              # Get first 5 elements
LLEN mylist                    # Get list length

# Modify elements
LSET mylist 0 "new value"      # Set element at index
LTRIM mylist 0 99              # Keep only elements 0-99

# Insert elements
LINSERT mylist BEFORE "pivot" "new"
LINSERT mylist AFTER "pivot" "new"

# Remove elements
LREM mylist 2 "value"          # Remove first 2 occurrences
LREM mylist -1 "value"         # Remove last occurrence
LREM mylist 0 "value"          # Remove all occurrences

# Block until element available
BRPOPLPUSH source dest 30      # Move element between lists
```

### List Use Cases
```bash
# Message queue
LPUSH queue:emails '{"to": "john@example.com", "subject": "Hello"}'
BRPOP queue:emails 30          # Worker processes messages

# Activity feed
LPUSH feed:user123 '{"action": "post", "content": "Hello World"}'
LRANGE feed:user123 0 9        # Get latest 10 activities

# Chat messages
LPUSH chat:room1 '{"user": "John", "message": "Hello!", "time": "2023-01-01T10:00:00Z"}'
LRANGE chat:room1 0 49         # Get latest 50 messages
LTRIM chat:room1 0 99          # Keep only latest 100 messages

# Undo/redo functionality
LPUSH undo:user123 "action_data"
LPOP undo:user123              # Undo last action

# Task queue with priority
LPUSH queue:high_priority "urgent_task"
RPUSH queue:low_priority "background_task"
```

---

## Set Operations

### Set Commands
```bash
# Add and remove members
SADD myset "member1" "member2" "member3"
SREM myset "member2"

# Check membership and get members
SISMEMBER myset "member1"      # Check if member exists
SMEMBERS myset                 # Get all members
SCARD myset                    # Get set size

# Random operations
SRANDMEMBER myset              # Get random member
SRANDMEMBER myset 3            # Get 3 random members
SPOP myset                     # Remove and return random member
SPOP myset 2                   # Remove and return 2 random members

# Set operations
SUNION set1 set2               # Union
SINTER set1 set2               # Intersection
SDIFF set1 set2                # Difference
SUNIONSTORE result set1 set2   # Store union result
SINTERSTORE result set1 set2   # Store intersection result
SDIFFSTORE result set1 set2    # Store difference result

# Move member between sets
SMOVE source dest "member"

# Scan set members
SSCAN myset 0 MATCH "pattern*"
```

### Set Use Cases
```bash
# Tags for articles
SADD article:123:tags "redis" "database" "nosql"
SADD article:456:tags "javascript" "nodejs" "database"
SINTER article:123:tags article:456:tags  # Common tags

# Online users
SADD online_users "user123" "user456" "user789"
SISMEMBER online_users "user123"      # Check if user is online
SCARD online_users                    # Count online users

# Unique visitors
SADD visitors:2023-01-01 "192.168.1.1" "10.0.0.1"
SCARD visitors:2023-01-01            # Count unique visitors

# Friend recommendations
SADD friends:john "alice" "bob" "charlie"
SADD friends:alice "bob" "david" "eve"
SDIFF friends:alice friends:john     # Alice's friends not John's friends

# Voting system
SADD votes:post123 "user1" "user2" "user3"
SCARD votes:post123                  # Vote count
SISMEMBER votes:post123 "user1"     # Check if user voted
```

---

## Sorted Set Operations

### Sorted Set Commands
```bash
# Add members with scores
ZADD leaderboard 100 "player1" 200 "player2" 150 "player3"
ZADD leaderboard 175 "player4"

# Get members by rank
ZRANGE leaderboard 0 -1                    # All members (ascending)
ZRANGE leaderboard 0 -1 WITHSCORES        # With scores
ZREVRANGE leaderboard 0 2                 # Top 3 (descending)
ZREVRANGE leaderboard 0 2 WITHSCORES

# Get members by score
ZRANGEBYSCORE leaderboard 100 200         # Score between 100-200
ZRANGEBYSCORE leaderboard "(100" "200"    # Exclusive range
ZRANGEBYSCORE leaderboard -inf +inf       # All members by score

# Count and rank
ZCARD leaderboard                          # Number of members
ZCOUNT leaderboard 100 200                # Count in score range
ZRANK leaderboard "player1"               # Rank of member (0-based)
ZREVRANK leaderboard "player1"            # Reverse rank
ZSCORE leaderboard "player1"              # Get score

# Remove members
ZREM leaderboard "player1"                # Remove by member
ZREMRANGEBYRANK leaderboard 0 2           # Remove by rank range
ZREMRANGEBYSCORE leaderboard 0 100        # Remove by score range

# Increment score
ZINCRBY leaderboard 10 "player1"          # Increase score by 10

# Set operations
ZUNIONSTORE result 2 set1 set2            # Union with score sum
ZINTERSTORE result 2 set1 set2            # Intersection with score sum

# Lexicographic operations (when scores are same)
ZRANGEBYLEX myset "[a" "[z"               # Range by lexicographic order

# Scan sorted set
ZSCAN leaderboard 0 MATCH "player*"
```

### Sorted Set Use Cases
```bash
# Leaderboard
ZADD leaderboard 1000 "player1" 1500 "player2" 800 "player3"
ZREVRANGE leaderboard 0 9 WITHSCORES     # Top 10 players

# Time-based data
ZADD timeline 1640995200 "event1" 1640995260 "event2"  # Unix timestamps
ZRANGEBYSCORE timeline 1640995200 1640995300           # Events in time range

# Priority queue
ZADD tasks 1 "low_priority_task" 5 "high_priority_task"
ZPOPMAX tasks                             # Get highest priority task

# Auto-expiring data
ZADD recent_activities $(date +%s) "user123_login"
ZREMRANGEBYSCORE recent_activities 0 $(($(date +%s) - 3600))  # Remove old activities

# Rate limiting with sliding window
ZADD rate_limit:user123 $(date +%s) $(uuidgen)
ZCOUNT rate_limit:user123 $(($(date +%s) - 3600)) +inf  # Count requests in last hour
ZREMRANGEBYSCORE rate_limit:user123 0 $(($(date +%s) - 3600))  # Clean old entries

# Geographic data (with geospatial commands)
GEOADD cities:usa -74.006 40.7128 "New York" -118.2437 34.0522 "Los Angeles"
GEODIST cities:usa "New York" "Los Angeles" km
GEORADIUS cities:usa -74.006 40.7128 100 km
```

---

## Advanced Data Types

### Streams
```bash
# Add entries to stream
XADD mystream * field1 value1 field2 value2
XADD mystream 1640995200000-0 sensor temp 23.5

# Read from stream
XREAD STREAMS mystream 0                   # Read all entries
XREAD COUNT 2 STREAMS mystream 0          # Read 2 entries
XREAD BLOCK 5000 STREAMS mystream $       # Block for new entries

# Consumer groups
XGROUP CREATE mystream mygroup $ MKSTREAM
XREADGROUP GROUP mygroup consumer1 COUNT 1 STREAMS mystream >

# Stream information
XLEN mystream                              # Number of entries
XINFO STREAM mystream                      # Stream info
XINFO GROUPS mystream                      # Consumer group info

# Acknowledge messages
XACK mystream mygroup 1640995200000-0

# Trim stream
XTRIM mystream MAXLEN 1000                 # Keep only 1000 entries
```

### Bitmaps
```bash
# Set bits
SETBIT bitmap 0 1                          # Set bit 0 to 1
SETBIT bitmap 7 1                          # Set bit 7 to 1
SETBIT bitmap 123 1                        # Set bit 123 to 1

# Get bits
GETBIT bitmap 0                            # Get bit 0

# Count set bits
BITCOUNT bitmap                            # Count all set bits
BITCOUNT bitmap 0 1                        # Count in byte range

# Bit operations
BITOP AND result bitmap1 bitmap2
BITOP OR result bitmap1 bitmap2
BITOP XOR result bitmap1 bitmap2
BITOP NOT result bitmap1

# Find first bit
BITPOS bitmap 1                            # First set bit
BITPOS bitmap 0                            # First unset bit
```

### HyperLogLog
```bash
# Add elements
PFADD hll element1 element2 element3

# Count unique elements (approximate)
PFCOUNT hll

# Merge HyperLogLogs
PFMERGE result hll1 hll2 hll3

# Use case: Unique visitor counting
PFADD visitors:2023-01-01 "192.168.1.1" "10.0.0.1" "192.168.1.2"
PFCOUNT visitors:2023-01-01               # Approximate unique count
```

---

## Key Management

### Key Operations
```bash
# Key existence and type
EXISTS key1 key2 key3             # Check multiple keys
TYPE mykey                        # Get data type

# Key expiration
EXPIRE mykey 60                   # Expire in 60 seconds
EXPIREAT mykey 1640995200         # Expire at timestamp
PEXPIRE mykey 60000               # Expire in 60000 milliseconds
TTL mykey                         # Time to live in seconds
PTTL mykey                        # Time to live in milliseconds
PERSIST mykey                     # Remove expiration

# Key patterns and scanning
KEYS pattern                      # Find keys (avoid in production)
SCAN 0 MATCH "user:*" COUNT 10   # Scan keys efficiently
SCAN 0 MATCH "session:*"

# Key renaming and moving
RENAME old_key new_key            # Rename key
RENAMENX old_key new_key          # Rename if new key doesn't exist
MOVE mykey 1                      # Move key to database 1

# Key serialization
DUMP mykey                        # Serialize key
RESTORE new_key 0 serialized_data # Restore serialized data

# Random key
RANDOMKEY                         # Get random key

# Key sorting
SORT mylist                       # Sort list values
SORT mylist DESC ALPHA LIMIT 0 5  # Complex sort with options
```

---

## Persistence

### RDB (Redis Database) Snapshots
```bash
# Configuration (redis.conf)
save 900 1      # Save if at least 1 key changed in 900 seconds
save 300 10     # Save if at least 10 keys changed in 300 seconds
save 60 10000   # Save if at least 10000 keys changed in 60 seconds

# Manual snapshot
BGSAVE                           # Background save
SAVE                             # Foreground save (blocks)
LASTSAVE                         # Last save timestamp

# RDB configuration
dbfilename dump.rdb              # RDB filename
dir /var/lib/redis               # Directory for RDB file
rdbcompression yes               # Compress RDB file
rdbchecksum yes                  # Checksum RDB file
```

### AOF (Append Only File)
```bash
# Configuration (redis.conf)
appendonly yes                   # Enable AOF
appendfilename "appendonly.aof"  # AOF filename
appendfsync everysec            # Sync every second (default)
# appendfsync always            # Sync every write (slow but safe)
# appendfsync no                # Let OS decide when to sync

# AOF rewrite
BGREWRITEAOF                    # Rewrite AOF in background
auto-aof-rewrite-percentage 100 # Rewrite when AOF is 100% larger
auto-aof-rewrite-min-size 64mb  # Minimum size before rewrite

# Check AOF integrity
redis-check-aof --fix appendonly.aof
```

### Backup Strategies
```bash
# RDB backup
cp /var/lib/redis/dump.rdb /backup/dump-$(date +%Y%m%d).rdb

# AOF backup
cp /var/lib/redis/appendonly.aof /backup/appendonly-$(date +%Y%m%d).aof

# Live backup using replication
redis-cli --rdb backup.rdb      # Stream RDB to file

# Point-in-time recovery
# 1. Stop Redis
# 2. Replace dump.rdb or appendonly.aof
# 3. Start Redis
```

---

## Replication

### Master-Slave Configuration
```bash
# Slave configuration (redis.conf)
replicaof 192.168.1.100 6379    # Master IP and port
masterauth password              # Master password
replica-read-only yes            # Read-only replica

# Runtime replication
REPLICAOF 192.168.1.100 6379    # Start replication
REPLICAOF NO ONE                 # Stop replication (promote to master)

# Replication info
INFO replication                 # Replication status
ROLE                            # Current role (master/slave)

# Master configuration
# bind 0.0.0.0                  # Accept connections from any IP
# requirepass password          # Password for slaves
```

### Monitoring Replication
```bash
# Master monitoring
INFO replication                 # Shows connected replicas
CLIENT LIST TYPE replica        # List replica connections

# Replica monitoring
INFO replication                # Shows master connection status
LASTSAVE                        # Last successful sync

# Replication offset
INFO replication | grep master_repl_offset  # Master offset
INFO replication | grep slave_repl_offset   # Slave offset
```

---

## Clustering

### Redis Cluster Setup
```bash
# Cluster configuration (redis.conf)
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000

# Create cluster (6 nodes: 3 masters, 3 slaves)
redis-cli --cluster create \
  192.168.1.100:7000 192.168.1.100:7001 192.168.1.100:7002 \
  192.168.1.101:7000 192.168.1.101:7001 192.168.1.101:7002 \
  --cluster-replicas 1

# Connect to cluster
redis-cli -c -h 192.168.1.100 -p 7000

# Cluster commands
CLUSTER NODES                   # Show cluster nodes
CLUSTER INFO                    # Cluster information
CLUSTER SLOTS                   # Show slot assignments

# Add/remove nodes
redis-cli --cluster add-node new-node:7000 existing-node:7000
redis-cli --cluster del-node cluster-node:7000 node-id

# Rebalance cluster
redis-cli --cluster rebalance cluster-node:7000

# Reshard cluster
redis-cli --cluster reshard cluster-node:7000
```

### Hash Tags
```bash
# Keys with same hash tag go to same slot
SET {user:123}:profile "data"
SET {user:123}:preferences "data"
SET {user:123}:settings "data"

# Multi-key operations work on same slot
MGET {user:123}:profile {user:123}:preferences
```

---

## Pub/Sub

### Basic Pub/Sub
```bash
# Subscribe to channels
SUBSCRIBE channel1 channel2      # Subscribe to specific channels
PSUBSCRIBE news:*               # Subscribe to pattern

# Publish messages
PUBLISH channel1 "Hello World"   # Publish to channel
PUBLISH news:sports "Goal!"      # Publish to pattern channel

# Check subscriptions
PUBSUB CHANNELS                 # List active channels
PUBSUB CHANNELS news:*          # List channels matching pattern
PUBSUB NUMSUB channel1          # Number of subscribers
PUBSUB NUMPAT                   # Number of pattern subscriptions

# Unsubscribe
UNSUBSCRIBE channel1            # Unsubscribe from specific channel
UNSUBSCRIBE                     # Unsubscribe from all channels
PUNSUBSCRIBE news:*             # Unsubscribe from pattern
```

### Stream-based Messaging
```bash
# Producer
XADD events * type "user_signup" user_id 123 email "john@example.com"

# Consumer group
XGROUP CREATE events processors $ MKSTREAM
XREADGROUP GROUP processors worker1 COUNT 1 STREAMS events >

# Process message and acknowledge
XACK events processors message_id

# Dead letter queue for failed messages
XCLAIM events processors worker2 3600000 message_id  # Claim abandoned message
```

---

## Transactions

### MULTI/EXEC Transactions
```bash
# Basic transaction
MULTI                           # Start transaction
SET key1 "value1"
SET key2 "value2"
INCR counter
EXEC                           # Execute all commands

# Discard transaction
MULTI
SET key1 "value1"
DISCARD                        # Cancel transaction

# Watch keys for changes
WATCH mykey
val = GET mykey
MULTI
SET mykey "new_value_based_on_old"
EXEC                           # Will fail if mykey changed

# Example: Transfer between accounts
WATCH account:A account:B
balance_A = GET account:A
balance_B = GET account:B
if balance_A >= amount:
    MULTI
    DECRBY account:A amount
    INCRBY account:B amount
    EXEC
else:
    UNWATCH
```

### Optimistic Locking Example
```bash
# Optimistic locking pattern
WATCH inventory:product123
current_stock = GET inventory:product123
if current_stock > 0:
    MULTI
    DECR inventory:product123
    EXEC
    # Success if EXEC returns list, null if failed
else:
    UNWATCH
```

---

## Lua Scripting

### Basic Lua Scripts
```bash
# Simple script
EVAL "return redis.call('set', KEYS[1], ARGV[1])" 1 mykey myvalue

# Script with logic
EVAL "
local current = redis.call('get', KEYS[1])
if current == false then
    return redis.call('set', KEYS[1], ARGV[1])
else
    return current
end
" 1 mykey myvalue

# Load and execute script
SCRIPT LOAD "return redis.call('get', KEYS[1])"  # Returns SHA
EVALSHA sha1 1 mykey                              # Execute by SHA

# Script management
SCRIPT EXISTS sha1 sha2                           # Check if scripts exist
SCRIPT FLUSH                                      # Remove all scripts
SCRIPT KILL                                       # Kill running script
```

### Advanced Lua Examples
```lua
-- Atomic counter with limit
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local current = tonumber(redis.call('get', key) or 0)

if current < limit then
    return redis.call('incr', key)
else
    return -1
end

-- Rate limiter (sliding window)
local key = KEYS[1]
local window = tonumber(ARGV[1])
local limit = tonumber(ARGV[2])
local now = tonumber(ARGV[3])

-- Remove old entries
redis.call('zremrangebyscore', key, 0, now - window)

-- Count current entries
local current = redis.call('zcard', key)

if current < limit then
    -- Add new entry
    redis.call('zadd', key, now, now)
    redis.call('expire', key, window)
    return 1
else
    return 0
end

-- Distributed lock
local key = KEYS[1]
local value = ARGV[1]
local ttl = ARGV[2]

if redis.call('set', key, value, 'NX', 'EX', ttl) then
    return 1
else
    return 0
end
```

---

## Administration

### Server Information
```bash
# Server stats
INFO                           # All information
INFO server                    # Server information
INFO memory                    # Memory usage
INFO clients                   # Client connections
INFO replication              # Replication status
INFO stats                    # Statistics

# Memory usage
MEMORY USAGE keyname          # Memory used by key
MEMORY STATS                  # Memory statistics
MEMORY DOCTOR                 # Memory optimization advice

# Client management
CLIENT LIST                   # List connected clients
CLIENT KILL ip:port          # Kill specific client
CLIENT KILL TYPE normal      # Kill all normal clients
CLIENT SETNAME myapp         # Set client name
CLIENT GETNAME               # Get client name

# Configuration
CONFIG GET parameter         # Get configuration
CONFIG SET parameter value   # Set configuration
CONFIG REWRITE              # Rewrite config file
```

### Performance Monitoring
```bash
# Latency monitoring
LATENCY LATEST              # Latest latency spikes
LATENCY HISTORY command     # Latency history for command
LATENCY RESET              # Reset latency data

# Slow log
SLOWLOG GET 10             # Get 10 slow queries
SLOWLOG LEN                # Slow log length
SLOWLOG RESET              # Clear slow log

# Monitor commands
MONITOR                    # Monitor all commands (debugging only)

# Statistics
INFO commandstats          # Command statistics
INFO keyspace             # Keyspace statistics
```

### Database Maintenance
```bash
# Memory management
MEMORY PURGE              # Free memory from lazy deletion
MEMORY MALLOC-STATS       # Memory allocator stats

# Background operations
BGREWRITEAOF             # Rewrite AOF file
BGSAVE                   # Background snapshot

# Debug commands
DEBUG OBJECT keyname     # Internal representation of key
DEBUG SEGFAULT          # Crash server (testing only)

# Keyspace operations
FLUSHDB                 # Clear current database
FLUSHALL                # Clear all databases
DBSIZE                  # Number of keys in database
RANDOMKEY               # Get random key
```

---

## Security

### Authentication
```bash
# Configuration (redis.conf)
requirepass mypassword          # Set password
# bind 127.0.0.1 192.168.1.100 # Bind to specific IPs

# Authentication
AUTH password                   # Authenticate
AUTH username password          # Authenticate with username (Redis 6+)

# User management (Redis 6+)
ACL SETUSER alice on >password ~* +@all    # Create user
ACL DELUSER alice                           # Delete user
ACL LIST                                    # List users
ACL USERS                                   # List usernames
ACL WHOAMI                                  # Current user
```

### Access Control Lists (ACL) - Redis 6+
```bash
# User creation with permissions
ACL SETUSER app_user on >app_password ~app:* +get +set +del
ACL SETUSER readonly_user on >read_password ~* +@read
ACL SETUSER admin_user on >admin_password ~* +@all

# ACL patterns
# ~pattern: Key patterns user can access
# +command: Commands user can execute  
# +@category: Command categories (@read, @write, @admin, etc.)
# -command: Deny specific command

# Load ACL from file
ACL LOAD                        # Load from redis.conf
ACL SAVE                        # Save current ACL to file

# Check permissions
ACL DRYRUN username command key # Test if user can run command
```

### SSL/TLS Configuration
```bash
# Configuration (redis.conf)
port 0                          # Disable non-TLS port
tls-port 6380                   # TLS port
tls-cert-file redis.crt         # Certificate file
tls-key-file redis.key          # Private key file
tls-ca-cert-file ca.crt         # CA certificate

# Connect with TLS
redis-cli --tls --cert redis.crt --key redis.key --cacert ca.crt
```

### Network Security
```bash
# Bind to specific interfaces
bind 127.0.0.1 192.168.1.100   # Only allow local and private network

# Protected mode
protected-mode yes              # Require AUTH if no bind or password

# Rename dangerous commands
rename-command FLUSHDB ""       # Disable command
rename-command FLUSHALL SECRET_FLUSHALL  # Rename command
rename-command CONFIG SECRET_CONFIG

# Disable commands
rename-command DEBUG ""
rename-command EVAL ""
```

---

## Monitoring & Performance

### Performance Metrics
```bash
# Key performance indicators
INFO stats | grep -E "instantaneous_ops_per_sec|used_memory_human"
INFO clients | grep connected_clients
INFO replication | grep master_repl_offset

# Redis Benchmark
redis-benchmark -h localhost -p 6379 -n 100000 -c 50
redis-benchmark -t set,get -n 100000 -q  # Quick benchmark

# Memory analysis
MEMORY USAGE key                # Memory used by specific key
INFO memory | grep used_memory  # Total memory usage
```

### Monitoring Tools
```bash
# Built-in monitoring
redis-cli --stat              # Real-time stats
redis-cli --latency          # Latency monitoring  
redis-cli --latency-history  # Latency over time
redis-cli --latency-dist     # Latency distribution

# Log analysis
tail -f /var/log/redis/redis-server.log

# System monitoring
htop                         # CPU and memory usage
iotop                       # I/O monitoring
```

### Performance Tuning
```bash
# Memory optimization (redis.conf)
maxmemory 2gb                       # Set memory limit
maxmemory-policy allkeys-lru        # Eviction policy
hash-max-ziplist-entries 512       # Optimize small hashes
list-max-ziplist-size -2           # Optimize small lists

# Network optimization
tcp-keepalive 60                    # TCP keepalive
timeout 300                         # Client timeout

# Persistence optimization
save ""                             # Disable RDB if using AOF
appendfsync everysec               # AOF sync policy
no-appendfsync-on-rewrite yes      # Don't sync during rewrite

# Slow log configuration
slowlog-log-slower-than 10000     # Log commands slower than 10ms
slowlog-max-len 128               # Keep 128 slow commands
```

### Common Performance Issues
```bash
# Large key detection
redis-cli --bigkeys

# Memory fragmentation
INFO memory | grep mem_fragmentation_ratio

# Slow commands
SLOWLOG GET 10

# CPU usage by command
INFO commandstats

# Key expiration issues
INFO keyspace | grep expires
```

---

## Integration Patterns

### Connection Pooling (Node.js)
```javascript
const redis = require('redis');

// Connection pool configuration
const client = redis.createClient({
    host: 'localhost',
    port: 6379,
    password: 'password',
    db: 0,
    retry_strategy: (options) => {
        if (options.error && options.error.code === 'ECONNREFUSED') {
            return new Error('The server refused the connection');
        }
        if (options.total_retry_time > 1000 * 60 * 60) {
            return new Error('Retry time exhausted');
        }
        if (options.attempt > 10) {
            return undefined;
        }
        return Math.min(options.attempt * 100, 3000);
    }
});

// Caching pattern
async function getUser(userId) {
    const cacheKey = `user:${userId}`;
    
    // Try cache first
    const cached = await client.get(cacheKey);
    if (cached) {
        return JSON.parse(cached);
    }
    
    // Fetch from database
    const user = await database.getUser(userId);
    
    // Cache for 1 hour
    await client.setex(cacheKey, 3600, JSON.stringify(user));
    
    return user;
}

// Session storage
async function createSession(userId, sessionData) {
    const sessionId = generateUniqueId();
    const sessionKey = `session:${sessionId}`;
    
    await client.hmset(sessionKey, sessionData);
    await client.expire(sessionKey, 3600); // 1 hour
    
    return sessionId;
}

// Rate limiting
async function checkRateLimit(userId, limit = 100, window = 3600) {
    const key = `rate_limit:${userId}`;
    const current = await client.incr(key);
    
    if (current === 1) {
        await client.expire(key, window);
    }
    
    return current <= limit;
}
```

### Python Integration (redis-py)
```python
import redis
import json
from datetime import datetime, timedelta

# Connection pool
pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    password='password',
    db=0,
    decode_responses=True,
    max_connections=20
)

r = redis.Redis(connection_pool=pool)

# Caching decorator
def cache_result(expiration=3600):
    def decorator(func):
        def wrapper(*args, **kwargs):
            cache_key = f"{func.__name__}:{hash(str(args) + str(kwargs))}"
            
            # Try to get from cache
            cached = r.get(cache_key)
            if cached:
                return json.loads(cached)
            
            # Execute function and cache result
            result = func(*args, **kwargs)
            r.setex(cache_key, expiration, json.dumps(result))
            
            return result
        return wrapper
    return decorator

@cache_result(expiration=1800)  # 30 minutes
def get_expensive_calculation(param1, param2):
    # Expensive operation here
    return {"result": param1 * param2}

# Distributed lock
class DistributedLock:
    def __init__(self, redis_client, key, timeout=10):
        self.redis = redis_client
        self.key = f"lock:{key}"
        self.timeout = timeout
        self.identifier = str(uuid.uuid4())
    
    def acquire(self):
        end_time = time.time() + self.timeout
        while time.time() < end_time:
            if self.redis.set(self.key, self.identifier, nx=True, ex=self.timeout):
                return True
            time.sleep(0.001)
        return False
    
    def release(self):
        lua_script = """
        if redis.call("get", KEYS[1]) == ARGV[1] then
            return redis.call("del", KEYS[1])
        else
            return 0
        end
        """
        return self.redis.eval(lua_script, 1, self.key, self.identifier)

# Usage
with DistributedLock(r, 'critical_section'):
    # Critical section code
    pass

# Queue implementation
class RedisQueue:
    def __init__(self, redis_client, name):
        self.redis = redis_client
        self.name = name
    
    def put(self, item):
        self.redis.lpush(self.name, json.dumps(item))
    
    def get(self, block=True, timeout=None):
        if block:
            item = self.redis.brpop(self.name, timeout=timeout)
            return json.loads(item[1]) if item else None
        else:
            item = self.redis.rpop(self.name)
            return json.loads(item) if item else None
    
    def size(self):
        return self.redis.llen(self.name)

# Usage
queue = RedisQueue(r, 'tasks')
queue.put({'task': 'process_data', 'params': {'file': 'data.csv'}})
task = queue.get()  # Blocks until task available
```

### Java Integration (Jedis)
```java
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;
import redis.clients.jedis.Transaction;

public class RedisClient {
    private JedisPool jedisPool;
    
    public RedisClient() {
        JedisPoolConfig config = new JedisPoolConfig();
        config.setMaxTotal(20);
        config.setMaxIdle(10);
        config.setMinIdle(5);
        config.setTestOnBorrow(true);
        
        jedisPool = new JedisPool(config, "localhost", 6379, 2000, "password");
    }
    
    // Caching
    public String getCachedData(String key) {
        try (Jedis jedis = jedisPool.getResource()) {
            String cached = jedis.get(key);
            if (cached != null) {
                return cached;
            }
            
            // Fetch from source
            String data = fetchFromDatabase(key);
            
            // Cache for 1 hour
            jedis.setex(key, 3600, data);
            
            return data;
        }
    }
    
    // Transaction example
    public boolean transferFunds(String fromAccount, String toAccount, double amount) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.watch(fromAccount, toAccount);
            
            double fromBalance = Double.parseDouble(jedis.get(fromAccount));
            if (fromBalance < amount) {
                jedis.unwatch();
                return false;
            }
            
            Transaction multi = jedis.multi();
            multi.decrByFloat(fromAccount, amount);
            multi.incrByFloat(toAccount, amount);
            
            List<Object> results = multi.exec();
            return results != null; // null if transaction failed
        }
    }
    
    // Pub/Sub
    public void subscribeToChannel(String channel, JedisPubSub subscriber) {
        try (Jedis jedis = jedisPool.getResource()) {
            jedis.subscribe(subscriber, channel);
        }
    }
}
```

---

## Performance Tuning Tips

1. **Use appropriate data structures**: Choose the right data type for your use case
2. **Optimize memory usage**: Use hash-max-ziplist-entries and similar settings
3. **Pipeline commands**: Group multiple commands to reduce network roundtrips
4. **Use connection pooling**: Reuse connections instead of creating new ones
5. **Monitor slow queries**: Use SLOWLOG to identify problematic commands
6. **Avoid KEYS command**: Use SCAN for key iteration in production
7. **Set appropriate timeouts**: Configure client and server timeouts
8. **Use read replicas**: Distribute read load across multiple instances
9. **Consider persistence trade-offs**: Choose between RDB and AOF based on needs
10. **Monitor memory fragmentation**: Restart if fragmentation ratio is high

---

## Resources
- [Redis Official Documentation](https://redis.io/documentation)
- [Redis Commands Reference](https://redis.io/commands)
- [Redis Best Practices](https://redis.io/topics/memory-optimization)
- [RedisInsight](https://redis.com/redis-enterprise/redis-insight/) - GUI management tool
- [Redis CLI Reference](https://redis.io/topics/rediscli)
- [Redis Persistence Guide](https://redis.io/topics/persistence)

---
*Comprehensive Redis reference for in-memory data structure operations. Contributions welcome!*