# OrthoNow GTM & Analytics Documentation

Welcome to the Google Tag Manager (GTM) and GA4 event schema documentation project for **OrthoNow Clinics**. This repository contains the analytics implementation schema, the frontend dataLayer specifications for the dynamic booking funnel, the campaign attribution tracking setups, and a demo CRO landing page.

## Repository Structure

* **[README.md](file:///Users/hemankthakur/Desktop/task/OrthoNow%20GTM%20Documentation/README.md)**: This overview file.
* **[gtm-event-schema.md](file:///Users/hemankthakur/Desktop/task/OrthoNow%20GTM%20Documentation/gtm-event-schema.md)**: The master GTM & GA4 Event Schema table detailing all core clinic website tracking events, parameters, GA4 reporting, audiences, and conversions.
* **[booking-funnel-json.md](file:///Users/hemankthakur/Desktop/task/OrthoNow%20GTM%20Documentation/booking-funnel-json.md)**: Technical developer-facing documentation for the 3-step dynamic appointment booking form, with sample production-quality `dataLayer.push` snippets and instructions on building GA4 Funnel Explorations.
* **[google-ads-conversion.md](file:///Users/hemankthakur/Desktop/task/OrthoNow%20GTM%20Documentation/google-ads-conversion.md)**: Marketing analytics strategy outlining which event to import as a primary Google Ads bidding signal and why other engagement events are treated as secondary.
* **[index.html](file:///Users/hemankthakur/Desktop/task/index.html)** (at the root of the workspace): A high-converting CRO landing page featuring a 2-field lead form, responsive structure, validation rules, and built-in `consultation_form_submitted` dataLayer pushes.

## Key Implementation Guidelines
1. **PII Warning**: Ensure no raw user-identifying info (like Name/Phone) is passed in standard GA4 parameters. Use flags or SHA-256 hashed fields as documented.
2. **GTM Setup**: Create custom triggers listening for events pushed to the `dataLayer` (such as `appointment_booking_complete` or `consultation_form_submitted`).
3. **GA4 Registering**: Custom parameters like `clinic_location` or `phone_length` must be manually registered in the Google Analytics 4 admin panel as custom dimensions or custom metrics before building custom reports.


ANSWER OF THE QUESTION ASKED IN TASK

1. How would you architect this integration end-to-end? What connects to what, in what order, and what technology sits at each step? Be specific - name the actual tools/methods (HubSpot Forms API vs. native embed vs. Zapier vs. Make vs. direct API call - pick one and justify it).

Answer:

I would use a custom backend with Node.js/Express, the HubSpot CRM API, the Karix WhatsApp Business API, and Google Tag Manager (GTM) integrated with Google Ads. I would avoid using Zapier or Make because they add unnecessary latency, have limited customization, and are less suitable for a production healthcare application.

The integration flow would be:

The user fills out the consultation form and clicks Submit.
The frontend sends the form data (Name, Phone, Clinic Preference) to a secure Node.js/Express backend via an HTTPS POST request.
The backend validates and sanitizes the data.
The backend searches HubSpot using the CRM Search API with the patient's phone number. Since HubSpot's default deduplication works on email, not phone number, this custom search prevents duplicate contacts.
If the phone number already exists, the contact is updated; otherwise, a new contact is created using the HubSpot CRM API.
The backend stores additional properties such as:
Source = "Google Ads - Consultation Landing Page"
Lead Status = "New Enquiry"
After HubSpot confirms success, the backend calls the Karix WhatsApp Business API to send a confirmation message to the patient.
At the same time, the frontend fires the consultation_form_submitted event through Google Tag Manager, which is imported into Google Ads as a conversion for campaign optimization.

This architecture provides better security, scalability, faster execution, and complete control over error handling compared to low-code automation tools.



2. What is the single biggest failure point in this setup, and how would you build a fallback for it?

Answer:

The biggest failure point is contact deduplication in HubSpot.

HubSpot automatically deduplicates contacts using email addresses, but this consultation form collects only the patient's phone number. Without custom logic, multiple submissions from the same patient could create duplicate contacts.

To solve this:

Before creating a contact, the backend searches HubSpot using the phone number through the CRM Search API.
If a matching phone number is found, the existing contact is updated.
If no record exists, a new contact is created.

As a fallback:

If the HubSpot API is temporarily unavailable, the lead is stored in a database or message queue.
The system retries the API request automatically using exponential backoff.
Every failed request is logged, and alerts are sent to the development team.
This ensures that no patient enquiry is lost even if HubSpot experiences temporary downtime.



3. The WhatsApp message must fire within 2 minutes. What could break this SLA and how would you monitor it?

Answer:

Several factors could prevent the WhatsApp confirmation from being sent within the required two-minute SLA:

HubSpot API delays or downtime.
Karix WhatsApp API failures or rate limits.
Network connectivity issues.
High server load or queue congestion.
Unexpected backend application errors.

To ensure the SLA is maintained:

The WhatsApp request is processed asynchronously immediately after the CRM operation succeeds.
Failed requests are automatically retried.
API response times, queue size, and delivery status are continuously monitored.
Application logs are stored in a centralized logging system.
Automated alerts are triggered if a message has not been delivered within 90 seconds, allowing engineers to investigate before the two-minute SLA is breached.

This monitoring strategy ensures reliable message delivery, quick failure detection, and minimal impact on the patient experience.