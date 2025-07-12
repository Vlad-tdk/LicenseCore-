# 📚 LicenseCore++ - Main Library Documentation

## 📋 Overview

The main LicenseCore++ library provides a complete C++ API for license management. It includes `HardwareFingerprint`, `LicenseManager`, `HMACValidator` classes and a comprehensive exception system.

---

## 🔧 Project Integration

### CMake (Recommended)

```cmake
cmake_minimum_required(VERSION 3.16)
project(MyProject)

# Find LicenseCore++
find_package(LicenseCore REQUIRED)

# Create executable
add_executable(myapp main.cpp)

# Link library
target_link_libraries(myapp LicenseCore::licensecore)

# Set C++ standard
set_property(TARGET myapp PROPERTY CXX_STANDARD 17)
```

### Makefile

```makefile
CXX = g++
CXXFLAGS = -std=c++17 -Wall -Wextra -O2
INCLUDES = -I/path/to/licensecore/include
LIBS = -L/path/to/licensecore/lib -llicensecore

# Platform-dependent libraries
ifeq ($(shell uname -s),Darwin)
    PLATFORM_LIBS = -framework IOKit -framework CoreFoundation
else ifeq ($(shell uname -s),Linux)
    PLATFORM_LIBS = -lpthread
else
    PLATFORM_LIBS = -liphlpapi -lole32 -loleaut32
endif

myapp: main.cpp
 $(CXX) $(CXXFLAGS) $(INCLUDES) -o $@ $< $(LIBS) $(PLATFORM_LIBS)
```

---

## 💻 API Reference

### HardwareFingerprint Class

```cpp
#include "license_core/hardware_fingerprint.hpp"
using namespace license_core;
```

#### Configuration

```cpp
struct HardwareConfig {
    bool use_cpu_id = true;                              // CPU identifier
    bool use_mac_address = true;                         // MAC address
    bool use_volume_serial = true;                       // Volume serial number
    bool use_motherboard_serial = false;                 // Motherboard serial number

    // Caching settings
    std::chrono::seconds cache_lifetime{300};            // Cache lifetime (5 minutes)
    bool enable_caching = true;                          // Enable caching
    bool thread_safe_cache = true;                       // Thread safety
};
```

#### Constructor

```cpp
HardwareFingerprint(const HardwareConfig& config = HardwareConfig{});
```

#### Core Methods

```cpp
// Hardware fingerprint generation
std::string get_fingerprint() const;           // May throw exceptions
std::string get_fingerprint_safe() const;      // Safe version, no exceptions

// Individual component access
std::string get_cpu_id() const;               // CPU identifier
std::string get_cpu_id_safe() const;          // Safe version
std::string get_mac_address() const;          // MAC address
std::string get_mac_address_safe() const;     // Safe version
std::string get_volume_serial() const;        // Volume serial number
std::string get_motherboard_serial() const;   // Motherboard serial number

// Cache management
void clear_cache() const;                      // Clear cache
void invalidate_cache() const;                 // Invalidate cache
bool is_cache_valid() const;                   // Check cache validity
CacheStats get_cache_stats() const;           // Get cache statistics
```

### LicenseManager Class

```cpp
#include "license_core/license_manager.hpp"
using namespace license_core;
```

#### License Information Structure

```cpp
struct LicenseInfo {
    std::string user_id;                                         // User identifier
    std::string hardware_hash;                                   // Hardware fingerprint
    std::vector<std::string> features;                          // Feature list
    std::chrono::system_clock::time_point expiry;               // Expiration date
    std::chrono::system_clock::time_point issued_at;            // Issuance date
    std::string license_id;                                      // License identifier
    uint32_t version = 1;                                        // Format version
    bool valid = false;                                          // Validity flag
    std::string error_message;                                   // Error message
};
```

#### Constructor

```cpp
LicenseManager(const std::string& secret_key);
```

#### Core Methods

```cpp
// Load and validate license from JSON
LicenseInfo load_and_validate(const std::string& license_json);

// Validate license with specified hardware ID
bool validate_license(const std::string& license_json, const std::string& hardware_id) const;

// Check if feature exists in current license
bool has_feature(const std::string& feature) const;

// Require feature presence (throws exception if feature is missing)
void require_feature(const std::string& feature) const;

// Generate signed license
std::string generate_license(const LicenseInfo& info) const;

// Check if current license is expired
bool is_expired() const;

// Get list of available features
std::vector<std::string> get_available_features() const;

// Get current Hardware ID
std::string get_current_hwid() const;

// Configuration
void set_hardware_config(const HardwareConfig& config);
void set_strict_validation(bool strict = true);
```

#### Utility methods for date handling

```cpp
static std::chrono::system_clock::time_point parse_iso8601(const std::string& date_str);
static std::string format_iso8601(const std::chrono::system_clock::time_point& time_point);
```

### HMACValidator Class

```cpp
#include "license_core/hmac_validator.hpp"
using namespace license_core;
```

#### Constructor

```cpp
HMACValidator(const std::string& secret_key);
```

#### Methods

```cpp
// Create HMAC signature for data
std::string sign(const std::string& data) const;

// Verify HMAC signature
bool verify(const std::string& data, const std::string& signature) const;
void verify_or_throw(const std::string& data, const std::string& signature) const;

// Work with JSON licenses
std::string sign_json(const std::string& json_without_signature) const;
bool verify_json(const std::string& json_with_signature) const;
void verify_json_or_throw(const std::string& json_with_signature) const;

// License validation
bool validate_license(const LicenseInfo& license_info, const std::string& hardware_id) const;
```

### Exception System

```cpp
#include "license_core/exceptions.hpp"
using namespace license_core;
```

#### Exception Hierarchy

