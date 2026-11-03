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

3. **Photos & Videos / Media** (`READ_MEDIA_IMAGES`, `READ_MEDIA_VIDEO`, `READ_MEDIA_VISUAL_USER_SELECTED`):
   - Targets **Android 14+ (API 34+)**.
   - Handles the new Android 14 visual user-selected partial media access permission (`READ_MEDIA_VISUAL_USER_SELECTED`) along with standard media permissions.

---

## 💻 Code Explanation: How Permissions Are Handled

Here is how the permission flow and retry logic are implemented in `MainActivity.kt`:

### 1. Request Launcher & Retry Counter Logic

We use `rememberLauncherForActivityResult` with AndroidX activity result contracts. When the user denies the request, we decrement the remaining tries counter and notify them with a Snackbar:

```kotlin
// Launcher for camera permission
val cameraLauncher = rememberLauncherForActivityResult(
