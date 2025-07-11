# LicenseCore++ Embedded Edition - Complete Guide

## 📦 Overview

LicenseCore++ Embedded Edition is a lightweight, static library for integrating licensing into applications without external dependencies. Perfect for:

- Embedded systems and IoT devices
- Desktop applications with minimal dependencies  
- B2B solutions with simple licensing
- Rapid prototyping of licensing systems

## 🏗️ Architecture

### Components

```
embedded/
├── liblicense_core.a      # Static library
├── include/
│   └── license_core_stub.h # C API header
├── src/                   # Source codes (optional)
├── test/                  # Test examples
└── example_integration.cpp # Integration example
```

### Architecture Diagram

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Your App      │───▶│ license_core.a  │───▶│  Hardware ID    │
│                 │    │                 │    │                 │
│ lc_validate()   │    │ • HMAC-SHA256   │    │ • CPU ID        │
│ lc_has_feature()│    │ • JSON parsing  │    │ • MAC address   │
│ lc_get_hwid()   │    │ • HW binding    │    │ • Volume serial │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

## 🔧 API Reference

### Core Functions

#### `int lc_validate_license(const char* license_json)`

Validates license from JSON string. The function automatically retrieves current Hardware ID and compares it with the license.

**Parameters:**
- `license_json` - JSON string with license data

**Returns:**
- `1` - license is valid
- `0` - license is invalid or error occurred

**Example:**
```c
const char* license = R"({
    "user_id": "Company Inc",
    "license_id": "lic-12345",
    "hardware_hash": "ABC123DEF456",
    "features": ["basic", "premium"],
    "expiry": "2025-12-31T23:59:59Z",
    "issued_at": "2024-01-01T00:00:00Z",
    "version": 1,
    "hmac_signature": "sha256_hmac_signature"
})";

if (lc_validate_license(license)) {
    printf("License is valid\n");
} else {
    printf("License validation failed\n");
}
```

#### `int lc_has_feature(const char* feature_name)`

Checks for specific feature availability in the loaded license.

**Parameters:**
- `feature_name` - feature name to check

**Returns:**
- `1` - feature is available
- `0` - feature is not available

**Example:**
```c
if (lc_has_feature("premium")) {
    enable_premium_features();
}

if (lc_has_feature("advanced_reporting")) {
    show_reports_menu();
}
```

#### `const char* lc_get_hwid(void)`

Returns unique Hardware ID of the current machine.

**Returns:**
- String with Hardware ID (do not free the memory)
- `NULL` in case of error

**Example:**
```c
const char* hwid = lc_get_hwid();
if (hwid) {
    printf("Hardware ID: %s\n", hwid);
}
```

## 📥 Installation and Integration

### Step 1: Prepare Files

```bash
# Copy necessary files to your project
cp embedded/liblicense_core.a your_project/lib/
cp embedded/include/license_core_stub.h your_project/include/
```

### Step 2: CMake Integration

```cmake
# CMakeLists.txt
cmake_minimum_required(VERSION 3.16)
project(YourApp)

# Add header path
include_directories(include)

# Add library
add_library(license_core STATIC IMPORTED)
set_target_properties(license_core PROPERTIES
    IMPORTED_LOCATION ${CMAKE_SOURCE_DIR}/lib/liblicense_core.a
)

# Create executable
add_executable(your_app main.cpp)

# Link with library
target_link_libraries(your_app license_core)
```

### Step 3: Makefile Integration

```makefile
# Makefile
CXX = g++
CXXFLAGS = -std=c++17 -O2 -Iinclude
LDFLAGS = -Llib -llicense_core

SOURCES = main.cpp
TARGET = your_app

$(TARGET): $(SOURCES)
	$(CXX) $(CXXFLAGS) -o $@ $^ $(LDFLAGS)

clean:
	rm -f $(TARGET)
```

### Step 4: Direct Compilation

```bash
# GCC/Clang
g++ -std=c++17 -O2 -Iinclude main.cpp -Llib -llicense_core -o your_app

# MSVC
cl /std:c++17 /O2 /Iinclude main.cpp /link lib/license_core.lib
```

## 💻 Usage Examples

### Basic Integration

```cpp
#include <iostream>
#include <fstream>
#include <string>
#include "license_core_stub.h"

int main() {
    // Load license from file
    std::ifstream file("license.json");
    std::string license_content((std::istreambuf_iterator<char>(file)),
                               std::istreambuf_iterator<char>());
    
    // Validate license
    if (!lc_validate_license(license_content.c_str())) {
        std::cerr << "Invalid license!" << std::endl;
        return 1;
    }
    
    std::cout << "License valid!" << std::endl;
    
    // Check available features
    if (lc_has_feature("premium")) {
        std::cout << "Premium features enabled" << std::endl;
    }
    
    if (lc_has_feature("export")) {
        std::cout << "Export functionality available" << std::endl;
    }
    
    // Get Hardware ID
    const char* hwid = lc_get_hwid();
    if (hwid) {
        std::cout << "Hardware ID: " << hwid << std::endl;
    }
    
    return 0;
}
```

