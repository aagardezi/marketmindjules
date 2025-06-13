# MarketMind

![MarketMind Logo](images/mmlogo1.png)

MarketMind is a sophisticated financial analyst chatbot designed to provide insightful market analysis and data-driven investment strategies. It features an intuitive user interface built with Streamlit and leverages the power of Gemini AI models for advanced analytical capabilities. MarketMind seamlessly integrates with leading financial data APIs, including Finnhub and Alpha Vantage, and can connect to Google BigQuery for robust data storage and retrieval.

## Features

- **Analyst Reports:** Generate comprehensive analyst reports for US stocks.
- **Report Comparison:** Compare analyst reports side-by-side.
- **Interactive Chat:** Chat with data and responses to gain deeper insights.
- **Symbol Lookup:** Easily find company ticker symbols.
- **Comprehensive Company Data:** Access company news, profiles, basic financials, peer analysis, insider sentiment, and SEC filings.
- **Real-time Share Prices:** Retrieve current stock prices.
- **Flexible AI Models:** Supports various Gemini models (e.g., gemini-1.5-pro-002, gemini-1.5-flash-002, gemini-2.0-flash-exp), configurable by the user.
- **Optional User Authentication:** Secure access with optional user authentication.
- **Asynchronous Operations:** Utilizes asynchronous agent capabilities for handling long-running tasks efficiently.

## Getting Started

This section will guide you through setting up and running MarketMind on your local machine or Google Cloud.

### Prerequisites

Before you begin, ensure you have the following:

- **Python:** Python 3.x (e.g., Python 3.8 or newer).
- **Google Cloud Project:** A Google Cloud Platform project with billing enabled.
- **Enabled Google Cloud APIs:**
    - Vertex AI API
    - Pub/Sub API (required if you plan to use the asynchronous agent feature for long-running tasks)
    - Secret Manager API (for securely storing API keys and other secrets)
- **Service Account:** A Google Cloud service account with the following roles (or equivalent permissions):
    - `Vertex AI User`: To interact with Gemini models.
    - `Pub/Sub Publisher`: To publish messages for asynchronous tasks.
    - `Secret Manager Secret Accessor`: To access API keys stored in Secret Manager.

### Installation

1.  **Clone the Repository:**
    ```bash
    git clone https://github.com/your-username/marketmind.git # Replace with the actual repository URL
    ```
2.  **Navigate to Project Directory:**
    ```bash
    cd marketmind
    ```
3.  **Install Dependencies:**
    It's recommended to create a virtual environment first.
    ```bash
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    pip install -r requirements.txt
    ```

### Configuration

Proper configuration is key to running MarketMind successfully.

#### API Keys & Secrets

MarketMind requires API keys for financial data providers. These keys must be stored securely in Google Cloud Secret Manager within your Google Cloud project.

Create the following secrets in Secret Manager:

-   `FinHubAccessKey`: Your API key for Finnhub.
-   `AlphaVantageKey`: Your API key for Alpha Vantage.
-   `AssetMPlatformKey`: (Required if `USEAUTH=True`) The content of your OAuth 2.0 client credentials JSON file obtained from Google Cloud Console. This is used for user authentication.

#### Environment Variables

MarketMind uses environment variables for various settings. These can be set in your shell, a `.env` file (if using a library like `python-dotenv`), or directly in your deployment environment (e.g., Cloud Run).

-   `PROJECT_ID`: Your Google Cloud Project ID. The application will attempt to automatically determine this using `helpercode.get_project_id()`, but setting it explicitly is recommended.
-   `USEAUTH`: Set to `True` to enable user authentication or `False` to disable. Defaults to `True`.
-   `TOPICID`: The Google Cloud Pub/Sub topic ID used for asynchronous agent operations. If not set, it defaults to `marketmind-async-topic`. Ensure this topic exists in your Google Cloud project if you intend to use asynchronous features.

#### BigQuery

The application is configured to potentially interact with Google BigQuery. For example, it might look for a dataset named `lseg_data_normalised` (as referenced in parts of the codebase).

-   Ensure any BigQuery datasets and tables that MarketMind expects are available in your project.
-   You may need to adjust table or dataset names in the code or configuration if your setup differs.

#### OAuth Consent Screen & Credentials

If you are using the user authentication feature (`USEAUTH=True`):

1.  **Configure OAuth Consent Screen:** In the Google Cloud Console, navigate to "APIs & Services" -> "OAuth consent screen." Configure it for your application.
2.  **Create OAuth 2.0 Client ID:**
    -   Go to "APIs & Services" -> "Credentials."
    -   Click "Create Credentials" -> "OAuth client ID."
    -   Select "Web application" as the application type.
    -   **Authorized JavaScript origins:** Add URIs like `http://localhost:8501` (for local development) and your deployed application's URL (e.g., `https://your-cloud-run-service-url.run.app`).
    -   **Authorized redirect URIs:** Add URIs like `http://localhost:8501/` and `https://your-cloud-run-service-url.run.app/`. The application will construct the full redirect path (e.g., `/login/google/authorized`).
    -   After creation, download the JSON credentials. The content of this JSON file should be stored as the `AssetMPlatformKey` secret in Google Cloud Secret Manager.

