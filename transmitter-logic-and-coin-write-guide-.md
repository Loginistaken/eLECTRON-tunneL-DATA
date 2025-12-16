// ============================================================================
// EDUCATIONAL BINARY TRANSLATOR + DEMO CODE-LISTER WITH SIMULATED TRIGGER
// Clean, Safe, Non-Destructive Version + Email & Modem Simulation
// Updated: 2025-12-16
// Target: C++11
// Notes: Fixes for portability, correctness, and safety.
// ============================================================================
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <sstream>
#include <map>
#include <thread>
#include <chrono>
#include <iomanip>
#include <ctime>
#include <algorithm>
#include <cctype>
#include <cstring>
#include <bitset>

#ifdef _WIN32
  #include <io.h>    // _mkdir
  #include <direct.h>
#else
  #include <sys/stat.h>  // mkdir on Unix/Mac
  #include <sys/types.h>
  #include <errno.h>
#endif

// libcurl (C API is already C-compatible)
#include <curl/curl.h>

// -------------------- Utility: Binary -> ASCII --------------------
std::string binaryToAscii(const std::string& bits) {
    std::string out;
    for (size_t i = 0; i + 8 <= bits.size(); i += 8) {
        std::string byte = bits.substr(i, 8);
        if (byte.find_first_not_of("01") != std::string::npos) continue;
        std::bitset<8> bs(byte);
        char c = static_cast<char>(bs.to_ulong());
        out += c;
    }
    return out;
}

// Read a text file containing 0/1 characters (ignores whitespace)
std::string loadBinaryFile(const std::string& filePath) {
    std::ifstream fin(filePath.c_str());
    if (!fin) {
        std::cerr << "[ERROR] Could not open file: " << filePath << "\n";
        return "";
    }
    std::string bits((std::istreambuf_iterator<char>(fin)),
                      std::istreambuf_iterator<char>());
    bits.erase(std::remove_if(bits.begin(), bits.end(),
        [](char ch){ return std::isspace(static_cast<unsigned char>(ch)); }),
        bits.end());
    return bits;
}

// -------------------- Safe Directory Path (Manual Creation) --------------------
bool ensureDirectory(const std::string& path) {
#ifdef _WIN32
    int r = _mkdir(path.c_str());
    if (r == 0 || errno == EEXIST) return true;
    return false;
#else
    if (mkdir(path.c_str(), 0755) == 0) return true;
    if (errno == EEXIST) return true;
    return false;
#endif
}

std::string getOutputPath() {
    std::string base = "./SafeOutput";
    ensureDirectory(base);
    return base + "/translated_output.txt";
}

// -------------------- Light Obfuscation (Safe, Transparent) --------------------
std::string simpleObfuscate(const std::string& input) {
    std::string out = input;
    for (char& c : out) c ^= 0x03;  // Small reversible transform
    return out;
}

// -------------------- Friendly Spin Interpreter --------------------
class SpinInterpreter {
public:
    SpinInterpreter() {
        map_['U'] = "01";
        map_['D'] = "10";
        map_['L'] = "11";
        map_['-'] = "00";
    }

    void run(const std::string& stream) {
        std::ofstream log("spin_log.txt", std::ios::app);
        for (char s : stream) {
            std::string b = decode(s);
            if (log) log << b << "\n";
            if (b == "01") std::cout << "[UP] Action: Data acknowledgement\n";
            else if (b == "10") std::cout << "[DOWN] Action: Reset event\n";
            else if (b == "11") std::cout << "[LOOP] Action: Synchronization signal\n";
            else std::cout << "[IDLE] No action\n";
            std::this_thread::sleep_for(std::chrono::milliseconds(150));
        }
    }

private:
    std::map<char, std::string> map_;
    std::string decode(char c) const {
        auto it = map_.find(c);
        return (it != map_.end()) ? it->second : "00";
    }
};

