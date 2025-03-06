# CKB Lightning Network Visualization

![CKB Lightning Network Visualization](https://img.shields.io/badge/license-MIT-blue.svg)  
A web-based visualization tool for the CKB Lightning Network, built with D3.js. This project provides an interactive force-directed graph to explore nodes, channels, and paths within the CKB Lightning Network, with enhanced features like real-time filtering, pathfinding with animated highlights, and detailed node information.

## Table of Contents
- [Overview](#overview)
- [Features](#features)
- [Demo](#demo)
- [Installation](#installation)
- [Usage](#usage)
- [Development](#development)
- [Example Data](#example-data)
- [Troubleshooting](#troubleshooting)
- [Contributing](#contributing)
- [License](#license)

## Overview

The CKB Lightning Network Visualization tool is designed to visualize the topology of the CKB (Common Knowledge Base) Lightning Network. It fetches real-time data from a JSON-RPC endpoint (`graph_nodes` and `graph_channels`) and renders an interactive graph using D3.js. This tool is ideal for developers, researchers, and users who want to analyze the network's structure, identify active nodes, and explore connectivity.

Key enhancements include animated path highlighting, public node identification with a crown icon, and a customizable RPC proxy for handling CORS issues.

## Features

- **Real-time Network Overview**:
   - Displays total nodes, total channels, CKB capacity, and UDT capacity in an overview panel.

- **Topology Visualization**:
   - Dynamically renders nodes and channels as a force-directed graph.
   - Supports zoom and drag interactions for detailed exploration.
   - Nodes are color-coded: green for active (updated within 3 days), dark gray-green for inactive.

- **Node Activity Status**:
   - Nodes' activity status is determined by their last update timestamp.

- **Query Functionality**:
   - **Single Node Query**: Highlights and centers the target node with a blinking effect.
   - **Dual Node Query**: Finds and highlights the shortest path between two nodes with pulsing green nodes and glowing edges.

- **Channel Filtering**:
   - Options to display all channels, CKB-only channels, or UDT-only channels.

- **Public Node Identification**:
   - Public nodes are marked with a crown icon above the node circle.

- **Custom RPC Support**:
   - Allows users to specify a custom RPC service address via an input field, with a proxy server to handle CORS.

- **Detailed Node Information**:
   - Click a node to view its public key, name, addresses, connected channels, CKB/UDT capacity, average fee rate, last updated time, and more.

## Demo

You can try the live demo here:  
[CKB Lightning Network Visualization](https://gpblockchain.github.io/CkbLightningNetworkVisualization/)

**Note**: The demo uses the default RPC endpoint (`https://crossorigin.me/http://18.162.235.225:8227`), which may be unavailable. Use a local RPC endpoint (e.g., `http://127.0.0.1:8229`) with the provided proxy for best results.

## Installation

### Prerequisites
- **Python 3.x**: For running the proxy server.
- **A modern browser**: Chrome, Firefox, Safari, etc.
- **An accessible RPC service**: Default is `http://127.0.0.1:8229` (e.g., a local Fiber RPC service).

### Steps
1. **Clone or Download the Project**:
   ```bash
   git clone https://github.com/gpBlockchain/CkbLightningNetworkVisualization.git
   cd CkbLightningNetworkVisualization
   ```

2. **Save the Frontend File**:
   - Save the provided `index.html` file to the project directory.

3. **Configure the Proxy Server**:
   - Create a file named `server.py` in the project directory with the following content:
     ```python
     import http.server
     import socketserver
     import urllib.request
     import json

     PORT = 8000
     RPC_URL = "http://127.0.0.1:8229"  # Replace with your RPC service URL

     class ProxyHTTPRequestHandler(http.server.SimpleHTTPRequestHandler):
         def do_POST(self):
             if self.path == '/proxy':
                 content_length = int(self.headers['Content-Length'])
                 post_data = self.rfile.read(content_length)
                 req = urllib.request.Request(RPC_URL, data=post_data, headers={'Content-Type': 'application/json'})
                 try:
                     with urllib.request.urlopen(req) as response:
                         data = response.read()
                         self.send_response(200)
                         self.send_header('Content-Type', 'application/json')
                         self.send_header('Access-Control-Allow-Origin', '*')
                         self.send_header('Access-Control-Allow-Methods', 'POST, GET, OPTIONS')
                         self.send_header('Access-Control-Allow-Headers', 'Content-Type')
                         self.end_headers()
                         self.wfile.write(data)
                 except Exception as e:
                     self.send_response(500)
                     self.send_header('Content-Type', 'application/json')
                     self.send_header('Access-Control-Allow-Origin', '*')
                     self.end_headers()
                     self.wfile.write(json.dumps({'error': str(e)}).encode())
             else:
                 super().do_POST()

         def do_OPTIONS(self):
             self.send_response(200)
             self.send_header('Access-Control-Allow-Origin', '*')
             self.send_header('Access-Control-Allow-Methods', 'POST, GET, OPTIONS')
             self.send_header('Access-Control-Allow-Headers', 'Content-Type')
             self.end_headers()

     Handler = ProxyHTTPRequestHandler
     with socketserver.TCPServer(("", PORT), Handler) as httpd:
         print(f"Serving at http://localhost:{PORT}")
         httpd.serve_forever()
     ```
   - Update `RPC_URL` to point to your CKB Lightning Network RPC service.

4. **Run the Proxy Server**:
   ```bash
   python3 server.py
   ```

5. **Access the Application**:
   - Open your browser and navigate to `http://localhost:8000/index.html`.
   - The page will automatically load data and render the topology.

## Usage

1. **View Network Overview**:
   - The top section displays total nodes, channels, and CKB/UDT capacities.

2. **Interact with the Topology**:
   - Zoom with the mouse wheel or pinch gestures.
   - Drag the graph to adjust the view.
   - Node colors: Green (active, updated within 3 days), dark gray-green (inactive).

3. **Query Nodes**:
   - Enter a single node public key (e.g., `0268847a...`) and click "Query" to highlight and center it.
   - Enter two node public keys (e.g., `0268847a...` and `02d0ab8c...`) to find and highlight the path.

4. **Filter Channels**:
   - Use the dropdown to select:
      - "Show All Channels"
      - "Show CKB Channels Only"
      - "Show UDT Channels Only"

5. **Customize RPC**:
   - Update the "RPC Address" input field (e.g., `http://localhost:8229`) and click "Fetch RPC Data" to reload data.

6. **View Node Details**:
   - Click a node to open the information panel with details like public key, capacity, and last update time.

## Development

### Technologies Used
- **Frontend**: HTML, CSS, JavaScript, D3.js (v7) for graph rendering.
- **Proxy Server**: Python with `http.server` and `urllib` to handle RPC requests and CORS.

### Project Structure
```
CkbLightningNetworkVisualization/
├── index.html        # Main HTML file with embedded JavaScript and CSS
├── server.py         # Python proxy server script
└── README.md         # Project documentation
```

### Modifying the Code
- Edit `index.html` for frontend changes:
   - Adjust `<style>` for visual styles (e.g., node colors, path animations).
   - Modify `<script>` for logic (e.g., `findPath`, `highlightPath`).
- Key functions:
   - `fetchRPC()`: Fetches data from the RPC endpoint via the proxy.
   - `processData()`: Processes `graph_channels` and `graph_nodes` data.
   - `renderGraph()`: Renders the D3.js graph with zoom and drag.
   - `findPath()`: Uses BFS to find the shortest path.
   - `highlightPath()`: Highlights paths with animated nodes and edges.

### Running Locally
Serve the project with the proxy server (`python3 server.py`) and access via `http://localhost:8000/index.html`.

## Example Data
Test with the following sample nodes:
- **Node 1**: `0268847a4791595d7d4ddaa853163839abc08f97b6950280f0dcf758cf36fb513f`
- **Node 2**: `02d0ab8ceb40e5f076519c2c00049abee8fa750b507bdd7e382ddf822825996a71`
- **Action**: Enter these public keys in the "Start Node" and "Target Node" fields and click "Query" to see a green highlighted path.

## Troubleshooting

- **Blank Page**:
   - Check the browser console (F12) for CORS or network errors.
   - Ensure the proxy server is running and the RPC URL is correct.

- **Path Not Found**:
   - Verify the public keys exist in the data.
   - Ensure the current filter includes the relevant channels.

- **Node Colors Incorrect**:
   - Check timestamp units (milliseconds expected; multiply by 1000 if in seconds).
   - Verify the current date (`March 04, 2025`) in the code.

- **Performance Issues**:
   - Large datasets may slow rendering. Reduce the `limit` in the RPC request (e.g., change `'0x1ffff'` to `'0x1000'`) or adjust simulation parameters in `renderGraph`.

## Contributing

We welcome contributions! To contribute:
1. **Fork the Repository**:
   ```bash
   git clone https://github.com/gpBlockchain/CkbLightningNetworkVisualization.git
   ```
2. **Create a Branch**:
   ```bash
   git checkout -b feature/your-feature-name
   ```
3. **Make Changes**:
   - Add features, fix bugs, or improve documentation.
   - Test your changes locally.
4. **Commit and Push**:
   ```bash
   git add .
   git commit -m "Describe your changes"
   git push origin feature/your-feature-name
   ```
5. **Submit a Pull Request**:
   - Open a PR on GitHub and describe your changes.

## License

This project is licensed under the [MIT License](LICENSE).
