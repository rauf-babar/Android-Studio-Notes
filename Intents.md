
# Android Intents

## 1. Core Concepts

**Intent**  
An abstract description of an operation to be performed. It acts as a bridge between Android app components.

### Types of Intents

1. **Explicit Intent**
   - Used for navigation within the same application.
   - You know the exact class name of the component to start.

2. **Implicit Intent**
   - Used for communication between different applications.
   - You specify an action, and Android selects the best app to handle it.

---

## 2. Explicit Intents (Internal Navigation)

### A. Sender Side (Starting an Activity)

#### Basic Navigation

```java
Intent intent = new Intent(MainActivity.this, SecondActivity.class);
startActivity(intent);
````

#### Passing Data (Sender)

Data is passed using **key–value pairs** via `putExtra()`.

```java
Intent intent = new Intent(MainActivity.this, SecondActivity.class);

// Primitive Data Types
intent.putExtra("USER_NAME", "Ali");
intent.putExtra("USER_AGE", 25);
intent.putExtra("IS_VERIFIED", true);

// Bundle (Grouped Data)
Bundle bundle = new Bundle();
bundle.putDouble("GPA", 3.8);
intent.putExtras(bundle);

startActivity(intent);
```

---

### B. Receiver Side (Retrieving Data)

Place this code inside `onCreate()` of the target Activity.

```java
Intent intent = getIntent();

// Primitive Data
String name = intent.getStringExtra("USER_NAME");
int age = intent.getIntExtra("USER_AGE", 0);
boolean verified = intent.getBooleanExtra("IS_VERIFIED", false);

// Bundle Data
Bundle args = intent.getExtras();
if (args != null) {
    double gpa = args.getDouble("GPA");
}
```

---

## 3. Implicit Intents (Common Actions)

### A. Location (Maps)

**Action:** `Intent.ACTION_VIEW`

#### Style 1: Specific Coordinates

```java
Uri location = Uri.parse("geo:37.7749,-122.4194");
Intent mapIntent = new Intent(Intent.ACTION_VIEW, location);
startActivity(mapIntent);
```

#### Style 2: Search Address

```java
Uri address = Uri.parse("geo:0,0?q=1600+Amphitheatre+Parkway");
Intent mapIntent = new Intent(Intent.ACTION_VIEW, address);
startActivity(mapIntent);
```

---

### B. Web Browser

**Action:** `Intent.ACTION_VIEW`

```java
Uri webpage = Uri.parse("https://www.google.com");
Intent webIntent = new Intent(Intent.ACTION_VIEW, webpage);
startActivity(webIntent);
```

---

### C. Phone Dialer

#### Style 1: Open Dial Pad (Safe)

**Action:** `Intent.ACTION_DIAL`

```java
Uri number = Uri.parse("tel:03001234567");
Intent dialIntent = new Intent(Intent.ACTION_DIAL, number);
startActivity(dialIntent);
```

#### Style 2: Direct Call (Permission Required)

**Action:** `Intent.ACTION_CALL`

**Manifest Permission**

```xml
<uses-permission android:name="android.permission.CALL_PHONE"/>
```

```java
Uri number = Uri.parse("tel:03001234567");
Intent callIntent = new Intent(Intent.ACTION_CALL, number);
startActivity(callIntent);
```

⚠️ App crashes if runtime permission is missing.

---

## 4. Advanced Implicit Scenarios

### A. Sharing Content (Share Sheet)

**Action:** `Intent.ACTION_SEND`

```java
Intent shareIntent = new Intent(Intent.ACTION_SEND);
shareIntent.setType("text/plain");

shareIntent.putExtra(Intent.EXTRA_SUBJECT, "Email Subject");
shareIntent.putExtra(Intent.EXTRA_TEXT, "This is the body text.");

// Force chooser dialog
startActivity(Intent.createChooser(shareIntent, "Share via"));
```

#### MIME Type Filtering

```java
shareIntent.setType("image/jpeg"); // Images
shareIntent.setType("video/*");    // Videos
```

#### Multiple MIME Types (Image OR PDF)

```java
Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
intent.setType("*/*");

String[] mimeTypes = {"image/*", "application/pdf"};
intent.putExtra(Intent.EXTRA_MIME_TYPES, mimeTypes);

filePickerLauncher.launch(intent);
```

---

### EXTRA Attributes Summary

| Attribute     | Primary Use     | Email           | WhatsApp |
| ------------- | --------------- | --------------- | -------- |
| EXTRA_SUBJECT | Context / Title | Subject line    | Ignored  |
| EXTRA_TEXT    | Main content    | Body            | Message  |
| EXTRA_EMAIL   | Recipient       | "To" field      | Ignored  |
| EXTRA_STREAM  | Attachments     | File attachment | Preview  |

---

### B. Receiving Implicit Intents (Share Receiver)

#### AndroidManifest.xml

```xml
<activity android:name=".ShareReceiverActivity">
    <intent-filter>
        <action android:name="android.intent.action.SEND" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:mimeType="text/plain" />
    </intent-filter>
