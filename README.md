# Openwebui-Vertex-ai-Pipe
A collection of pipe functions that intergrate Gemini and Imagen models with Openwebui 

Yes, I have a clear understanding of the correct approach now.

Based on our discussion and your `docker-compose.yml`, it's evident that you are using the more advanced and flexible method of on-the-fly package installation, which is a great way to handle this.

Here is a completely rewritten README file, tailored to this superior method. This version is more accurate and significantly simpler for the end-user, as it doesn't require them to build custom Docker images.

***

# Vertex AI Imagen 3 & 4 Hybrid Function for OpenWebUI

**Author:** Daniel Parsons
**Version:** 1.1.1

## Overview

This guide provides instructions for installing the Vertex AI Imagen 3 & 4 Hybrid Function in OpenWebUI. This function connects directly to Google's Vertex AI platform, allowing you to generate and upscale images from within the OpenWebUI chat interface.

It is designed to be installed without needing to build custom Docker images, instead leveraging OpenWebUI's built-in environment variable for installing Python packages at runtime.

### Features

* **Direct Google AI Integration:** Adds "Vertex AI Imagen 3/4" as a selectable model in your UI.
* **Powerful Image Generation:** Create high-quality images from text prompts.
* **Image Upscaling:** Upscale existing images to 2x or 4x their original resolution.
* **In-Chat Parameter Control:** Use flags directly in your prompt to control image count (`--count`), aspect ratio (`--aspect-ratio`), negative prompts, watermarking, and more.
* **Real-time Feedback:** Receive status updates as your images are processed.

---

## Prerequisites

Before you begin, ensure you have the following:

1.  **A running OpenWebUI instance** using `docker-compose`.
2.  **A Google Cloud Project** with an active billing account.
3.  **The Vertex AI API enabled** in your Google Cloud Project.
4.  **A Service Account with a JSON key** for secure authentication.

---

## Installation Guide

Follow these three steps to configure your environment and install the function.

### Step 1: Configure Your Google Cloud Project

First, set up the necessary credentials and permissions in your Google Cloud Console.

#### A. Enable the Vertex AI API

1.  Navigate to the [Google Cloud Console](https://console.cloud.google.com/).
2.  Using the top search bar, find and select the **Vertex AI API**.
3.  If the API is not already enabled, click the **Enable** button.

#### B. Create a Service Account & Download the Key

1.  In the Google Cloud Console, navigate to **IAM & Admin > Service Accounts**.
2.  Click **+ CREATE SERVICE ACCOUNT**.
3.  Provide a name (e.g., "openwebui-vertexai") and click **CREATE AND CONTINUE**.
4.  In the "Role" dropdown, search for and select **Vertex AI User**.
5.  Click **CONTINUE**, then **DONE**.
6.  Back on the Service Accounts page, find the account you just created. Click the three vertical dots under the "Actions" column and select **Manage keys**.
7.  Click **ADD KEY > Create new key**.
8.  Select **JSON** as the key type and click **CREATE**. A JSON file will be downloaded to your computer.

### Step 2: Configure Your OpenWebUI Environment

Now, you'll modify your `docker-compose.yml` to give OpenWebUI the necessary dependencies and access to your key file.

#### A. Place the Service Account Key

1.  On the machine running Docker, navigate to the directory containing your `docker-compose.yml` file.
2.  Create a new folder here named `credentials`.
3.  Move the downloaded JSON key file into this new `credentials` folder and rename it to `service-account-key.json`.

#### B. Modify `docker-compose.yml`

1.  Open your `docker-compose.yml` file in a text editor.
2.  Locate the service definition for `open-webui`.
3.  **Add the following environment variable** under the `environment:` section. This tells OpenWebUI to install the required Python packages when it starts.

    ```yaml
    - OPENWEBUI_PYTHON_PACKAGES=google-auth-oauthlib google-auth-transport-requests httpx pydantic
    ```
4.  **Add the following volume mount** under the `volumes:` section. This makes your key file securely available inside the container.

    ```yaml
    - ./credentials:/app/credentials:ro
    ```
    *The `:ro` makes the file read-only, which is a security best practice.*

    After your changes, the `open-webui` service in your `docker-compose.yml` should look similar to this:

    ```yaml
    services:
      open-webui:
        image: ghcr.io/open-webui/open-webui:main # Or your specific version
        container_name: open-webui
        volumes:
          - openwebui-data:/app/backend/data
          - ./credentials:/app/credentials:ro  # <-- ADD THIS LINE
        environment:
          # ... your other environment variables ...
          - OPENWEBUI_PYTHON_PACKAGES=google-auth-oauthlib google-auth-transport-requests httpx pydantic # <-- ADD THIS LINE
        # ... rest of your service definition ...
    ```

5.  Save the `docker-compose.yml` file and restart your OpenWebUI stack to apply the changes.

    ```bash
    docker-compose down && docker-compose up -d
    ```

### Step 3: Install the Function in the UI

The final step is to add the script code into the OpenWebUI interface.

1.  Open your OpenWebUI instance in a web browser.
2.  Navigate to **Settings > Functions**.
3.  Click on **Add function**.
4.  **Copy the entire Python script** (the original, detailed one for Imagen 3/4) and paste it into the **Code** block.
5.  Scroll down to the **Valves** section. These are the settings for the function.
6.  Fill in the following required values:
    * **`PROJECT_ID`**: Your Google Cloud Project ID.
    * **`SERVICE_ACCOUNT_KEY_FILE_PATH`**: The path **inside the container**. This must be exactly `/app/credentials/service-account-key.json`.
7.  Adjust any other default values (like `MODEL`, `ASPECT_RATIO`, etc.) as you see fit.
8.  Click **Save**.

The function is now installed and ready for use!

---

## Usage Guide

To use the function, simply select **Vertex AI Imagen 3/4** from the model dropdown list in the chat interface.

* **Simple Generation:**
    > a photorealistic portrait of a steampunk librarian

* **Generate 2 images with a specific aspect ratio:**
    > a logo for a coffee shop called "The Daily Grind" --count: 2 --aspect-ratio: 16:9

* **Generate with a negative prompt (to exclude things):**
    > a beautiful winter landscape --negative-prompt: "no houses, no roads"

* **Upscale an existing image:**
    1.  Upload an image to the chat.
    2.  In the prompt box for that image, type: `--mode: upscale --upscale-factor: x4`

---

## Troubleshooting

* **Authentication Errors / "Service Account key file not found":**
    * Verify the `SERVICE_ACCOUNT_KEY_FILE_PATH` in the function's Valves is exactly `/app/credentials/service-account-key.json`.
    * Ensure your `docker-compose.yml` correctly mounts the `./credentials` directory to `/app/credentials`.
    * Confirm the `service-account-key.json` file is present in the `credentials` folder on your host machine.

* **`ImportError: No module named 'httpx'` (or similar):**
    * This means the dependencies were not installed. Check that the `OPENWEBUI_PYTHON_PACKAGES` environment variable is correctly spelled and formatted in your `docker-compose.yml`.
    * Restart your Docker stack with `docker-compose down && docker-compose up -d` to force the installation script to run again.

* **API Errors (e.g., 403 Forbidden):**
    * Ensure the **Vertex AI API** is enabled in your Google Cloud Project.
    * Confirm the Service Account has the **Vertex AI User** role.
