# Tracking Specification: 3-Step Appointment Booking Form

This document provides GTM developers and frontend engineers with the technical requirements to track OrthoNow's dynamic 3-step appointment booking funnel. 

---

## Why GTM Cannot Automatically Detect Funnel Steps

Most modern multi-step booking funnels do not trigger standard browser navigation (page loads) when transitioning between steps. Instead:
1. **Dynamic DOM Manipulation**: The form components are rendered client-side (e.g., using React, Vue, or Angular) within a single Page URL. Traditional pageview-based triggers will not fire.
2. **Form Submission Interception**: Modern validation scripts intercept browser form submissions using `event.preventDefault()` to validate inputs before making an AJAX/fetch call. This blocks GTM's native **Form Submission** trigger from firing.
3. **No Direct DOM Hooks**: Elements representing errors or success states might appear/disappear instantly, making DOM Element Visibility triggers unreliable and prone to race conditions.

To ensure data integrity, we rely on developer-implemented **`dataLayer.push()`** events.

---

## Frontend Implementation: dataLayer Specifications

Frontend developers must trigger the following dataLayer pushes at each stage of the booking funnel.

> [!WARNING]
> **PII (Personally Identifiable Information) Compliance**: Never pass raw customer names, phone numbers, or emails directly in GTM custom parameters to Google Analytics, as this violates Google's privacy policy. Instead, use boolean flags (`has_name_entered`, etc.) for engagement tracking. If Google Ads Enhanced Conversions are required, see the hashed format in Step 2.

### Step 1: Location + Specialty Selected
* **Description**: Triggered immediately after the user selects a clinic location and specialty, and clicks to proceed from Step 1.
* **Code Block**:
```javascript
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  event: 'appointment_booking_start',
  step_number: 1,
  step_name: 'location_specialty_selection',
  form_id: 'orthonow_booking_funnel',
  form_name: 'OrthoNow Booking Funnel',
  clinic_location: 'Downtown OrthoNow', // e.g., 'Downtown OrthoNow', 'Northside OrthoNow'
  specialty_selected: 'Sports Medicine'  // e.g., 'Sports Medicine', 'Spine Care', 'Physical Therapy'
});
```

### Step 2: Name + Phone + Preferred Date Entered
* **Description**: Triggered when the user enters their contact details and preferred date, and clicks to proceed from Step 2.
* **Code Block**:
```javascript
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  event: 'appointment_booking_step',
  step_number: 2,
  step_name: 'contact_date_entry',
  form_id: 'orthonow_booking_funnel',
  form_name: 'OrthoNow Booking Funnel',
  booking_session_id: 'sess_9876543210',  // Unique session identifier to tie steps together
  preferred_date: '2026-07-15',          // Format: YYYY-MM-DD
  has_name_entered: true,                // PII-safe verification flag
  has_phone_entered: true,               // PII-safe verification flag
  // Optional: For Google Ads Enhanced Conversions / GA4 User-Provided Data (SHA-256 hashed only)
  hashed_phone: 'f268297b4b1a457c669145fb4b94fcf232759e66c7f8976bcf6e4d5fb5880b91' 
});
```

### Step 3: Booking Confirmed (Final Conversion)
* **Description**: Triggered when the server returns a successful booking confirmation and the user sees the thank-you/success state.
* **Code Block**:
```javascript
window.dataLayer = window.dataLayer || [];
window.dataLayer.push({
  event: 'appointment_booking_complete',
  step_number: 3,
  step_name: 'booking_confirmed',
  form_id: 'orthonow_booking_funnel',
  form_name: 'OrthoNow Booking Funnel',
  booking_id: 'APT-2026-9871',
  clinic_location: 'Downtown OrthoNow',
  specialty_selected: 'Sports Medicine',
  appointment_date: '2026-07-15'
});
```

---

## GTM Trigger Configuration

In Google Tag Manager, configure three separate **Custom Event** triggers to capture these dataLayer pushes.

### 1. Step 1 Trigger Setup
* **Trigger Type**: Custom Event
* **Event Name**: `appointment_booking_start`
* **This Trigger Fires On**: All Custom Events

### 2. Step 2 Trigger Setup
* **Trigger Type**: Custom Event
* **Event Name**: `appointment_booking_step`
* **This Trigger Fires On**: All Custom Events

### 3. Step 3 Trigger Setup
* **Trigger Type**: Custom Event
* **Event Name**: `appointment_booking_complete`
* **This Trigger Fires On**: All Custom Events

---

## Building Funnel Exploration in GA4

Once the data is flowing into GA4, build the Funnel Exploration report using these steps:

1. **Navigate to Explore**: Open GA4 and click on the **Explore** tab in the left-hand navigation.
2. **Create New Exploration**: Choose **Funnel exploration** (or start with a Blank exploration and select the "Funnel exploration" technique).
3. **Configure Variables**:
   * Add the following events to your variables list: `appointment_booking_start`, `appointment_booking_step`, `appointment_booking_complete`.
4. **Define Steps**: Under **Tab Settings**, double-click on **Steps** (or click the edit pencil icon) to open the funnel editor:
   * **Step 1 (Start - Location & Specialty)**: Name it "Location/Specialty Selection" -> Select event `appointment_booking_start`.
   * **Step 2 (Contact & Date)**: Name it "Contact Info & Date" -> Select event `appointment_booking_step` -> Filter by parameter `step_number` equals `2`.
   * **Step 3 (Booking Complete)**: Name it "Booking Confirmed" -> Select event `appointment_booking_complete`.
5. **Apply Settings**: Click **Apply** in the top right.
6. **Interpret the Funnel**:
   * **Completion Rate**: View the percentage of users who make it from Step 1 to Step 3.
   * **Drop-off Analysis**: Inspect which step has the highest drop-off rate to optimize UX (e.g., if Step 2 has a 70% drop-off, the contact details form might be too long/intrusive).