### Application Class Integration

```cpp
class LicensedApplication {
private:
    bool license_valid_;
    std::vector<std::string> available_features_;
    
public:
    bool initialize(const std::string& license_path) {
        // Load license
        std::ifstream file(license_path);
        if (!file.is_open()) {
            return false;
        }
        
        std::string license_content((std::istreambuf_iterator<char>(file)),
                                   std::istreambuf_iterator<char>());
        
        // Validate license
        license_valid_ = lc_validate_license(license_content.c_str());
        
        if (license_valid_) {
            // Determine available features
            const char* features[] = {"basic", "premium", "export", "import", "api"};
            for (const char* feature : features) {
                if (lc_has_feature(feature)) {
                    available_features_.push_back(feature);
                }
            }
        }
        
        return license_valid_;
    }
    
    bool is_feature_enabled(const std::string& feature) const {
        return std::find(available_features_.begin(), 
                        available_features_.end(), 
                        feature) != available_features_.end();
    }
    
    void run() {
        if (!license_valid_) {
            std::cout << "Application requires valid license" << std::endl;
            return;
        }
        
        std::cout << "Starting licensed application..." << std::endl;
        
        if (is_feature_enabled("premium")) {
            enable_premium_ui();
        }
        
        if (is_feature_enabled("export")) {
            enable_export_menu();
        }
        
        main_loop();
    }
};
```

### Embedded License

```cpp
// Embed license directly in code
const char* embedded_license = R"({
    "user_id": "OEM Partner",
    "license_id": "lic-embedded-001",
    "hardware_hash": "*",
    "features": ["basic", "embedded"],
    "expiry": "2025-12-31T23:59:59Z",
    "issued_at": "2024-01-01T00:00:00Z",
    "version": 1,
    "hmac_signature": "embedded_signature_here"
})";

int main() {
    // Validate embedded license
    if (lc_validate_license(embedded_license)) {
        std::cout << "Embedded license is valid" << std::endl;
        
        // Run application
        run_application();
    } else {
        std::cout << "Embedded license validation failed" << std::endl;
        return 1;
    }
    
    return 0;
}
```

## 🔒 Security

### Protection Levels

#### Basic Protection (enabled by default):
- ✅ HMAC-SHA256 license signing
- ✅ Hardware binding by CPU ID, MAC address, disk serial number
- ✅ Symbol hiding (`-fvisibility=hidden`)
- ✅ Debug information removal

#### Additional Protection (recommended):
- 🔧 String obfuscation with keys
- 🔧 API function renaming
- 🔧 UPX compression
- 🔧 Wrapper with anti-debug protection

### Security Recommendations

#### 1. Secret Key Obfuscation

```cpp
// Bad - key visible in disassembler
const char* secret_key = "my_secret_key_123";

// Good - simple XOR obfuscation
const char* get_secret_key() {
    static char key[] = {0x6D^0xAA, 0x79^0xAA, 0x5F^0xAA, 0x73^0xAA, 0x65^0xAA, 
                        0x63^0xAA, 0x72^0xAA, 0x65^0xAA, 0x74^0xAA, 0x5F^0xAA,
                        0x6B^0xAA, 0x65^0xAA, 0x79^0xAA, 0x5F^0xAA, 0x31^0xAA,
                        0x32^0xAA, 0x33^0xAA, 0x00^0xAA};
    
    static bool decrypted = false;
    if (!decrypted) {
        for (int i = 0; key[i]; i++) {
            key[i] ^= 0xAA;
        }
        decrypted = true;
    }
    return key;
}
```

#### 2. API Function Renaming

```cpp
// Add aliases with less obvious names
extern "C" {
    int app_check_auth(const char* data) {
        return lc_validate_license(data);
    }
    
    int app_has_capability(const char* cap) {
        return lc_has_feature(cap);
    }
    
    const char* app_get_machine_id(void) {
        return lc_get_hwid();
    }
}
```

#### 3. Anti-Debug Protection

```cpp
bool is_debugger_present() {
#ifdef _WIN32
    return IsDebuggerPresent();
#elif defined(__linux__)
    char buf[4096];
    const int status_fd = open("/proc/self/status", O_RDONLY);
    if (status_fd == -1) return false;
    
    const ssize_t num_read = read(status_fd, buf, sizeof(buf) - 1);
    close(status_fd);
    
    if (num_read <= 0) return false;
    buf[num_read] = '\0';
    
    return strstr(buf, "TracerPid:\t0") == nullptr;
#else
    return false;
#endif
}

int main() {
    if (is_debugger_present()) {
        std::cout << "Debugging not allowed" << std::endl;
        return 1;
    }
    
    // Rest of the code...
}
```

