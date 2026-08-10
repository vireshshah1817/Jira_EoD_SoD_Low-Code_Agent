---
title: Multi-Agent Automated EoD Report Generator
---

# Multi-Agent Automated EoD Report Generator

A low-code, multi-agent AI system designed to completely automate End of Day (EoD) reporting. Built for Gemini Enterprise Agent Designer (or compatible low-code AI platforms), this system fetches daily Jira activity, checks Google Calendar for Out-of-Office (OOO) status, validates Jira ticket hygiene, and posts a beautifully formatted summary directly to a specific Google Chat space thread.

## 🌟 Architecture Overview

This solution utilizes a multi-agent orchestration pattern consisting of three distinct agents:

*   **EoD Workflow Orchestrator (Main Agent):** Acts as the central router. Takes the user request, coordinates the sub-agents, and passes warnings forward.
*   **Pre-requisites & Guideline Validator (Sub-agent):** Fetches active Jira tickets and cross-references them against organizational guidelines (checking for Acceptance Criteria, Start Dates, 30-day limits, and max In-Progress limits). Generates non-blocking warnings for non-compliance.
*   **EoD Report Generator (Sub-agent):** Checks user availability via Calendar, extracts today's worklogs and formatted comments from Jira, synthesizes a contextual summary, formats it strictly against a Knowledge Base template, and dynamically threads it in Google Chat.

## 📋 Prerequisites

Before setting up the agents, ensure you have the following:

*   Access to a low-code AI Agent Builder (e.g., Gemini Enterprise Agent Designer).
*   Admin or integration access to your Jira Workspace.
*   A Google Workspace account (for Calendar and Chat integrations).
*   The specific Space ID of your target Google Chat Space where reports will be sent.

## 🏗️ Step 1: Creating the Gemini Enterprise (GE) App

Before configuring the individual agents, you need to create the container app that will house them.

1.  Navigate to your Gemini Enterprise Agent Designer dashboard.
2.  Click on `Create New App` (or "New Agent Project").
3.  Name your app something recognizable (e.g., `EoD Automation App`) and provide a brief description.
4.  Save the app. You will use this newly created app workspace to configure the Data Sources, Knowledge Base, and Agents in the following steps.

## ⚙️ Step 2: Connector Configuration

Our application already includes pre-configured data sources for the necessary integrations. You do not need to set up custom schemas or authentication manually; simply ensure these existing data sources are linked to your GE App workspace:

1.  **Jira Data Source**
    *   `Jira_evonence`: Use this data source to query issues assigned to a user and to fetch daily comments/worklogs.

2.  **Google Workspace Data Sources**
    *   `Google Calendar`: Use the `common_calendar` data source to check the user's schedule for OOO or Out of Office events for the current date.
    *   `Google Chat`: Use the `common_gchat` data source to fetch the daily EoD thread and post the final formatted report.

## 📚 Step 3: Knowledge Base Preparation

Create two text/markdown files and upload them to the Knowledge Base of your GE App environment. The agents will use these for grounding.

1.  **Jira Ticket Guidelines Document:**
    *   Define your team's rules (e.g., tickets must have AC, valid Start/Due dates, daily comments, max 5 In-Progress tickets unless in the 'LA' space, max 30-day lifespan).
    *   *(Refer to `https://docs.google.com/document/d/1d417HBcVFM2F99mBpEPptXTL7UZOxDHogzb3IRRom3g/edit?usp=sharing`)*

2.  **EoD Format:**
    *   Define the exact literal string template the Generator must use. **CRITICAL:** Use exact markdown spacing and asterisks as expected by Google Chat.
    *   *(Refer to `https://docs.google.com/document/d/1uuyBvs0HZpWnLKOcVZrWAOBy2g-NmAezcRIXSqMSWzw/edit?usp=sharing` for the expected format structure)*

## 🤖 Step 4: Creating the Agents

You will find the specific system instructions for each agent in the `/instructions` folder of this repository (or as provided in the context). Follow these steps to build them inside your GE App:

1.  **Create the Main Orchestrator Agent:**
    *   Name it: `EoD Workflow Orchestrator`.
    *   Paste the Orchestrator system instructions.
    *   Link the two sub-agents (`Pre-requisites and Guideline Validator` and `EoD Report Generator`) to this Main Agent so it can route tasks to them.
    *   Attach the Calendar, Chat and Jira data sources.
    *   Attach the `Jira Ticket Guidelines Document` and `EoD Format` documents to its knowledge base.


2.  **Create the Validator Sub-agent:**
    *   Name it: `Pre-requisites and Guideline Validator`.
    *   Paste the Validator system instructions.
    *   Attach the `Jira_evonence` and `common_chat` data source.

3.  **Create the Generator Sub-agent:**
    *   Name it: `EoD Report Generator`.
    *   Paste the Generator system instructions (from `/run/media/viresh-shah/New Volume/JIRA_EoD_SoD_Low-Code_Agent/EoD Report Generation Logic.md`).
    *   Attach the `Jira_evonence`, `common_calendar`, and `common_gchat` data sources.
    *   Attach the `EoD Format` document to its knowledge base.
    *   **Important Configuration:** Edit the system prompt to replace the `space_id` placeholder with the specific Google Chat Space ID where you want the EoD reports sent. (e.g., `space_id`: "AAQAHPmO4jw" as per instructions).


## 🚀 Usage

Once deployed, users simply interact with the Main Orchestrator Agent via the chat interface:

**User:** "Generate my EoD report for today."

The Orchestrator will automatically:

1.  Call the Validator to check Jira hygiene and capture warnings.
2.  Pass those warnings to the Generator.
3.  The Generator will check the Calendar, read today's formatted Jira comments, synthesize the data using the strict Knowledge Base template, locate the correct thread for today, and post it perfectly in Google Chat.

## 🛠️ Troubleshooting

*   **Formatting Issues in Chat:** If Google Chat formatting breaks or asterisks appear as plain text, ensure the Generator agent's instructions strictly enforce "literal string replacement" and explicitly forbid the LLM from altering the markdown syntax from the EoD Format document.
*   **Chat Tool Failing/Not Threading:** Ensure the `space_id` is correct in the instructions. The agent must successfully execute `fetch_chat_messages` to grab the `thread_id`. If the tool fails entirely, ensure the app's service account has permissions to read/write in that specific Google Chat Space.
