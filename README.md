Formatted version of this page is available online: http://squizzle.me/php/predis/doc/

# Predis - a PHP interface to Redis

This documentation is based on [Predis](https://github.com/nrk/predis) 1.1.2-dev.

Source Markdown files are available [on GitHub](https://github.com/ProgerXP/predis-doc) or directly at [this path](md/Commands.md).

Please submit corrections [via GitHub](https://github.com/ProgerXP/predis-doc/issues).

## Documentation map

* [Classes](Classes.md) - in-depth reference to Predis classes
* [Commands](Commands.md) - Redis command reference and their Predis implementation details
* [Configuration](Configuration.md) - server connection and Predis options

### Calling Redis commands

```PHP
$oldValue = $predis->getset('skey', 'newvalue');
var_dump($oldValue);    //=> string(3) "foo"
```

[Command reference](Commands.md): supported Redis commands (`GETSET`, `SORT`, `HGET`, etc.) and their arguments.

[The `Client` class API](Classes.md#client).

### Connection configuration

```PHP
$connections = [
  ['database' => 3, 'alias' => 'primary'],
  ['host' => 'backupredis.example.com', 'scheme' => 'tls'],
];

$predis = new Predis\Client($connections);
```

[Using a connection string](Classes.md#parse):
```PHP
$predis = new Predis\Client('unix:///var/run/redis.sock?database=5');
```

[The **connections** option](Configuration.md#connections): backends (`PhpiredisStreamConnection`) and their schemes (`tcp`, `unix`, etc.).

### Predis options

```PHP
$options = ['exceptions' => false];
$predis = new Predis\Client([], $options);
```

[Configuration reference](Configuration.md): supported Predis options for `Client` constructor.

### Default connection parameters

```PHP
$connections = [
  ['alias' => 'primary'],
  ['port' => 6380, 'alias' => 'backup'],
];

$options = [['parameters' => ['host' => 'redis.example.com', 'scheme' => 'tls']];

$predis = new Predis\Client($connections, $options);
```

The above is equivalent to:
```PHP
$connections = [
  [
    'host' => 'redis.example.com',
    'scheme' => 'tls',
    'alias' => 'primary',
  ],
  [
    'host' => 'redis.example.com',
    'scheme' => 'tls',
    'port' => 6380,
    'alias' => 'backup',
  ],
];

$predis = new Predis\Client($connections);
```

[The **parameters** option](Configuration.md#parameters).

### Pipelining

[Basic](Classes.md#pipeline):
```PHP
$res = $predis->pipeline(function ($p) {
  $p->set('skey', 'v');
  $p->hmset('hkey', ['k' => 'v', 'k2' => 'v2']);
  $p->get('skey');
  $p->get('skey2');
});

// $res has 4 members - one per each command ran.
```

Combined with a transaction ([options](Classes.md#pipeline)):
```PHP
$res = $predis->pipeline(['atomic' => true], function ($p) {
  ...
});
```

[Custom transaction nesting](Classes.md#nested):
```PHP
$res = $predis->pipeline(function ($p) {
  $p->get('notintrans');

  $p->multi();
  $p->set('skey', 'intrans');
  $p->exec();

  $p->get('againnotintrans');
});

// $res has 5 members, with array 4th (EXEC) containng 1 member.
```

Using fluent interface:
```PHP
$res = $predis->pipeline(['atomic' => true])
  ->get('skey')
  ->set('skey', 'value')
  ->execute();
```

### Transactions

[Basic](Classes.md#transaction):
```PHP
$res = $predis->transaction(function ($t) {
  $t->get('skey');
  $t->hset('hkey', 'k', 'v');
});

// $res has 2 members.
```

Faining on key(s) changes (`WATCH`) with up to two retries:
```PHP
try {
  $res = $predis->transaction(['watch' => 'wkey', 'retry' => 2], function ($t) {
    ...
  });
} catch (Transaction\AbortedMultiExecException $e) {
  die("Still couldn't save the changes after 3 attempts.");
}
```

[Compare-And-Swap](Classes.md#cas):
```PHP
$res = $predis->transaction(['watch' => 'wkey', 'cas' => true], function ($t) {
  $value = $t->get('wkey');
  $t->multi();
  $t->set('wkey', $value.'foo');
});
```

Using fluent interface:
```PHP
$res = $predis->transaction(['exceptions' => false])
  ->set('k1', 'v1')
  ->set('k2', 'v2')
  ->execute();

// With 'exceptions' unset, failed commands will return an ErrorInterface object instead of throwing ServerException.
```

### Pub/Sub subscription

[Basic](Classes.md#pubsubloop):
```PHP
$predis->pubSubLoop(['subscribe' => 'chan'], function ($l, $msg) {
  if ($msg->payload === 'Q') {
    return false;
  } else {
    echo "$msg->payload on $msg->channel", PHP_EOL;
  }
});
```
Using a [consumer object](Classes.md#abstractconsumer):
```PHP
$l = $predis->pubSubLoop(['subscribe' => 'chan']);

foreach ($l as $msg) {
  if ($msg->payload === 'unsub') {
    $l->unsubscribe('chan');
  } elseif ($msg->payload === 'psub') {
    $l->psubscribe('chan:*');
  } elseif ($msg->payload === 'Q') {
    $l->stop();
  }
}

echo 'pub/sub has stopped.';
```

Using [per-channel callbacks](Classes.md#dispatcherloop):
```PHP
$l = predis->pubSubLoop();
$dl = new Predis\PubSub\DispatcherLoop($l);

$dl->attachCallback('chan1', function ($payload) {
  echo "Got $payload on chan1.", PHP_EOL;
});

$dl->attachCallback('ctlchan', function ($payload) use ($dl) {
  echo "Received a message on ctlchan, stopping.";
  $dl->stop();
});

$dl->defaultCallback(function ($msg) {
  echo "Received a message on $msg->channel.", PHP_EOL;
});

$l->psubscribe('foo:*');

$l->run();
```

### Iterating over keys and members

[All keys in a database](Classes.md#keyspace) (`SCAN`):
```PHP
$it = new Predis\Collection\Iterator\Keyspace($predis);

foreach ($it as $key) {
  echo "Found a key named '$key'", PHP_EOL;
}
```

[Hash fields](Classes.md#hashkey) (`HSCAN`):
```PHP
$it = new Predis\Collection\Iterator\HashKey($predis, 'hkey');

foreach ($it as $key => $value) {
  echo "Found a field '$key', value '$value' in hash key 'hkey'", PHP_EOL;
}
```

[Set members](Classes.md#setkey) (`SSCAN`):
```PHP
$it = new Predis\Collection\Iterator\SetKey($predis, 'setkey');

foreach ($it as $member) {
  echo "Found a member '$member' in set key 'setkey'", PHP_EOL;
}
```

[Sorted set members](Classes.md#sortedsetkey) (`ZSCAN`):
```PHP
$it = new Predis\Collection\Iterator\SortedSetKey($predis, 'zkey');

foreach ($it as $member => $score) {
  echo "Found a member '$member', score '$score' in sorted set key 'zkey'", PHP_EOL;
}
```

[List values](Classes.md#listkey) (emulation with `LRANGE`):
```PHP
$it = new Predis\Collection\Iterator\ListKey($predis, 'lkey');

foreach ($it as $member) {
  echo "Found a member '$member' in list key 'lkey'", PHP_EOL;
}
```

### Registering Lua scripts

```PHP
class EchoScript extends Predis\Command\ScriptCommand {
  function getScript() {
    return 'return ARGV[1]';
  }
}

$predis->getProfile()->defineCommand('echolua', 'EchoScript');

echo $predis->echolua('foo');
  //=> foo
```

[The `ScriptCommand` Class](Classes.md#scriptcommand).

### Session handler

```PHP
$h = new Session\Handler($predis);
$h->register();
session_start();
```

[The `Session\Handler` class](Classes.md#session): using Redis for storing session data.

### Other topics

* [List of commands](Configuration.md#supported) supported by target Redis server (Predis "profile")
* [List of Predis exceptions](Classes.md#list)
* [List of Predis interfaces](Classes.md#list-of)
* Details about default [StreamConnection](Classes.md#streamconnection) used for handling `tcp`, `tls`, `unix` and `redis`/`rediss` schemes,
* ...about [WebdisConnection](Classes.md#webdisconnection) handling `http` scheme, about `phpiredis` using [PHP streams](Classes.md#phpiredisstreamconnection) and [sockets extension](Classes.md#phpiredissocketconnection)


## Syntax conventions

Function definitions include arguments type hinting similar to those used in the PHP manual. In cases no type hint is present, a string type - `str` - is implied. Usually if an invalid type is given there will be a PHP error or implicit coersion so don't do that.

Sometimes argument `$name` is omitted and only its type hint is left for clarity. This is often done with object arguments: `SomeInterface $some` is written as just `SomeInterface`.

The `$key` argument always contains name of a Redis key entry such as `foo:123`. `$src` and `$dest` arguments are similar but additionally indicate that the operation will take data from (`$src`) or put data into (`$dest`) them.

If arguments are omitted with an ellipsis: `brpop(...)` - they are entirely dictated by another source indicated by the `See` reference in the description.

Many functions are variadic, i.e. accepting arbitrary number of arguments. Some functions even have several "variadic" argument groups. Variability is indicated by the `[...]` pattern (where square brackets mean optional data as per standard BNF notation). For example:
```PHP
eval($script, $numKeys[, $key[...]][, $arg[...]]);
```

The definition above contains 2 variadic groups: `$key` and `$arg`. Examples of proper invocation:
```PHP
eval('s', 1, 'k', 'v', 'v2');
eval('s', 1, 'k');
eval('s', 0, 'v', 'v2');
eval('s', 0);
```

Examples in the Commands section omit object reference from method invokaction for clarity. Thus, the above example in real code looks like this:
```PHP
$predis = new Predis;
$predis->eval('s', 1, 'k', 'v', 'v2');
...
```

All examples assume their base namespace as `Predis` so that `Command\HashGet` refers to `Predis\Command\HashGet`. In some cases class names are further reduced since most are unique Predis-wise. Thus `ConnectionException` refers to `Predis\Connection\ConnectionException`.

The `$predis` variable in the examples refers to a previously set up `Predis\Client` object.



