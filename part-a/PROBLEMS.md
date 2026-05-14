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
 
---
 
## Problem 4: CAPTCHA Loop on Login [Self-Discovered]
 
**What is broken:**
The login CAPTCHA frequently rejects correct inputs or fails to load the image properly, trapping users in an infinite loop of refreshing and re-entering CAPTCHAs.
 
**Affected users:**
All users attempting to log in, particularly those with slower internet connections or visual impairments struggling with distorted text.
 
**Frequency:**
Very high, occurring in almost every session, especially during peak booking hours.
 
**Current flow — step by step:**
1. User navigates to irctc.co.in and clicks "Login".
2. User enters a valid username and password.
3. User views the CAPTCHA image provided.
4. User types the exact alphanumeric string shown in the image.
5. User clicks "Sign In".
6. The page reloads, clearing the password field and displaying "Invalid CAPTCHA".
7. User has to re-enter the password and attempt a new CAPTCHA, often failing multiple times.
 
**Where exactly it breaks:**
Step 6. The CAPTCHA validation server is either out of sync with the rendered image or times out, treating a valid input as incorrect.
 
**How I found it:**
I was on the homepage trying to log into my account to begin a booking flow.
 
**Screenshot or description:**
The login modal overlay on the homepage. The password field is blanked out, a new CAPTCHA image is loaded, and a red text error "Invalid Captcha. Please enter the correct captcha." is displayed above the username field.
 
---
 
## Problem 5: Vikalp Scheme UI is Confusing [Self-Discovered]
 
**What is broken:**
The Vikalp (Alternate Train Accommodation) option is presented as a simple checkbox with no contextual explanation, leading users to accidentally opt-in without understanding the implications.
 
**Affected users:**
Waitlisted passengers who might be unexpectedly shifted to a different train, significantly altering their travel schedule.
 
**Frequency:**
Constant presence on the passenger details page for all waitlisted tickets.
 
**Current flow — step by step:**
1. User selects a train that currently has a Waitlisted (WL) status.
2. User clicks "Book Now" and proceeds to Passenger Details.
3. User fills in basic passenger information.
4. User scrolls down to the "Other Preferences" section.
5. User sees a checkbox labeled "Opt for Vikalp" with no tooltip or clear explanation.
6. User checks the box, assuming it generally increases confirmation chances.
7. User completes booking, later realizing their ticket might be confirmed on a different train departing 12 hours later.
 
**Where exactly it breaks:**
Step 5. There is a complete lack of inline documentation, tooltips, or progressive disclosure explaining what Vikalp entails before the user commits to it.
 
**How I found it:**
Exploring the passenger details form (Step 2 of the booking flow) specifically looking for confusing labels or missing information.
 
**Screenshot or description:**
The "Other Preferences" section of the passenger details page. A lone checkbox says "Opt for Vikalp" next to "Consider for Auto Upgradation". There is no info icon, hyperlink, or helper text nearby explaining the scheme.
 
---
 
## Problem 6: Session Expiry Erases All Entered Data [Self-Discovered]
 
**What is broken:**
If a session expires while a user is filling out a long passenger details form, the user is abruptly redirected to the login page, and all entered data is irreversibly lost.
 
**Affected users:**
Users booking for large groups (e.g., 6 passengers) who take longer to carefully enter names, ages, and ID details, or users who switch tabs to verify information.
 
**Frequency:**
High for group bookings or users with slower typing speeds due to the strict idle timeout.
 
**Current flow — step by step:**
1. User initiates a booking for a group of 6 passengers.
2. User takes 5-10 minutes to gather and enter details for all passengers.
3. Session invisibly expires in the background due to the strict idle timeout.
4. User finally clicks "Continue" to proceed to payment.
5. User is abruptly redirected to the Login page with an "Invalid Session" error.
6. User logs back in successfully.
7. User must start the entire train search and data entry process from scratch because no draft was saved.
 
**Where exactly it breaks:**
Step 4 and 5. The system enforces a strict timeout without providing a visible pre-expiry warning to the user, and fails to cache the form state locally.
 
**How I found it:**
I left the passenger details screen open while writing notes on other issues. When I clicked proceed, I was kicked out completely.
 
**Screenshot or description:**
The standard login page with a red alert banner at the top reading "Session Expired. Please login again to continue." The previously entered passenger data is completely wiped.