</activity>
```

#### Java (ShareReceiverActivity)

```java
Intent intent = getIntent();

if (Intent.ACTION_SEND.equals(intent.getAction())
        && "text/plain".equals(intent.getType())) {

    String sharedText = intent.getStringExtra(Intent.EXTRA_TEXT);
}
```

---

## 5. Activity Result Launcher (Modern API)

#### Launcher Setup 

ActivityResultLauncher<Intent> filePickerLauncher = registerForActivityResult(
    new ActivityResultContracts.StartActivityForResult(),
    new ActivityResultCallback<ActivityResult>() {
        @Override
        public void onActivityResult(ActivityResult result) {
            if (result.getResultCode() == Activity.RESULT_OK) {
                Intent data = result.getData();
                if (data != null) {
                    Uri fileUri = data.getData();
                    // Handle file here...
                }
            }
        }
    }
);

#### Launcher Setup (Lambda)

```java
ActivityResultLauncher<Intent> filePickerLauncher =
        registerForActivityResult(
                new ActivityResultContracts.StartActivityForResult(),
                result -> {
                    if (result.getResultCode() == Activity.RESULT_OK) {
                        Intent data = result.getData();
                        if (data != null) {
                            Uri fileUri = data.getData();
                        }
                    }
                }
        );
```

#### Launch Image Picker

```java
public void uploadImage() {
    Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
    intent.setType("image/*");
    filePickerLauncher.launch(intent);
}
```
# Handling Image & PDF Selection

### 1. The Launcher (Receiver)
This code sits at the top of your Activity. It waits for the user to pick a file.

```java
// Assumed UI elements
ImageView myImageView = findViewById(R.id.imgPreview);
TextView myPdfTextView = findViewById(R.id.pdfNamePreview);

ActivityResultLauncher<Intent> filePickerLauncher = registerForActivityResult(
    new ActivityResultContracts.StartActivityForResult(),
    result -> {
        if (result.getResultCode() == Activity.RESULT_OK) {
            Intent data = result.getData();
            if (data != null) {
                // 1. Get the File Path (Uri)
                Uri selectedUri = data.getData();
                
                // 2. Check: Is it Image or PDF?
                String type = getContentResolver().getType(selectedUri);
                
                if (type != null) {
                    if (type.startsWith("image/")) {
                        // HANDLE IMAGE
                        // Simplest way to show image from Uri
                        myImageView.setImageURI(selectedUri);
                        
                        // Hide PDF text
                        myPdfTextView.setVisibility(View.GONE);
                        myImageView.setVisibility(View.VISIBLE);
                        
                    } else if (type.equals("application/pdf")) {
                        // HANDLE PDF
                        // We cannot "show" a PDF in an ImageView. 
                        // Instead, we show an icon or the file name.
                        myPdfTextView.setText("PDF Selected: " + selectedUri.getLastPathSegment());
                        
                        // Hide Image view
                        myImageView.setVisibility(View.GONE);
                        myPdfTextView.setVisibility(View.VISIBLE);
                    }
                }
            }
        }
    }
);

#### Launch Image or PDF Picker

```java
public void uploadMixed() {
    Intent intent = new Intent(Intent.ACTION_GET_CONTENT);
    intent.setType("*/*");

    String[] mimeTypes = {"image/*", "application/pdf"};
    intent.putExtra(Intent.EXTRA_MIME_TYPES, mimeTypes);

    filePickerLauncher.launch(intent);
}
```

---

### B. Explicit Usage (Returning Result from Activity)

#### MainActivity (Sender)

```java
ActivityResultLauncher<Intent> myActivityLauncher =
        registerForActivityResult(
                new ActivityResultContracts.StartActivityForResult(),
                result -> {
                    if (result.getResultCode() == Activity.RESULT_OK) {
                        Intent data = result.getData();
                        String message = data.getStringExtra("RESULT_KEY");
                    }
                }
        );

public void openActivityB() {
    Intent intent = new Intent(MainActivity.this, SecondActivity.class);
    myActivityLauncher.launch(intent);
}
```

#### SecondActivity (Receiver)

```java
Intent resultIntent = new Intent();
resultIntent.putExtra("RESULT_KEY", "Task Completed Successfully");

setResult(Activity.RESULT_OK, resultIntent);
finish();
```

## 6. Result Codes Explained

When an Activity returns data, it sends a status code (`resultCode`) to tell the parent what happened.

