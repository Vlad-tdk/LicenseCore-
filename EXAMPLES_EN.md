# 💻 Main LicenseCore++ Library Usage Examples

## 📁 Examples Structure

All examples for the main library are located in the `examples/` folder:
- `simple_example` - basic usage
- `license_generator` - license creation
- `hwid_tool` - Hardware ID retrieval

## 🚀 Basic Usage

### Simple Example

```cpp
#include <iostream>
#include "license_core/hardware_fingerprint.hpp"
#include "license_core/license_manager.hpp"

using namespace license_core;

int main() {
    try {
        // 1. Get Hardware ID
        HardwareConfig config;
        HardwareFingerprint fingerprint(config);
        std::string hardware_id = fingerprint.get_fingerprint();
        
        std::cout << "Hardware ID: " << hardware_id << std::endl;
        
        // 2. Validate license from JSON
        LicenseManager manager("your-secret-key");
        
        std::string license_json = R"({
            "user_id": "customer-123",
            "hardware_hash": ")" + hardware_id + R"(",
            "features": ["basic", "premium"],
            "expiry": "2025-12-31T23:59:59Z",
            "issued_at": "2024-01-01T00:00:00Z",
            "license_id": "lic-456",
            "version": 1,
            "hmac_signature": "c4ef45e6..."
        })";
        
        // 3. Load and validate license
        auto info = manager.load_and_validate(license_json);
        
        if (info.valid) {
            std::cout << "✅ License is valid!" << std::endl;
            std::cout << "👤 User: " << info.user_id << std::endl;
            
            if (manager.has_feature("premium")) {
                std::cout << "🌟 Premium features available!" << std::endl;
            }
        } else {
            std::cout << "❌ License is invalid: " << info.error_message << std::endl;
        }
        
    } catch (const LicenseException& e) {
        std::cerr << "Error: " << e.what() << std::endl;
        return 1;
    }
    
    return 0;
}
```

## 🔧 Advanced Usage

### Custom Hardware Configuration

```cpp
#include "license_core/hardware_fingerprint.hpp"

HardwareConfig create_secure_config() {
    HardwareConfig config;
    
    // Components for fingerprint
    config.use_cpu_id = true;              
    config.use_mac_address = true;         
    config.use_motherboard_serial = false; // Disabled for compatibility
    config.use_volume_serial = true;         
    
    // Caching settings
    config.enable_caching = true;             
    config.cache_lifetime = std::chrono::minutes(30);
    config.thread_safe_cache = true;                
    
    return config;
}

void hardware_example() {
    try {
        HardwareConfig config = create_secure_config();
        HardwareFingerprint fingerprint(config);
        
        // Get full fingerprint
        std::string hardware_id = fingerprint.get_fingerprint();
        std::cout << "Full Hardware ID: " << hardware_id << std::endl;
        
        // Get individual components
        std::string cpu_id = fingerprint.get_cpu_id();
        std::string mac_addr = fingerprint.get_mac_address();
        
        std::cout << "CPU ID: " << cpu_id << std::endl;
        std::cout << "MAC Address: " << mac_addr << std::endl;
        
        // Safe versions (don't throw exceptions)
        std::string safe_hardware_id = fingerprint.get_fingerprint_safe();
        std::cout << "Safe Hardware ID: " << safe_hardware_id << std::endl;
        
        // Cache statistics
        auto stats = fingerprint.get_cache_stats();
        std::cout << "Cache hit rate: " << (stats.hit_rate() * 100) << "%" << std::endl;
        
    } catch (const HardwareDetectionException& e) {
        std::cerr << "Hardware error: " << e.what() << std::endl;
    }
}
```

### Working with License Files

```cpp
#include <fstream>
#include <chrono>

class LicenseFileHandler {
private:
    LicenseManager manager_;
    
public:
    LicenseFileHandler(const std::string& secret_key) 
        : manager_(secret_key) {}
    
    bool loadLicenseFromFile(const std::string& file_path) {
        try {
            std::ifstream file(file_path);
            if (!file.is_open()) {
                std::cerr << "Failed to open file: " << file_path << std::endl;
                return false;
            }
            
            std::string license_json((std::istreambuf_iterator<char>(file)),
                                    std::istreambuf_iterator<char>());
            
            // Load and validate license
            auto info = manager_.load_and_validate(license_json);
            
            if (info.valid) {
                std::cout << "✅ License loaded successfully!" << std::endl;
                std::cout << "👤 User: " << info.user_id << std::endl;
                std::cout << "🆔 License ID: " << info.license_id << std::endl;
                
                // Display expiry time
                auto expiry_time = std::chrono::system_clock::to_time_t(info.expiry);
                std::cout << "⏰ Expires: " << std::ctime(&expiry_time);
                
                return true;
            } else {
                std::cout << "❌ License error: " << info.error_message << std::endl;
                return false;
            }
            
        } catch (const std::exception& e) {
            std::cerr << "Error loading license: " << e.what() << std::endl;
            return false;
        }
    }
    
    std::vector<std::string> getAvailableFeatures() {
        return manager_.get_available_features();
    }
    
    bool hasFeature(const std::string& feature) {
        return manager_.has_feature(feature);
    }
};
```

