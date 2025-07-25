# Lock

The RoadRunner lock plugin is a powerful tool that enables you to manage resource locks in their applications using the 
RPC protocol. By leveraging the benefits of using GO with PHP, it provides a lightweight, fast, and reliable way to
acquire, release, and manage locks. With this plugin, you can easily manage critical sections of your application and 
prevent race conditions, data corruption, and other synchronization issues that can occur in multiprocess environments.

{% hint style="warning" %}
RoadRunner lock plugin uses an in-memory storage to store information about locks at this moment. When multiple
instances of RoadRunner are used, each instance will have its own in-memory storage for locks. As a result, if a
process
acquires a lock on one instance of RoadRunner, it will not be aware of the lock state on the other instances.
{% endhint %}

## PHP client

The RoadRunner lock plugin comes with a convenient PHP package that simplifies the process of integrating the
plugin with your PHP application.

### Installation

To get started, you can install the package via Composer using the following command:

{% code %}

```bash
composer require roadrunner-php/lock
```

{% endcode %}

### Usage

After the installation, you can create an instance of the `RoadRunner\Lock\Lock` class, which will allow you to use the
available class methods.

**Here is an example:**

{% code title="app.php" %}

```php
use RoadRunner\Lock\Lock;
use Spiral\Goridge\RPC\RPC;

require __DIR__ . '/vendor/autoload.php';

$lock = new Lock(RPC::create('tcp://127.0.0.1:6001'));
```

{% endcode %}

{% hint style="warning" %}
To interact with the RoadRunner lock plugin, you will need to have the RPC defined in the rpc configuration
section. You can refer to the documentation page [here](../php/rpc.md) to learn more about the configuration and
installation.
{% endhint %}

The `RoadRunner\Lock\Lock` class provides four methods that allow you to manage locks:

#### Acquire lock

Locks a resource so that it can be accessed by one process at a time. When a resource is locked, other processes that
attempt to lock the same resource will be blocked until the lock is released.

{% code title="app.php" %}

```php
$id = $lock->lock('pdf:create');

// Acquire lock with ttl - 10 microseconds
$id = $lock->lock('pdf:create', ttl: 10);
// or
$id = $lock->lock('pdf:create', ttl: new \DateInterval('PT10S'));

// Acquire lock and wait 5 microseconds until lock will be released
$id = $lock->lock('pdf:create', waitTTL: 5);
// or
$id = $lock->lock('pdf:create', waitTTL: new \DateInterval('PT5S'));

// Acquire lock with id - 14e1b600-9e97-11d8-9f32-f2801f1b9fd1
$id = $lock->lock('pdf:create', id: '14e1b600-9e97-11d8-9f32-f2801f1b9fd1');
```

{% endcode %}

#### Acquire read lock

Locks a resource for shared access, allowing multiple processes to access the resource simultaneously. When a resource
is locked for shared access, other processes that attempt to lock the resource for exclusive access will be blocked
until all shared locks are released.

{% code title="app.php" %}

```php
$id = $lock->lockRead('pdf:create', ttl: 100000);
// or
$id = $lock->lockRead('pdf:create', ttl: new \DateInterval('PT10S'));

// Acquire lock and wait 5 microseconds until lock will be released
$id = $lock->lockRead('pdf:create', waitTTL: 5);
// or
$id = $lock->lockRead('pdf:create', waitTTL: new \DateInterval('PT5S'));

// Acquire lock with id - 14e1b600-9e97-11d8-9f32-f2801f1b9fd1
$id = $lock->lockRead('pdf:create', id: '14e1b600-9e97-11d8-9f32-f2801f1b9fd1');
```

{% endcode %}

#### Release lock

Releases an exclusive lock or read lock on a resource that was previously acquired by a call to `lock()`
or `lockRead()`.

{% code title="app.php" %}

```php
// Release lock after task is done.
$lock->release('pdf:create', $id);

// Force release lock
$lock->forceRelease('pdf:create');
```

{% endcode %}

#### Check lock

Checks if a resource is currently locked and returns information about the lock.

{% code title="app.php" %}

```php
$status = $lock->exists('pdf:create');
if($status) {
    // Lock exists
} else {
    // Lock not exists
}
```

{% endcode %}

#### Update TTL

Updates the time-to-live (TTL) for the locked resource.

{% code title="app.php" %}

```php
// Add 10 microseconds to lock ttl
$lock->updateTTL('pdf:create', $id, 10);
// or
$lock->updateTTL('pdf:create', $id, new \DateInterval('PT10S'));
```

{% endcode %}

## Symfony integration

### Installation

You can install the package via composer:

{% code %}

```bash
composer require roadrunner-php/symfony-lock-driver
```

{% endcode %}

### Usage

{% code title="app.php" %}

```php
use RoadRunner\Lock\Lock;
use Spiral\Goridge\RPC\RPC;
use Spiral\RoadRunner\Symfony\Lock\RoadRunnerStore;
use Symfony\Component\Lock\LockFactory;

require __DIR__ . '/vendor/autoload.php';

$lock = new Lock(RPC::create('tcp://127.0.0.1:6001'));
$factory = new LockFactory(
    new RoadRunnerStore($lock)
);
```

{% endcode %}

Read more about using a Symfony Lock component [here](https://symfony.com/doc/current/components/lock.html).

## API

### Protobuf API

To make it easy to use the Lock proto API in PHP, we provide
a [GitHub repository](https://github.com/roadrunner-php/roadrunner-api-dto), that contains all the generated
PHP DTO classes proto files, making it easy to work with these files in your PHP application.

- [API](https://buf.build/roadrunner-server/api/file/main:lock/v1beta1/lock.proto)

### RPC API

RoadRunner provides an RPC API, which allows you to manage locks in your applications using remote procedure calls. The 
RPC API provides a set of methods that map to the available methods of the `RoadRunner\Lock\Lock` class in PHP.

### Lock

Acquires an exclusive lock on a resource so that it can be accessed by one process at a time. When a resource is locked,
other processes that attempt to lock the same resource will be blocked until the lock is released.

{% code %}

```go
func (r *rpc) Lock(req *lockApi.Request, resp *lockApi.Response) error {}
```

{% endcode %}

### LockRead

Acquires a read lock on a resource, allowing multiple processes to access the resource simultaneously. When a resource
is locked for shared access, other processes that attempt to lock the resource for exclusive access will be blocked
until all shared locks are released.

{% code %}

```go
func (r *rpc) LockRead(req *lockApi.Request, resp *lockApi.Response) error {}
```

{% endcode %}

#### Release

Releases an exclusive lock or a read lock on a resource that was previously acquired by a call to `Lock` or `LockRead`.

{% code %}

```go
func (r *rpc) Release(req *lockApi.Request, resp *lockApi.Response) error {}
```

{% endcode %}

#### ForceRelease

Releases all locks on a resource, regardless of which process acquired them.

{% code %}

```go
func (r *rpc) ForceRelease(req *lockApi.Request, resp *lockApi.Response) error {}
```

{% endcode %}

#### Exists

Checks if a resource is currently locked and returns information about the lock.

{% code %}

```go
func (r *rpc) Exists(req *lockApi.Request, resp *lockApi.Response) error {}
```

{% endcode %}

#### UpdateTTL

Updates the time-to-live (TTL) for the locked resource.

{% code %}

```go
func (r *rpc) UpdateTTL(req *lockApi.Request, resp *lockApi.Response) error {}
```

{% endcode %}
