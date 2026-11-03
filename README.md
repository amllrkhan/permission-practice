# Permissions Practice - Mini Project

This is an Android application built using Jetpack Compose to practice Android runtime permissions handling according to the assignment requirements.

---

## 📹 Demo Video

<details>
<summary>▶️ Click to expand/collapse demo video (Max 40s)</summary>

<!-- 
NOTE FOR SUBMISSION:
Drag and drop your recorded screen video (MP4) right here when editing this file on GitHub, or paste the video link below.
-->

https://github.com/user-attachments/assets/sample-demo-video.mp4

</details>

---

## 📱 Features & Behavior

The app consists of a single **Permissions Settings** screen with 3 items:

1. **Camera** (`CAMERA`):
   - Clicking prompts the system permission dialog.
   - If denied, a Snackbar shows: *"Permission denied, you've n amount of tries left."* (2 tries total).
   - Once all 2 tries are exhausted, tapping the item again automatically redirects the user to the app's system settings page.
   - When granted in settings and returning to the app, the status automatically updates to "Granted".

2. **Geolocation** (`ACCESS_FINE_LOCATION`, `ACCESS_COARSE_LOCATION`):
   - Requests location permissions using `RequestMultiplePermissions()`.
   - Manifest also declares `ACCESS_BACKGROUND_LOCATION`.
   - Follows the same 2-attempt denial countdown before navigating to system settings.
