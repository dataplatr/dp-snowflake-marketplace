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