// -------------------- Timestamp Helper --------------------
std::string getTimeStamp() {
    std::time_t now = std::time(nullptr);
    std::tm tbuf;
#if defined(_WIN32)
    localtime_s(&tbuf, &now);
    std::tm* t = &tbuf;
#else
    std::tm* t = std::localtime(&now);
#endif
    std::ostringstream oss;
    oss << (t->tm_year + 1900) << '-'
        << std::setfill('0') << std::setw(2) << (t->tm_mon + 1) << '-'
        << std::setfill('0') << std::setw(2) << t->tm_mday << ' '
        << std::setfill('0') << std::setw(2) << t->tm_hour << ':'
        << std::setfill('0') << std::setw(2) << t->tm_min << ':'
        << std::setfill('0') << std::setw(2) << t->tm_sec;
    return oss.str();
}

// -------------------- Simulated Modem Activation at 137Hz --------------------
void simulateModemActivation() {
    std::cout << "[SIM] Activating modem at 137Hz (simulated pulse)...\n";
    for (int i = 0; i < 137; ++i) {
        std::cout << "." << std::flush;
        std::this_thread::sleep_for(std::chrono::milliseconds(7));  // rough sim
    }
    std::cout << "\n[SIM] Modem activated - ready for trigger.\n";
}

// -------------------- Auto Email Send Function (Libcurl with TLS) --------------------
struct upload_status {
    size_t bytes_read;
    const char* data;
    size_t total;
};

static size_t read_cb(void* ptr, size_t size, size_t nmemb, void* userp) {
    upload_status* upload_ctx = static_cast<upload_status*>(userp);
    size_t room = size * nmemb;
    if (upload_ctx->bytes_read >= upload_ctx->total) return 0;
    size_t left = upload_ctx->total - upload_ctx->bytes_read;
    size_t to_copy = (left < room) ? left : room;
    std::memcpy(ptr, upload_ctx->data + upload_ctx->bytes_read, to_copy);
    upload_ctx->bytes_read += to_copy;
    return to_copy;
}

bool sendAutoEmail() {
    CURL* curl = nullptr;
    CURLcode res = CURLE_OK;
    struct curl_slist* recipients = NULL;
    upload_status upload_ctx = {0, nullptr, 0};

    // NOTE: DO NOT hardcode real credentials in source.
    // Replace placeholders below with secure retrieval (env, vault, prompt).
    std::string body = "This is a letter from the year 2000.\nTime capsule activated in " + getTimeStamp() + ".\n";
    std::string full_payload =
        "Date: " + getTimeStamp() + "\r\n"
        "To: <ericlindau@hotmail.com>\r\n"
        "From: <YOUR_SENDER_EMAIL@outlook.com>\r\n"
        "Subject: LETTER FROM THE YEAR 2000\r\n"
        "\r\n" + body + "\r\n";

    upload_ctx.data = full_payload.c_str();
    upload_ctx.total = std::strlen(upload_ctx.data);
    upload_ctx.bytes_read = 0;

    curl_global_init(CURL_GLOBAL_ALL);
    curl = curl_easy_init();
    if (!curl) {
        curl_global_cleanup();
        std::cerr << "[ERROR] curl initialization failed\n";
        return false;
    }

    // Credentials (REPLACE THESE - use app password if MFA on)
    curl_easy_setopt(curl, CURLOPT_USERNAME, "YOUR_SENDER_EMAIL@outlook.com");
    curl_easy_setopt(curl, CURLOPT_PASSWORD, "YOUR_APP_PASSWORD");

    // Use STARTTLS SMTP
    curl_easy_setopt(curl, CURLOPT_URL, "smtp://smtp.live.com:587");
    curl_easy_setopt(curl, CURLOPT_USE_SSL, (long)CURLUSESSL_ALL);
    curl_easy_setopt(curl, CURLOPT_SSL_VERIFYPEER, 1L);
    curl_easy_setopt(curl, CURLOPT_SSL_VERIFYHOST, 2L);

    // Sender and recipient
    curl_easy_setopt(curl, CURLOPT_MAIL_FROM, "<YOUR_SENDER_EMAIL@outlook.com>");
    recipients = curl_slist_append(recipients, "<ericlindau@hotmail.com>");
    curl_easy_setopt(curl, CURLOPT_MAIL_RCPT, recipients);

    // Payload callback
    curl_easy_setopt(curl, CURLOPT_READFUNCTION, read_cb);
    curl_easy_setopt(curl, CURLOPT_READDATA, &upload_ctx);
    curl_easy_setopt(curl, CURLOPT_UPLOAD, 1L);
    curl_easy_setopt(curl, CURLOPT_VERBOSE, 0L);  // toggle for debug

    res = curl_easy_perform(curl);
    if (res != CURLE_OK) {
        std::cerr << "[ERROR] Email send failed: " << curl_easy_strerror(res) << "\n";
    } else {
        std::cout << "[SUCCESS] Email triggered and sent.\n";
    }

    if (recipients) curl_slist_free_all(recipients);
    curl_easy_cleanup(curl);
    curl_global_cleanup();
    return (res == CURLE_OK);
}