| Code | Value | Meaning |
| :--- | :--- | :--- |
| **RESULT_OK** | `-1` | The operation succeeded. The user picked a file, took a photo, or submitted data. |
| **RESULT_CANCELED** | `0` | The user backed out. They pressed the "Back" button or closed the picker without selecting anything. |
| **RESULT_FIRST_USER** | `1` | Start of custom user-defined codes. You can use this if you want to define specific custom results (e.g., `EDIT_COMPLETED = 1`, `DELETE_COMPLETED = 2`). |

**Usage Example:**
```java
if (result.getResultCode() == Activity.RESULT_CANCELED) {
    // User pressed back without selecting anything
    Toast.makeText(this, "Selection Cancelled", Toast.LENGTH_SHORT).show();
}

```

---

## 7. Runtime Permissions (The Modern Way)

This uses `ActivityResultLauncher` with the `RequestPermission` contract. It returns a `Boolean` (True/False) instead of an `Intent`.

### A. Setup (Manifest)

**Crucial:** You must always declare the permission in `AndroidManifest.xml` first.

```xml
<uses-permission android:name="android.permission.RECORD_AUDIO" />

```

### Common Android Permissions

| Permission Name | Usage |
| :--- | :--- |
| `INTERNET` | Allows the app to access the Internet (APIs, Web). |
| `ACCESS_NETWORK_STATE` | Checks if the device is connected to WiFi or Mobile Data. |
| `READ_EXTERNAL_STORAGE` | accessing files on shared storage (Android 12 and below). |
| `READ_MEDIA_IMAGES` | Accessing photos/images only (Android 13+). |
| `READ_MEDIA_VIDEO` | Accessing video files only (Android 13+). |
| `CAMERA` | Accessing the camera sensor for photos or video. |
| `RECORD_AUDIO` | Accessing the microphone for voice notes or calls. |
| `ACCESS_FINE_LOCATION` | Precise location using GPS satellites. |
| `ACCESS_COARSE_LOCATION` | Approximate location using WiFi/Cell towers. |
| `READ_CONTACTS` | Reading user's address book data. |
| `CALL_PHONE` | Initiating a phone call directly without user confirmation. |
| `SEND_SMS` | Sending text messages in the background. |
| `READ_SMS` | Reading messages from the user's Inbox. |
| `BLUETOOTH_CONNECT` | Connecting to paired Bluetooth devices (Headphones, etc.). |

### B. The Launcher (Receiver)

Define this variable at the top of your class (outside methods).

```java
private final ActivityResultLauncher<String> requestPermissionLauncher =
    registerForActivityResult(new ActivityResultContracts.RequestPermission(),
        new ActivityResultCallback<Boolean>() {
            @Override
            public void onActivityResult(Boolean isGranted) {
                if (isGranted) {
                    startRecording();
                } else {
                    handleDenial();
                }
            }
        });

```

### C. The Trigger Logic (Asking User)

Do not launch immediately. Check if you already have it first.

```java
public void onRecordButtonClick() {
    // 1. Check if permission is ALREADY granted
    if (ContextCompat.checkSelfPermission(this, Manifest.permission.RECORD_AUDIO) 
            == PackageManager.PERMISSION_GRANTED) {
        
        // We have it -> Go!
        startRecording();
        
    } else {
        // We don't have it -> Ask the user
        // This triggers the System Dialog (or auto-denies if blocked)
        requestPermissionLauncher.launch(Manifest.permission.RECORD_AUDIO);
    }
}

```

---

## 8. Handling "Don't Ask Again" (Settings Redirect)

If the user selects "Don't Ask Again" (or denies twice on modern Android), the system blocks the popup. You must detect this and send them to Settings.

**Logic Flow:**

1. Launcher returns `false`.
2. check `shouldShowRequestPermissionRationale()`.
* **True:** User denied normally. Show a Toast.
* **False:** User is blocked ("Don't Ask Again"). Show Dialog to open Settings.



**Implementation:**

```java
private void handleDenial() {
    // Check if the system blocked the popup
    if (shouldShowRequestPermissionRationale(Manifest.permission.RECORD_AUDIO)) {
        // Case 1: Normal Denial
        Toast.makeText(this, "Permission is needed to record audio.", Toast.LENGTH_SHORT).show();
    } else {
        // Case 2: "Don't Ask Again" Checked (Permanent Denial)
        showSettingsDialog();
    }
}

private void showSettingsDialog() {
    new AlertDialog.Builder(this)
        .setTitle("Permission Required")
        .setMessage("You have permanently disabled this feature. Please enable it in Settings.")
        .setPositiveButton("Go to Settings", (dialog, which) -> {
            // Intent to open App Settings Page
            Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
            Uri uri = Uri.fromParts("package", getPackageName(), null);
            intent.setData(uri);
            startActivity(intent);
        })
        .setNegativeButton("Cancel", null)
        .show();
}

```



