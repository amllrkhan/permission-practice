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
    contract = ActivityResultContracts.RequestPermission()
) { granted ->
    if (granted) {
        isCameraGranted = true
        scope.launch {
            snackbarHostState.showSnackbar("Camera permission granted!")
        }
    } else {
        if (cameraTries > 0) {
            cameraTries--
        }
        scope.launch {
            snackbarHostState.showSnackbar(
                "Permission denied, you've $cameraTries amount of tries left."
            )
        }
    }
}
```

### 2. Handling Tries Exhaustion & Redirecting to Settings

When an item is clicked, we first check if it is already granted. If not, and there are tries remaining (`cameraTries > 0`), we trigger the system dialog. If tries reach 0, we redirect the user directly to the app details in system settings:

```kotlin
if (isCameraGranted) {
    scope.launch {
        snackbarHostState.showSnackbar("Camera permission is already granted.")
    }
} else if (cameraTries > 0) {
    cameraLauncher.launch(Manifest.permission.CAMERA)
} else {
    // 0 tries left: redirect to settings
    scope.launch {
        snackbarHostState.showSnackbar("Tries exhausted. Navigating to settings...")
    }
    openAppSettings()
}
```

The helper function to open system settings:

```kotlin
fun openAppSettings() {
    val intent = Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS).apply {
        data = Uri.fromParts("package", context.packageName, null)
    }
    context.startActivity(intent)
}
```

### 3. API 34+ Media Permission Support

Android 14 (API 34) introduced selective photo/video sharing. In `AndroidManifest.xml` and in the launcher, we inspect the SDK version dynamically:

```kotlin
val permissionsToRequest = when {
    Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE -> arrayOf(
        Manifest.permission.READ_MEDIA_IMAGES,
        Manifest.permission.READ_MEDIA_VIDEO,
        Manifest.permission.READ_MEDIA_VISUAL_USER_SELECTED
    )
    Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU -> arrayOf(
        Manifest.permission.READ_MEDIA_IMAGES,
        Manifest.permission.READ_MEDIA_VIDEO
    )
    else -> arrayOf(
        Manifest.permission.READ_EXTERNAL_STORAGE
    )
}
mediaLauncher.launch(permissionsToRequest)
```

### 4. Detecting Permission Changes on App Resume

When the user leaves the app to grant permissions in device settings and returns, the `LifecycleEventObserver` catches the `ON_RESUME` event and automatically re-evaluates `checkSelfPermission()` so the UI status immediately switches to "Granted":

```kotlin
