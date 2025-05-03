[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/shoofio-brave-search-mcp-sse-badge.png)](https://mseep.ai/app/shoofio-brave-search-mcp-sse)

# Brave Search MCP/SSE Server

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker Hub](https://img.shields.io/docker/v/shoofio/brave-search-mcp-sse/latest?sort=semver&label=Docker%20Hub)](https://hub.docker.com/r/shoofio/brave-search-mcp-sse)
[![Helm Chart](https://img.shields.io/badge/Helm%20Chart-1.0.10-blue?link=https://shoofio.github.io/brave-search-mcp-sse/)](https://shoofio.github.io/brave-search-mcp-sse/)
<!-- Add other badges here if applicable (e.g., build status, Docker pulls) -->

An implementation of the Model Context Protocol (MCP) using Server-Sent Events (SSE) that integrates the [Brave Search API](https://brave.com/search/api/), providing AI models and other clients with web and local search capabilities through a streaming interface.

## Overview

This server acts as a tool provider for Large Language Models that understand the Model Context Protocol. It exposes Brave's powerful web and local search functionalities via an SSE connection, allowing for real-time streaming of search results and status updates.

**Key Design Goals:**

*   **Centralized Access:** Designed with centrality in mind, allowing organizations or individuals to manage a single Brave Search API key and provide controlled access to multiple internal clients or applications.
*   **Observability:** Features robust logging to track requests, API interactions, errors, and rate limits, providing visibility into usage and aiding debugging.
*   **Flexible Deployment:** Can be deployed privately within a network or optionally exposed publicly via methods like Kubernetes Ingress or direct Docker port mapping.

## Features

*   **Web Search**: Access Brave's independent web search index for general queries, news, articles, etc. Supports pagination and filtering controls.
*   **Local Search**: Find businesses, restaurants, and services with detailed information like address, phone number, and ratings.
*   **Smart Fallbacks**: Local search automatically falls back to a filtered web search if no specific local results are found for the query.
*   **Server-Sent Events (SSE)**: Efficient, real-time streaming of search results and tool execution status.
*   **Model Context Protocol (MCP)**: Adheres to the MCP standard for seamless integration with compatible clients.
*   **Docker Support**: Includes a `Dockerfile` for easy containerization and deployment.
*   **Helm Chart**: Provides a Helm chart for straightforward deployment to Kubernetes clusters.

## Prerequisites

Depending on your chosen deployment method, you will need some of the following:

*   **Brave Search API Key**: Required for all deployment methods. See "Getting Started" below.
*   **Docker**: Required if deploying using Docker.
*   **kubectl & Helm**: Required if deploying to Kubernetes using Helm.
*   **Node.js & npm**: Required *only* for local development (Node.js v22.x or later recommended).
*   **Git**: Required for cloning the repository for local development or building custom Docker images.

## Getting Started

### 1. Obtain a Brave Search API Key

1.  Sign up for a [Brave Search API account](https://brave.com/search/api/).
2.  Choose a plan (a free tier is available).
3.  Generate your API key from the [developer dashboard](https://api.search.brave.com/app/keys).

### 2. Configuration

The server requires the Brave Search API key to be set via the `BRAVE_API_KEY` environment variable.

Other potential environment variables (check `src/config/config.ts` for details):
*   `PORT`: The port the server listens on (defaults to `8080`).
*   `LOG_LEVEL`: Logging verbosity (e.g., `info`, `debug`).

Set these variables in your environment or using a `.env` file in the project root for local development.

## Installation & Usage

Choose the deployment method that best suits your needs:

### Option 1: Docker (Recommended for Deployment)

**Prerequisites:** Docker installed.

1.  **Obtain a Brave Search API Key:** Follow the steps in the "Getting Started" section.
2.  **Pull the Docker image:**
    Pull the latest image from Docker Hub:
    ```bash
    docker pull shoofio/brave-search-mcp-sse:latest
    ```
    Or pull a specific version tag (e.g., `1.0.10`):
    ```bash
    docker pull shoofio/brave-search-mcp-sse:1.0.10
    ```
    *(Alternatively, you can build the image locally if needed. Clone the repository and run `docker build -t brave-search-mcp-sse:custom .`)*
3.  **Run the Docker container:**
    Use the tag you pulled (e.g., `latest` or `1.0.10`):
    ```bash
    docker run -d --rm \
      -p 8080:8080 \
      -e BRAVE_API_KEY="YOUR_API_KEY_HERE" \
      -e PORT="8080" # Optional: Define the port if needed
      # -e LOG_LEVEL="info" # Optional: Set log level
      --name brave-search-server \
      shoofio/brave-search-mcp-sse:latest # Or your specific tag
    ```
    This runs the server in detached mode, mapping port 8080 on your host to the container.

### Option 2: Helm (Kubernetes Deployment)

**Prerequisites:** `kubectl` connected to your cluster, Helm installed.

1.  **Obtain a Brave Search API Key:** Follow the steps in the "Getting Started" section.
2.  **Add the Helm repository:**
    ```bash
    helm repo add brave-search-mcp-sse https://shoofio.github.io/brave-search-mcp-sse/
    helm repo update
    ```
3.  **Prepare API Key Secret (Recommended):**
    Create a Kubernetes secret in the target namespace:
    ```bash
    kubectl create secret generic brave-search-secret \
      --from-literal=api-key='YOUR_API_KEY_HERE' \
      -n <your-namespace>
    ```
4.  **Install the Helm chart:**
    The chart version corresponds to the application version (latest is `1.0.10`). Install using the secret:
    ```bash
    helm install brave-search brave-search-mcp-sse/brave-search-mcp-sse \
      -n <your-namespace> \
      --set braveSearch.existingSecret=brave-search-secret
      # Optionally specify a version: --version 1.0.10
    ```
    Or provide the key directly (less secure):
    ```bash
    helm install brave-search brave-search-mcp-sse/brave-search-mcp-sse \
      -n <your-namespace> \
      --set braveSearch.apiKey="YOUR_API_KEY_HERE"
    ```
5.  **Chart Configuration:**
    You can customize the deployment by overriding default values. Create a YAML file (e.g., `dev-values.yaml`, `prod-values.yaml`) with your desired settings and use the `-f` flag during installation: `helm install ... -f dev-values.yaml`.

    Refer to the chart's default [`values.yaml`](./helm/brave-search-mcp-sse/values.yaml) file to see all available configuration options and their default settings.

### Option 3: Local Development

**Prerequisites:** Node.js and npm (v22.x or later recommended), Git.

1.  **Obtain a Brave Search API Key:** Follow the steps in the "Getting Started" section.
2.  **Clone the repository:**
    ```bash
    git clone <repository_url> # Replace with the actual URL
    cd brave-search-mcp-sse
    ```
3.  **Install dependencies:**
    ```bash
    npm install
    ```
4.  **Set Environment Variables:**
    Create a `.env` file in the root directory:
    ```env
    BRAVE_API_KEY=YOUR_API_KEY_HERE
    PORT=8080
    # LOG_LEVEL=debug
    ```
5.  **Build the TypeScript code:**
    ```bash
    npm run build
    ```
6.  **Run the server:**
    ```bash
    npm start
    # Or for development with auto-reloading (if nodemon/ts-node-dev is configured)
    # npm run dev
    ```
    The server will start listening on the configured port (default `8080`).

## API / Protocol Interaction

Clients connect to this server via HTTP GET request to establish an SSE connection. The specific endpoint depends on your deployment (e.g., `http://localhost:8080/`, `http://<k8s-service-ip>:8080/`, or through an Ingress).

Once connected, the server and client communicate using MCP messages over the SSE stream.

### Available Tools

The server exposes the following tools to connected clients:

1.  **`brave_web_search`**
    *   **Description**: Performs a general web search using the Brave Search API.
    *   **Inputs**:
        *   `query` (string, required): The search query.
        *   `count` (number, optional): Number of results to return (1-20, default 10).
        *   `offset` (number, optional): Pagination offset (0-9, default 0).
        *   *(Other Brave API parameters like `search_lang`, `country`, `freshness`, `result_filter`, `safesearch` might be supported - check `src/services/braveSearchApi.ts`)*
    *   **Output**: Streams MCP messages containing search results (title, URL, snippet, etc.).

2.  **`brave_local_search`**
    *   **Description**: Performs a search for local businesses and places using the Brave Search API. Falls back to web search if no local results are found.
    *   **Inputs**:
        *   `query` (string, required): The local search query (e.g., "pizza near me", "cafes in downtown").
        *   `count` (number, optional): Maximum number of results (1-20, default 5).
    *   **Output**: Streams MCP messages containing local business details (name, address, phone, rating, etc.).

*(Example using `curl` - Note: Actual MCP interaction requires a client library)*
```bash
# Example: Connect to SSE endpoint (won't show MCP messages directly)
curl -N http://localhost:8080/ # Or your deployed endpoint
```

### Client Configuration Example (Cursor)

To use this server with an MCP client like Cursor, you need to configure the client to connect to the server's SSE endpoint.

Add the following configuration to your Cursor settings (`mcp.json` or similar configuration file), replacing the URL with the actual address and port where your `brave-search-mcp-sse` server is accessible:

```json
{
  "mcpServers": {
    "brave-search": {
      "transport": "sse",
      "url": "http://localhost:8080/sse"
    }
  }
}
```

**Explanation:**

*   `transport`: Must be set to `"sse"` for this server.
*   `url`: This is the crucial part.
    *   If running locally via Docker (as shown in the example), `http://localhost:8080/sse` is likely correct.
    *   If running in Kubernetes, replace `localhost:8080` with the appropriate Kubernetes Service address/port or the Ingress hostname/path configured to reach the server's port 8080.
    *   Ensure the URL path ends with `/sse`.

*(Similar configuration steps might apply to other MCP clients that support the SSE transport, like recent versions of Claude Desktop, but refer to their specific documentation.)*

## Project Structure

```
.
├── Dockerfile             # Container build definition
├── helm/                  # Helm chart for Kubernetes deployment
│   └── brave-search-mcp-sse/
├── node_modules/        # Project dependencies (ignored by git)
├── src/                   # Source code (TypeScript)
│   ├── config/            # Configuration loading
│   ├── services/          # Brave API interaction logic
│   ├── tools/             # Tool definitions for MCP
│   ├── transport/         # SSE/MCP communication handling
│   ├── types/             # TypeScript type definitions
│   ├── utils/             # Utility functions
│   └── index.ts           # Main application entry point
├── dist/                  # Compiled JavaScript output (ignored by git)
├── package.json           # Project metadata and dependencies
├── tsconfig.json          # TypeScript compiler options
├── .env.example           # Example environment file
├── .gitignore
└── README.md              # This file
```

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request with your changes. Ensure your code adheres to the existing style and includes tests where applicable. I will review PRs as time permits.

## License

This MCP server is licensed under the[MIT License](./LICENSE). This means you are free to use, modify, and distribute the software, subject to the terms and conditions of the MIT License. For more details, please see the LICENSE file in the project repository.