## Usage

This section describes how to run and interact with the MarketMind application.

## Technical Overview

This section provides a brief overview of the main components and directories within the MarketMind project:

-   **`main.py`**: The main entry point for the Streamlit web application. It initializes the user interface, manages session state, handles user input, and orchestrates calls to the Gemini AI models and various helper functions.

-   **`gemini*handler.py`** (e.g., `gemini15handler.py`, `gemini20handler.py`): These modules are responsible for interfacing with specific versions or configurations of the Gemini API. They manage the conversation history, send requests to the AI model, and process the responses, including handling any function calls (tool usage) requested by the model.

-   **`helper*.py`** (e.g., `helperfinhub.py`, `helperalphavantage.py`, `helperbqfunction.py`, `helpercode.py`, `helperstreamlit.py`): This collection of utility modules provides abstraction layers for various external services and internal functionalities:
    -   `helperfinhub.py`: Interacts with the Finnhub API for financial data.
    -   `helperalphavantage.py`: Interacts with the Alpha Vantage API for financial data.
    -   `helperbqfunction.py`: Handles interactions with Google BigQuery.
    -   `helpercode.py`: Contains common utility functions used across the application.
    -   `helperstreamlit.py`: Provides helper functions specifically for Streamlit UI elements and interactions.

-   **`gemini*function*.py`** (e.g., `geminifunctionfinhub.py`, `gemini20functiongeneral.py`): These files define the function declarations (tools) that are made available to the Gemini models. These declarations specify how the AI can request the application to perform actions, such as fetching data from Finnhub or looking up company information.

-   **`Dockerfile`**: Contains instructions to build a Docker image for the MarketMind application. This allows for packaging the application and its dependencies into a container for easy deployment and consistent execution across different environments.

-   **`.streamlit/config.toml`**: A configuration file for Streamlit. It can be used to customize the appearance (themes), set server options, and define other behaviors of the Streamlit application.

-   **`requirements.txt`**: Lists all the Python packages and their versions that are required to run MarketMind. This file is used by `pip` to install the dependencies.

-   **`images/`**: A directory used to store static image files, such as the project logo (`mmlogo1.png`), which are used within the application or documentation.

### Running the Application

1.  **Ensure Prerequisites and Configuration are Met:** Before running, make sure you have completed all steps in the "Getting Started" section, including installing dependencies and configuring API keys and environment variables.
2.  **Navigate to the Project Directory:**
    ```bash
    cd marketmind
    ```
3.  **Run the Streamlit Application:**
    ```bash
    streamlit run main.py
    ```
    This will typically open the application in your default web browser.

### Interacting with MarketMind

Once the application is running, you'll see a chat interface powered by Streamlit.

-   **Chat Interface:** Type your financial questions or commands directly into the input box. MarketMind will process your request and display the results.
-   **Example Queries/Commands:**
    Here are a few examples of what you can ask MarketMind (these are based on the capabilities demonstrated in the application's help information):
    *   `Can you create an analyst report for the company ALPHABET INC-CL A that includes basic financials, company news for the year 2024 and company profile. Include the actual numbers as well. Include a summary of the analysis as well.`
    *   `Can you create an analyst report for the company META`
    *   `Can you compare the above analyst reports and give me a summary list pros and cons with a rating (Buy, Sell, Hold)`
    *   To get current share price: `GOOGL for the last 6 months` (Note: The "last 6 months" part might be a placeholder for how you'd ask for price, the core is requesting a symbol like `GOOGL`)
    *   Similarly for another company: `META for the last 6 months`

### Selecting AI Models

MarketMind allows you to choose from different Gemini AI models for analysis.

-   You can typically find an option in the user interface (e.g., a dropdown or a dialog box triggered by a button like "Select Model") to switch between available models such as `gemini-1.5-pro-latest`, `gemini-1.5-flash-latest`, etc. The exact list of models may vary based on availability and configuration.

### Authentication

-   If user authentication is enabled (i.e., the `USEAUTH` environment variable is set to `True`), you will be prompted to log in with your Google account when you first access the application.
-   If `USEAUTH` is `False`, the application will be accessible without a login.

### Asynchronous Agent

-   MarketMind includes an "Async Agent" feature, which can be useful for analyses that might take a longer time to complete.
-   You may find a toggle or an option in the UI to enable or disable this feature for specific requests. When enabled, the application can handle these long-running tasks in the background without making you wait, potentially notifying you when complete (depending on the implementation).

## Contributing

Contributions are welcome! If you have suggestions for improvements or new features, please feel free to:
1. Open an issue to discuss what you would like to change.
2. Fork the repository and create a pull request with your contributions.

## License

This project is currently not licensed. All rights reserved.
