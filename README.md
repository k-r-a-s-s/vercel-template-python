# My Vercel Python API Template

Welcome! This repository serves as a template for building Python-based APIs deployed as Serverless Functions on Vercel.

## Project Structure

```
my-vercel-python-api/
│
├── api/                  # Vercel Serverless Functions
│   ├── __init__.py       # Makes 'api' a package (optional but good practice)
│   ├── webhook_handler.py # Entry point: /api/webhook_handler
│   └── another_endpoint.py # Example: /api/another_endpoint
│
├── utils/                # Shared utility code
│   ├── __init__.py
│   ├── db.py             # Lightweight storage logic (e.g., SQLite, Supabase client)
│   └── helpers.py        # Shared logic across endpoints
│
├── tests/                # Unit and integration tests
│   ├── __init__.py
│   ├── test_webhook_handler.py
│   └── test_helpers.py
│
├── requirements.txt      # Python dependencies
├── vercel.json           # Vercel configuration (optional: rewrites, env)
├── .gitignore            # Files to ignore in git
├── README.md             # This file
└── LICENSE               # Your chosen license (e.g., MIT)
```

## How Vercel Serverless Functions Work (Python)

Vercel allows you to deploy individual Python files within the `/api` directory as serverless functions. You **do not** need a web framework like Flask or Django to handle routing for basic APIs.

1.  **Entry Point:** Each `.py` file in the `api/` directory becomes an API endpoint. The path to the file determines the URL path. For example, `api/webhook_handler.py` becomes accessible at `/api/webhook_handler`.
2.  **Handler Function:** Vercel looks for a specific function within your Python file to handle incoming HTTP requests. By default, it looks for a function named `handler`. This function typically accepts request details (like `request` from the `http.server` module or similar, depending on the runtime) and should return a response. You can configure the entrypoint function name in `vercel.json` if needed.
3.  **Dependencies:** List your project's dependencies in `requirements.txt`. Vercel automatically installs these dependencies during the build process.
4.  **Shared Code:** Place reusable code, like database interactions (`utils/db.py`) or helper functions (`utils/helpers.py`), outside the `api/` directory and import them into your endpoint files.

**Example (`api/webhook_handler.py` structure):**

```python
# Example using standard library http.server for structure
# Vercel might use a different base class depending on runtime settings
from http.server import BaseHTTPRequestHandler, HTTPServer
import json
from urllib.parse import urlparse, parse_qs

# You can import shared utilities
# from utils.helpers import some_helper_function

class handler(BaseHTTPRequestHandler):

    def do_POST(self):
        content_length = int(self.headers.get('Content-Length', 0))
        post_data = self.rfile.read(content_length)
        data = {}
        try:
            data = json.loads(post_data)
        except json.JSONDecodeError:
            self.send_response(400)
            self.send_header('Content-type','application/json')
            self.end_headers()
            self.wfile.write(json.dumps({"error": "Invalid JSON"}).encode('utf-8'))
            return

        # Process the webhook data (e.g., call helpers)
        print(f"Received webhook data: {data}")
        # response_data = some_helper_function(data)
        response_data = {"status": "received", "your_data": data}


        # Send response
        self.send_response(200)
        self.send_header('Content-type','application/json')
        self.end_headers()
        self.wfile.write(json.dumps(response_data).encode('utf-8'))
        return

    def do_GET(self):
        # Handle GET requests if needed, perhaps for health checks
        self.send_response(200)
        self.send_header('Content-type','application/json')
        self.end_headers()
        self.wfile.write(json.dumps({"message": "Hello from the webhook handler GET!"}).encode('utf-8'))
        return

# Vercel often runs this in a way where you don't need the typical
# if __name__ == '__main__': block for running a local server.
# The `handler` class itself is the target.
```

## Using Vercel

1.  **Sign Up:** Create an account at [vercel.com](https://vercel.com/).
2.  **Install Vercel CLI (Optional but Recommended):**
    ```bash
    npm install -g vercel
    ```
3.  **Connect Your Repository:**
    *   Push this repository to GitHub, GitLab, or Bitbucket.
    *   Go to your Vercel dashboard and import the project. Vercel should automatically detect it as a Python project.
4.  **Deployment:**
    *   **Automatic:** Vercel automatically deploys pushes to your main branch (and previews for other branches/PRs).
    *   **Manual (CLI):** Navigate to your project's root directory (`my-vercel-python-api`) in your terminal and run:
        ```bash
        vercel
        ```
        The first time, you'll link the local directory to your Vercel project. Subsequent runs of `vercel` will deploy changes. Use `vercel --prod` to deploy directly to production.
5.  **Environment Variables:**
    *   Go to your project settings on the Vercel dashboard.
    *   Navigate to "Environment Variables".
    *   Add any secrets (API keys, database URLs) here. **Do not commit secrets directly into your code or `.env` files if the repository is public.** Use environment variables for production, preview, and development environments as needed. Your serverless functions can access these using standard Python methods (e.g., `os.environ.get('MY_VARIABLE')` after `import os`).
6.  **Logs:** View real-time and historical logs for your deployments and function invocations directly from the Vercel dashboard.

## Development

1.  **Install Dependencies:**
    ```bash
    python -m venv venv        # Create a virtual environment (optional but recommended)
    source venv/bin/activate   # On Windows use `venv\Scripts\activate`
    pip install -r requirements.txt
    ```
2.  **Local Testing:** You can test individual function logic locally. For testing the HTTP handling part, you might use `vercel dev` (requires Vercel CLI and Node.js) which simulates the Vercel environment locally, or set up a minimal local server using Python's built-in `http.server` or a micro-framework if necessary for more complex testing.
    ```bash
    vercel dev
    ```
3.  **Add Endpoints:** Create new `.py` files in the `api/` directory.
4.  **Add Tests:** Write tests for your endpoints and utility functions in the `tests/` directory using a framework like `pytest`.

Happy Coding! 