// -------------------- Main --------------------
int main(int argc, char* argv[]) {
    std::cout << "=== Safe Educational Binary Translator + Trigger Sim ===\n";
    std::cout << "Runtime Timestamp: " << getTimeStamp() << "\n";
    std::cout << "-------------------------------------------\n";

    // Simulated transmitter binary code (wireless intro signal)
    std::string transmitter_binary =
        "010000010100001101010100010010010101011001000001010101000100010101001100001100010011001101110011";  // "ACTIVATE137"
    std::cout << "[SIM] Receiving wireless transmitter signal (binary): " << transmitter_binary << "\n";
    std::string decoded_trigger = binaryToAscii(transmitter_binary);
    std::cout << "[Decoded Trigger] " << decoded_trigger << "\n";

    if (decoded_trigger == "ACTIVATE137") {
        std::cout << "[TRIGGER] Match found - jogging old logic (2002+ sim).\n";
        simulateModemActivation();
        std::cout << "[SIM] Triggering GPU per se for task initiation...\n";  // GPU sim placeholder
        // Email will fail unless valid credentials are provided and network access allowed.
        sendAutoEmail();
    } else {
        std::cout << "[WARN] No trigger match - skipping email/modem.\n";
    }

    if (argc > 1) {
        std::string filePath = argv[1];
        std::cout << "[INFO] Loading binary file: " << filePath << "\n";
        std::string bits = loadBinaryFile(filePath);
        if (!bits.empty()) {
            std::string ascii = binaryToAscii(bits);
            std::cout << "[Decoded Text] " << ascii << "\n";
            std::string outPath = getOutputPath();
            std::ofstream fout(outPath.c_str());
            if (fout) {
                fout << ascii;
                std::cout << "[INFO] Saved translation to: " << outPath << "\n";
            } else {
                std::cerr << "[ERROR] Could not write output file: " << outPath << "\n";
            }
        } else {
            std::cout << "[WARN] File contained no usable binary.\n";
        }
    } else {
        std::cout << "[INFO] No binary file provided.\n";
        std::cout << "      Usage: program <binary_file.txt>\n";
    }

    std::cout << "\n--- Spin Interpreter Demo ---\n";
    SpinInterpreter spin;
    spin.run("UDL-UL");

    std::cout << "\n[INFO] Program finished safely.\n";
    return 0;
}
// Cleaned and consolidated version of CPP-writer-wBinarytranslator.cpp



CPP-writer-wBinarytranslator.cpp
//

// Notes:
// - Consolidated duplicated functions and removed duplicate mains.
// - Kept core functionality: binary <-> ascii translation, simple CPPWriter payload,
//   vault path resolution + safe file write/append, simple qubit interpreter simulation.
// - Removed any automatic external download triggers that execute system commands.
// - Platform-dependent code remains minimal and guarded.

#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <sstream>
#include <map>
#include <thread>
#include <chrono>
#include <filesystem>
#include <algorithm>
#include <iomanip>
#include <ctime>

#ifdef _WIN32
#  include <windows.h>
#  include <shlobj.h>
#elif defined(__APPLE__) || defined(__linux__)
#  include <pwd.h>
#  include <unistd.h>
#endif

namespace fs = std::filesystem;

// -------------------- Utility: Binary <-> ASCII --------------------
std::string binaryToAscii(const std::string& binaryStream) {
    std::string output;
    size_t len = binaryStream.size();
    for (size_t i = 0; i + 8 <= len; i += 8) {
        std::string byte = binaryStream.substr(i, 8);
        // skip non-binary bytes (defensive)
        if (byte.find_first_not_of("01") != std::string::npos) continue;
        char chr = static_cast<char>(std::stoi(byte, nullptr, 2));
        output += chr;
    }
    return output;
}

