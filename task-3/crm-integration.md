# Task 3  Integration Architecture

---

## End-to-End Architecture

When a patient submits the consultation form, three things happen in this order:

**1. Google Ads conversion fires (client-side, immediate)**
The GTM dataLayer push on form submit triggers the Google Ads conversion tag directly in the browser. This happens instantly and does not depend on any backend call.

**2. Form data is sent to a lightweight backend endpoint**
A JavaScript fetch call sends the patient's name and phone to a backend endpoint (a Cloudflare Worker works well here - no server to maintain, deploys in seconds, runs at the edge for low latency). This is where the HubSpot and WhatsApp calls are made server-side, keeping API keys out of the browser.

**3. HubSpot contact is created or updated**
The backend calls the HubSpot Contacts Search API first, searching by phone number. If a contact with that phone already exists, it updates the record. If not, it creates a new one. Fields set: Name, Phone, Source = "Google Ads - Consultation Landing Page", Lead Status = "New Enquiry".

I chose direct HubSpot API over Zapier or Make because it gives full control over error handling, avoids the latency of a third-party automation layer, and adds no additional paid dependency to the stack.

**4. WhatsApp confirmation is sent via Karix**
Immediately after the HubSpot call succeeds, the backend calls the Karix WhatsApp Business API to send the patient a confirmation message. Firing this after HubSpot (not in parallel) ensures we only message patients whose records are confirmed in the CRM.

---
 
## Biggest Failure Point

HubSpot deduplicates contacts on email by default - not phone number. Since this form collects no email address, submitting the same phone number twice would create two separate contacts in HubSpot unless explicitly handled. To fix this: always call the Contacts Search API before creating anything. Search by phone, and if a record exists, update it. If not, create new. Never POST a new contact without checking first.

---

## WhatsApp 2-Minute SLA

What could break it: Karix API downtime, rate limiting on the WhatsApp Business account, or the backend endpoint timing out before reaching the Karix call.

How to protect the SLA: log a timestamp and response code for every Karix call. If no 200 response within 30 seconds, retry once automatically. If the retry also fails, send a Slack alert immediately so the team can follow up manually. If Karix failures exceed 5% in any 10-minute window, that triggers a higher-priority alert. For a harder fallback, an SMS via a secondary provider can be queued if the WhatsApp call fails entirely.