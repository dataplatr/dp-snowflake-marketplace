# AI-ELT Native App — Consumer Installation & Quick Start Guide

## Overview

The **AI-ELT Native App** is a Snowflake Native Application running on **Snowflake Container Services (SPCS)** with a Streamlit UI.
It helps consumers generate and manage AI-driven ELT workflows and integrates securely with external services such as:

* **GitHub** – for repository access and file pushes
* **Google Cloud Vertex AI** – for LLM-powered generation

All credentials are managed using **Snowflake Secrets** and injected into the container as **environment variables**.
No secrets are stored in code, Streamlit configuration files, or SQL queries.

---

## Architecture Summary

The application is built using **Streamlit** and runs securely inside **Snowpark Container Services (SPCS)**.
It relies on Snowflake-managed authentication, secrets, and external access controls to enable AI-driven ELT workflows while maintaining strict security boundaries.

![SNF AI-ELT Platform Architecture](https://github.com/dataplatr/dp-snowflake-marketplace/blob/main/Architecture.jpg)


## Core Application Modules

The application is organized into five primary functional areas accessible via the sidebar navigation menu:

* **Home**
  The landing page provides an overview and acts as the entry point for all workflows.

* **Metadata Enricher**
  A specialized module for managing and enhancing data metadata to ensure high-quality downstream processing.

* **L1 Generator (Raw / Landing)**
  Focused on the first layer of data processing, transforming landing data into structured raw tables.

* **L2 Generator (Silver / Cleansed)**
  Handles cleansing and standardization of data from the L1 layer.

* **L3 Gold Layer**
  Manages the final processing stage to produce high-quality, business-ready Gold data using an interactive and conversational AI-driven workflow.

---

## Session & State Management

Upon startup, the application attempts to establish a Snowflake session using `get_session()`.
If a session cannot be established, the application displays an error and stops to prevent unauthorized or broken workflows.

The app runs inside Snowpark Container Services, where Snowflake automatically injects authentication credentials, including:

* OAuth token provided at `/snowflake/session/token`
* Environment variables such as `SNOWFLAKE_HOST` and `SNOWFLAKE_ACCOUNT`

Navigation across modules is managed using a sidebar radio button interface, with callbacks ensuring session state and navigation remain synchronized throughout the application lifecycle.

---

## GitHub & AI Integrations

### GitHub Integration

GitHub serves as the primary landing zone for AI-generated outputs.
Once an AI model is generated, the platform automatically writes the code and logic to a designated GitHub repository, allowing the models to be seamlessly integrated into dbt projects for final transformation and deployment.

To enable secure programmatic writes, the application requires a GitHub Personal Access Token, which is resolved securely through Snowflake Secrets or environment variables.

### Vertex AI Integration

The application integrates with **Google Cloud Vertex AI** to perform AI-powered SQL generation and metadata enrichment.
Authentication is handled using a Snowflake-stored service account secret, ensuring credentials are never exposed in the UI or application code.

---

## Prerequisites

Before installing the application, ensure you have:

### Snowflake

* A Snowflake account with:

  * Native App installation privileges
  * Permission to create and grant `SECRET` objects
  * Permission to create `NETWORK RULE` and `EXTERNAL ACCESS INTEGRATION`
  * Permission to create a compute pool

### Compute Resources

* Compute Pool: `CPU_X64_XS`
* Minimum Memory: `2 GiB`

### GitHub

* A GitHub Personal Access Token (PAT) with `repo` scope

### Google Cloud

* A Google Cloud service account with Vertex AI permissions
* A downloadable JSON key file

---

## Step-by-Step Installation

After installing the application from the Snowflake Marketplace, the Account Administrator must perform the following steps.

---

### Step 1: Create Secure Secrets

Sensitive credentials must be stored in Snowflake’s encrypted Secret objects.

```sql
USE ROLE ACCOUNTADMIN;

CREATE OR REPLACE SECRET snf_git_secret
  TYPE = PASSWORD
  USERNAME = '<your_github_username>'
  PASSWORD = '<your_github_personal_access_token>';

CREATE OR REPLACE SECRET vertex_sa
  TYPE = GENERIC_STRING
  SECRET_STRING = $$
  {
    "type": "service_account",
    "project_id": "your-project-id",
    "private_key_id": "...",
    "private_key": "...",
    "client_email": "...",
    "client_id": "...",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_x509_cert_url": "..."
  }
  $$;
```

---

### Step 2: Define the Network Rule

Specify the egress endpoints allowed for the application.

```sql
CREATE OR REPLACE NETWORK RULE github_pypi_gcp_network_rule
  TYPE = HOST_PORT
  MODE = EGRESS
  VALUE_LIST = (
    'github.com:443',
    'api.github.com:443',
    'raw.githubusercontent.com:443',
    'pypi.org:443',
    'files.pythonhosted.org:443',
    'aiplatform.googleapis.com:443',
    'us-central1-aiplatform.googleapis.com:443',
    'generativelanguage.googleapis.com:443',
    'oauth2.googleapis.com:443',
    'sts.googleapis.com:443',
    'iamcredentials.googleapis.com:443'
  );
```

---

### Step 3: Create the External Access Integration (EAI)

This bridge connects the Network Rule and the Secrets.

```sql
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION snf_native_eai
  ALLOWED_NETWORK_RULES = (github_pypi_gcp_network_rule)
  ALLOWED_AUTHENTICATION_SECRETS = (snf_git_secret, vertex_sa)
  ENABLED = TRUE;

GRANT USAGE ON SECRET snf_git_secret TO APPLICATION <APP_NAME>;
GRANT USAGE ON SECRET vertex_sa TO APPLICATION <APP_NAME>;
```

---

### Step 4: Bind the EAI to the Application

```sql
CALL <APP_NAME>.CONFIG.REGISTER_EAI_CALLBACK(
  'external_access_reference',
  'ADD',
  SYSTEM$REFERENCE(
    'EXTERNAL ACCESS INTEGRATION',
    'SNF_NATIVE_EAI',
    'PERSISTENT',
    'USAGE'
  )
);
```

---

### Step 5: Verify the Configuration

```sql
SHOW REFERENCES IN APPLICATION <APP_NAME>;
```

---

## Post-Installation: Quick Start Guide

* Launch the application from Snowflake.
* The Streamlit application opens on the Home page.
* Use the sidebar to navigate between modules.
* Start by enriching metadata using the **Metadata Enricher**.
* Register source datasets and generate **L1 (Raw)** views.
* Create **L2 (Silver)** incremental tables.
* Generate **Gold** tables interactively using the AI-powered conversational interface.

  * SQL can be edited directly or modified using plain English prompts.
  * SQL is validated and only pushed to GitHub after review and approval.


---

## Installation Summary Checklist

* Snowflake compute pool created
* Network Rule and External Access Integration configured
* Secrets created and granted to the application
* GitHub and Vertex AI integrations verified

---
