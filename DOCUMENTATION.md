# Elastos Carrier Classic Crawler - Complete Documentation

## Project Overview

The **Elastos Carrier Classic Crawler** is a network analysis tool designed to discover and map the topology of the Elastos Carrier network. It systematically crawls the decentralized peer-to-peer network to collect information about active nodes, their geographical locations, and network connectivity patterns.

### Key Features

- **Multi-threaded crawling**: Runs multiple concurrent crawler instances (configurable, default: 5)
- **Network topology discovery**: Maps the Carrier network by sending `get_nodes` requests
- **Geographical mapping**: Integrates with IP2Location database to resolve node locations
- **Data persistence**: Saves discovered nodes to timestamped files
- **Real-time monitoring**: Provides logging and status updates during crawling
- **Bootstrap support**: Uses predefined bootstrap nodes to join the network

## Architecture & Technical Details

### Core Components

#### 1. **Main Crawler Engine** (`src/main.c`)
- **Crawler Structure**: Each crawler instance maintains:
  - Tox client instance for network communication
  - DHT (Distributed Hash Table) access
  - Dynamic nodes list with automatic resizing
  - Send pointer tracking for request management
  - Timestamps for timeout management

#### 2. **Configuration Management** (`src/config.c`, `src/config.h`)
- Parses configuration files using libconfig
- Manages bootstrap nodes, timeouts, intervals, and data directories
- Supports flexible configuration for different deployment scenarios

#### 3. **Dependencies**
- **c-toxcore**: Core networking and DHT functionality
- **libconfig**: Configuration file parsing
- **libsodium**: Cryptographic operations
- **IP2Location**: Geographical location resolution
- **libcrystal**: Utility functions (logging, encoding)

### How the Crawler Works

#### Network Discovery Process

1. **Initialization**:
   - Creates Tox client instances for each crawler thread
   - Connects to bootstrap nodes to join the network
   - Initializes DHT callback handlers

2. **Node Discovery Loop**:
   ```
   For each crawler instance:
   ├── Send get_nodes requests to discovered nodes
   ├── Process responses containing new node information
   ├── Add unique nodes to the local nodes list
   ├── Track geographical locations using IP2Location
   └── Continue until timeout or network exhaustion
   ```

3. **Request Strategy**:
   - Sends requests to `requests_per_interval` nodes per interval
   - Includes random requests to improve network coverage
   - Implements exponential backoff and timeout handling

4. **Data Collection**:
   - Records node public keys, IP addresses, and ports
   - Resolves geographical locations when IP2Location database is available
   - Maintains statistics on discovery progress

#### Thread Management

- **Controller Thread**: Manages crawler lifecycle and spawning
- **Crawler Threads**: Independent discovery workers (detached threads)
- **Synchronization**: Uses atomic operations and mutexes for thread safety

#### Data Output Format

Each completed crawl generates a file with format: `{timestamp}.lst`
```
{node_public_key_base58}, {ip_address}, {geographical_location}
```

Example:
```
89vny8MrKdDKs7Uta9RdVmspPjnRMdwMmaiEW27pZ7gh, 13.58.208.50, United States, Ohio, Columbus
G5z8MqiNDFTadFUPfMdYsYtkUDbX5mNCMVHMZtsCnFeb, 18.216.102.47, United States, Ohio, Columbus
```

## Configuration Reference

### Configuration File Structure (`elacrawler.conf`)

```ini
# Crawler timing settings
interval = 60                    # Minutes between crawler launches
timeout = 60                     # Seconds to wait for new nodes before stopping

# Resource management  
max_crawlers = 5                 # Maximum concurrent crawler instances
data_dir = "~/.elacrawler"       # Directory for output files and cache

# Logging configuration
log_level = 4                    # Log verbosity (1=Error, 4=Verbose)
log_file = "crawler.log"         # Optional log file path

# IP geolocation (optional)
ip2location_database = "IP2LOCATION-LITE-DB9.BIN"

# Bootstrap nodes for network entry
bootstraps = (
  {
    ipv4 = "13.58.208.50"
    port = 33445
    public_key = "89vny8MrKdDKs7Uta9RdVmspPjnRMdwMmaiEW27pZ7gh"
  },
  # ... additional bootstrap nodes
)
```

### Advanced Configuration Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `interval` | uint32 | 60 | Minutes between new crawler launches |
| `timeout` | uint32 | 60 | Seconds to wait for new nodes before stopping |
| `max_crawlers` | uint32 | 5 | Maximum concurrent crawler instances |
| `initial_nodes_list_size` | uint32 | 4096 | Initial capacity for nodes list |
| `request_interval` | uint32 | 1 | Seconds between request batches |
| `requests_per_interval` | uint32 | 8 | Number of requests per batch |
| `random_requests` | uint32 | 32 | Additional random requests per node |

## Building from Source

### Prerequisites

- **CMake** 3.5 or higher
- **C compiler** (GCC, Clang)
- **Git** for dependency management
- **autotools** (autoconf, automake, libtool)
- **pkg-config**

### Build Process

```bash
# Clone the repository
git clone https://github.com/elastos/Elastos.NET.Carrier.Crawler
cd Elastos.CarrierClassic.Crawler

# Create build directory
mkdir build
cd build

# Configure build
cmake -DCMAKE_INSTALL_PREFIX=dist ..

# Build and install
make install
```

### Build Output

