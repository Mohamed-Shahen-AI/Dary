# Dary: Automated Property Outreach System

## Project Overview
Dary is an automation workflow designed to streamline communication between property platforms and potential clients. Built using **n8n**, the system monitors incoming lead data via **Telegram**, verifies records against a **Google Sheets** database to prevent duplicate outreach, and executes automated messaging through the **Evolution API**.

---

## Technical Architecture
The workflow follows a linear logic path with integrated error handling and data persistence:

1.  **Trigger**: Receives a phone number via a Telegram Bot command.
2.  **Validation**: Queries a Google Sheet to check if the phone number has been contacted previously.
3.  **Conditional Logic**: 
    * **True (Duplicate)**: If the number exists, the workflow assigns a "Duplicate" status and notifies the administrator.
    * **False (New Lead)**: If the number is unique, the workflow proceeds to the messaging phase.
4.  **Communication**: Dispatches a predefined Arabic template via WhatsApp using the Evolution API.
5.  **Logging**: Appends the results (Success or Failure) and the phone number to the Google Sheet for tracking.
6.  **Reporting**: Sends a final status confirmation back to the Telegram user.

---

## Core Components

| Node Name | Function | Service / Tool |
| :--- | :--- | :--- |
| **Telegram Trigger** | Entry point for receiving phone numbers. | Telegram API |
| **Check Presence** | Verification of existing records to avoid spam. | Google Sheets API |
| **Dary Message** | Formats the outreach script in Arabic. | n8n Set Node |
| **Evolution API** | Handles the delivery of WhatsApp messages. | Evolution API |
| **Log Results** | Records transaction history and message status. | Google Sheets API |
| **Merge & Notify** | Consolidates status paths for a final report. | n8n Merge Node |

---

## Setup Requirements

### API Credentials
* **Telegram Bot Token**: Required for the Trigger and Notification nodes.
* **Google OAuth2**: Required for read/write access to the tracking spreadsheet.
* **Evolution API Instance**: Required for WhatsApp integration.

### Database Configuration
The Google Sheet must contain the following headers in the primary sheet (`gid=0`):
* `Phone Number`
* `Status of Messaging`
* `Status of Acceptance`

---

## Workflow Logic Summary
* **Error Handling**: The `Send text` node is configured with `continueErrorOutput`, ensuring that even if a WhatsApp message fails, the error is logged to the sheet and reported back to the administrator.
* **Data Integrity**: Uses a strict conditional check on `row_number` to ensure the "If" node correctly identifies unique vs. existing entries.
