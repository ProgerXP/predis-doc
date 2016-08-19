Formatted version of this page is available online: http://squizzle.me/php/predis/doc/Classes

[Index](README.md) • **Classes** • [Commands](Commands.md) • [Configuration](Configuration.md)

# Classes

This section documents most of Predis classes that can be directly used by the client code.


## List of exceptions

Hierarchy:
```
Exception
|
+ EmptyRingException                  <-
|
+ A PredisException
  |
  + AbortedMultiExecException         <- WATCH'ed key change
  |
  + A CommunicationException
  | |
  | + RoleException                   <-
  | |
  | + ProtocolException               <-
  | |
  | + ConnectionException             <- connection timeout, etc.
  |
  + NotSupportedException             <-
  |
  + ClientException                   <-
  | |
  | + MissingMasterException          <-
  |
  + ServerException                   <- erroneous Redis response (-ERR)
    ( ErrorInterface )
```

### Transaction\AbortedMultiExecException

Thrown when a `MULTI`/`EXEC` transaction has failed:

* due to unexpected Redis response to a command to be queued.
* due to a `WATCH`'ed key change.

#### getTransaction()

Returns the offending `MultiExec` object.

### abstract CommunicationException

A base class for exceptions identifying a low-level protocol problem, unexpected reply, etc.

#### getConnection()

Retrusn a related `NodeConnectionInterface`.

#### shouldResetConnection()

Returns `true` if the connection should be closed when this exception is produced, to recover from unexpected condition. `false` if the connection can be kept.

#### static handle(CommunicationException)

Throws the argument, disconnecting if its `shouldResetConnection()` is `true`.

### abstract PredisException

A base class for most exceptions produced by Predis.

### NotSupportedException

Thrown when client code attempts to use a feature unsupported by current Predis setup, such as command unsupported by Predis profile or connection scheme (e.g. due to `http`/Webdis not supporting transactions).

### ClientException

A generic exception, usually thrown when the client code attempts to use Predis in a wrong way.

### Replication\MissingMasterException

Thrown when client code has configured replication but no master was specified or autodiscovered.

### Cluster\Distributor\EmptyRingException

Thrown when using `HashRight` key distribution without configured nodes.

### Response\ServerException