// Read binary stream from file (as '0' and '1' chars, ignoring whitespace)
std::string readElectronStreamFromFile(const std::string& filename) {
    std::ifstream fin(filename.c_str());
    if (!fin) {
        std::cerr << "[ERROR] Could not open file: " << filename << std::endl;
        return "";
    }
    std::string bits((std::istreambuf_iterator<char>(fin)), std::istreambuf_iterator<char>());
    bits.erase(std::remove(bits.begin(), bits.end(), ' '), bits.end());
    bits.erase(std::remove(bits.begin(), bits.end(), '\n'), bits.end());
    bits.erase(std::remove(bits.begin(), bits.end(), '\r'), bits.end());
    bits.erase(std::remove(bits.begin(), bits.end(), '\t'), bits.end());
    return bits;
}

// Simulate quantum noise (random bitflip in a string of bits)
void applyQuantumNoise(std::string& bits, int num_flips = 1) {
    if (bits.empty() || num_flips <= 0) return;
    std::srand(static_cast<unsigned>(std::time(nullptr)));
    for (int i = 0; i < num_flips && !bits.empty(); ++i) {
        size_t idx = std::rand() % bits.size();
        bits[idx] = (bits[idx] == '0') ? '1' : '0';
    }
}

// -------------------- Vault & File Helpers --------------------
std::string getHomeDirectory() {
#ifdef _WIN32
    char path[MAX_PATH];
    if (SUCCEEDED(SHGetFolderPathA(NULL, CSIDL_PROFILE, NULL, 0, path))) {
        return std::string(path);
    }
    return "";
#else
    const char* homeDir = getenv("HOME");
    if (homeDir != nullptr) return std::string(homeDir);
    struct passwd* pw = getpwuid(getuid());
    if (pw) return std::string(pw->pw_dir);
    return "";
#endif
}

std::string getVaultFilePath() {
#ifdef _WIN32
    char appdata[MAX_PATH];
    if (SUCCEEDED(SHGetFolderPathA(NULL, CSIDL_LOCAL_APPDATA, NULL, 0, appdata))) {
        std::string vaultDir = std::string(appdata) + "\\LocalLow\\System\\Vault";
        fs::create_directories(vaultDir);
        return vaultDir + "\\time_vault.dat";
    }
    return getHomeDirectory() + "\\time_vault.dat";
#elif defined(__APPLE__)
    std::string vaultDir = getHomeDirectory() + "/Library/Preferences/.vault";
    fs::create_directories(vaultDir);
    return vaultDir + "/time_vault.dat";
#elif defined(__linux__)
    std::string vaultDir = getHomeDirectory() + "/.config/.vault";
    fs::create_directories(vaultDir);
    return vaultDir + "/time_vault.dat";
#else
    return "time_vault.dat";
#endif
}

std::string encryptPayload(const std::string& input, char key = 'K') {
    std::string output = input;
    for (auto& c : output) c ^= key;
    return output;
}

void writePayload(const std::string& filepath, const std::string& payload) {
    std::ofstream outFile(filepath, std::ios::binary);
    if (!outFile) {
        std::cerr << "[ERROR] Failed to open vault file for writing: " << filepath << "\n";
        return;
    }
    outFile.write(payload.data(), static_cast<std::streamsize>(payload.size()));
}

void appendPayload(const std::string& filepath, const std::string& payload) {
    std::ofstream outFile(filepath, std::ios::binary | std::ios::app);
    if (!outFile) {
        std::cerr << "[ERROR] Failed to open vault file for appending: " << filepath << "\n";
        return;
    }
    outFile.write(payload.data(), static_cast<std::streamsize>(payload.size()));
}

// Minimal safe placeholder for off-board update trigger (no system calls)
void triggerOffBoardUpdate() {
    std::cout << "[INFO] (placeholder) Off-board update trigger would be initiated here.\n";
}

// -------------------- CPPWriter (lightweight) --------------------
const char* cppWriterCode = R"cpp(
// CPP-Writer: lightweight payload (string literal included into vault)
#include <iostream>
int main() {
    std::cout << "CPP-Writer payload (embedded example)\\n";
    return 0;
}
)cpp";