### Error Handling

```cpp
void comprehensive_error_handling() {
    try {
        HardwareConfig config;
        HardwareFingerprint fingerprint(config);
        
        std::string hardware_id = fingerprint.get_fingerprint();
        
        LicenseManager manager("secret-key");
        std::string license_json = "..."; // License JSON
        
        auto info = manager.load_and_validate(license_json);
        
        if (info.valid) {
            std::cout << "License is valid" << std::endl;
        }
        
    } catch (const InvalidSignatureException& e) {
        // Invalid HMAC signature
        std::cerr << "❌ Invalid license signature: " << e.what() << std::endl;
        
    } catch (const ExpiredLicenseException& e) {
        // License expired
        std::cerr << "⏰ License expired: " << e.what() << std::endl;
        
    } catch (const HardwareMismatchException& e) {
        // Hardware ID mismatch
        std::cerr << "🖥️ License for different hardware: " << e.what() << std::endl;
        
    } catch (const MalformedLicenseException& e) {
        // Corrupted JSON or invalid format
        std::cerr << "📄 Malformed license: " << e.what() << std::endl;
        
    } catch (const HardwareDetectionException& e) {
        // Hardware detection errors
        std::cerr << "🔧 Hardware detection error: " << e.what() << std::endl;
        
    } catch (const CryptographicException& e) {
        // Cryptographic errors
        std::cerr << "🔐 Cryptographic error: " << e.what() << std::endl;
        
    } catch (const LicenseException& e) {
        // Base class - all other library errors
        std::cerr << "⚠️ General LicenseCore error: " << e.what() << std::endl;
        
    } catch (const std::exception& e) {
        // Standard C++ exceptions
        std::cerr << "💥 Standard error: " << e.what() << std::endl;
    }
}
```

### Multi-threaded Usage

```cpp
#include <thread>
#include <vector>
#include <mutex>

class ThreadSafeLicenseManager {
private:
    HardwareFingerprint fingerprint_;
    LicenseManager license_manager_;
    mutable std::mutex results_mutex_;
    std::vector<bool> validation_results_;
    
public:
    ThreadSafeLicenseManager(const std::string& secret_key) 
        : fingerprint_(HardwareConfig{.thread_safe_cache = true}),
          license_manager_(secret_key) {}
    
    void validateLicenseAsync(const std::string& license_json, size_t thread_id) {
        bool result = false;
        
        try {
            auto info = license_manager_.load_and_validate(license_json);
            result = info.valid;
            
        } catch (const std::exception& e) {
            std::cerr << "Error in thread " << thread_id << ": " << e.what() << std::endl;
        }
        
        // Thread-safe result storage
        std::lock_guard<std::mutex> lock(results_mutex_);
        if (validation_results_.size() <= thread_id) {
            validation_results_.resize(thread_id + 1);
        }
        validation_results_[thread_id] = result;
    }
    
    void runConcurrentValidation(const std::vector<std::string>& licenses) {
        std::vector<std::thread> threads;
        
        for (size_t i = 0; i < licenses.size(); ++i) {
            threads.emplace_back(&ThreadSafeLicenseManager::validateLicenseAsync,
                               this, std::ref(licenses[i]), i);
        }
        
        // Wait for all threads to complete
        for (auto& thread : threads) {
            thread.join();
        }
        
        // Display results
        std::lock_guard<std::mutex> lock(results_mutex_);
        for (size_t i = 0; i < validation_results_.size(); ++i) {
            std::cout << "License " << i << ": " 
                      << (validation_results_[i] ? "✅ Valid" : "❌ Invalid") 
                      << std::endl;
        }
    }
};
```

### Creating Custom License Generator

