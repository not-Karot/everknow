# 🏗️ Using Google Artifact Registry for Python Packages

This guide provides a comprehensive walkthrough on how to publish and install Python packages using Google Artifact Registry (GAR).

## 🌟 Introduction

Google Artifact Registry is a fully managed package management solution that helps you store, manage, and secure build artifacts, container images, and language packages. This guide focuses on its use for Python packages.

## 📋 Prerequisites

- Google Cloud Platform (GCP) account
- `gcloud` CLI installed and configured
- Python environment (preferably a virtual environment)
- Poetry (for building the package)
- Twine (for uploading the package)

## 🚀 Publishing a Package

### 1. Create a Repository in GAR

If you haven't already, create a new repository in Google Artifact Registry:

```bash
gcloud artifacts repositories create REPOSITORY_NAME \
    --repository-format=python \
    --location=REGION \
    --description="Description of the Python repository"
```

Replace `REPOSITORY_NAME`, `REGION`, and the description with your specific details.

### 2. Build Your Package

Navigate to your package directory and build it:

```bash
cd your_package_directory
poetry build
```

### 3. Configure Authentication

Ensure you're authenticated with Google Cloud:

```bash
gcloud auth application-default login
```

Install necessary Python packages:

```bash
pip install keyring keyrings.google-artifactregistry-auth
```

### 4. Configure Repository Settings

Retrieve your repository settings:

```bash
gcloud artifacts print-settings python \
    --project=PROJECT \
    --repository=REPOSITORY \
    --location=LOCATION
```

Follow the printed instructions to update your `pip.conf` and `.pypirc` files.

### 5. Upload the Package

Use Twine to upload your built package:

```bash
twine upload --repository REPOSITORY_NAME --verbose dist/*
```

## 📥 Installing a Package from GAR

1. **Authenticate** with Google Cloud:

   ```bash
   gcloud auth application-default login
   ```

2. **Install required packages**:

   ```bash
   pip install keyring keyrings.google-artifactregistry-auth
   ```

3. **Configure pip**: Ensure your `pip.conf` is correctly set up as per the settings retrieved earlier.

4. **Install the package**:

   ```bash
   pip install your_package_name
   ```

## 🔍 Troubleshooting

- If you encounter authentication issues, double-check your Google Cloud credentials and ensure you have the necessary permissions.
- For installation problems, verify that your `pip.conf` is correctly configured and that you're using the correct package name.

## 🌐 Additional Resources

- [Google Artifact Registry documentation for Python](https://cloud.google.com/artifact-registry/docs/python)
- [Authentication for Artifact Registry](https://cloud.google.com/artifact-registry/docs/python/authentication)

Remember to replace placeholders like `REPOSITORY_NAME`, `REGION`, `PROJECT`, `LOCATION`, and `your_package_name` with your specific details when using this guide.

Happy packaging! 📦🚀
