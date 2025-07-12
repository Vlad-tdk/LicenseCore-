# Hardware Fingerprint Caching System

## Overview

Caching in LicenseCore improves performance by storing hardware component detection results in memory for a specified time. This is especially important for applications that frequently check hardware fingerprints.

## Configuration

### Basic Configuration

```cpp
HardwareConfig config;
config.enable_caching = true;                          // Enable caching
config.cache_lifetime = std::chrono::minutes(5);       // Cache lifetime
config.thread_safe_cache = true;                       // Thread-safe cache
```

### Cache Lifetime Options

```cpp
// Various cache lifetime options
config.cache_lifetime = std::chrono::seconds(30);      // 30 seconds
config.cache_lifetime = std::chrono::minutes(5);       // 5 minutes (default)
config.cache_lifetime = std::chrono::hours(1);         // 1 hour
```

## Usage Examples

### 1. Basic Caching

```cpp
HardwareConfig config;
config.enable_caching = true;
config.cache_lifetime = std::chrono::minutes(5);

HardwareFingerprint fingerprint(config);

// First call - cache miss (slow)
std::string fp1 = fingerprint.get_fingerprint();

// Subsequent calls - cache hit (fast)
std::string fp2 = fingerprint.get_fingerprint();
std::string fp3 = fingerprint.get_fingerprint();
```

### 2. Performance Comparison

```cpp
auto measure_time = [](auto func) {
    auto start = std::chrono::high_resolution_clock::now();
    func();
    auto end = std::chrono::high_resolution_clock::now();
    return std::chrono::duration_cast<std::chrono::microseconds>(end - start);
};

// With caching
HardwareConfig cached_config;
cached_config.enable_caching = true;
HardwareFingerprint cached_fp(cached_config);

// Without caching
HardwareConfig no_cache_config;
no_cache_config.enable_caching = false;
HardwareFingerprint no_cache_fp(no_cache_config);

// First call (same time)
auto time1 = measure_time([&]() { cached_fp.get_fingerprint(); });
auto time2 = measure_time([&]() { no_cache_fp.get_fingerprint(); });

// Second call (cache works only for cached_fp)
auto time3 = measure_time([&]() { cached_fp.get_fingerprint(); });  // Fast
auto time4 = measure_time([&]() { no_cache_fp.get_fingerprint(); }); // Slow
```

### 3. Cache Statistics

```cpp
HardwareFingerprint fingerprint(config);

// Perform several calls
fingerprint.get_fingerprint();
fingerprint.get_fingerprint();
fingerprint.get_fingerprint();

// Check statistics
auto stats = fingerprint.get_cache_stats();
std::cout << "Cache hits: " << stats.cache_hits << std::endl;
std::cout << "Cache misses: " << stats.cache_misses << std::endl;
std::cout << "Hit rate: " << (stats.hit_rate() * 100) << "%" << std::endl;
```

### 4. Cache Management

```cpp
HardwareFingerprint fingerprint(config);

// Check cache validity
if (fingerprint.is_cache_valid()) {
    std::cout << "Cache is valid" << std::endl;
}

// Force cache clear
fingerprint.clear_cache();

// Or invalidation (same thing)
fingerprint.invalidate_cache();

// Last cache update time
auto cache_time = fingerprint.get_cache_time();
```

## Thread Safety

### Thread-Safe Mode (Default)

```cpp
HardwareConfig config;
config.thread_safe_cache = true; // Default

HardwareFingerprint fingerprint(config);

// Safe to use from different threads
std::thread t1([&]() { fingerprint.get_fingerprint(); });
std::thread t2([&]() { fingerprint.get_fingerprint(); });
t1.join();
t2.join();
```

### Non-Thread-Safe Mode (Faster)

```cpp
HardwareConfig config;
config.thread_safe_cache = false; // Faster, but not thread-safe

HardwareFingerprint fingerprint(config);

// Use only from one thread!
std::string fp = fingerprint.get_fingerprint();
```

## Cache Expiration

### Automatic Expiration

```cpp
HardwareConfig config;
config.cache_lifetime = std::chrono::seconds(2);

HardwareFingerprint fingerprint(config);

// First call
std::string fp1 = fingerprint.get_fingerprint(); // Cache miss

// Second call (within lifetime)
std::string fp2 = fingerprint.get_fingerprint(); // Cache hit

// Wait for cache expiration
std::this_thread::sleep_for(std::chrono::seconds(3));

// Third call (cache expired)
std::string fp3 = fingerprint.get_fingerprint(); // Cache miss
```

### Manual Cache Control

```cpp
HardwareFingerprint fingerprint(config);

// Build cache
fingerprint.get_fingerprint();

// Check validity
if (fingerprint.is_cache_valid()) {
    std::cout << "Cache is still valid" << std::endl;
}

// Force clear
fingerprint.clear_cache();

// Check again
if (!fingerprint.is_cache_valid()) {
    std::cout << "Cache was cleared" << std::endl;
}
```

## Performance Benefits

### Typical Performance Gains

- **First call**: 1000-5000 μs (system dependent)
- **Cached calls**: 1-10 μs
- **Speedup**: 100-1000x for cached calls

### When to Use Caching

**Use caching when:**

- Application frequently checks hardware fingerprint
- Performance is critical
- Hardware fingerprint doesn't change often

**Disable caching when:**

- Need guarantee of fresh data on every call
- Memory usage is critical
- Hardware may change dynamically

## Configuration Examples

### High-Frequency Checks

```cpp
// For applications with frequent checks
HardwareConfig config;
config.enable_caching = true;
config.cache_lifetime = std::chrono::minutes(30);
config.thread_safe_cache = true;
```

### Security-Critical Applications

```cpp
// For security-critical applications
HardwareConfig config;
config.enable_caching = true;
config.cache_lifetime = std::chrono::seconds(30); // Short lifetime
config.thread_safe_cache = true;
```

### Single-Threaded Performance

```cpp
// For maximum performance (single thread)
HardwareConfig config;
config.enable_caching = true;
config.cache_lifetime = std::chrono::minutes(10);
config.thread_safe_cache = false; // Disable mutexes
```

### Real-Time Applications

```cpp
// For real-time applications
HardwareConfig config;
config.enable_caching = false; // Disable caching for predictability
```

## Best Practices

1. **Cache lifetime selection**: Balance between performance and data freshness
2. **Thread safety**: Enable only if needed (overhead ~10-20%)
3. **Hit rate monitoring**: Aim for >90% for effective caching
4. **Cache clearing**: When hardware configuration changes
5. **Memory usage**: Cache takes ~1KB per instance

## Error Handling with Caching

```cpp
try {
    HardwareFingerprint fingerprint(config);
    std::string fp = fingerprint.get_fingerprint();

    // Cache automatically stores successful results
    // On errors, cache is not updated

} catch (const HardwareDetectionException& e) {
    // Error is not cached - next call will try again
    std::cerr << "Hardware detection failed: " << e.what() << std::endl;
}
```

Caching in LicenseCore provides significant performance improvement while maintaining reliability and security of the licensing system.