```cpp
#include <chrono>
#include <ctime>
#include <iomanip>
#include <sstream>

class LicenseGenerator {
private:
    LicenseManager manager_;
    
public:
    LicenseGenerator(const std::string& secret_key) : manager_(secret_key) {}
    
    LicenseInfo createLicense(const std::string& customer_id,
                             const std::string& hardware_id,
                             const std::vector<std::string>& features,
                             int validity_days = 365) {
        LicenseInfo license;
        
        license.user_id = customer_id;
        license.license_id = "lic-" + std::to_string(std::time(nullptr));
        license.hardware_hash = hardware_id;
        license.features = features;
        license.version = 1;
        
        // Set issue time
        license.issued_at = std::chrono::system_clock::now();
        
        // Set expiry date
        license.expiry = license.issued_at + std::chrono::hours(24 * validity_days);
        
        return license;
    }
    
    std::string generateSignedLicense(const LicenseInfo& license) {
        return manager_.generate_license(license);
    }
    
    bool validateGeneratedLicense(const std::string& license_json) {
        try {
            auto info = manager_.load_and_validate(license_json);
            return info.valid;
        } catch (const std::exception&) {
            return false;
        }
    }
};

// License generator usage example
void license_generation_example() {
    LicenseGenerator generator("my-secret-key");
    
    // Get customer's Hardware ID
    HardwareFingerprint fingerprint;
    std::string customer_hardware = fingerprint.get_fingerprint_safe();
    
    // Generate license
    auto license = generator.createLicense(
        "enterprise-customer-001",
        customer_hardware,
        {"basic", "premium", "enterprise"},
        365  // 1 year
    );
    
    // Create signed license
    std::string signed_license = generator.generateSignedLicense(license);
    
    std::cout << "Generated license:" << std::endl;
    std::cout << signed_license << std::endl;
    
    // Validate generated license
    bool is_valid = generator.validateGeneratedLicense(signed_license);
    std::cout << "License " << (is_valid ? "✅ valid" : "❌ invalid") << std::endl;
}
```

## 🎯 Practical Scenarios

### Desktop Application with Activation

```cpp
class DesktopAppLicensing {
private:
    std::unique_ptr<HardwareFingerprint> fingerprint_;
    std::unique_ptr<LicenseManager> manager_;
    bool is_activated_;
    
public:
    DesktopAppLicensing(const std::string& secret_key) 
        : is_activated_(false) {
        HardwareConfig config;
        config.enable_caching = true;
        config.cache_lifetime = std::chrono::hours(1);
        
        fingerprint_ = std::make_unique<HardwareFingerprint>(config);
        manager_ = std::make_unique<LicenseManager>(secret_key);
    }
    
    std::string getActivationCode() {
        return fingerprint_->get_fingerprint_safe();
    }
    
    bool activateWithLicense(const std::string& license_json) {
        try {
            auto info = manager_->load_and_validate(license_json);
            
            if (!info.valid) {
                std::cout << "❌ License is invalid: " << info.error_message << std::endl;
                return false;
            }
            
            // Check hardware binding
            std::string current_hardware = fingerprint_->get_fingerprint();
            if (info.hardware_hash != current_hardware) {
                std::cout << "❌ License doesn't match this hardware" << std::endl;
                return false;
            }
            
            // Check expiry
            if (manager_->is_expired()) {
                std::cout << "❌ License has expired" << std::endl;
                return false;
            }
            
            is_activated_ = true;
            std::cout << "✅ Application activated successfully!" << std::endl;
            return true;
            
        } catch (const std::exception& e) {
            std::cerr << "Activation error: " << e.what() << std::endl;
            return false;
        }
    }
    
    bool hasFeature(const std::string& feature) {
        if (!is_activated_) {
            return false;
        }
        return manager_->has_feature(feature);
    }
    
    void showFeatureStatus() {
        if (!is_activated_) {
            std::cout << "❌ Application not activated" << std::endl;
            return;
        }
        
        std::cout << "🎯 Available features:" << std::endl;
        auto features = manager_->get_available_features();
        for (const auto& feature : features) {
            bool available = hasFeature(feature);
            std::cout << "  " << (available ? "✅" : "❌") << " " << feature << std::endl;
        }
    }
};
```

### Using HMAC Validator

```cpp
#include "license_core/hmac_validator.hpp"

void hmac_validation_example() {
    try {
        HMACValidator validator("my-secret-key");
        
        // Sign data
        std::string data = "important license data";
        std::string signature = validator.sign(data);
        
        std::cout << "Data: " << data << std::endl;
        std::cout << "Signature: " << signature << std::endl;
        
        // Verify signature
        bool is_valid = validator.verify(data, signature);
        std::cout << "Signature " << (is_valid ? "✅ valid" : "❌ invalid") << std::endl;
        
        // Working with JSON
        std::string json_without_signature = R"({
            "user_id": "test-user",
            "features": ["basic", "premium"],
            "expiry": "2025-12-31T23:59:59Z"
        })";
        
        std::string signed_json = validator.sign_json(json_without_signature);
        std::cout << "Signed JSON:" << std::endl << signed_json << std::endl;
        
        // Verify JSON
        bool json_valid = validator.verify_json(signed_json);
        std::cout << "JSON signature " << (json_valid ? "✅ valid" : "❌ invalid") << std::endl;
        
    } catch (const CryptographicException& e) {
        std::cerr << "Cryptographic error: " << e.what() << std::endl;
    }
}
```