Thrown when Redis returns an erroneous response (e.g. `-ERR Something went wrong`) to some command. Some of these cases can be disabled with `exceptions` option for `Client` (see [Configuration](Configuration.md#exceptions)) or for `MultiExec`/`transaction()`.

Implements `ErrorInterface`, see its API for details.

### Protocol\ProtocolException

Thrown on Redis communication protocol parsing errors.

### Connection\ConnectionException.php

Thrown when the connection to Redis couldn't be established or has broken or timed out.

## List of interfaces

### Command\CommandInterface

Indicates command objects that can be sent to Redis and which response can be parsed.

Implemented by classes under `Command` namespace.

### Command\PrefixableCommandInterface

Indicates commands that provide special processing when `prefix` (option)[Configuration.md#prefix] is enabled (for prefixing keys with a fixed value).

#### prefixKeys($prefix)

When called, must prefix all keys used by this command with `$prefix`.

### Configuration\OptionInterface

Indicates Predis configuration objects. See (Configuration)[Configuration.md].

Implemented by classes under `Configuration` namespace.

### Configuration\OptionsInterface

Indicates a Predis configuration container, as returned by `Client->getOptions()`.

Implemented by `Options`.

### Profile\ProfileInterface

Indicates objects representing Redis server profiles with the list of supported command IDs (e.g. `SCAN`).

Implemented by classes under `Profile` namespace.

### Cluster\Hash\HashGeneratorInterface

Indicates an object calculating key hashes used for key distribution over a Redis cluster.

### Cluster\Distributor\DistributorInterface

Indicates an implementation of key distribution logic.

### Cluster\StrategyInterface

Indicates a strategy used to calculate key hashes for client-side sharging.

### Response\ErrorInterface

Represents an erroneous Redis response to a command. Such responses are of `-ERR Something went wrong` format, as opposed to `+strings` and other types in Redis protocol.

#### getErrorType()

Returns the error identifier, in upper-case, such as `WRONGTYPE` or a generic `ERR`.

#### toErrorResponse()

Returns a new `Response\Error` object (implementing `ErrorInterface`). These objects are usually used when exceptions were configured to be ignored, not bailing out (see `exceptions` [Configuration](Configuration.md#exceptions)).

### Response\ResponseInterface

Represents a Redis response to a command, successful or not.

### ClientContextInterface

Indicates that the object can be used in a similar way to the main `Client` object. For example, it's implemented by objects returned from `pipeline()` and `transaction()`.

Such objects are guaranteed to support registered Predis command invokation via `__call`, and `executeCommand`.

```PHP
$p = $predis->pipeline();
$v = $p->get('skey');
$p->set('skey', 'value');

// Just like $predis->get() and $predis->set().
```

Implemented by `Pipeline` and `MultiExec`.

### ClientInterface

Represents a main Predis interface object.

Implemented by `Client`.

### Protocol\ResponseReaderInterface

Implements a standard Redis protocol reader.

### Protocol\ProtocolProcessorInterface

Implements data serialization and unserialization for standard Redis protocol.

### Protocol\Text\Handler\ResponseHandlerInterface

Implements a protocol parser for a particular type of Redis response.

### Protocol\RequestSerializerInterface

Implements data serialization for standard Redis protocol.

### Connection\CompositeConnectionInterface

Identifies a connection with a single Redis server using an external protocol processor.

### Connection\NodeConnectionInterface

Identifies a connection with a single Redis erver.

### Connection\AggregateConnectionInterface

Identifies a virtual connection aggregating multiple Redis servers.

### Connection\ParametersInterface

Implements a container with connection options such as `read_write_timeout`. Can be set by `connections` and related (options)[Configuration.md#connection].

Implemented by `Connection\Parameters`.

### Connection\Aggregate\ReplicationInterface

Identifies a group of Redis servers in a replication (master/slave) setup.

### Connection\Aggregate\ClusterInterface

Identifies a cluster of multiple Redis servers.

### Connection\FactoryInterface

Represents a factory for managing Predis connections. Can be obtained by `Client->getOptions()->connections`.

Implemented by `Connection\Factory`.

### Connection\ConnectionInterface

Represents a base Predis connection backend.

Implemented by classes under `Connection` namespace.


## Root namespace

All Predis classes are grouped under the `Predis` namespace: `Predis\Client`, `Predis\Connection\PhpiredisStreamConnection`, etc.

### Client

**Implements:** `ClientInterface`, `IteratorAggregate`.

The main class you will use to connect to Redis and execute commands.

When using a connection pool (`PredisCluster` or `RedisCluster`), it's possible to iterate over a `Client` object with `foreach`: keys are connection identifiers, e.g. `1.2.3.4:555`, values are new `Client` instances with only that connection from the pool. If this object uses another connection class, a `ClientException` is thrown on `foreach`.

#### __construct([mixed $parameters[, mixed $options]])

`$parameters` specify connection settings, set up aggregate connections and so on. Can be:

* a `ConnectionInterface` - used as is.
* a `ParametersInterface`, a string, an indexed array - given to `connections` option's `create()`.
* an associative array - **TODO**.
* a callable - invoked with current options (`OptionsInterface`), should return a `ConnectionInterface`.
* other values cause an exception.

`$options` configure Predis behaviour. Can be:

* an array - given to `Options->__construct()`.
* an `OptionsInterface` - used as is.
* other values cause an exception.

#### getProfile()

Returns a `ProfileInterface`. A profile describes the server Predis is talking to, such as Redis version and special processing (such as key prefixing).

#### getOptions()

Returns an `OptionsInterface` that tweak Predis behaviour.

#### getClientFor($connectionID)

**TODO**.

#### connect()

Connects to the Redis server. Usually happens automatically behind the scenes.

#### disconnect()

Disconnects from the Redis server.

#### quit()

An alias to `disconnect`, like Redis' `QUIT`.

#### isConnected()

Returns a boolean indicating if Predis is connected to the Redis server or not.

#### getConnection()

Returns a `ConnectionInterface` (even if disconnected).

#### getConnectionById($connectionID)

**TODO**.

#### executeRaw(array $arguments[, &$error])

Executes a Redis command (`RawCommand`) without any pre- and post-processing as if you have invoked it directly on the server. Doesn't throw an exception on invalid response, even if `exceptions` option is set.

`$arguments` - array of command ID (case-insensitive Redis command name like `HSET`) followed by its arguments.

Optional `$error` is given by reference. After `executeRaw` returns, `$error` is set to `true` if Redis returned an error response, `false` if it was executed successfully.

Returns a string regardless of `$error`.

Example:
```PHP
var_dump($predis->executeRaw(['command', 'count'], $error), $e);
```

Output:
```
int(170)
bool(false)
```

Erroneous call:
```PHP
var_dump($predis->executeRaw(['command', 'foo'], $error), $e);
```

Output:
```
string(52) "ERR Unknown subcommand or wrong number of arguments."
bool(true)
```

#### __call($commandID[, array $arguments])

It's possible to invoke Predis commands directly on a `Client` instance.

`$commandID` is case-insensitive, i.e. `SORT`, `sort` and `SorT` are identical.

Equivalent to calling `executeCommand(createCommand(...))`.

Returns whatever was returned by the command (typically a string or an array).

```PHP
var_dump($predis->renamenx('oldkey', 'newkey'));
  //=> int(1)
var_dump($predis->renameNX('nonexkey', 'newkey'));
  // ServerException: 'ERR no such key'.
var_dump($predis->RenameNX('oldkey', 'exkey'));
  //=> int(0)
```

#### createCommand($commandID, array $arguments)

Asks current server profile to construct a command instance.

`$commandID` is case-insensitive, i.e. `SORT`, `sort` and `SorT` are identical.

Equivalent to calling `getProfile()->createCommand(...)`.

Returns a `CommandInterface`.

#### executeCommand(CommandInterface)

Asks current connection to execute the given command. If Redis responds with an error and `exceptions` option is set, a `ServerException` is thrown; if it's unset, an `ErrorInterface` is returned.

**Note:** unlike `executeRaw()` this works with registered Predis commands (including custom scripts), not necessary native Redis command names. Regular processing rules such as key prefixing and error handling apply.

The `NOSCRIPT` error in response to a `ScriptCommand` (base class for custom Lua scripts) is transparently handled by resubmitting the same script with `EVAL` instead of `EVALSHA`. **Note:** this doesn't work in pipeline and transaction modes.

Equivalent to calling `getConnection()->executeCommand(...)`.

Returns whatever was returned by the command (typically a string or an array).

See `__call`.

#### pipeline([array $options][, callable $callable])

Creates a pipeline context. A pipeline combines multiple commands into a single network packet, drastically reducing latency if the Redis server is on another host. Responses to those commands are combined into a single response packet sent by Redis. Multiple pipeline contexts can exist at a time.

Unlike transactions (`MULTI`-`EXEC` blocks), pipelining doesn't provide guarantees that its commands will run in succession without another connection's commands running in between.

Like transactions, response to any pipelined command is unknown before the entire pipeline is ran. When it happens, all responses are returned as a single indexed array.

If `$callable` is provided, it's invoked with the intermediate pipeline object (`Pipeline\Pipeline`) and upon return the pipeline is dispatched (`execute`). `pipeline()` itself **returns** an array of responses (each called command gets one entry). `execute()` must not be called inside `$callable` or a `ClientException` will be thrown.

If `$callable` is not provided, a `ClientContextInterface` is **returned** that can be used to enqueue commands and call them later. This interface mimics the `Client` class but isn't entirely compatible; it should only be used for invoking Predis commands. To dispatch the pipeline, `execute()` must be called (**not `exec()`**).

If `exceptions` Predis option is set and any queued command received an erroneous response from Redis, Predis calls `disconnect()` and throws a `ServerException` (other responses are not processed nor returned). If the option is unset, returned response array will contain `ErrorInterface` instances for failed commands.

Pipelined commands are always sent to the master connection.

##### $options

`$options` is an associative array of these keys:

* `atomic` - boolean; if set starts a pipeline and enters a transaction
* `fire-and-forget` - boolean, ignored if `atomic` is set; if set doesn't read Redis response after executing the pipeline and so doesn't throw any exceptions even if it fails

Example:
```PHP
$res = $predis->pipeline(['atomic' => true], function ($p) {
  $p->set('k1', 'foo');
  $p->hset('h', 'k', 'bar');
});

// $res[0] is a Status (ResponseInterface).
var_dump((string) $res[0]);
  //=> string(2) "OK"

var_dump($res[1]);
  //=> int(0)
```

The above is schematically identical to:
```
START_PIPELINE
MULTI
SET k1 foo
HSET h k bar
EXEC
END_PIPELINE
PROCESS_RESPONSES
```

A `fire-and-forget`, if used, would be the same but without `PROCESS_RESPONSES`.

A simple example without any of these modifiers:
```PHP
$predis->pipeline(function ($p) {
  $p->get('k');
  $p->set('k2', 'v2');
});
```

Note the absense of `MULTI`/`EXEC`:
```
START_PIPELINE
GET k
SET k2 v2
END_PIPELINE
PROCESS_RESPONSES
```

##### Fluent call examples (without $callable)

Without `$options`:
```PHP
$res = $predis->pipeline()
  ->get('k')
  ->set('k2', 'v2')
  ->execute();
```

With `$options`:
```PHP
$res = $predis->pipeline(['atomic' => true])
  ->set('k1', 'foo')
  ->hset('h', 'k', 'bar')
  ->execute();
```

##### A combination example

It's possible to combine fluent and closure call forms as long as `$callable` doesn't invoke `execute()`:
```PHP
$pipe = $predis->pipeline(['atomic' => true]);
$pipe->set('k', 'v');
...

$res = $pipe->execute(function ($pipe) {
  $pipe->hset('h', 'k', 'v');
});
```

##### Nested modes

You cannot enter transaction after entering a pipeline or vice-versa using `pipeline()` and `transaction()` methods. This is invalid:
```PHP
$predis->pipeline(function ($p) use ($predis) {
  // No such method 'transaction'.
  $p->transaction(...);

  // This will work but transaction will be executed outside of the pipeline.
  $predis->transaction(...);
});
```

However, it's possible to initiate transaction by calling `MULTI`, `EXEC`, `WATCH` and other related methods directly:
```
$res = $predis->pipeline(function ($p) {
  $p->get('k1');

  $p->multi();
  $p->get('k2');
  $p->set('k3', 'v2');
  $p->exec();

  $p->get('k4');
]);
```

`$res` will contain a nested array for each `exec()`:
```
array(6) {
  [0] =>
  string(3) "foo of k1"

  [1] =>
  class Predis\Response\Status#11 (1) {
    private $payload =>
    string(2) "OK"
  }
  [2] =>
  class Predis\Response\Status#12 (1) {
    private $payload =>
    string(6) "QUEUED"
  }
  [3] =>
  class Predis\Response\Status#12 (1) {
    private $payload =>
    string(6) "QUEUED"
  }
  [4] =>
  array(2) {
    [0] =>
    string(3) "quux of k2"
    [1] =>
    class Predis\Response\Status#11 (1) {
      private $payload =>
      string(2) "OK"
    }
  }

  [5] =>
  string(3) "bar of k4"
}
```

As a more convenient alternative, if you don't need to have anything but the transaction inside the pipeline, pass the `atomic` option:
```PHP
$res = $predis->pipeline(['atomic' => true], function ($p) {
  ...
});
```

##### Reusing the pipeline

It's possible to use the same pipeline object more than once - it's flushed after each successful `execute()`. It's not recommended to reuse an object that has thrown an exception during execution as previously recorded commands might be left in the queue.

```PHP
$pipe = $predis->pipeline();
$pipe->set('k', 'v');
$res1 = $pipe->execute();

...

$pipe->hset('h', 'k', 'v');
$res2 = $pipe->execute();
```

This reusage is not recommended:
```PHP
...
try {
  $pipe->execute();
} catch (...) {
  // Do nothing.
}

// $pipe might be reused after an exception in execute().
// Leftover commands might be left in its queue.
$pipe->...
```

**Note:** nested `execute()` is not allowed:
```PHP
$predis->pipeline(function ($pipe) {
  // This will throw a ClientException.
  $pipe->execute();
});
```

##### ConnectionErrorProof

This class makes the pipeline resistant to `ConnectionException`: whenever it occurs, remaining commands are not ran and their responses are returned as a `CommunicationException` instance that has occurred.

For cluster connections, commands continue to run if they are meant for execution on a connection that is yet to fail.

This class is currently not reachable via `pipeline()` and should be used directly. It's interface is identical to the one described under the `pipeline()` method.

It only supports connections implementing `NodeConnectionInterface` or `ClusterInterface` and throws `NotSupportedException` for others.

```PHP
$res = (new Pipeline\ConnectionErrorProof($predis))
  ->execute(function ($cep) {
    $cep->set('k');
    ...
  });
```

`$res` is an array of the usual command responses. If a connection error occurs, that command's response and all the following are set to that exception's object and not sent to Redis.

For a cluster connection, suppose we have 3 individual connections S1, S2, S3. Last two have failed at some point with different exceptions E2, E3. Suppose we are submitting 12 commands in such a way that each connection handles 4 of them; commands are named C1x for S1's commands, etc. and their successful return values - R1x, etc. Then:
```
Command   Result
C11       R11
 C21      R21
 C22      E2
  C31     R31
C12       R12
C13       R13
 C23      not ran; E2
 C24      not ran; E2
C14       R14
  C32     R32
  C33     R33
  C34     E3
```


#### transaction([array $options][, callable $callable])

Creates a transaction context. Commands ran inside a transaction are guaranteed to be serialized, i.e. no other connection will be able to run commands in between serialized commands. Responses to those commands are delayed until the transaction ends, after which they are returned in a single indexed array. Multiple transaction contexts can exist at a time.

Using transactions in Predis is very similar to using pipelines, therefore see `pipeline()` for more details and examples.

Transaction cannot be started on an aggregate connection or with a profile that doesn't support `MULTI`, `EXEC` or `DISCARD`  (`NotSupportedException` is thrown).

If a watched key changes before `EXEC`, the transaction is re-run up to `retries` times (default: 0), after which an `AbortedMultiExecException` is thrown.

An error in any non-user command (`WATCH`, `MULTI`, `EXEC`, `UNWATCH`, `DISCARD`) immediately causes `ServerException` regardless of the `exception` option (transaction's or `Client`'s). **Note:** `EXEC` returns empty response if a watched key has changed, not an error.

If Redis reports an error while queueing a command an `AbortedMultiExecException` is thrown, or if Redis sends a non-`QUEUED` response Predis disconnects and throws a `ProtocolException`.

The transaction is discarded on an exception within `$callable`.

For this transaction object only, `exec` acts as `execute` (but `exec` doesn't accept `$callable`).

If there were no queued commands when `execute` was called, watched keys (if any) are unwatched and no `MULTI`/`EXEC` are sent.

##### $options

`$options` is an associative array with the following optional keys:

* `exceptions` - if null, `Client`'s `exceptions` option is used; if non-null, is a boolean indicating if a `ServerException` should be thrown if `EXEC` returns an erroneous response to any queued command (other cases e.g. when a watched key has changed always throw an exception); if `false` such commands will have a `ErrorInterface` object in the returned array
* `cas` - boolean; if set enables "Compare-And-Swap Mode" - **TODO**
* `watch` - a string key name or an array of names to `WATCH` before `EXEC`; watching begins when a first command is called on this transaction object (it's enqueued unless in CAS mode); throws `NotSupportedException` if current profile doesn't support `WATCH`
* `retry` - max number of automatic re-runs of the transaction on a watched key change before giving up and failing (so called "optimistic locking"); default is 0 meaning no re-runs; if non-zero and a `$callable` is not supplied, a `ClientException` is thrown; only makes sense when also using `watch`

Unlike pipelines, reusing transaction object with a non-empty queue is prohibited (`ClientException` is thrown):

```PHP
$trans = $predis->transaction();
$trans->get('k');
// This throws an exception:
$trans->execute(function ...);
```

##### CAS, watching and retries

Imagine a setup with many concurrent connections doing the same operation: reading a value, changing it and writing back. Without transactions you cannot guarantee that nobody else has read or written the value while you are in the middle of this operation:
```
client1 GET k
client2 GET k
client1 SET k
client2 SET k
```

To avoid this, the key `k` must be watched for changes between `GET` and `SET`:
```PHP
$res = $predis->transaction(['watch' => 'k', 'cas' => true], function ($t) {
  // In CAS mode, before multi() has been manually called all commands are immediately sent to Redis and results fetched.
  $value = $t->get('k');
  // Change the value somehow...
  $value .= mt_rand();

  // Begin the transaction. 'k' was put on the watch list immediately before calling get() above.
  $t->multi();
  $t->set('k', $value);
});
```

The above example will only write `k` if this key (i.e. no keys in the `watch` option) has not changed from `WATCH` to `EXEC`.
If it did, the transaction will fail with `AborderMultiExecException` after the closure returns.

This constitutes CAS - Compare-And-Swap where you first fetch values, do the calculates and then save results only if those values were not changed by another client.

Optimistic locking simply means that the entire CAS operation is repeated until it succeeds (i.e. a lucky chance was taken when nobody else was writing the values in the parallel). To specify the number of retries to re-run the operation `retry` option is used:
```PHP
$predis->transaction(['watch' => 'k', 'cas' => true, 'retry' => 5], function ($t) {
  // Identical to the above example.
});
```

Here, the transaction is allowed to be repeated up to 4 times and if 5th repetition successfully saves the data the transaction will succeed.

#### pubSubLoop([array $options][, callable $callable])

Enters a pub/sub (publish-subscribe) context to read messages in the subscribed channels. A "channel" is like a pipe: another connection (the publisher) pushes messages from one end and this connection and all other subscribers take them from the other end. It's possible to subscribe and unsubscribe while this context is active. Multiple pub/sub contexts can be running at a time.

If `$callable` is provided, it's invoked with the intermediate pub/sub object (`PubSub\Consumer`) and message for every new message. The code blocks if there's no available message in any of the subscribed channels. If `$callable` returns `false`, the pub/sub context ends and execution continues in the caller.

If `$callable` is not provided, an `AbstractConsumer` is **returned** and can be used for custom processing.

Unlike transaction and pipeline contexts that buffer responses until they can be submitted at once, pub/sub changes underlying connection context. Do not call commands on the main `Client` object with active pub/sub - they will produce exceptions or return unexpected results. To return to normal context use `stop()`.

See also `DispatcherLoop` for a callback-based message handling.

Pub/sub cannot be started on an aggregate connection or with a profile that doesn't support `PUBLISH`, `SUBSCRIBE`, `UNSUBSCRIBE`, `PSUBSCRIBE` or `PUNSUBSCRIBE` (`NotSupportedException` is thrown).

##### $options

`$options` is an associative array with the following optional keys:

* `subscribe` - a string channel name or an array of strings
* `psubscribe` - a string pattern or an array of strings

Keys are this object's method names which receive one argument - that key's value so other options can be theoretically used.

##### Possible messages

A "message" is a generic object (`stdClass`) with these properties:

* `kind` - message type such as `subscribe`
* `channel` (absent for `pong` kind) - exact name of the channel that produced the message
* `payload` - additional details (mixed type), see Redis documentation
* other properties can be set by specific message kinds

Standard message kinds:

* `subscribe` - in response to subscription by name, per each name. `channel` is the subscribed channel's name, `payload` is the number of currently subscribed channels (0 if this context is not subscribed to any), with one pattern reporting as one channel.
* `unsubscribe` - in response to unsubscription by name, per each name. See `subscribe` for details.
* `psubscribe` - in response to subscription by pattern, per each pattern. See `subscribe` for details.
* `punsubscribe` - in response to unsubscription by pattern, per each pattern. See `subscribe` for details.
* `message` - when a publisher has pushed a new message onto one of the subscribed channels. `channel` indicates the origin, `payload` is the message.
* `pmessage` - when a publisher has pushed a new message onto one of the channels subscribed by pattern (see `message`). Extra property `pattern` contains original subscription pattern that matched this `channel`.
* `pong` - in response to `ping()`. `channel` is not present. `payload` is the same string that was given to `ping()` or empty (`''`).

Other kinds produce a `ClientException`.

##### Examples

Convenient callback-based looping:
```PHP
$predis->pubSubLoop(['psubscribe' => 'foo:*'], function ($l, $msg) {
  if ($msg->kind === 'pong') {
    die('ouch!');
  } elseif ($msg->kind === 'pmessage' and $msg->payload === 'Q') {
    return false.
  }
});

echo 'I will be reached after you send "Q" to any foo:* channel.';
$predis->get('strkey');   // now is fine.
```

Customized processing using the pub/sub object:
```PHP
$l = $predis->pubSubLoop(['subscribe' => ['a', 'b']]);
// Ignore two initial 'subscribe' messages.
$l->current();
$l->current();
$l->ping();
$l->psubscribe('foo:*', '*bar');

foreach ($l as $msg) {
  // Receive 'pong', two 'psubscribe', then new messages.
  ...
}

$l->stop();

$predis->get('strkey');
```

#### monitor()

Enters a command monitor context. Redis will echo commands from other clients. Returns a `Monitor\Consumer` object. Monitor is stopped (`stop()`) when this object is destroyed. Multiple monitors can be running at a time.

Monitor cannot be started on an aggregate connection or with a profile that doesn't support `MONITOR` (`NotSupportedException` is thrown).

The monitor object implements `Iterator` so it can be used in `foreach`. However, it doesn't support rewinding (rewind calls are ignored) - this means if a `foreach` loop stops, next `foreach` will resume from the first command not handled by previous loop.

Each monitored command is represented by a generic object (`stdClass`) with these properties:

* `timestamp` - time when the command was ran; float like `timestamp.ms` - the same format is used by PHP's `microtime(true)`.
- `database` - database number on which the command was ran.
- `client` (`null` in Redis before 2.6) - string like `host:port` of the originating connection. **Note:** it's client's local port, not server's port; two clients of the same server will always have different ports.
- `command` - Redis command name, without arguments. It's not normalized and can be in any character case, just like issued by that client (e.g. `SeT`).
- `arguments` - null or string - arguments exactly as given by that client (e.g. `"key foo" value_bar`).

```PHP
foreach ($predis->monitor() as $cmd) {
  echo "Client [$cmd->client] has issued a ", strtoupper($cmd->command),
       " on DB #$cmd->database at ", date(DATE_RSS, $cmd->timestamp),
       " with arguments:", PHP_EOL;
  echo $cmd->arguments, PHP_EOL;
}
```

##### Connection context

**Note:** there's no way to stop the monitor other than closing the connection. The `stop()` method does that.

Like `pubSubLoop()`, monitoring changes underlying connection context. Do not call commands on the main `Client` object with active monitor - they will produce exceptions or return unexpected results. Use `stop()` to close the connection and have `Client` reconned when a regular Redis command is issued.

Correct usage:
```PHP
$m = $predis->monitor();

foreach ($m as $cmd) {
  ...
}

$m->stop();

$predis->set('k', 'v');
```

Wrong usage:
```PHP
$m = $predis->monitor();
$predis->set('k', 'v');
  // This is wrong!

```


### ClientContextInterface

Commands that enter another context - mode of operation (pipeline, transaction, pub-sub) typically implement this interface to allow calling Predis commands the same way it's done on the main `Client` instance.

#### executeCommand(CommandInterface)

Executes a pre-constructed command in this context. This doesn't always mean the command is immediately send to Redis - e.g. pipeline will record them for bulk submission.

#### __call($commandID[, array $arguments])

Similarly to `Client->__call()`, creates and executes a registered Predis command (case-insensitive `$commandID`, such as `llen`) in this context.

#### execute([$callable])

Starts execution of this context. If `$callable` is given, it's invoked with this object (its return value is ignored). After it returns, accumulated commands are sent to Redis.


## Collection\Iterator\

Classes of this group provide high-level abstraction over `SCAN` family of Redis commands. There's also a list iterator class which is not provided with `SCAN`. Iteration doesn't have to be exhausted; you can abandon the loop at any time, or even not start it without a penalty.

To use them, construct them directly. They are not returned nor used by other Predis classes.

### CursorBasedIterator

**Implements:** `Iterator`.

An abstract base class for concrete iterators that make use of `SCAN` commands provided by Redis. Iteration order is undefined.

Unlike `KEYS` and `SMEMBERS` commands that do similar tasks, `SCAN` commands do not block the server for a long time. However, this means there might be inconsistency if the database or target key is actively being updated: new keys might be missing from `SCAN` results while removed keys might be present.

#### __construct(ClientInterface[, $match[, int $count]])

Creates a new iterator on the given `Client` object. `$match` is an optional pattern such as `foo:*`. `$count` sets a recommended result set size for Redis (see Redis documentation for details).

#### protected getScanOptions()

Returns an array of options accepted by corresponding `scan()` Predis command. Example: `['MATCH' => 'foo:*', 'COUNT' => 10]`.

#### protected executeCommand()

Fetches next set from Redis.

#### rewind(), current(), key(), next(), valid()

`Iterator` methods rewinding the scan to the beginning, returning current element, its key, advancing to the next element and returning `true` if there are more elements or `false` if not.

### HashKey

**Extends:** `CursorBasedIterator`.

An interface to `HSCAN` for iterating over hash key fields. Iteration order is undefined.

Iteration key is hash field name, value is that field's value.

Throws `NotSupportedException` if current profile doesn't support `HSCAN`.

```PHP
$predis->hmset('hkey', ['k' => 'v', 'k2' => 'v2']);
$it = new Collection\Iterator\HashKey($predis, 'hkey');

foreach ($it as $field => $value) {
  echo "$field = $value", PHP_EOL;
}

// The above cycle will print:
//   k = v
//   k2 = v2
```

#### __construct(ClientInterface, $key[, $match[, int $count]])

Creates a new hash iterator on the given `Client` object. `$key` is the name of the hash key which fields should be iterated over. `$match` is an optional pattern such as `foo:*`. `$count` sets a recommended result set size for Redis (see Redis documentation for details).

### Keyspace

**Extends:** `CursorBasedIterator`.

An interface to `SCAN` for iterating over all keys in the database.

Iteration value is current key name.

Throws `NotSupportedException` if current profile doesn't support `SCAN`.

```PHP
$predis->set('skey', 'str');
$predis->hset('hkey', 'field', 'value');
$it = new Predis\Collection\Iterator\Keyspace($predis);

foreach ($it as $key) {
  echo "$key", PHP_EOL;
}

// The above cycle will print:
//   skey
//   hkey
```

### ListKey

**Implements:** `Iterator`.

A class for iterating over list keys in the same way as provided by `CursorBasedIterator`-based classes. Since there's no built-in Redis command for iterating over lists, `ListKey` emulates it with `LRANGE`.

See `CursorBasedIterator` for method details and (lack of) guarantees.

Throws `NotSupportedException` if current profile doesn't support `LRANGE`.

```PHP
$predis->lpush('lkey', 'v1', 'v2', 'v3');
$it = new Predis\Collection\Iterator\ListKey($predis, 'lkey');

foreach ($it as $value) {
  echo "$value", PHP_EOL;
}

// The above cycle will print:
//   v1
//   v2
//   v3
```

#### __construct(ClientInterface, $key[, $match[, int $count = 20]])

Creates a new list iterator on the given `Client` object. `$key` is the name of the list key which values should be iterated over. `$match` is an optional pattern such as `foo:*`. `$count` sets the number of elements to fetch from Redis with `lrange` and hold in memory at once.

### SetKey

**Extends:** `CursorBasedIterator`.

An interface to `SSCAN` for iterating over sets.

Iteration value is current set member's name.

Throws `NotSupportedException` if current profile doesn't support `SSCAN`.

```PHP
$predis->sadd('skey', 'm1', 'm2');
$it = new Predis\Collection\Iterator\SetKey($predis, 'skey');

foreach ($it as $member) {
  echo "$member", PHP_EOL;
}

// The above cycle will print:
//   m1
//   m2
```

#### __construct(ClientInterface, $key[, $match[, int $count]])

Creates a new set iterator on the given `Client` object. `$key` is the name of the set key which members should be iterated over. `$match` is an optional pattern such as `foo:*`. `$count` sets a recommended result set size for Redis (see Redis documentation for details).

### SortedSetKey

Extends `CursorBasedIterator`.

An interface to `ZSCAN` for iterating over hash key fields.

Iteration key is set's member name, value is that member's score.

Throws `NotSupportedException` if current profile doesn't support `ZSCAN`.

```PHP
$predis->zadd('zkey', 0.1, 'm1', 0.2, 'm2', 0.3, 'm3');
$it = new Predis\Collection\Iterator\SortedSetKey($predis, 'zkey');

foreach ($it as $member => $score) {
  echo "$member = $score", PHP_EOL;
}

// The above cycle will print:
//   m1 = 0.1
//   m2 = 0.2
//   m3 = 0.3
```

#### __construct(ClientInterface, $key[, $match[, int $count]])

Creates a new sorted set iterator on the given `Client` object. `$key` is the name of the sorted set key which members should be iterated over. `$match` is an optional pattern such as `foo:*`. `$count` sets a recommended result set size for Redis (see Redis documentation for details).



## Command\

This namespace contains Predis implementations of Redis commands. Each supported command is a separate class, for example, `ListPopFirstBlocking` implements `BLPOP`.

Client code can register new commands using `ProfileInterface->defineCommand()` (see `ScriptCommand` examples below).

### ScriptCommand

An abstraction for implementing server-side Lua scripts called with `EVAL` or `EVALSHA`. This class is registered as a regular Predis command and so can be called directly on a `Client` object.

Basic script:
```PHP
class ReturnFooScript extends Predis\Command\ScriptCommand {
  function getScript() {
    return 'return "foo"';
  }
}

$predis->getProfile()->defineCommand('retfoo', 'ReturnFooScript');
echo $predis->retfoo();
  //=> foo
```

More complex example that accepts a key, a value and validates arguments:
```PHP
class GetSetScript extends Predis\Command\ScriptCommand {
  function getScript() {
    return 'return redis.call("GETSET", KEYS[1], ARGV[1])';
  }

  protected function getKeysCount() {
    return 1;
  }

  protected function filterArguments(array $args) {
    if (count($args) < 2) {
      throw new InvalidArgumentException(__CLASS__." expects at least 2 arguments.");
    }

    return parent::filterArguments($args);
  }
}

$predis->getProfile()->defineCommand('mygetset', 'GetSetScript');
$predis->set('k', 'oldk');
echo $predis->mygetset('k', 'newk');
  //=> oldk, old value returned
echo $predis->get('k');
  //=> newk, current value as given to mygetset()
```

#### abstract getScript()

Returns a string - Lua script.

#### protected getKeysCount()

Returns how many of the command's arguments are keys. Redis calling convention that must be followed is to pass all key names the script will work on (read or write) before other arguments.

Can return a negative number meaning "all arguments but last N are keys". So if the script takes 5 arguments, returning -2 is the same as returning 3.


## Configuration\

This namespace Predis option classes (see [Configuration](Configuration.md)). Each option is a separate class, but not all options need a class.

Options are hardcoded and cannot be registered by the client code.

### Options

**Implements:** `OptionsInterface`.

This is a container for Predis options typically given as an array to `Client->__construct()`. Each option is represented by a separate class, typically under `Configuration` namespace, and a name-class link is called a "handler".

The list of standard configuration options is described in the separate section.

An instance of this class can be obtained by `Client->getOptions()`.

#### __construct([array $options])

Initializes the options container with the given user option values (they are passed through corresponding option's normalization).

#### getDefault($option)

Returns a mixed default value used if there was no value supplied by the user.

```PHP
$predis->getOptions()->getDefault('exceptions');
  //=> true
```

#### defined($option)

Returns `true` if `$option` was given by the user or is a standard option (which always has a default value), or `false` otherwise.

#### __isset($option)

Returns `true` if `$option` is `defined()` and its value is not `null`.

```PHP
$options = new Options(['custom' => 1]);

isset($options['exceptions']);
  //=> true, even though it wasn't explictly given to the constructor

isset($options['prefix']);
  //=> false

isset($options['custom']);
  //=> true, even though it's not a standard option as demonstrated below

$options = new Options;
isset($options['custom']);
  //=> false
```

#### __get($option)

If `$option` was given by user and if it's an object with `__invoke` such as a Closure (**note:** callable array won't work), it's called and return value used as if it was given directly.

If `$option` was given by user and it's a standard Predis option, it is normalized (`filter()`) and returned value used instead of the user value. `filter` might throw an exception.

If `$option` was given by user and it's not a standard option, it's returned as is.

Finally, if `$option` was not given by user and it's a standard option, its default value (`getDefault()`) is used.

If `$option` was not given by user and it's not a standard option, `null` is returned.

**Note:** this returns option's _value_, not option's object - it cannot be obtained using `Options` public interface.

```PHP
$options = new Options(['custom' => 1]);

$options->exceptions
  //=> true

$options->custom
  //=> 1

$options->prefix
  //=> null

$options->replication
  //=> a MasterSlaveReplication
```


## Connection\

Classes under this namespace handle low-level Redis connection details including aggregate (cluster) connections.

Hierarchy:

```
I ConnectionInterface
|
+ I AggregateConnectionInterface
| |
| + I ClusterInterface
| | |
| | + PredisCluster                         <-
| | |
| | + RedisCluster                          <-
| |
| + I ReplicationInterface
|   |
|   + MasterSlaveReplication                <-
|   |
|   + SentinelReplication                   <-
|
+ I NodeConnectionInterface
  |
  + I CompositeConnectionInterface
  |
  + WebdisConnection                        <- default for http
  |
  + A AbstractConnection
    |
    + PhpiredisSocketConnection             <-
    |
    + StreamConnection                      <- default for tcp, tls, unix
      |
      + CompositeStreamConnection           <-
      | ( CompositeConnectionInterface )
      |
      + PhpiredisStreamConnection           <-
```

### ConnectionInterface

The basic interface that every Predis connection backend should implement.

#### connect()

Connects to Redis server, unless already connected.

#### disconnect()

Disconnects from Redis server, if connected.

#### isConnected()

Returns `true` if this backend has currently open connection, `false` if not.

#### writeRequest(CommandInterface)

Sends the command with arguments to Redis. Doesn't read response. Returns nothing.

#### readResponse(CommandInterface)

Reads Redis response to the given command. Returns mixed value as parsed by that command. It's supposed that this command was indeed written just before reading the response, or unexpected things will happen.

#### executeCommand(CommandInterface)

Sends the command with arguments to Redis and reads its response. Returns mixed value as parsed by the command. A shortcut to `writeRequest(); return readResponse()`.

### NodeConnectionInterface

Represents a connection to a single Redis server, as opposed to an aggregate pool of connections.

#### __toString()

Returns a string identifying this connection, such as `tls://redis.host?database=5` or `/var/run/redis.sock` (it doesn't have to be a proper URI).

#### getResource()

Returns an elementary resource used by this backend to represent the connection, for example, a PHP file handle.

#### getParameters()

Returns a `ParametersInterface of original parameters used to initialize this connection.

#### addConnectCommand(CommandInterface)

Schedules given command to be executed after (re-)establishing a new connection to Redis server. Usually used for `AUTH` and `DAATABASE` commands.

New commands are not executed if a connection is already established but will be executed if it's reconnected.

#### read()

Performs a low-level read from the connection. Similar to `readResponse()` but doesn't process the result in any way.

### AbstractConnection

**Implements:** `NodeConnectionInterface`.

A base class for connections implemented using an elementary resource type, such as a PHP socket.

#### __construct(ParametersInterface)

Creates new connection instance. Doesn't connect - this happens with the first communication attempt. Throws `InvalidArgumentException` on invalid parameters.

#### __destruct()

Closes the connection when the object is destroyed.

#### protected onConnectionError($message[, $code = null])

Is called to throw `ConnectionException` on a low-level connection error, such as inability to resolve target hostname, read/write required data, timeout occurrence, etc.

Is also caused by a failed command from `addConnectCommand` (including those which received an erroneous response).

#### protected onProtocolError($message)

Is called to throw `ProtocolException` on a Redis protocol processing, such as unexpected format.

#### __toString()

Returns a cached string identifier, or builds it, caches and returns.

#### protected getIdentifier()

Returns a string identifier of this connection: `/var/run/redis.sock` for `unix` scheme, `host:port` for others.

### StreamConnection

**Extends:** `AbstractConnection`.

A concrete class used to handle numerous schemes from `tcp` to `unix` using PHP streams.

Constructor fails if:

* `scheme` is not one of `tcp`, `tls`, `unix`, `redis`, `rediss`.
* `persistent` parameter is set and `scheme` is `tls` or `rediss` but PHP version is below `7.0.0beta`.

Treats `redis` and `rediss` as aliases to `tcp` and `tls`.

#### __destruct()

Doesn't disconnect if `persistent` parameter is set.

### PhpiredisStreamConnection

**Extends:** `StreamConnection`.

A concrete class used to remove overhead of parsing and building Redis protocol in PHP by using `phpiredis` PHP extension in C.

Throws `NotSupportedException` if `phpiredis` extension is not loaded.

### PhpiredisSocketConnection

**Extends:** `Abstractonnection`.

Similar to `PhpiredisStreamConnection` but uses more flexible PHP `socket` extension instead of streams PHP API.

Throws `NotSupportedException` if `phpiredis` or `sockets` extensions are not loaded, if scheme is not `tcp`, `redis` or `unix` or if `persistent` parameter is set.

Always sets `TCP_NODELAY` and `SO_REUSEADDR` flags.

### WebdisConnection

**Implements:** `NodeConnectionInterface`.

A concrete class for communicating with http://webd.is (a read-only Redis-JSON web frontend).

Throws `InvalidArgumentException` if scheme is not `http`. Throws `NotSupportedException` if `curl` or `phpiredis` extensions are not loaded or when calling Redis commands not supported by Webdis:

* `AUTH`
* `SELECT`
* `MULTI`
* `EXEC`
* `WATCH`
* `UNWATCH`
* `DISCARD`
* `MONITOR`
* `addConnectCommand`

Always sets `TCP_NODELAY` and `SO_REUSEADDR` flags.

#### __toString()

Returns `host:port`.

### Factory

**Implements**: `FactoryInterface`.

This class is accessible as `Client->getOptions()->connections`. It manages a pool of Predis connections, possibly over different schemes and with paraameters.

#### define($scheme, mixed $initializer)

Overrides initializer for the scheme.

An initializer must be a string class name implementing `NodeConnectionInterface or`a callable (lazily invoked when it's time to initialize a scheme) returning such a class name.

```PHP
$connections->define('myyth', 'MyCorp\\MyMyyth');

$connections->define('http', function () {
  return 'MyCorp\\MyHttpRedisConnection';
});
```

#### undefine($scheme)

Forgets about the scheme. Attempting to instantiate a connection with that scheme will throw an exception.

```PHP
$connections->undefine('http');
```

#### create(mixed $parameters)

`$parameters` can be:

* string - turned to array by `Parameters::parse()` and given to `Parameters` constructor.
* array - given to `Parameters` constructor.

Default parameters (if set) are used for keys missing in the input. See the `parameters` configuration option.

Constructed parameters object (a `ParametersInterface`) is given to the scheme initializer which returns a `NodeConnectionInterface`, and that object is returned by `create()`.

If parameters specify connection password, `AUTH` command is scheduled for execution whenever that connection reconnects. If the database parametre is set, a `SELECT` command is also scheduled.

```PHP
$connections->create('tcp://localhost?password=Sekretno')
$connections->create(['host' => 'go.go.go', 'database' => 10]);
```

#### aggregate(AggregateConnectionInterface $aggr, array $parameters)

Aggregates connections from `$parameters` (creating them as necssary) into a single `$aggr` connection.

`$parameters` is an array of `NodeConnectionInterfaces` (aggregated as is) and other types (given to `create()` as creation parameters).

```PHP
$conn1 = $connections->create(...);
$conn2 = 'http://localhost:8081/webdis?timeout=10';
$conn3 = ['scheme' => 'redis'];
$connections->aggregate($aggr, [ $conn1, $conn2, $conn3 ]);
```

#### setDefaultParameters(array $parameters)

Sets default parameters used when creating a new connection. This doesn't merge with defaults set earlier.

See the `parameters` configuration option and the `create()` method for the format.

### Parameters

**Implements**: `ParametersInterface`.

This object holds key/value pairs used by newly created Predis connections (see `Connection\Factory->create()`).

It stores global defaults in the private static variable `$defaults` which is defined as follows:
```PHP
[
  'scheme' => 'tcp',
  'host' => '127.0.0.1',
  'port' => 6379,
]
```

#### __construct([array $parameters])

Creates a new `Parameters` instance. If you wish to use a shorthand string parameters format, use `create()` or `parse()`.

#### static create(mixed $parameters)

Creates a new `Parameters` instance. `$parameters` can be a string (given to `parse()`) or an ready-made array.

`$parameters` can be an empty string or array to use only defaults:
```PHP
new Parameters();
  // or
Parameters::create('');
  // or
Parameters::create([]);
  //=> ['scheme' => 'tcp', 'host' => '127.0.0.1', 'port' => 6379]
```

#### static parse($uri)

Parses an URI specification into separate parameter values.

When using the `redis` and `rediss` schemes the URI is parsed according to the rules defined by the provisional registration documents approved by IANA. If the URI has a password in its `user-information` part or a database number in the `path` part these values override the values of `password` and `database` if they are present in the `query` part.

See (redis)[http://www.iana.org/assignments/uri-schemes/prov/redis] and (rediss)[http://www.iana.org/assignments/uri-schemes/prov/rediss] specs.

Throws `InvalidArgumentException` is `$uri` is not a valid URI.

Standard PHP's `parse_uri()` is used and so any input it recognizes is supported.

Generally, URI has this format (ignore spaces):
```
[scheme: [//]] [ [user]:[pass] ] host [:port] [/path] [?query]
```

Two `//` after `scheme:` can sometimes be omitted but it's not recommended as many URIs won't be properly recognized.

IPv6 hosts are specified in square brackets: `tcp://[::1]:777`.

`query` part has this format:
```
key=value [& key=value [...]]
```

Keys from the `query` part are treated like parameters (they override URI components by the same name): `localhost:123?port=6379` and `localhost:6379` are equivalent, as are `tcp:localhost` and `localhost?scheme=tcp`.

**TODO:** document redis/rediss URI

Some valid URI examples:

* `unix:///var/run/redis.sock`
* `unix:/var/run/redis.sock` - the same as above
* `tcp://4.4.4.4:6000`
* `tls://:RedisPassword@4.4.4.4`
* `tls://[::1]?pass=RedisPassword`
* `http://dev.local/webdisroot/webdis?timeout=10`

```PHP
Parameters::parse('tcp://localhost:10240');
  //=> ['scheme' => 'tcp', 'host' => 'localhost', 'port' => 10240]
```

#### get($parameter)

Returns mixed value for the parameter, or `null` if it's undefined. Default parameters are also examined.

#### isset($parameters)

Returns `true` if the parameter is defined or it has a default value, `false` otherwise.

#### toArray()

Returns array of all parameters including defaults.

```PHP
$p = Parameters::create('localhost:10240');
$p->toArray();
  //=> ['scheme' => 'tcp', 'host' => 'localhost', 'port' => 10240]
  // 'scheme' comes from global defaults.
```


## Profile\

This namespace contains Redis server profile definitions.

### Factory

Manages Redis server profiles - their versions, understood commands and so on. It's a static class that can't be instantiated.

#### static getDefault()

Returns a default `ProfileInterface`. It's usually last stable version, at this time - of Redis 3.2.

#### static getDevelopment()

Returns `ProfileInterface` of an latest unstable Redis version. Usually it's the same as `getDefault`.

#### static define($alias, $class)

Registers new server profile named `$alias` and handled by `$class` (must be a subclass of `ProfileInterface` or this will fail). Can be used to override a default profile.

```PHP
Profile\Factory::define('dev', 'MyRedisProfile');
```

#### static get($alias)

Returns a singleton instance of server profile `$alias`. Throws `ClientException` if `$alias` is not registered.

`getDefault` and `getDevelopment` are convenient shortcuts to `get('default')` and `get('dev')`.


## PubSub\

This namespace contains abstractions for receiving messages on Redis pub/sub channels.

### AbstractConsumer

**Implements**: `Iterator`.

This is the base class used to implement subscriber's logic in pub/sub context.

It implements `Iterator` so it can be used in `foreach`. However, it doesn't support rewinding because pub/sub can only go forward (rewind calls are ignored) - this means if a `foreach` loop stops, next `foreach` will resume from the first message not handled by previous loop.

```PHP
foreach ($predis->pubSubLoop(['subscribe' => 'x']) as $i => $msg) {
  // First $i is null, then 1 2 3 and so on.
  // $msg is an object with at least kind, channel and payload properties.
}
```

The context is automatically stopped and connection closed (by `stop(true)`) when the object is destroyed.

#### isFlagSet($value)

Returns `true` if this context has all `$value` state bits set, `false` otherwise.

A pub/sub context can be in these states:

* `STATUS_VALID` - the pub/sub context is active, not stopped; it begins in this state
* `STATUS_SUBSCRIBED` - if this context has subscribed to any channels by exact name
* `STATUS_PSUBSCRIBED` - if this context has subscribed to any channels by pattern

The following example returns `true` if this context is valid and has subscribed to at least one channel by pattern:
```PHP
$pubSub->isFlagSet($pubSub::STATUS_VALID | $pubSub::STATUS_PSUBSCRIBED);
```

#### subscribe($channel[, $channel[...]])

Subscribes to one or more channels by exact name.

#### unsubscribe([$channel[, $channel[...]]])

Unsubscribes from one or more channels by exact name (skips those that it didn't subscribe to).

Without arguments unsubscribes from all channels that were subscribed to by exact name.

#### psubscribe($pattern[, $pattern[...]])

Subscribes to one or more channels by pattern (e.g. `foo:*`).

#### punsubscribe([$pattern[, $pattern[...]]])

Unsubscribes from one or more channels by pattern (skips those that it didn't subscribe to).

Without arguments unsubscribes from all channels that were subscribed to by pattern.

#### ping([$payload])

Pings the server; if the connection is alive Redis will respond with `PONG`. If `$payload` is given, it's included in the response.

Response can be retrieved as any other pub/sub message:
```PHP
$predis->pubSubLoop(function ($l, $msg) {
  $l->ping('got you');
  // Look for $msg->kind === 'pong' and $msg->payload === 'got you'.
  ...
});
```

#### stop([bool $drop = false])

Closes this pub/sub context.

If `$drop` is `true`, closes the main connection (used by `Client` object). Useful to hard-reset connection context.

If it's `false`, only unsubscribes from all channels (by name and pattern) so it's possible to issue regular Redis commands, or subscribe again.

**Returns** `true` if the context remains open, `false` if not (inclluding if the context was already closed).


Example of reusing subscription:
```PHP
$l = $predis->pubSubLoop();
...
$l->stop();   //=> true
$l->subscribe('new:chan');

// $l can be used again to read messages.
// But don't issue Redis commands - result will be unexpected:
$predis->get('s');
  //=> ['subscribe', 'x', 1]
  // or Redis will reply with aa invalid-in-this-context error.
```

Example of calling normal Redis commands while reusing the connection:
```PHP
$l = $predis->pubSubLoop();
...
$l->stop();   //=> true

// $l can be used again to read messages or to issue Redis commands.
$predis->get('s');
  //=> string(1) "svalue"
```

Example of hard-resetting the connection:
```PHP
$l = $predis->pubSubLoop();
...
$l->stop(true);   //=> false

// $l can no more be used but $predis can be.
$predis->get('s');
  //=> string(1) "svalue"

// Throws a ServerException about bad context.
$l->subscribe('foo');
```

#### protected disconnect()

Close the main connection used by `Client`. This is a protected method, use `stop(true)` instead.

#### valid()

Returns `true` if this context is open and it has subscribed to at least one channel by name or pattern.

### DispatcherLoop

A wrapper around Predis `Client->pubSubLoop()` object that allows registration of callbacks based on the channel's name.

#### __construct(Consumer)

Creates a dispatcher instance.

```PHP
new DispatcherLoop($predis->pubSubLoop())
```

#### getPubSubConsumer()

Returns a `Consumer` instance this dispatcher was created with.

#### subscriptionCallback([callable $callback = null])

Sets or unsets (if `null`) a callback invoked after subscribing to or unsubscribing from a channel.

The callback is given entire message object with `kind`, `channel` and other properties (see `pubSubLoop()`). Result is ignored.

#### defaultCallback([callable $callback = null])

Sets or unsets (if `null`) a callback invoked when a message arrives on a channel without its own callback.

The callback is given entire message object with `kind`, `channel` and other properties (see `pubSubLoop()`). Result is ignored.

#### attachCallback($channel, callable $callback)

Subscribes to `$channel` by exact name (to `message` and `pmessage` message kinds). Whenever a new message arrives, `$callback` is invoked, but only for that channel. Other channels' messages are given to `defaultCallback()`, if any.

`$callback` cannot be `null` - use `detachCallback()` to unsubscribe.

The callback is given message's `payload`.

#### detachCallback($channel)

Unsubscribes from a channel, if it was subscribed to. Its callback will no more be called.

#### run()

Starts the loop, receiving messages and invoking registered callbacks.

#### stop()

Stops the running pub/sub loop.


## Session\

This namespace contains a handler for managing PHP sessions using Predis.

### Handler

**Implements**: `\SessionHandlerInterface`.

A supporting class for storing PHP sessions using Predis in one or more Redis servers.

```PHP
$h = new Session\Handler($predis);
$h->register();

session_start();
$_SESSION['foo'] = 'value';

echo $predis->get('foo');
  //=> value
```

#### __construct(ClientInterface[, array $options])

Creates a new instance but doesn't register new handler with PHP.

`$options` is an associative array with the following optional key:

* `gc_maxlifetime` - number of seconds any session value is kept at most (Redis' `TTL`); if unset `session.gc_maxlifetime` is used from `php.ini` (which defaults to 24 minutes).

#### register()

Registers this instance as the current PHP session handler. After this `session_start()` can be called.

#### getClient()

Returns the original `Client` object this handler uses to manage session data.

#### getMaxLifeTime()`

Returns the number of seconds any session value is kept for at most. Corresponds to Redis' `TTL`.