## 🚀 Performance

### Characteristics

| Metric | Value |
|---------|-------|
| Library size | ~150KB |
| Validation time | <1ms |
| Memory usage | <10KB |
| Dependencies | C++17 stdlib only |
| Thread safety | ❌ (add mutex if needed) |

### Size Optimization

```bash
# Remove debug information
strip --strip-unneeded liblicense_core.a

# Remove unused sections
g++ -ffunction-sections -fdata-sections -Wl,--gc-sections

# UPX compression (for final application)
upx --best your_app
```

### Thread Safety

The library is not thread-safe by default. For multi-threaded usage, add synchronization:

```cpp
#include <mutex>

class ThreadSafeLicense {
private:
    mutable std::mutex mutex_;
    
public:
    bool validate_license(const char* license_json) {
        std::lock_guard<std::mutex> lock(mutex_);
        return lc_validate_license(license_json);
    }
    
    bool has_feature(const char* feature_name) {
        std::lock_guard<std::mutex> lock(mutex_);
        return lc_has_feature(feature_name);
    }
    
    std::string get_hwid() {
        std::lock_guard<std::mutex> lock(mutex_);
        const char* hwid = lc_get_hwid();
        return hwid ? std::string(hwid) : std::string();
    }
};
```

## 🔧 Building from Source

### Requirements

- **Compiler:** GCC 9+, Clang 10+, MSVC 2019+
- **Standard:** C++17
- **Build System:** CMake 3.16+ or Make

### Build Process

```bash
# Go to embedded folder
cd embedded

# Build with Make
make clean
make

# Or with CMake
mkdir build && cd build
cmake ..
make

# Test build
./test/test_embedded
```

### Compilation Settings

```makefile
# Makefile settings
CXX = g++
CXXFLAGS = -std=c++17 -O2 -fvisibility=hidden -ffunction-sections -fdata-sections
LDFLAGS = -Wl,--gc-sections -static-libgcc -static-libstdc++

# For maximum security
SECURITY_FLAGS = -fstack-protector-strong -D_FORTIFY_SOURCE=2 -Wformat -Wformat-security
```

## 🧪 Testing

### Running Tests

```bash
cd embedded/test
./run_tests.sh
```

### Test Examples

```cpp
#include <cassert>
#include "license_core_stub.h"

void test_valid_license() {
    const char* valid_license = R"({
        "user": "Test User",
        "product": "TestApp",
        "version": "1.0",
        "expiry": "2025-12-31",
        "features": ["basic", "test"],
        "hwid": "*",
        "signature": "valid_signature"
    })";
    
    assert(lc_validate_license(valid_license) == 1);
    assert(lc_has_feature("basic") == 1);
    assert(lc_has_feature("test") == 1);
    assert(lc_has_feature("nonexistent") == 0);
}

void test_hwid() {
    const char* hwid = lc_get_hwid();
    assert(hwid != nullptr);
    assert(strlen(hwid) > 0);
}

int main() {
    test_valid_license();
    test_hwid();
    
    std::cout << "All tests passed!" << std::endl;
    return 0;
}
```

## 🌍 Platform Support

### Windows
- **Compilers:** MSVC 2019+, MinGW
- **Architectures:** x86, x64
- **Features:** WMI support for Hardware ID

### macOS
- **Compilers:** Xcode 12+, Clang
- **Architectures:** Intel x64, Apple Silicon (ARM64)
- **Features:** Universal binaries

### Linux
- **Compilers:** GCC 9+, Clang 10+
- **Architectures:** x86, x64, ARM, ARM64
- **Distributions:** Ubuntu 20.04+, CentOS 8+, Alpine Linux

### Embedded Systems
- **Raspberry Pi** (ARM)
- **BeagleBone** (ARM)
- **Intel Edison** (x86)

## 📚 FAQ

### Q: Can I use the library in commercial projects?
A: Yes, the library is designed for commercial use.

### Q: Do I need additional licenses for distribution?
A: No, the static library is embedded into your application.

### Q: Is online license validation supported?
A: Embedded Edition supports offline validation only. Online validation is available in Enterprise version.

### Q: Can I modify the Hardware ID algorithm?
A: Yes, source codes allow algorithm customization.

### Q: What to do if Hardware ID changes?
A: Use wildcard "*" in hwid field or implement tolerance to changes.

## 📞 Support

- **Email:** support@licensecore.tech
- **Documentation:** https://docs.licensecore.tech
- **Examples:** https://github.com/licensecore/examples
- **Forum:** https://forum.licensecore.tech

## 📄 License

© 2024 LicenseCore Technologies
Licensed under Commercial License Agreement
