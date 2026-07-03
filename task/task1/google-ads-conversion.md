# Google Ads Conversion Strategy & Recommendations

This document outlines the conversion tracking strategy for OrthoNow's Google Ads campaigns. It provides a formal recommendation on which single event to import as a primary conversion action to optimize campaign bidding, along with the technical and strategic rationale for excluding other tracking events from primary bidding.

---

## Primary Recommendation

### Recommended Conversion: `appointment_booking_complete`
We recommend importing **`appointment_booking_complete`** (GA4 Event) as the **sole primary conversion action** in Google Ads.

### Strategic Rationale
Google Ads Smart Bidding (such as Target CPA or Maximize Conversions) relies heavily on machine learning to optimize bids toward users most likely to take high-value actions. 
* **High-Intent Signal**: A completed appointment booking represents a user who has committed to a specific time, clinic location, and medical specialty. This is the ultimate bottom-funnel action on the site.
* **Low Noise**: This event is only fired via a successful API callback on step 3 of the booking form, meaning it represents a validated lead with zero false positives.
* **Smart Bidding Efficiency**: Optimizing campaigns for this event ensures Google Ads targets users ready to convert into patients, maximizing the return on ad spend (ROAS) and lowering cost per acquisition (CPA) for actual clinic visits.

---

## Evaluation of Alternative Events (Why They Should Be Secondary or Excluded)

### 1. Why Not Call Button Click (`click_to_call`)
* **The Issue (Click vs. Connection)**: Click-to-call events track the action of clicking a phone link, not the phone call itself. A user might click the button but cancel before the call dials, dial and hang up immediately, or reach a busy signal.
* **Bidding Risk**: If used as a primary conversion, Google Ads will optimize for users who are prone to clicking phone buttons, which often leads to spam, accidental clicks, or non-viable inquiries.
* **Best Practice**: Treat `click_to_call` as a **Secondary Conversion** for observation only. For actual phone call tracking, use Google Ads Call Forwarding numbers (which track call duration, e.g., only counting calls lasting >60 seconds) rather than GTM click tracking.

### 2. Why Not WhatsApp Click (`whatsapp_chat_start`)
* **The Issue (Drop-off after redirection)**: A WhatsApp click tracks a redirect to the WhatsApp application. Once the user leaves the website, the site loses visibility. We cannot programmatically verify if the user sent a message, if they were answered, or if they eventually booked.
* **Bidding Risk**: Redirection clicks are highly susceptible to bot traffic and low-intent clickers who leave immediately upon opening the app. Optimizing bidding algorithms for this action will exhaust budget on unqualified mobile traffic.
* **Best Practice**: Set as a **Secondary Conversion**. Observe chat volume changes, but do not allow Google Ads Smart Bidding to optimize toward it.

### 3. Why Not Patient Guide Download (`file_download`)
* **The Issue (Top/Mid-Funnel Intent)**: Downloading a Patient Guide is a classic top-of-funnel informational touchpoint. The user is researching symptoms or options, not booking an immediate consultation.
* **Bidding Risk**: If Google Ads optimizes for PDF downloads, the algorithm will find users who like downloading guides, who are generally in the early research phase. This will dilute campaign traffic with low-intent prospects who have no immediate need for clinical appointments, raising the CPA of actual bookings.
* **Best Practice**: Treat guide downloads as a **Secondary Conversion** (or set up as a micro-conversion/soft lead in GA4). It is excellent for tracking content marketing engagement, but detrimental as a primary bidding signal.

---

## Conversion Setup Matrix

| GTM / GA4 Event | Google Ads Conversion Action Category | Action Optimization | Attribution Model |
| :--- | :--- | :--- | :--- |
| `appointment_booking_complete` | **Submit lead form** | **Primary** (Optimizes bidding) | Data-driven |
| `click_to_call` | **Phone call lead** | **Secondary** (Observation only) | Data-driven |
| `whatsapp_chat_start` | **Outgoing click** | **Secondary** (Observation only) | Data-driven |
| `lead_form_submit` (Guide Form) | **Submit lead form** | **Secondary** (Observation only) | Data-driven |
| `file_download` (Guide PDF) | **Other** | **Secondary** (Observation only) | Data-driven |