After successful build:
```
build/
├── dist/
│   ├── bin/
│   │   └── elacrawler              # Main executable
│   └── etc/
│       └── carrier/
│           └── crawler.conf        # Configuration file
└── intermediates/                  # Build artifacts and libraries
```

## Running the Crawler

### Basic Usage

```bash
# From project root directory
cd /Users/weili/blockchain/ela/Elastos.CarrierClassic.Crawler

# Run with default configuration
./build/dist/bin/elacrawler -c build/dist/etc/carrier/crawler.conf

# Run with custom configuration
./build/dist/bin/elacrawler --config=/path/to/custom/crawler.conf

# Show help
./build/dist/bin/elacrawler --help
```

### Command Line Options

```
Usage: elacrawler [OPTION]...

  -c, --config=CONFIG_FILE  Set config file path
      --once                Run once then exit (Note: may not be implemented)
      --debug               Wait for debugger attach after start
```

### Runtime Behavior

#### Normal Operation
1. **Startup**: Loads configuration and initializes IP2Location database
2. **Network Connection**: Connects to bootstrap nodes
3. **Discovery Phase**: Launches crawler threads according to configuration
4. **Data Collection**: Continuously discovers and records nodes
5. **File Output**: Saves results when crawlers complete or timeout

#### Output Files Location
- **Data Directory**: `~/.elacrawler/` (configurable)
- **File Format**: `{YYYYMMDD}/{timestamp}.lst`
- **Content**: CSV format with node ID, IP address, and location

#### Log Messages
```
2025-09-08 15:56:00 - INFO    : Crawler[1] - created and running.
2025-09-08 15:56:07 - INFO    : Crawler[1] - connection status: Connected/UDP
2025-09-08 15:56:15 - VERBOSE : Crawler[1] - 89vny8MrKdDKs7Uta9RdVmspPjnRMdwMmaiEW27pZ7gh, 13.58.208.50, United States, Ohio, Columbus - 1
```

## Optional IP2Location Setup

### Database Installation

1. **Download Database**:
   ```bash
   # Visit https://www.ip2location.com/
   # Download IP2LOCATION-LITE-DB9.BIN (free) or commercial version
   ```

2. **Install Database**:
   ```bash
   # Place in same directory as crawler.conf
   cp IP2LOCATION-LITE-DB9.BIN build/dist/etc/carrier/
   ```

3. **Verify Configuration**:
   ```ini
   # In crawler.conf
   ip2location_database = "IP2LOCATION-LITE-DB9.BIN"
   ```

### Without IP2Location

The crawler works without IP2Location database:
- **Warning message**: "IP2Location - open database failed, check config file!"
- **Functionality**: All network discovery features work normally
- **Output**: Location field shows "Unknown" instead of geographical data

## Troubleshooting

### Common Issues

#### 1. **Build Failures**
```bash
# Clean and rebuild
rm -rf build/
mkdir build && cd build
cmake -DCMAKE_INSTALL_PREFIX=dist ..
make install
```

#### 2. **Executable Not Found**
```bash
# Ensure you're in the correct directory
cd /Users/weili/blockchain/ela/Elastos.CarrierClassic.Crawler
ls -la build/dist/bin/elacrawler
```

#### 3. **Configuration Errors**
```bash
# Verify configuration file exists and is readable
cat build/dist/etc/carrier/crawler.conf
```

#### 4. **Network Connection Issues**
- **Check bootstrap nodes**: Verify bootstrap node addresses are reachable
- **Firewall settings**: Ensure UDP port 33445 is not blocked
- **Network connectivity**: Test basic internet connectivity

#### 5. **Permission Issues**
```bash
# Ensure data directory is writable
mkdir -p ~/.elacrawler
chmod 755 ~/.elacrawler
```

### Debug Mode

```bash
# Run with debug flag (waits for debugger attachment)
./build/dist/bin/elacrawler -c build/dist/etc/carrier/crawler.conf --debug
```

### Verbose Logging

Increase log level in configuration:
```ini
log_level = 5    # Maximum verbosity
log_file = "debug.log"
```

## Performance Considerations

### Resource Usage
- **Memory**: ~50MB per crawler instance
- **CPU**: Low to moderate, depends on network activity
- **Network**: Moderate bandwidth usage for DHT requests
- **Disk**: Minimal, only for result files

### Optimization Tips
1. **Adjust crawler count**: Balance between speed and resource usage
2. **Tune timeouts**: Longer timeouts discover more nodes but take more time
3. **Request intervals**: Lower intervals increase discovery speed but network load

## Development and Contribution

### Code Structure
```
src/
├── main.c          # Core crawler implementation
├── config.c        # Configuration management
├── config.h        # Configuration structures
└── elacrawler.conf # Default configuration
```

### Key Functions
- `crawler_thread_routine()`: Main crawler loop
- `getnodes_response_callback()`: Handles node discovery responses
- `crawler_send_node_requests()`: Manages network requests
- `crawler_dump_nodes()`: Saves results to files

### Dependencies Management
The project uses CMake's ExternalProject to manage dependencies:
- Automatic download and compilation of required libraries
- Cross-platform compatibility
- Version pinning for reproducible builds

## License and Attribution

- **License**: MIT License (Elastos Foundation)
- **Based on**: toxcrawler by JFreegman
- **Dependencies**: Various open-source libraries (see THANKS.md)

This crawler is part of the Elastos ecosystem and contributes to understanding and monitoring the Carrier network's health and topology.

