Formatted version of this page is available online: http://squizzle.me/php/predis/doc/Commands

[Index](README.md) • [Classes](Classes.md) • **Commands** • [Configuration](Configuration.md)

# Commands

This section lists all existing Predis commands. Some might not be supported by all [server profiles](Configuration.md#profile).

```PHP
$predis->append('skey', 'tail');
```

## Common call forms

### Variadic arguments

If a function is given exactly 2 arguments and the second is an array then it is flattened:
```PHP
hdel('hashkey', ['hash2', 'hash3todel']);
hdel('hashkey', 'hash2', 'hash3todel');
```

### List arguments

If a function is given exactly 1 argument and that's an array, its values are treated as if they were given to the function as arguments.
```PHP
pfcount(['key1', 'key2']);
pfcount('key1', 'key2');
```


## No special processing

These commands accept the same arguments you would give to Redis over `redis-cli` and they return raw Redis response.

Their arguments and responses are documented here for the sake of convenience; they must match Redis documentation exactly.

### append($key, $value)

Append a value to a key.

### auth($password)

Authenticate to the server.

### bitcount($key[, int $start, int $end])

Count set bits in a string.

### bitpos($key, int $bit[, int $start][, int $end])

Find first bit set or clear in a string.

### bitfield

**Undocumented in Redis.**

### brpoplpush($src, $dest, int $timeout)

Pop a value from a list, push it to another list and return it; or block until one is available.

### command($subcommand[, $arg[...]])

Execute a Redis command-related task.

### dbsize()

Return the number of keys in the selected database.

### decr($key)

Decrement the integer value of a key by one.

### decrby($key, $decrement)

Decrement the integer value of a key by the given number `$decrement`.

### discard()

Discard all commands issued after MULTI.

### dump($key)

Return a serialized version of the value stored at the specified key.

### echo($message)

Echo the given string.

### eval($script, $numKeys[, $key[...]][, $arg[...]])

Execute a Lua script server-side.

### evalsha($sha1, $numKeys[, $key[...]][, $arg[...]])

Execute a Lua script server-side by its SHA-1 hash.

### exec()

Execute all commands issued after MULTI.

### exists($key[, $key[...]])

Determine if a key exists.

### expire($key, int $seconds)

Set a key's time to live in seconds.

### expireat($key, int $timestamp)

Set a key's time to live in seconds.

### flushall()

Remove all keys from all databases.

### flushdb()

Remove all keys from the current database.

### geodist($key, $member1, $member2[, $unit])

Returns the distance between two members of a geospatial index.

### get($key)

Get the value of a key.

### getbit($key, int $offset)

Returns the bit value at offset in the string value stored at key.

### getrange($key, int $start, int $end)

Get a substring of the string stored at a key.

### getset($key, $value)

Set the string value of a key and return its old value.

### hexists($key, $field)

Determine if a hash field exists.

### hget($key, $field)

Get the value of a hash field.

### hincrby($key, $field, int $increment)

Increment the integer value of a hash field by the given number (`$increment`).

### hincrbyfloat($key, $field, int $increment)

Increment the float value of a hash field by the given amount (`$increment`).

### hkeys($key)

Get all the fields in a hash.

### hlen($key)

Get the number of fields in a hash.

### hset($key, $field, $value)

Set the string value of a hash field.

### hsetnx($key, $field, $value)

Set the value of a hash field, only if the field does not exist.

### hstrlen($key, $field)

Get the length of the value of a hash field.

### hvals($key)

Get all the values in a hash.

### incr($key)

Increment the integer value of a key by one.

### incrby($key, int $increment)

Increment the integer value of a key by the given amount (`$increment`).

### incrbyfloat($key, int $increment)

Increment the float value of a key by the given amount (`$increment`).

### keys($pattern)

Find all keys matching the given pattern.

### lastsave()

Get the UNIX time stamp of the last successful save to disk.

### lindex($key, int $index)

Get an element from a list by its index.

### linsert($key, $place, int $pivot, $value)

Insert an element before or after another element in a list. `$place` is either `BEFORE` or `AFTER`.

### llen($key)

Get the length of a list.

### lpop($key)

Remove and get the first element in a list.

### lpushx($key, $value)

Prepend a value to a list, only if the list exists.

### lrange($key, int $start, int $stop)

Get a range of elements from a list.

### lrem($key, int $count, $value)

Remove elements from a list.

### lset($key, int $index, $value)

Set the value of an element in a list by its index.

### ltrim($key, int $start, int $stop)

Trim a list to the specified range.

### monitor()

Listen for all requests received by the server in real time.

### move($key, int $db)

Move a key to another database.

### multi()

Mark the start of a transaction block.

### object($subcommand[, $arg[...]])

Inspect the internals of Redis objects.

### persist($key)

Remove the expiration from a key.

### pexpire($key, int $ms)

Set a key's time to live in milliseconds.

### pexpireat($key, int $msTimestamp)

Set the expiration for a key as a UNIX timestamp specified in milliseconds.

### ping()
Ping the server.

### psetex($key, int $ms, $value)

Set the value and expiration in milliseconds of a key.

### pttl($key)

Get the time to live for a key in milliseconds.

### publish($channel, $message)

Post a message to a channel.

### quit()

Close the connection.

### rename($key, $newKey)

Rename a key.

### renamenx($key, $newKey)

Rename a key, only if the new key does not exist.

### restore($key, int $ttl, $serializedValue[, 'REPLACE'])

Create a key using the provided serialized value, previously obtained using `DUMP`.

### rpop($key)

Remove and get the last element in a list.

### rpoplpush($src, $dest)

Remove the last element in a list, prepend it to another list and return it.

### rpushx($key, $value)

Append a value to a list, only if the list exists.

### save()

Synchronously save the dataset to disk.

### scard($key)

Get the number of members in a set.

### script($subcommand[, $arg[...]])

Execute a Lua server-side script-related task.

### select(int $index)

Change the selected database for the current connection.

### set($key, $value[, $option[...]])

Set the string value of a key. Options are given inline and can be:

* `'EX', seconds`
* `'PX', ms`
* `'NX'` or `'XX'`

### setbit($key, int $offset, int value)

Sets or clears the bit at offset in the string value stored at key.

### setex($key, int $seconds, $value)

Set the value and expiration of a key.

### setnx($key, $value)

Set the value of a key, only if the key does not exist.

### setrange($key, int $offset, $value)

Overwrite part of a string at key starting at the specified offset.

### shutdown([$mode])

Synchronously save the dataset to disk and then shut down the server. `$mode` can be either `NOSAVE` or `SAVE`.

### sismember($key, $member)

Determine if a given value is a member of a set.

### smembers($key)

Get all the members in a set.

### smove($src, $dest, $member)

Move a member from one set to another.

### spop($key[, int $count])

Remove and return one or multiple random members from a set.

### srandmember($key[, int $count])

Get one or multiple random members from a set.

### strlen($key)

Get the length of the value stored in a key.

### substr

**Undocumented in Redis.**

### time()

Return the current server time.

### ttl($key)

Get the time to live for a key.

### type($key)

Determine the type stored at key.

### unwatch()

Forget about all watched keys.

### zcard($key)

Get the number of members in a sorted set.

### zcount($key, int $min, int $max)

Count the members in a sorted set with scores within the given values.

### zincrby($key, int $increment, $member)

Increment the score of a member in a sorted set by `$increment`.

### zlexcount($key, int $min, int $max)

Count the number of members in a sorted set between a given lexicographical range.

### zremrangebylex($key, int $min, int $max)

Remove all members in a sorted set between the given lexicographical range.

### zremrangebyrank($key, int $min, int $max)

Remove all members in a sorted set within the given indexes.

### zremrangebyscore($key, int $min, int $max)

Remove all members in a sorted set within the given scores.

### zrevrank($key, $member)

Determine the index of a member in a sorted set, with scores ordered from high to low.

### zscore($key, $member)

Get the score associated with the given member in a sorted set.

### zsetrank

**Undocumented in Redis.**


## With special processing

These commands have custom arguments and/or process Redis response in a special way.

Most of the time Predis will let you call these as using native Redis syntax, e.g. giving `SORT` options directly as function arguments - however, using custom syntax is typically more convenient and natural:

```PHP
sort('keytosort', 'LIMIT', 20, 5, 'ALPHA');
sort('keytosort', ['LIMIT' => [20, 5], 'ALPHA' => true]);
```

### geoadd($key, float $long, float $lat, $member[, float $long, float $lat, $member[...]])
### geoadd($key, array $memberLongLat)

Add one or more geospatial items in the geospatial index represented using a sorted set.

Members and their long/lat can be given inline (first form) or as a single array with each member and its into in a subarray (second form):
```PHP
geoadd('geokey', 0.1, 0.1, 'm', 0.2, 0.2, 'm2');
geoadd('geokey', [ [0.1, 0.1, 'm'], [0.2, 0.2, 'm2'] ]);
```

### geohash($key, $member[, $member[...]])
### geohash($key, array $members)

Returns members of a geospatial index as standard geohash strings.

Members can be given inline (first form) or as a single array (second form):
```PHP
geohash('geokey', 'm', 'm2');
geohash('geokey', ['m', 'm2']);
```

### geopos($key, $member[, $member[...]])
### geopos($key, array $members)

Returns longitude and latitude of members of a geospatial index.

Members can be given inline (first form) or as a single array (second form):
```PHP
geopos('geokey', 'm', 'm2');
geopos('geokey', ['m', 'm2']);
```

### georadius($key, float $long, float $lat, float $radius, $unit[, $option[...]])
### georadius($key, float $long, float $lat, float $radius, $unit, array $options)

Query a sorted set representing a geospatial index to fetch members matching a given maximum distance from a point. `$unit` is one of: `m`, `km`, `ft`, `mi`.

Options can be given inline (first form) or as an array (second form).

Array `$options` accepts optional keys (key names are case-insensitive):

* WITHCOORD - boolean
* WITHDIST - boolean
* WITHHASH - boolean
* COUNT - integer
* SORT - string (case-insensitvie)
* STORE - string
* STOREDIST - string

Inline options are not normalized.

```PHP
georadius('k', 0.1, 0.1, 0.1, 'COUNT', 10);
georadius('k', 0.1, 0.1, 0.1, ['COUNT' => 10]);
```

### georadiusbymember(...)

Query a sorted set representing a geospatial index to fetch members matching a given maximum distance from a member.

See `georadius`.

### hdel($key, $field[, $field[...]])

Delete one or more hash fields.

See **variadic arguments**.

### hgetall($key)

Get all the fields and values in a hash.

Redis returns result as a flat array: `['k', 'v', 'k2', 'v2']`. Predis parses it into an associative array: `['k' => 'v', 'k2' => 'v2']`.

```PHP
hmset('hashkey', ['k' => 'v', 'k2' => 'v2']);
hgetall('hashkey');
  //=> ['k' => 'v', 'k2' => 'v2']
```

### hmget($key, $field[, $field[...]])

Get the values of all the given hash fields.

See **variadic arguments**.

### hscan($key, $cursor[, $option[...]])
### hscan($key, $cursor, array $options)

Incrementally iterate hash fields and associated values.

Options can be given inline (first form) or as an array (second form).

Array `$options` accepts optional keys (key names are case-insensitive):

* MATCH - string
* COUNT - integer

Inline options are not normalized.

```PHP
hscan('hashkey', 0, 'MATCH', 'foo*');
hscan('hashkey', 0, ['MATCH' => 'foo*']);
```

Redis returns array of new cursor and flat array of key/values. Predis parses second array into an associative array: `['k' => 'v', 'k2' => 'v2']`.

### hmset($key, $key, $value[, $key, $value[...]])
### hmset($key, array $keyValues)

Set multiple hash fields to multiple values.

Values can be given as inline flat arguments (first form) or an associative array (second form).

```PHP
hmset('hashkey', 'k', 'v', 'k2', 'v2');
hmset('hashkey', ['k' => 'v', 'k2' => 'v2']);
```

### pfadd($key, $element[, $element[...]])

Adds the specified elements to the specified HyperLogLog.

See **variadic arguments**.

### pfcount($key[, $key[...]])

Return the approximated cardinality of the set(s) observed by the HyperLogLog at key(s).

See **list arguments**.

### pfmerge($dest, $src[, $src[...]])

Merge N different HyperLogLogs into a single one.

See **list arguments**.

### del($key[, $key[...]])

Delete a key.

See **list arguments**.

### migrate($host, int $port, $key, int $destDB, int $timeout, [, $option[...]])
### migrate($host, int $port, $key, int $destDB, int $timeout, array $options)

Atomically transfer a key from a Redis instance to another one. `$key` can be an empty string (`''`).

Options can be given as inline arguments (first form) or as an array (second form).

Array `$options` accepts optional keys (key names are case-insensitive):

* COPY - boolean
* REPLACE - boolean

Inline options are not normalized.

```PHP
migrate('hostname', 6379, 'keytomigrate', 0, 1000, 'COPY');
migrate('hostname', 6379, 'keytomigrate', 0, 1000, ['COPY' => true]);
```

### randomkey()

Return a random key from the keyspace.

`null` is returned if database has no keys.

### scan($cursor[, $option[...]])
### scan($cursor, array $options)

Incrementally iterate the keys space.

Options can be given as inline arguments (first form) or as an array (second form).

Array `$options` accepts optional keys (key names are case-insensitive):

* MATCH - string
* COUNT - integer

Inline options are not normalized.

```PHP
scan(0, 'MATCH', 'foo*');
scan(0, ['MATCH' => 'foo*']);
```

### sort($key[, array $options])

Sort the elements in a list, set or sorted set.

Options can only be given as an array, not inline.

Array `$options` accepts optional keys (key names are case-insensitive):

* BY - string
* GET - string or array of strings (patterns)
* LIMIT - array of 2 integers (offset and count)
* SORT - string (case-insensitive)
* ALPHA - boolean
* STORE - string

```PHP
sort('keytosort', ['LIMIT' => [20, 5], 'ALPHA' => true]);
```

### blpop($key[, $key[...]][, $timeout])
### blpop(array $keys, $timeout)

Remove and get the first element in a list, or block until one is available.

First form lists key names directly as arguments, with optional numeric `$timeout` as the last argument.

Second form passes all key names as a single array and requires `$timeout`.

```PHP
blpop('list1', 'list2', 60);
blpop(['list1', 'list2'], 60);
blpop('list1', 'list2');
// This is wrong:
blpop(['list1', 'list2']);
```

### brpop(...)

Remove and get the last element in a list, or block until one is available.

See `blpop`.

### lpush(...)

Prepend one or multiple values to a list.

See `rpush`.

### rpush($key, $value[, $value[...]])

Append one or multiple values to a list.

See **variadic arguments**.

### pubsub($subcommand[, $arg[...]])

Inspect the state of the Pub/Sub subsystem.

When `$subcommand` is `numsub`, Redis returns flat list of channels. Predis parses it into an associative array: `['chan1' => count, 'chan2' => count]`.

### subscribe($channel[, $channel[...]])

Listen for messages published to the given channels.

See **list arguments**.

### psubscribe($pattern[, $pattern[...]])

Listen for messages published to channels matching the given patterns.

See **list arguments**.

### unsubscribe($channel[, $channel[...]])

Stop listening for messages posted to the given channels.

See **list arguments**.

### punsubscribe($pattern[, $pattern[...]])

Stop listening for messages posted to channels matching the given patterns.

See **list arguments**.

### bgrewriteaof()

Asynchronously rewrite the append-only file.

Redis returns a textual message. Predis returns `true` on `Background append only file rewriting started`, `false` on any other response.

### bgsave()

Asynchronously save the dataset to disk.

Redis returns a textual message. Predis returns `true` on `Background saving started`, or original Redis response if it's different.

### client($subcommand[, $arg[...]])

Execute a Redis client connection-related task.

When `$subcommand` is `list`, Redis returns a textual list of clients and their properties. Predis parses it into an array of associative arrays.

### config($subcommand[, $arg[...]])

Execute a server configuration-related task.

Redis returns flat list of config options and values. Predis parses it into an associative array: `['dbfilename' => 'dump.rdb', 'requirepass' => '', ...]`.

### info([$section])

Get information and statistics about the server.

Redis returns a textual list of keys and values, with some comment lines. Predis parses it into an associative array as described below (1). Additionally, keys of form `dbN` where N is a number (these are present in `Keyspace` section), are parsed into a nested associative array: `db0: keys=4,expires=0` becomes `'db0' => ['keys' => 4, 'expires' => 0]`.

(1) Initial parsing is conditional:

* If first line in response begins with `#`, result is a grouped associative array of arrays: comment lines like `# Section` (`^# \w+$`) are outer array keys, lines in between them like `key: value` form inner arrays of the last section.
* If it doesn't, result is an ungrouped associative array. Lines that do not contain `:` are ignored.

Example raw output (shortened with ellipsises):
```
# Server
redis_version:3.1.103
redis_git_sha1:00000000
redis_git_dirty:0
...

# Clients
connected_clients:2
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:846528
used_memory_human:826.69K
...

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
...

# Stats
total_connections_received:57
total_commands_processed:350
instantaneous_ops_per_sec:0
...

# Replication
role:master
connected_slaves:0
...

# CPU
used_cpu_sys:58.20
used_cpu_user:0.40
used_cpu_sys_children:0.19
used_cpu_user_children:0.00

# Cluster
cluster_enabled:0

# Keyspace
db0:keys=4,expires=0,avg_ttl=0
db1:keys=1,expires=0,avg_ttl=0
db15:keys=6,expires=0,avg_ttl=0
```

Predis output (shortened):
```PHP
[
  ...,
  'CPU' => [
    'used_cpu_sys' => '59.04',
    'used_cpu_user' => '0.41',
    'used_cpu_sys_children' => '0.19',
    'used_cpu_user_children' => '0.00',
  ],
  'Cluster' => [
    'cluster_enabled' => '0',
  ],
  'Keyspace' => [
    'db0' => [
      'keys' => '4',
      'expires' => '0',
      'avg_ttl' => '0',
    ],
    'db1' => [
      'keys' => '1',
      'expires' => '0',
      'avg_ttl' => '0',
    ],
    'db15' => [
      'keys' => '6',
      'expires' => '0',
      'avg_ttl' => '0',
    ],
  ],
]
```

### sentinel($subcommand[, $arg[...]])

**Undocumented in Redis.**

When `$subcommand` is `masters` or `slaves`, Redis returns a listt of flat list of properties. Predis parses it into a list of associative arrays.

### slaveof($host, $port)
### slaveof(['NO ONE'])

Make the server a slave of another instance (first form), or promote it as master (second form - no arguments or a single string `NO ONE`).

```PHP
slaveof('8.8.8.8', 123);
slaveof('NO ONE');
slaveof();
```

### slowlog($subcommand[, $arg[...]])

Manages the Redis slow queries log.

When Redis returns list of flat lists of properties (e.g. when `$subcommand` is `GET`). Predis parses it into a list of associative arrays with keys:

* `id`
* `timestamp`
* `duration`
* `command`

### sadd($key, $member[, $member[...]])

Add one or more members to a set.

See **variadic arguments**.

### sdiff($key[, $key[...]])

Subtract multiple sets.

See **list arguments**.

### sdiffstore(...)

Subtract multiple sets and store the resulting set in a key.

See `sinterstore`.

### sinter($key[, $key[...]])

Intersect multiple sets.

See **list arguments**.

### sinterstore($dest, $key[, $key[...]])
### sinterstore($dest, array $keys)

Intersect multiple sets and store the resulting set in a key.

Key names can be given directly as function arguments (first form) or a single array of key names (second form).

```PHP
sinterstore('destkey', 'src', 'src2');
sinterstore('destkey', ['src', 'src2']);
```

### srem($key, $member[, $member[...]])

Remove one or more members from a set.

See **variadic arguments**.

### sscan(...)

Incrementally iterate Set elements.

See `hscan`.

### sunion($key[, $key[...]])

Add multiple sets.

See **list arguments**.

### sunionstore(...)

Add multiple sets and store the resulting set in a key.

See `sinterstore`.

### bitop($operation, $dest, $key[, $key[...]])
### bitop($operation, $dest, array $keys)

Perform bitwise operations between strings.

Source key names can be given directly as function arguments (first form) or a single array of key names (second form).

```PHP
bitop('OR', 'dest', 'k1', 'k2');
bitop('OR', 'dest', ['k1', 'k2']);
bitop('NOT', 'k', 'k');
```

### mget($key, $field[, $field[...]])

Get the values of all the given hash fields.

See **list arguments**.

### mset($key, $value[, $key, $value[...]])
### mset(array $values)

Set multiple keys to multiple values.

Key names and their values to set can be given directly as function arguments one after another (first form) or as a single associative array (second form).

```PHP
mset('k', 'v', 'k2', 'v2');
mset(['k' => 'v', 'k2' => 'v2']);
```

### msetnx(...)

Set multiple keys to multiple values, only if none of the keys exist.

See `mset`.

### watch($key[, $key[...]])

Watch the given keys to determine execution of the MULTI/EXEC block.

See **variadic arguments**.

### zadd($key[, $nx][, 'CH'][, 'INCR'], int $score, $member[, int $score, $member[...]])
### zadd($key[, $nx][, 'CH'][, 'INCR'], array $memberScores)

Add one or more members to a sorted set, or update its score if it already exists. `$nx` can be either `NX` or `XX`.

Member scores can be given directly as arguments (first form) or as a single associative array (second form).

```PHP
zadd('zkey', 'XX', 'INCR', 1, 'm1', 2, 'm2');
zadd('zkey', 'XX', 'INCR', ['m1' => 1, 'm2' => 2]);
```

### zinterstore(...)

Intersect multiple sorted sets and store the resulting sorted set in a new key.

See `zunionstore`.

### zrange($key, int $start, int $stop[, mixed $options])

Return a range of members in a sorted set, by index.

Options can be a string `WITHSCORES` or an associative array of optional keys (key names are case-insensitive):

* WITHSCORES - boolean

If `WITHSCORES` is used, Redis returns a flat list of members and scores. Predis parses it into an associative list: `['m1' => score, 'm2' => score]`.

### zrangebylex($key, int $start, int $stop[, $option[...]])
### zrangebylex($key, int $start, int $stop, array $options)

Return a range of members in a sorted set, by lexicographical range.

Options can be given as inline arguments (first form) or as an array (second form).

Array `$options` accepts optional keys (key names are case-insensitive):

* LIMIT - array of 2 integers (offset and count) or associative array with keys `OFFSET` and `COUNT` (key names are case-insensitive)

Inline options are not normalized.

```PHP
zrangebylex('zkey', 1, 10, 'LIMIT', 20, 5);
zrangebylex('zkey', 1, 10, ['LIMIT' => [20, 5]]);

zrangebylex('zkey', 1, 10, [
  'LIMIT' => [
    'OFFSET' => 20,
    'COUNT' => 5,
  ],
]);
```

### zrangebyscore(...)

Return a range of members in a sorted set, by score.

See `zrange`.

Array `$options` additionally accepts the following keys:

* LIMIT - array of 2 integers (offset and count) or associative array with keys `OFFSET` and `COUNT` (key names are case-insensitive)

```PHP
zrangebyscore('zkey', 0, 5, ['WITHSCORES' => true, 'LIMIT' => [20, 5]]);
```

### zrem($key, $member[, $member[...]])

Remove one or more members from a sorted set.

See **variadic arguments**.

### zrevrange(...)

Return a range of members in a sorted set, by index, with scores ordered from high to low.

See `zrange`.

### zrevrangebylex(...)

Return a range of members in a sorted set, by lexicographical range, ordered from higher to lower strings.

See `zrangebylex`.

### zrevrangebyscore(...)

Return a range of members in a sorted set, by score, with scores ordered from high to low.

See `zrangebyscore`.

### zscan(...)

Incrementally iterate sorted sets elements and associated scores.

See `hscan`.

Additionally, Redis returns a flat list of members and their scores. Predis parses it into an associative list: `['m1' => score, 'm2' => score]`.

### zunionstore($dest, $numKeys, $key[, $key[...]][, array $options])
### zunionstore($dest, array $keys[, array $options])

Add multiple sorted sets and store the resulting sorted set in a new key.

First call form is close to native Redis syntax. Any number of key names can be given directly as arguments as long as it matches `$numKeys`.

Second call form accepts key names as a single array.

Array `$options` accepts optional keys (key names are case-insensitive):

* WEIGHTS - array of numerical weights
* AGGREGATE - string

```PHP
zunionstore('zdest', 2, 'zsrc1', 'zsrc2', ['AGGREGATE' => 'sum']);
zunionstore('zdest', ['zsrc1', 'zsrc2'], ['AGGREGATE' => 'sum']);
```



