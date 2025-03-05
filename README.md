# CKB Lightning Network Visualization

This is a visualization tool for the Lightning Network, built with D3.js, designed to display the topology of nodes and channels. It supports interactive features such as querying nodes, finding paths between nodes, and filtering channel types.

## Features
- **Real-time Network Overview**: Displays total nodes, total channels, ckb capacity, and UDT capacity.
- **Topology Visualization**: Dynamically renders nodes and channels with zoom and drag capabilities.
- **Node Activity Status**: Nodes are colored based on their last update time (green for active within 3 days, dark gray-green for inactive).
- **Query Functionality**:
    - Single Node Query: Highlights and centers the target node.
    - Dual Node Query: Displays the shortest path between two nodes (highlighted in green).
- **Channel Filtering**: Options to show all channels, ckb-only channels, or UDT-only channels.
- **Custom RPC**: Allows specifying the RPC service address via an input field.

## Technology Stack
- **Frontend**: HTML, CSS, JavaScript, D3.js (v7)
- **Backend Proxy**: Python (using `http.server` and `urllib`)
- **Data Source**: Fetches Lightning Network data via a JSON-RPC interface

## Installation and Setup

### Prerequisites
- Python 3.x
- A modern browser (e.g., Chrome, Firefox)
- An accessible RPC service (e.g., running at `http://127.0.0.1:8229`)

### Steps
1. **Clone or Download the Project**
   ```bash
   git clone https://github.com/gpBlockchain/CkbLightningNetworkVisualization.git
   cd CkbLightningNetworkVisualization
   ```

2. **Save the Frontend File**
    - Save the provided `index.html` file to the project directory.

3. **Configure the Proxy Server**
    - Create and save `server.py` in the same directory (see code below).
    - Ensure `RPC_URL` points to your Fiber RPC service (default: `http://127.0.0.1:8229`).

   ```python
   import http.server
   import socketserver
   import urllib.request
   import json

   PORT = 8000
   RPC_URL = "http://127.0.0.1:8229"

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

4. **Run the Proxy Server**
   ```bash
   python3 server.py
   ```

5. **Access the Page**
    - Open your browser and navigate to `http://localhost:8000/index.html`.
    - The page will automatically load data and display the topology.

## Usage
1. **View Network Overview**
    - The top section shows the total number of nodes, channels, and capacities (ckb and UDT).
2. **Interact with the Topology**
    - Use the mouse wheel to zoom, and drag to adjust the view.
    - Node colors: Green (active, updated within 3 days), Dark Gray-Green (inactive, older than 3 days).
3. **Query Nodes**
    - Enter one node public key (e.g., `0268847a...`) and click "Query" to highlight and center it.
    - Enter two node public keys (e.g., `0268847a...` and `02d0ab8c...`) to display the path between them.
4. **Filter Channels**
    - Use the dropdown menu to select "Show All Channels", "Show ckb Channels Only", or "Show UDT Channels Only".
5. **Customize RPC**
    - Modify the "RPC Address" input field and click "Fetch RPC Data" to reload data from a different source.

## Example Data
The code includes a sample channel:
- Node 1: `0268847a4791595d7d4ddaa853163839abc08f97b6950280f0dcf758cf36fb513f`
- Node 2: `02d0ab8ceb40e5f076519c2c00049abee8fa750b507bdd7e382ddf822825996a71`
- Test Path: Enter these two public keys to see a green connecting path.

## Notes
- **CORS**: If direct RPC requests (e.g., `http://127.0.0.1:8229`) fail due to CORS, use the provided `server.py` proxy or configure CORS on the RPC server.
- **Timestamp**: Assumes timestamps are in milliseconds. If in seconds, multiply `time1` and `time2` by 1000 in `processData`.
- **Node Spacing**: Set to `200` pixels by default. Adjust `distance` in `renderGraph` (e.g., to `300`) for larger spacing.
- **Performance**: Large datasets may slow rendering. Reduce `limit` in the RPC request or optimize simulation parameters.

## Troubleshooting
- **Blank Page**: Check the browser console (F12) for CORS or network errors.
- **Path Not Found**: Ensure the entered public keys exist in the data and match the current filter.
- **Node Colors**: Verify timestamp units and the current date (`March 04, 2025`) in the code.

## Contributing
Feel free to submit issues or pull requests via GitHub to improve the project!

## License
MIT License
