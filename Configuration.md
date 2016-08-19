Formatted version of this page is available online: http://squizzle.me/php/predis/doc/Configuration

[Index](README.md) • [Classes](Classes.md) • [Commands](Commands.md) • **Configuration**

# Configuration

Predis configuration is defined by `...Option` classes under `Configuration` namespace.

## cluster

Sets up an aggregate connection for multiple Redis instances, possibly with different profiles (versions) and other properties.

Accepted values:

* string: `predis` or `predis-cluster` - uses `PredisCluster`.
* string: `redis` or `redis-cluster` - uses `RedisCluster` over currently configured connections (see `connections` option).
* `ClusterInterface` - used as is.

Default value: `predis-cluster`.

Returns: a `ClusterInterface`.

Clustering description **TODO**.

```PHP
new Predis([], ['cluster' => 'redis']);
```

## connections

Configures actual underlying connection(s) used by Predis to run the commands. Also used in clustering.

Accepted values:

* `FactoryInterface` - used as is
* array - defines a connection pool, keys are schemes, values are initializers

If an array is used and `parameters` option is defined, it's set as default parameters for the pool.

An initializer must be a string class name implementing `NodeConnectionInterface or`a callable (lazily invoked when it's time to initialize a scheme) returning such a class name.

Default value: a `Connection\Factory` with default set of schemes and initializers (below).

Returns: a `Connection\FactoryInterface`.

```PHP
new Predis([], ['connections' => [
  'unix' => 'Predis\\Connection\\PhpiredisSocketConnection',
]]);
```

### Standard schemes

Common parameters for all schemes:

* `scheme` - required, string; type of connection (transport), as listed below (`tcp`, `unix`, etc.)
* `alias` - optional, string; connection name, if unset connection index is used as its identifier (for functions like `getConnectionById`); connection aggregation might use these for other needs, such as master connection in a replication scheme must have the `master` alias

#### tcp

##### StreamConnection (default)

Parameters:

* `host` - required, string; a hostname or an IPv4 or IPv6 address.
* `port` - required, number
* `async_connect` - boolean

Parameters common to `tcp` and `unix` schemes:

* `persistent` - boolean; if set doesn't close connection when script execution ends so that next request can reuse connection to the same server
* `timeout` - number of seconds to wait until remote server responds to the initial connection; a float, defaults to 5. Doesn't apply if `async_connect` is not set.
* `read_write_timeout` - timeout in seconds applied after connection establishment; uses PHP default. Negative value sets infinite timeout.
* `tcp_nodelay` - boolean, controls `TCP_NODELAY` option on the socket (`true` disables Nagle's algorithm); only effective in PHP 5.4+ and with available `sockets` extension.

##### PhpiredisStreamConnection

Supports the same parameters as `StreamConnection`.

##### PhpiredisSocketConnection

* `host` - required, string; a hostname or an IPv4 or IPv6 address.
* `port` - required, number

Parameters common to `tcp` and `unix` schemes (see their descriptions in `StreamSocket` parameters above):

* `read_write_timeout`
* `timeout`

#### unix

##### StreamConnection (default)

Parameters:

* `path` - required, string; path to Redis' Unix socket

See also common parameters in `tcp`.

##### PhpiredisStreamConnection

Supports the same parameters as `StreamConnection`.

##### PhpiredisSocketConnection

* `path` - required, string; path to Redis' Unix socket

See also common parameters in `tcp`.

#### tls

##### StreamConnection (default)

Parameters are the same as of `tcp`, plus:

* `ssl` - array of crypto options as accepted by PHP's `ssl` stream context: http://php.net/manual/en/context.ssl.php. If `crypto_type` is unset, it's set to `STREAM_CRYPTO_METHOD_TLS_CLIENT`.

```PHP
new Predis(['tls' => ['ssl' => ['verify_peer' => false]]]);
```

##### PhpiredisStreamConnection

Supports the same parameters as `StreamConnection`.

#### redis

Alias of `tcp`.

#### rediss

Alias of `tls`.

#### http

##### WebdisConnection (default)

Parameters:

* `host` - required, string; a hostname or an IPv4 or IPv6 address.
* `port` - required, number
* `user` and `pass` - strings; both must be set even if one is empty.
* `timeout` - see description in `StreamSocket` parameters above

## parameters

Accepted values:

* string - given to `Parameters::parse()`
* array

Default value: taken from private `Parameters::$defaults`, which are:

* `scheme` - `tcp`
* `host` - `127.0.0.1`
* `port` - 6379

Returns: raw string or array as given by the user, or `null`.

See the `connections` option and the `Connection\Parameters` class for format details.

## exceptions

Specifies what to do if a Redis command fails (also affects pipelines and transactions).

Accepted values:

* boolean `true` - fail with `ServerException` wherever Redis returns erroneous response.
* boolean `false` - return failed commands as `ErrorInterface` objects.

Default value: boolean `true`.

Returns: boolean.

```PHP
$predis = new Predis([], ['exceptions' => false]);
$predis->set('s', 'v');
$res = $predis->lpush('s', 'fail!');

echo get_class($res);
  //=> Predis\Response\Error

echo $res->getErrorType();
  //=> WRONGTYPE

echo $res->getMessage();
  //=> WRONGTYPE Operation against a key holding the wrong kind of value
```

## prefix

Transforms key names referenced by Predis commands.

Accepted values:

* `ProcessorInterface` - used as is
* string - key prefix given to `KeyPrefixProcessor`'s constructor

Default value: none (no prefixing).

Returns: `null` or a `ProcessorInterface`.

`KeyPrefixProcessor` works by prepending the prefix to all key references that it recognizes using a built-in command list except for commands that implement `PrefixableCommandInterface` - for them, their `prefixKeys()` method is called with the current prefix string.

Example of simple key prefixing using standard command list:
```PHP
$predis = new Predis([], ['prefix' => 'user118:']);
$predis->get('skey');
  // reads 'user118:skey' key.
```

### Customizing command list

`KeyPrefixProcessor` doesn't recognize custom commands or scripts registered on the main `Client` object. Use `setCommandHandler()` method to register custom prefixers or unregister default ones.

Callback is given a `CommandInterface` and the prefix string. Its return value is ignored. Callback can be any valid callable including a string (`MyClass::foo`) or array reference.

If `setCommandHandler` is called without or with a null callback, that handler is unregistered (command's keys not transformed).

**Note:** callbacks are never called for commands implementing `PrefixableCommandInterface`.

Example of custom prefixing:
```PHP
$pf = new Command\Processor\KeyPrefixProcessor('user118:');

// setPrefix() can be used to dynamically change the prefix.
$pf->setPrefix('user220:');

echo $pf->getPrefix();
  //=> string(8) "user220:"

// Register a custom handler for 'mycustomcmd':
$pf->setCommandHandler('mycustomcmd', function (CommandInterface $cmd, $prefix) {
  $a = $cmd->getArguments();
  $a[0] = $prefix.$a[0];
  $cmd->setRawArguments($a);
});

//$predis->mycustomcmd('key');
// Now the above will refer to 'user220:key'.

// Unregister a standard handler for 'sort':
$pf->setCommandHandler('sort');

// The same:
$pf->setCommandHandler('sort', null);
```

## profile

Sets a "server profile" - info about Redis server that Predis is going to talk to, such as its version.

Accepted values:

* string - profile identifier as recognized by `Profile\Factory::get()`.
* `ProfileInterface` - used as is; **note:** command processors defined by other `$options` (like `prefix`) won't be set to this argument

Default value: `Profile\Factory::getDefault()`.

Returns: `ProfileInterface`.

```PHP
new Predis([], ['profile' => '2.6']);

new Predis([], ['profile' => 'dev']);
// or
new Predis([], ['profile' => Profile\Factory::getDevelopment()]);
```

### Supported commands

#### 2.0

APPEND AUTH BGREWRITEAOF BGSAVE BLPOP BRPOP CONFIG DBSIZE DECR DECRBY DEL DISCARD ECHO EXEC EXISTS EXPIRE EXPIREAT FLUSHALL FLUSHDB GET GETSET HDEL HEXISTS HGET HGETALL HINCRBY HKEYS HLEN HMGET HMSET HSET HSETNX HVALS INCR INCRBY INFO KEYS LASTSAVE LINDEX LLEN LPOP LPUSH LRANGE LREM LSET LTRIM MGET MONITOR MOVE MSET MSETNX MULTI PING PSUBSCRIBE PUBLISH PUNSUBSCRIBE QUIT RANDOMKEY RENAME RENAMENX RPOP RPOPLPUSH RPUSH SADD SAVE SCARD SDIFF SDIFFSTORE SELECT SET SETEX SETNX SHUTDOWN SINTER SINTERSTORE SISMEMBER SLAVEOF SMEMBERS SMOVE SORT SPOP SRANDMEMBER SREM SUBSCRIBE SUBSTR SUNION SUNIONSTORE TTL TYPE UNSUBSCRIBE ZADD ZCARD ZCOUNT ZINCRBY ZINTERSTORE ZRANGE ZRANGEBYSCORE ZRANK ZREM ZREMRANGEBYRANK ZREMRANGEBYSCORE ZREVRANGE ZREVRANK ZSCORE ZUNIONSTORE

#### 2.2

Added commands:

BRPOPLPUSH GETBIT GETRANGE LINSERT LPUSHX OBJECT PERSIST RPUSHX SETBIT SETRANGE SLOWLOG STRLEN UNWATCH WATCH ZREVRANGEBYSCORE

#### 2.4

Added commands:

CLIENT

#### 2.6

Added commands:

BITCOUNT BITOP DUMP EVAL EVALSHA HINCRBYFLOAT INCRBYFLOAT MIGRATE PEXPIRE PEXPIREAT PSETEX PTTL RESTORE SCRIPT SENTINEL TIME

#### 2.8

Added commands:

BITPOS COMMAND HSCAN PFADD PFCOUNT PFMERGE PUBSUB SCAN SSCAN ZLEXCOUNT ZRANGEBYLEX ZREMRANGEBYLEX ZREVRANGEBYLEX ZSCAN

#### 3.0

No new commands supported by this profile. See 2.8.

#### 3.2

Added commands:

BITFIELD GEOADD GEODIST GEOHASH GEOPOS GEORADIUS GEORADIUSBYMEMBER HSTRLEN

## replication

Sets up an aggregate connection pool for master/slave replication.

Accepted values:

* `ReplicationInterface` - used as is
* `null` or `false` - not attempt replication
* `true` - attempt replication (depends on `autodiscovery` option)
* string `sentinel` - using `SentinelReplication`; needs `service` option

Default value: if `autodiscovery` option is set, uses autodiscovery to configure itself.

Returns: `MasterSlaveReplication` or `null`.

```PHP
$predis = new Predis([], ['replication' => 'sentinel']);
$predis->options->replication;
  // instance of SentinelReplication
```

## autodiscovery

If enabled, attempts automatic configuration of the replication (unless it was explicitly disabled with `replication` option).

Accepted values: boolean.

Default value: `false`.

Returns: raw user input.

## service

Used by `SentinelReplication`, if enabled with the `replication` option.

## aggregate

If present, configures an aggregate connection pool for `Client`.

Accepted values:

* callable - given `ParametersInterface` and `OptionsInterface`.

Default value: `null`.

Returns: callable or `null`.


