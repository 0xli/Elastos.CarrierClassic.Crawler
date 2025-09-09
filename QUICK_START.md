# Quick Start Guide - Elastos Carrier Classic Crawler

## ✅ Build Status: WORKING
The project has been successfully fixed and builds correctly on ARM64 macOS.

## 🚀 How to Run

### Step 1: Navigate to Project Directory
```bash
cd /Users/weili/blockchain/ela/Elastos.CarrierClassic.Crawler
```

### Step 2: Run the Crawler
```bash
# Run the crawler with the default configuration
./build/dist/bin/elacrawler -c build/dist/etc/carrier/crawler.conf
```

### Step 3: Monitor Output
The crawler will display status messages like:
```
2025-09-08 15:56:00 - INFO    : Crawler[1] - created and running.
2025-09-08 15:56:07 - INFO    : Crawler[1] - connection status: Connected/UDP
```

### Step 4: View Results
- **Output files**: `~/.elacrawler/{date}/{timestamp}.lst`
- **Format**: `node_id, ip_address, location`
- **Updates**: New files created when crawlers complete

## 📋 Command Options

```bash
# Show help
./build/dist/bin/elacrawler --help

# Use custom config file
./build/dist/bin/elacrawler -c /path/to/custom/config.conf

# Debug mode (waits for debugger)
./build/dist/bin/elacrawler -c build/dist/etc/carrier/crawler.conf --debug
```

## ⚙️ Configuration

Edit `build/dist/etc/carrier/crawler.conf` to customize:
- `interval = 60` - Minutes between crawler launches
- `max_crawlers = 5` - Number of concurrent crawlers  
- `timeout = 60` - Seconds to wait for new nodes
- `data_dir = "~/.elacrawler"` - Output directory

## 🛑 Stop the Crawler

Press `Ctrl+C` to gracefully stop all crawlers.

## 📁 Output Location

Results are saved to: `~/.elacrawler/{YYYYMMDD}/{timestamp}.lst`

Example file content:
```
89vny8MrKdDKs7Uta9RdVmspPjnRMdwMmaiEW27pZ7gh, 13.58.208.50, United States, Ohio, Columbus
G5z8MqiNDFTadFUPfMdYsYtkUDbX5mNCMVHMZtsCnFeb, 18.216.102.47, United States, Ohio, Columbus
```

## ⚠️ Notes

- **IP2Location Warning**: Normal if you don't have the database file
- **Network Required**: Needs internet connection to discover nodes
- **Patience**: Initial connection may take 30-60 seconds

## 🔧 Troubleshooting

**If executable not found:**
```bash
# Check if build completed successfully
ls -la build/dist/bin/elacrawler
```

**If permission denied:**
```bash
chmod +x build/dist/bin/elacrawler
```

**If config file not found:**
```bash
# Verify config file exists
ls -la build/dist/etc/carrier/crawler.conf
```

---
**Status**: ✅ Ready to use - All dependencies fixed and working on ARM64 macOS