// -------------------- Qubit Interpreter (simple, deterministic) --------------------
class QubitInterpreter {
    std::map<char, std::string> spinMap;
public:
    QubitInterpreter() {
        // Using ASCII keys for portability: U (up), D (down), L (loop), - (idle)
        spinMap['U'] = "01";
        spinMap['D'] = "10";
        spinMap['L'] = "11";
        spinMap['-'] = "00";
    }

    void processSpinStream(const std::string& spinStream) {
        for (char spin : spinStream) {
            std::string binary = interpretSpin(spin);
            logToLegacySystem(binary);
            executeLegacyFunction(binary);
            std::this_thread::sleep_for(std::chrono::milliseconds(200));
        }
    }

private:
    std::string interpretSpin(char spinState) const {
        auto it = spinMap.find(spinState);
        return (it != spinMap.end()) ? it->second : "00";
    }

    void logToLegacySystem(const std::string& binary) {
        std::ofstream legacyOut("legacy_output.log", std::ios::app);
        if (legacyOut) legacyOut << binary << '\n';
    }

    void executeLegacyFunction(const std::string& binary) {
        if (binary == "01") std::cout << "[U] Quantum pulse detected -> Initiate data capture\n";
        else if (binary == "10") std::cout << "[D] Quantum inverse -> Reset chamber\n";
        else if (binary == "11") std::cout << "[L] Looping spin -> Trigger phase sync\n";
        else std::cout << "[-] Idle state\n";
    }
};

void simulateElectronFeed(QubitInterpreter& qInt) {
    std::string qubitInput = "UDLU-LD"; // sample stream (U,D,L,-)
    qInt.processSpinStream(qubitInput);
}

// -------------------- Main --------------------
int main(int argc, char* argv[]) {
    std::cout << "[TIME-VAULT] Initializing...\n";

    // Resolve vault file
    std::string vaultFile = getVaultFilePath();
    std::cout << "[INFO] Vault file path: " << vaultFile << "\n";

    // Demonstrate binary translation (if an input file provided)
    if (argc > 1) {
        std::string inputPath = argv[1];
        std::cout << "[INFO] Reading electron stream file: " << inputPath << "\n";
        std::string bits = readElectronStreamFromFile(inputPath);
        if (!bits.empty()) {
            applyQuantumNoise(bits, 1); // gentle simulated noise
            std::string ascii = binaryToAscii(bits);
            std::cout << "[Decoded ASCII] " << ascii << "\n";
        } else {
            std::cout << "[WARN] No bits read from file.\n";
        }
    } else {
        std::cout << "[INFO] No input file given. To decode a binary file, run: program <binary_file>\n";
    }

    // Vault write example (encrypted)
    std::string rawPayload = "THE SECRET TIME CAPSULE PAYLOAD - SAMPLE";
    std::string encryptedPayload = encryptPayload(rawPayload);
    writePayload(vaultFile, encryptedPayload);

    // Append the CPP-Writer payload (encrypted) to vault
    std::string encryptedCppWriter = encryptPayload(std::string(cppWriterCode));
    appendPayload(vaultFile, encryptedCppWriter);

    std::cout << "[SUCCESS] Payload(s) written to vault (encrypted).\n";
    triggerOffBoardUpdate();

    // Simulate qubit feed & process
    std::cout << "\n=== Qubit Interpreter Boot ===\n";
    QubitInterpreter qCore;
    simulateElectronFeed(qCore);

    std::cout << "\n[INFO] Program complete. Exiting.\n";
    return 0;
}
// ============================================================================
// EDUCATIONAL BINARY TRANSLATOR + DEMO CODE-LISTER
// Clean, Safe, Non‑Destructive Version
// Timestamp of last modification: 2025‑12‑03T18:32:00 (UTC‑05:00)
// ----------------------------------------------------------------------------
// Purpose:
//   • Demonstrates how to read 8‑bit binary text and convert to ASCII.
//   • Demonstrates safe file output using a public folder.
//   • Demonstrates a friendly "spin interpreter" that prints messages.
//   • Contains **no** system calls, downloads, triggers, or hidden activity.
//   • Suitable for students, hobbyists, and demonstration environments.
// ============================================================================

