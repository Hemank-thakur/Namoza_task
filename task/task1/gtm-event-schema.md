# OrthoNow GA4 & Google Tag Manager Event Measurement Schema

This document defines the Google Analytics 4 (GA4) and Google Tag Manager (GTM) event schema for **OrthoNow**. This schema captures key user interactions—including the multi-step appointment booking funnel, micro-conversions (calls/WhatsApp), lead generation, location-based page views, and blog engagement scroll depth.

## GTM & GA4 Event Schema Table

| Event Name | Trigger Type | Trigger Condition | Minimum 3 Parameters | GA4 Report | Audience | Conversion (Yes/No) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`appointment_booking_start`** | Custom Event | `event` equals `booking_start` | `form_id` (string)<br>`form_name` (string)<br>`page_location` (string) | Engagement > Events | Appointment Initiators | No |
| **`appointment_booking_step`** | Custom Event | `event` equals `booking_step` | `step_number` (int)<br>`step_name` (string)<br>`booking_session_id` (string) | Engagement > Events | Booking Progressors | No |
| **`appointment_booking_complete`** | Custom Event | `event` equals `booking_complete` | `booking_id` (string)<br>`clinic_location` (string)<br>`appointment_specialty` (string) | Engagement > Conversions | Booked Patients | **Yes** |
| **`click_to_call`** | Just Clicks | Click URL starts with `tel:` | `click_url` (string)<br>`link_text` (string)<br>`page_location` (string) | Engagement > Events | Phone Call Inquirers | **Yes** (Micro-conversion) |
| **`whatsapp_chat_start`** | Just Clicks | Click URL contains `wa.me` or `api.whatsapp.com` | `click_url` (string)<br>`link_text` (string)<br>`page_location` (string) | Engagement > Events | WhatsApp Chatters | **Yes** (Micro-conversion) |
| **`lead_form_submit`** | Form Submission | Form ID equals `patient_guide_form` | `form_id` (string)<br>`form_name` (string)<br>`page_location` (string) | Engagement > Events | Patient Guide Leads | **Yes** |
| **`file_download`** | Just Clicks | Click URL path ends with `.pdf` | `file_name` (string)<br>`file_extension` (string)<br>`link_url` (string) | Engagement > Events | Guide Downloaders | No |
| **`location_page_view`** | Page View | Page Path matches regex `^/locations/.+` | `clinic_name` (string)<br>`clinic_city` (string)<br>`clinic_zip` (string) | Engagement > Pages and screens | Local Clinic Prospects | No |
| **`blog_engagement_scroll`** | Scroll Depth | Page Path starts with `/blog/` AND Scroll Depth equals `25, 50, 75, 90` | `percent_scrolled` (int)<br>`blog_post_title` (string)<br>`blog_category` (string) | Engagement > Events | Highly Engaged Blog Readers (if >= 75%) | No |

---

## Technical Recommendations for GTM Implementation

### 1. Appointment Booking Funnel (`appointment_booking_*`)
Since the booking form is a 3-step dynamic workflow, tracking should rely on `dataLayer.push()` calls triggered by the front-end application logic rather than DOM scrape.
* **Step 1 (Start)**: Triggered when the user interacts with the first field of the form.
* **Step 2 (Step Transition)**: Triggered on successful transition from Step 1 to Step 2, and Step 2 to Step 3.
* **Step 3 (Success)**: Triggered on final submission success callback.

### 2. Click-to-Call and WhatsApp Links
Use GTM's built-in **Click - Just Links** trigger. Ensure **Link Click** variables are enabled in GTM:
* Set `Click URL` starts with `tel:` for Phone calls.
* Set `Click URL` matches regex `(wa\.me\|whatsapp\.com)` for WhatsApp.

### 3. Patient Guide Download
To prevent duplicate conversion counts, the `lead_form_submit` event should be the primary lead conversion event. The subsequent `file_download` event tracks the utility of downloading the guide after submission.