## 🔍 Debugging and Diagnostics

### Logging System

```cpp
#include <fstream>

class LicenseLogger {
private:
    std::ofstream log_file_;
    
public:
    LicenseLogger(const std::string& log_path) : log_file_(log_path, std::ios::app) {}
    
    void logHardwareInfo() {
        HardwareFingerprint fingerprint;
        
        log_file_ << "=== Hardware Information ===" << std::endl;
        log_file_ << "Full ID: " << fingerprint.get_fingerprint_safe() << std::endl;
        log_file_ << "CPU ID: " << fingerprint.get_cpu_id_safe() << std::endl;
        log_file_ << "MAC Address: " << fingerprint.get_mac_address_safe() << std::endl;
        log_file_ << "Volume Serial: " << fingerprint.get_volume_serial() << std::endl;
        log_file_ << "Timestamp: " << std::time(nullptr) << std::endl;
        log_file_ << std::endl;
    }
    
    void logLicenseValidation(const LicenseInfo& info) {
        log_file_ << "=== License Validation ===" << std::endl;
        log_file_ << "License ID: " << info.license_id << std::endl;
        log_file_ << "User ID: " << info.user_id << std::endl;
        log_file_ << "Valid: " << (info.valid ? "YES" : "NO") << std::endl;
        if (!info.valid) {
            log_file_ << "Error: " << info.error_message << std::endl;
        }
        
        // Log expiry time
        auto expiry_time = std::chrono::system_clock::to_time_t(info.expiry);
        log_file_ << "Expiry: " << std::ctime(&expiry_time);
        
        log_file_ << "Features: ";
        for (const auto& feature : info.features) {
            log_file_ << feature << " ";
        }
        log_file_ << std::endl << std::endl;
    }
};
```

### Performance Testing

```cpp
#include <chrono>

void performance_test() {
    // Hardware Fingerprint performance test
    auto start = std::chrono::high_resolution_clock::now();
    
    HardwareFingerprint fingerprint;
    for (int i = 0; i < 1000; ++i) {
        fingerprint.get_fingerprint_safe(); // Should use cache
    }
    
    auto end = std::chrono::high_resolution_clock::now();
    auto duration = std::chrono::duration_cast<std::chrono::microseconds>(end - start);
    
    std::cout << "1000 calls to get_fingerprint_safe: " 
              << duration.count() << " microseconds" << std::endl;
    
    auto stats = fingerprint.get_cache_stats();
    std::cout << "Cache hit rate: " << (stats.hit_rate() * 100) << "%" << std::endl;
    std::cout << "Cache hits: " << stats.cache_hits << std::endl;
    std::cout << "Cache misses: " << stats.cache_misses << std::endl;
}
```

## 📁 Example Files

See ready examples in the `examples/` folder:

1. **`simple_example.cpp`** - basic usage
2. **`license_generator.cpp`** - license creation  
3. **`hwid_tool.cpp`** - Hardware ID retrieval
4. **`error_handling_example.cpp`** - error handling
5. **`caching_example.cpp`** - caching usage
6. **`performance_test.cpp`** - performance testing

Compiling examples:
```bash
cd build
./examples/simple_example
./examples/license_generator
./examples/hwid_tool
```

## 📝 Integration Notes

### License Manager Initialization

```cpp
// Proper initialization
class MyApplication {
private:
    std::unique_ptr<LicenseManager> license_manager_;
    
public:
    bool initialize() {
        try {
            // Create manager with secret key
            license_manager_ = std::make_unique<LicenseManager>("secure-secret-key");
            
            // Load license on startup
            return loadLicense();
            
        } catch (const std::exception& e) {
            std::cerr << "Initialization error: " << e.what() << std::endl;
            return false;
        }
    }
    
    bool loadLicense() {
        // Try loading license from different sources
        std::vector<std::string> license_paths = {
            "license.json",
            "config/license.json", 
            getAppDataPath() + "/license.json"
        };
        
        for (const auto& path : license_paths) {
            if (tryLoadLicenseFromFile(path)) {
                return true;
            }
        }
        
        return false; // License not found
    }
};
```

### Secure Secret Key Storage

```cpp
// Don't store secret key directly in code!
// ❌ Bad:
// const char* SECRET_KEY = "my-secret-key-123";

// ✅ Good - load from environment variable:
std::string getSecretKey() {
    const char* key = std::getenv("LICENSECORE_SECRET_KEY");
    if (!key) {
        throw std::runtime_error("Secret key not found in environment");
    }
    return std::string(key);
}

// ✅ Good - load from encrypted file:
std::string loadEncryptedSecretKey(const std::string& encrypted_file) {
    // Load and decrypt secret key
    // Implementation depends on chosen encryption method
    return decryptSecretKey(encrypted_file);
}
```