#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <sstream>
#include <map>
#include <thread>
#include <chrono>
#include <filesystem>
#include <algorithm>
#include <iomanip>
#include <ctime>

namespace fs = std::filesystem;

// -------------------- Utility: Binary → ASCII --------------------
std::string binaryToAscii(const std::string& bits) {
    std::string out;
    for (size_t i = 0; i + 8 <= bits.size(); i += 8) {
        std::string byte = bits.substr(i, 8);
        if (byte.find_first_not_of("01") != std::string::npos) continue;
        char c = static_cast<char>(std::stoi(byte, nullptr, 2));
        out += c;
    }
    return out;
}

// Read a text file containing 0/1 characters (ignores whitespace)
std::string loadBinaryFile(const std::string& filePath) {
    std::ifstream fin(filePath);
    if (!fin) {
        std::cerr << "[ERROR] Could not open file: " << filePath << "\n";
        return "";
    }
    std::string bits((std::istreambuf_iterator<char>(fin)),
                      std::istreambuf_iterator<char>());
    bits.erase(std::remove_if(bits.begin(), bits.end(), ::isspace), bits.end());
    return bits;
}

// -------------------- Safe Directory Path --------------------
std::string getOutputPath() {
    std::string base = fs::current_path().string() + "/SafeOutput";
    fs::create_directories(base);
    return base + "/translated_output.txt";
}

// -------------------- Light Obfuscation (Safe, Transparent) --------------------
std::string simpleObfuscate(const std::string& input) {
    std::string out = input;
    for (auto& c : out) c ^= 0x03; // small reversible transform
    return out;
}

// -------------------- Friendly Spin Interpreter --------------------
class SpinInterpreter {
public:
    SpinInterpreter() {
        map_['U'] = "01";
        map_['D'] = "10";
        map_['L'] = "11";
        map_['-'] = "00";
    }

    void run(const std::string& stream) {
        std::ofstream log("spin_log.txt", std::ios::app);
        for (char s : stream) {
            std::string b = decode(s);

            log << b << "\n";

            if (b == "01") std::cout << "[UP] Action: Data acknowledgement\n";
            else if (b == "10") std::cout << "[DOWN] Action: Reset event\n";
            else if (b == "11") std::cout << "[LOOP] Action: Synchronization signal\n";
            else std::cout << "[IDLE] No action\n";

            std::this_thread::sleep_for(std::chrono::milliseconds(150));
        }
    }

private:
    std::map<char, std::string> map_;

    std::string decode(char c) const {
        auto it = map_.find(c);
        return (it != map_.end()) ? it->second : "00";
    }
};

// -------------------- Timestamp Helper --------------------
std::string getTimeStamp() {
    std::time_t now = std::time(nullptr);
    std::tm* t = std::localtime(&now);

    std::ostringstream oss;
    oss << std::put_time(t, "%Y-%m-%d %H:%M:%S");
    return oss.str();
}

// -------------------- Main --------------------
int main(int argc, char* argv[]) {
    std::cout << "=== Safe Educational Binary Translator ===\n";
    std::cout << "Runtime Timestamp: " << getTimeStamp() << "\n";
    std::cout << "-------------------------------------------\n";

    // Translate binary file if given
    if (argc > 1) {
        std::string filePath = argv[1];
        std::cout << "[INFO] Loading binary file: " << filePath << "\n";

        std::string bits = loadBinaryFile(filePath);
        if (!bits.empty()) {
            std::string ascii = binaryToAscii(bits);
            std::cout << "[Decoded Text] " << ascii << "\n";

            std::string outPath = getOutputPath();
            std::ofstream fout(outPath);
            fout << ascii;
            std::cout << "[INFO] Saved translation to: " << outPath << "\n";
        } else {
            std::cout << "[WARN] File contained no usable binary.\n";
        }
    } else {
        std::cout << "[INFO] No binary file provided.\n";
        std::cout << "      Usage: program.exe <binary_file.txt>\n";
    }

    // Demonstrate spin interpreter
    std::cout << "\n--- Spin Interpreter Demo ---\n";
    SpinInterpreter spin;
    spin.run("UDL-UL");

    std::cout << "\n[INFO] Program finished safely.\n";
    return 0;
}
