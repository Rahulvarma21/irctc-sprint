# IRCTC Problem Discovery — Part A
 
## Summary
- Total problems documented: 6 (3 given + 3 self-discovered)
- Platform explored: irctc.co.in (live, as of May 2026)
- Devices used: Desktop Chrome, Mobile Safari
 
---
 
## Problem 1: Tatkal Booking Crashes at 10:00 AM [Given]
 
**What is broken:**
Server timeout and gateway errors occur exactly when Tatkal booking opens, preventing users from moving past the train list or payment gateway. There is no queue position or clear error message.
 
**Affected users:**
Users attempting last-minute emergency travel (Tatkal quota), estimated at hundreds of thousands daily.
 
**Frequency:**
Daily at exactly 10:00 AM (AC classes) and 11:00 AM (Non-AC classes).
 
**Current flow — step by step:**
1. User logs into IRCTC at 9:55 AM.
2. User enters source, destination, and travel date (next day).
3. User selects "Tatkal" from the quota dropdown.
4. User waits on the train list page and refreshes at exactly 10:00 AM.
5. The page freezes, showing a loading spinner with no progress indicator.
6. User clicks "Book Now" when the button finally appears.
7. System throws a generic "Service Unavailable" or "Session Timeout" error, and the user finds Tatkal quota is gone upon re-login.
 
**Where exactly it breaks:**
Step 5 and 7. The servers fail to handle the massive concurrent request spike. This results in dropped connections and session timeouts, leaving the user with no feedback.
 
---
 
## Problem 2: Search Filters Do Not Work Reliably [Given]
 
**What is broken:**
Applied filters (like specific class or departure time) reset unexpectedly when navigating back from a train detail view, forcing the user to re-filter the results.
 
**Affected users:**
Travel planners and users with specific preferences (e.g., overnight travel) trying to narrow down specific options from a large list of trains.
 
**Frequency:**
High frequency. It occurs consistently across standard booking sessions when modifying searches or using browser back navigation.
 
**Current flow — step by step:**
1. User searches for trains between New Delhi and Mumbai.
2. System displays a long list of available trains.
3. User applies "Sleeper (SL)" and "Morning Departure" filters on the left sidebar.
4. The list updates to show only the filtered results.
5. User clicks on a train to check route or fare details.
6. User clicks the "Back" button to return to the search results.
7. The search results reload without the previously applied filters, showing all trains again.
 
**Where exactly it breaks:**
Step 7. State management for applied filters is not persisted in the session or URL parameters, forcing users to reapply filters manually.
 
---

## Problem 3: Seat Selection Resets [Given]
 
**What is broken:**
A user's berth preference (e.g., Lower Berth) selected during the passenger details step is not retained and resets or ignores the preference without warning on the review page.
 
**Affected users:**
Elderly passengers, pregnant women, or users with mobility issues who specifically require a lower berth.
 
**Frequency:**
Moderate to high, especially prevalent on the mobile web experience.
 
**Current flow — step by step:**
1. User selects a train and class, then clicks "Book Now".
2. User lands on the Passenger Details page.
3. User enters passenger name, age, and gender.
4. User selects "Lower" from the Berth Preference dropdown.
5. User scrolls down and clicks "Proceed to Payment" (or Review Journey).
6. User lands on the Journey Review page.
7. The berth preference shows as "No Preference" or is completely blank in the summary details.
 
**Where exactly it breaks:**
Step 6 and 7. The form submission fails to correctly bind or pass the berth preference parameter to the confirmation state, leading to a silent failure.
