# AI-ELT Native App — Consumer Installation & Quick Start Guide

## Overview

The **AI-ELT Native App** is a Snowflake Native Application running on **Snowflake Container Services (SPCS)** with a Streamlit UI.  
It helps consumers generate and manage AI-driven ELT workflows and integrates securely with external services such as:

- **GitHub** – for repository access and file pushes
- **Google Cloud Vertex AI** – for LLM-powered generation

All credentials are managed using **Snowflake Secrets** and injected into the container as **environment variables**.  
No secrets are stored in code, Streamlit configuration files, or SQL queries.

---

## Architecture Summary 
<img width="892" height="487" alt="image" src="https://github.com/user-attachments/assets/e9a98def-41f9-4f8e-b779-f25c55412608" />



## Prerequisites

Before installing the application, ensure you have:

- A Snowflake account with:
  - Native App installation privileges
  - Permission to create and grant `SECRET` objects
- A **GitHub Personal Access Token (PAT)** with:
  - `repo` scope (required for pushes)
- A **Google Cloud service account** with:
  - Vertex AI permissions
  - A downloadable JSON key file

**Step-by-Step Installation**

After installing the application from the Snowflake Marketplace, the Account Administrator must perform the following steps to provision the infrastructure and security bridges required for the AI engine to function.

**Step 1**: Create Secure Secrets
Sensitive credentials must be stored in Snowflake’s encrypted Secret objects. This ensures the application can authenticate with GitHub and Google Vertex AI without exposing raw tokens in the UI.

```sql
USE ROLE ACCOUNTADMIN;

-- Create a secret for the GitHub Personal Access Token (PAT)
CREATE OR REPLACE SECRET snf_git_secret
  TYPE = PASSWORD
  USERNAME = '<your_github_username>'
  PASSWORD = '<your_github_personal_access_token>';

-- Create a secret for the GCP Service Account JSON Key
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

**Step 2**: Define the Network Rule
Specify the egress endpoints allowed for the application. This whitelist includes all necessary nodes for GitHub, PyPI, and Google Cloud’s AI and authentication services.

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

**Step 3**: Create the External Access Integration (EAI)
This bridge connects the Network Rule and the Secrets, allowing the containerized service to communicate securely with the outside world.

```sql
CREATE OR REPLACE EXTERNAL ACCESS INTEGRATION snf_native_eai
  ALLOWED_NETWORK_RULES = (github_pypi_gcp_network_rule)
  ALLOWED_AUTHENTICATION_SECRETS = (snf_git_secret, vertex_sa)
  ENABLED = TRUE;

-- Grant usage on secrets and integration to the application
GRANT USAGE ON SECRET snf_git_secret TO APPLICATION <APP_NAME>;
GRANT USAGE ON SECRET vertex_sa TO APPLICATION <APP_NAME>;
```

---
## Post Installation - Quick Start Guide:

- Configure the application by binding the External Access Integrations and netwrok rules.
- Launch the Application from the next window.
- The streamlit application opens up on the home page.
- On the left side bar, you can navigate to various pages.
- The first step in the AELT process would be enriching the metadata, so choose the source dataset to be enriched.
- The second step is registerning the bronze tables and creating the Silver L1 views, from the sidebar choose the source and target datasets.
- The third step is creating the silver L2 incremental tables for which choose your source L1 views and target L2 schema.
- The fourth step, create gold tables in an interactive and conversational way with the google-adk based agent that understands your data well and generates the SQL logic for the gold object.
  - You can edit the SQL in the editor or ask in plain english to get it modified.
  - Thw SQL is also validated and only after the review and approval phase, it pushes the final gold model to the dbt project from where it gets executed.

**Step-by-Step Configuration**

**Step 4**: Bind the EAI to the Application
Because the app uses a Native App Reference model, you must use the internal callback procedure to link the integration to the app’s external_access_reference.

```sql
CALL <APP_NAME>.CONFIG.REGISTER_EAI_CALLBACK(
  'external_access_reference', 
  'ADD',
  SYSTEM$REFERENCE('EXTERNAL ACCESS INTEGRATION', 'SNF_NATIVE_EAI', 'PERSISTENT', 'USAGE')
);
```

**Step 5**: Verify the Configuration
Confirm that the security references are correctly attached and active

```sql
SHOW REFERENCES IN APPLICATION <APP_NAME>;
```
