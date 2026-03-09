# @philiprehberger/cache-kit

In-memory LRU cache with TTL, stale-while-revalidate, and tag invalidation.

## Installation

```bash
npm install @philiprehberger/cache-kit
```

## Usage

### Basic

```ts
import { createCache } from '@philiprehberger/cache-kit';

const cache = createCache({ maxItems: 1000, defaultTTL: '5m' });

cache.set('user:123', userData);
cache.set('user:456', otherUser, { ttl: '10m' });

const user = cache.get('user:123'); // T | undefined
cache.has('user:123');              // boolean
cache.delete('user:123');           // boolean
cache.clear();
```

### TTL Durations

Supports string durations or milliseconds:

```ts
cache.set('key', value, { ttl: '30s' });  // 30 seconds
cache.set('key', value, { ttl: '5m' });   // 5 minutes
cache.set('key', value, { ttl: '1h' });   // 1 hour
cache.set('key', value, { ttl: '1d' });   // 1 day
cache.set('key', value, { ttl: 60000 });  // 60000ms
```

### Tag-Based Invalidation

```ts
cache.set('user:1', user1, { tags: ['users'] });
cache.set('user:2', user2, { tags: ['users'] });
cache.set('post:1', post1, { tags: ['posts'] });

cache.invalidateTag('users'); // removes user:1 and user:2
```

### Wrap (Memoize)

```ts
const getUser = cache.wrap(
  'user',
  (id: string) => db.users.findById(id),
  { ttl: '5m' },
);

const user = await getUser('123'); // fetches from DB
const same = await getUser('123'); // served from cache
```

### Stale-While-Revalidate

```ts
const getUser = cache.wrap(
  'user',
  (id: string) => db.users.findById(id),
  { ttl: '5m', staleWhileRevalidate: '1m' },
);
// Serves stale data immediately while refreshing in the background
```

### Stats

```ts
cache.stats();
// { hits: 150, misses: 23, hitRate: 0.867, size: 42 }
```

### Persistence

```ts
const data = cache.dump();   // serialize to JSON
cache.load(data);            // restore from JSON
```

## License

MIT
