# Task 1 GTM Event Schema for OrthoNow

---

## Event Schema Table

| Event Name | Trigger Type | Key Parameters | GA4 Report / Audience |
|---|---|---|---|
| consultation_form_submitted | Custom Event | patient_name, phone_number, source_page | Conversions report, Google Ads import |
| booking_step_complete | Custom Event | step_number, step_name, clinic_location, specialty | Funnel Exploration |
| call_now_click | Click - All Elements | click_location, clinic_name, device_type | Engagement report |
| whatsapp_chat_click | Click - Just Links | click_location, wa_number, device_type | Engagement report |
| patient_guide_download | Custom Event | file_name, clinic_interest, device_type | Conversions report |
| clinic_page_view | Page View | clinic_name, city, page_path | Location reports |
| blog_scroll_depth | Scroll Depth | scroll_threshold (25/50/75/90), article_title, article_category | Engagement report |

---

## Booking Form Funnel Tracking (3-Step)

GTM cannot natively detect multi-step form progress. The front-end developer must fire `window.dataLayer.push()` at each step transition. GTM only listens via a Custom Event trigger watching for the event name `booking_step_complete`. The dev team needs to be briefed to fire the push at the moment each step completes — GTM does not handle this automatically.

### Step 1 - User selects clinic and specialty

```json
{
  "event": "booking_step_complete",
  "step_number": 1,
  "step_name": "location_specialty_selected",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Knee Pain"
}
```

### Step 2 - User enters name, phone, preferred date

```json
{
  "event": "booking_step_complete",
  "step_number": 2,
  "step_name": "patient_details_entered",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Knee Pain",
  "preferred_date": "2025-07-15"
}
```

### Step 3 - User confirms booking

```json
{
  "event": "booking_step_complete",
  "step_number": 3,
  "step_name": "booking_confirmed",
  "clinic_location": "Koramangala, Bengaluru",
  "specialty": "Knee Pain",
  "preferred_date": "2025-07-15",
  "booking_id": "{{booking_id}}"
}
```

---

## GA4 Funnel Exploration Setup

In GA4, go to Explore → Funnel Exploration. Create 3 steps, each using `booking_step_complete` as the event, filtered by `step_number` equals 1, 2, and 3 respectively. This surfaces the exact drop-off percentage between each step. If drop-off is highest between step 1 and step 2, the clinic/specialty selection is the friction point. If it is between step 2 and step 3, the confirmation screen has a UX problem.

---

## Google Ads Conversion Action

**Recommended conversion:** `booking_step_complete` where `step_number = 3` (booking confirmed).

This is the highest-intent signal available, the patient has completed all three steps and committed to an appointment. Importing `consultation_form_submitted` instead would include incomplete journeys that never became actual bookings, inflating conversion volume and giving the campaign's bidding algorithm a misleading optimisation signal. Step 3 maps directly to a patient acquired, which is the outcome the paid campaign exists to drive.