```cpp
// Base class for all library exceptions
class LicenseException : public std::exception {
public:
    const char* what() const noexcept override;
    const std::string& message() const noexcept;
};

// Specific exceptions
class InvalidSignatureException : public LicenseException {};       // Invalid signature
class ExpiredLicenseException : public LicenseException {};         // Expired license
class HardwareMismatchException : public LicenseException {};       // Hardware mismatch
class MalformedLicenseException : public LicenseException {};       // Malformed license
class MissingFeatureException : public LicenseException {};         // Missing feature
class HardwareDetectionException : public LicenseException {};      // Hardware detection error
class CryptographicException : public LicenseException {};          // Cryptographic error
class JsonParsingException : public MalformedLicenseException {};   // JSON parsing error
class ValidationException : public LicenseException {};             // General validation error
class NotInitializedException : public LicenseException {};         // Component not initialized
```

---

## 🎯 Usage Examples

### Basic Usage

```cpp
#include <license_core/license_manager.hpp>
#include <iostream>

using namespace license_core;

int main() {
    try {
        // Create license manager
        LicenseManager manager("your-secret-key");

        // Load license from JSON string
        std::string license_json = R"({
            "user_id": "customer-123",
            "hardware_hash": "abc123def456...",
            "features": ["basic", "premium"],
            "expiry": "2025-12-31T23:59:59Z",
            "issued_at": "2024-01-01T00:00:00Z",
            "license_id": "lic-456",
            "version": 1,
            "hmac_signature": "c4ef45e6..."
        })";

        // Validate license
        auto info = manager.load_and_validate(license_json);

        if (info.valid) {
            std::cout << "✅ License is valid!" << std::endl;
            std::cout << "👤 User: " << info.user_id << std::endl;

            // Check available features
            if (manager.has_feature("premium")) {
                std::cout << "🌟 Premium features activated" << std::endl;
            }
        }

    } catch (const InvalidSignatureException& e) {
        std::cerr << "❌ Invalid license signature: " << e.what() << std::endl;
    } catch (const ExpiredLicenseException& e) {
        std::cerr << "❌ License expired: " << e.what() << std::endl;
    } catch (const HardwareMismatchException& e) {
        std::cerr << "❌ License doesn't match this hardware: " << e.what() << std::endl;
    } catch (const LicenseException& e) {
        std::cerr << "❌ License error: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

### Working with Hardware Fingerprint

```cpp
#include <license_core/hardware_fingerprint.hpp>
#include <iostream>

using namespace license_core;

int main() {
    try {
        // Configure hardware settings
        HardwareConfig config;
        config.use_cpu_id = true;
        config.use_mac_address = true;
        config.use_volume_serial = true;
        config.use_motherboard_serial = false; // Disable for stability
        config.enable_caching = true;
        config.cache_lifetime = std::chrono::minutes(5);

        HardwareFingerprint fingerprint(config);

        // Get complete fingerprint
        std::string hwid = fingerprint.get_fingerprint();
        std::cout << "Hardware ID: " << hwid << std::endl;

        // Get individual components
        std::cout << "CPU ID: " << fingerprint.get_cpu_id_safe() << std::endl;
        std::cout << "MAC Address: " << fingerprint.get_mac_address_safe() << std::endl;

        // Cache statistics
        auto stats = fingerprint.get_cache_stats();
        std::cout << "Cache hit rate: " << (stats.hit_rate() * 100) << "%" << std::endl;

    } catch (const HardwareDetectionException& e) {
        std::cerr << "Hardware detection error: " << e.what() << std::endl;
    }

    return 0;
}
```

### License Generation

```cpp
#include <license_core/license_manager.hpp>
#include <chrono>

using namespace license_core;

LicenseInfo create_license(const std::string& user_id,
                          const std::string& hardware_hash,
                          const std::vector<std::string>& features,
                          int validity_days = 365) {
    LicenseInfo info;
    info.user_id = user_id;
    info.hardware_hash = hardware_hash;
    info.features = features;
    info.license_id = "lic-" + std::to_string(std::time(nullptr));
    info.version = 1;

    // Set issuance time
    info.issued_at = std::chrono::system_clock::now();

    // Set expiration time
    info.expiry = info.issued_at + std::chrono::hours(24 * validity_days);

    return info;
}

int main() {
    try {
        LicenseManager manager("server-secret-key");

        // Create license for customer
        auto license = create_license(
            "enterprise-customer-001",
            manager.get_current_hwid(),
            {"basic", "premium", "enterprise"},
            365  // 1 year
        );

        // Generate signed license
        std::string signed_license = manager.generate_license(license);

        std::cout << "Signed license:" << std::endl;
        std::cout << signed_license << std::endl;

    } catch (const LicenseException& e) {
        std::cerr << "License generation error: " << e.what() << std::endl;
        return 1;
    }

    return 0;
}
```

Detailed code examples can be found in the `EXAMPLES.md` file in the same directory.

---

## 🧪 Testing

To run tests:

```bash
./build.sh
# or
cd build && make test
```

Available 56 automated tests, including:

- Unit tests (37 tests)
- Performance tests (10 tests)
- Multithreading tests (8 tests)

---

## 🛠️ Building

Main build is done via:

```bash
./build.sh
```

Result: `build/liblicensecore.a`

---

## 📋 System Requirements

- **Compiler**: GCC 9+ or Clang 10+ or MSVC 2019+
- **C++ Standard**: C++17
- **CMake**: 3.16+
- **Dependencies**: OpenSSL

---

## ⚠️ Important Notes

1. Use `*_safe()` methods in production code to avoid exceptions
2. Configure `HardwareConfig` according to security requirements
3. Store secret_key in a secure location
4. Regularly clear cache when hardware configuration changes
5. Handle all LicenseCore++ exceptions in your code
6. Use `std::chrono::time_point` for date handling instead of strings
