# Dary: Automated Property Outreach System

## Project Overview
Dary is an automation workflow designed to streamline communication between property platforms and potential clients. Built using **n8n**, the system monitors incoming lead data via Telegram, verifies records against a Google Sheets database, and executes automated messaging through the **Evolution API**.

![Dary Workflow](Dary%20Workflow%20Image.png)

## Technical Architecture
The updated workflow incorporates advanced logic to handle retries for failed messages and ensure database integrity:

1.  **Trigger**: Receives a phone number via a Telegram Bot command.
2.  **Validation (Duplicate Check)**: Queries Google Sheets to check if the phone number already exists with a "Done" status.
    *   **True (Already Exists)**: Notifies the admin that the number is a duplicate.
    *   **False (Proceed)**: Moves to the outreach phase.
3.  **Outreach Phase**:
    *   **Data Retrieval**: Pulls existing row data to check for previous "Fail" statuses.
    *   **Dynamic Messaging**: A JavaScript node generates a personalized Arabic outreach template.
    *   **WhatsApp Delivery**: Sends the message via Evolution API.
4.  **Conditional Data Persistence**:
    *   **Update**: If the lead was previously in the sheet but had a "Fail" status, the row is updated with the new result.
    *   **Append**: If the lead is entirely new, a new record is created.
5.  **Reporting**: A consolidated status (Done, Fail, or Duplicate) is sent back to the admin via Telegram.

## Core Components

| Node Name | Function | Service / Tool |
| :--- | :--- | :--- |
| **Telegram Trigger** | Entry point for receiving phone numbers. | Telegram API |
| **Check Presence** | Verification of existing "Done" records. | Google Sheets API |
| **Code in JavaScript** | Formats the professional Arabic outreach script. | n8n Code Node |
| **Evolution API** | Handles the delivery of WhatsApp messages. | Evolution API |
| **Update/Add Row** | Smart logic to either update failed attempts or add new leads. | Google Sheets API |
| **Merge & Notify** | Consolidates all logic paths for a final Telegram report. | n8n Merge Node |

## Setup Requirements

### API Credentials
*   **Telegram Bot Token**: Required for the Trigger and Notification nodes.
*   **Google OAuth2**: Required for read/write access to the tracking spreadsheet.
*   **Evolution API Instance**: Required for WhatsApp integration.

### Database Configuration
The Google Sheet must contain the following headers in the primary sheet (**gid=0**):
*   `Phone Number`
*   `Status of Messaging`
*   `Status of Acceptance`

## Workflow Logic Summary
*   **Intelligent Retries**: The system now distinguishes between a number that has already been successfully contacted ("Done") and one that previously failed, allowing for automated updates to existing rows rather than creating duplicates.
*   **Error Resilience**: The "Send text" node uses `continueErrorOutput`, ensuring that message delivery failures are captured, categorized as "Fail," and logged correctly for future attempts.
