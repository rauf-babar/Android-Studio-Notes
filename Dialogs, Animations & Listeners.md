# Topics
1. Infaltor
2. Alert Dialogs
3. Handlers
4. Animations
5. Listeners
6. Android Options Menu (The "3 Dots")


# 1. Mastering LayoutInflater

**Inflation** is the process of converting an XML layout file (e.g., `res/layout/item_row.xml`) into actual View objects (Java/Kotlin objects) that can be used in your code.

* **XML:** Static blueprint.
* **View Object:** Live component in memory.

---

## 2. Accessing the Inflater
There are two main ways to get a `LayoutInflater` instance depending on where you are writing code.

### A. Inside an Activity (`getLayoutInflater()`)
Since `Activity` is a Context, it has a built-in getter.
```java
LayoutInflater inflater = getLayoutInflater();

```

### B. Outside an Activity (`LayoutInflater.from(context)`)

Used in RecyclerView Adapters, Utility classes, or anywhere you only have a `Context` object.

```java
LayoutInflater inflater = LayoutInflater.from(context); // context could be 'this' or 'parent.getContext()'

```

---

## 3. The `inflate()` Method Syntax

The method signature is critical to understand to avoid crashes or UI bugs.

```java
public View inflate(int resource, ViewGroup root, boolean attachToRoot);

```

| Parameter | Meaning |
| --- | --- |
| **resource** | The ID of the XML file you want to convert (e.g., `R.layout.dialog_view`). |
| **root** | The parent ViewGroup that *might* contain this new view eventually. |
| **attachToRoot** | **Crucial Boolean.** See section below. |

---

## 4. The `attachToRoot` Boolean Explained

This is the most misunderstood part of inflation.

### A. When to use `true`

**Meaning:** "Create this view and immediately add it inside the parent `root`."
**Usage:** When you are manually adding a view to a layout container in your activity.

```java
// Adds 'banner_layout' inside 'mainContainer' instantly
inflater.inflate(R.layout.banner_layout, mainContainer, true);

```

### B. When to use `false` (Most Common)

**Meaning:** "Create the view object using the `root` for layout parameter calculations (width/height rules), but **DO NOT** add it yet."
**Usage:** Fragments, RecyclerViews. The system (or Adapter) will add it later.

```java
// Create view, prepare it for 'container', but wait to add it
inflater.inflate(R.layout.fragment_home, container, false);

```

### C. When to pass `null` as Root

**Meaning:** "I have no idea who the parent is yet."
**Usage:** Alert Dialogs.

```java
// Dialogs float over the screen, they don't have a standard parent view immediately
inflater.inflate(R.layout.custom_dialog, null);

```

---

## 5. Practical Implementation Examples

### Scenario 1: Fragments (Standard Pattern)

You are provided with an `inflater` object by the system. You almost **ALWAYS** use `false` here because the FragmentManager handles adding the view.

```java
@Override
public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
    // 1. Use the provided 'inflater'
    // 2. Pass 'container' so width/height match parent rules
    // 3. Pass 'false' because FragmentManager adds it later
    return inflater.inflate(R.layout.fragment_profile, container, false);
}

```

### Scenario 2: RecyclerView Adapter (From Context)

You don't have a global `getLayoutInflater()`, so you extract it from the parent context.

```java
@Override
public MyViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
    // 1. Get Inflater from parent's context
    LayoutInflater inflater = LayoutInflater.from(parent.getContext());
    
    // 2. Inflate Item Layout (attachToRoot = false is MANDATORY for lists)
    View itemView = inflater.inflate(R.layout.item_row, parent, false);
    
    return new MyViewHolder(itemView);
}

```

### Scenario 3: Custom Alert Dialog (Null Root)

Since a Dialog is a floating window, it is not "inside" your activity's layout XML.

```java
// 1. Get Inflater manually
LayoutInflater inflater = LayoutInflater.from(this); // or getLayoutInflater()

// 2. Inflate with NULL root
View dialogView = inflater.inflate(R.layout.dialog_design, null);

// 3. Set into Dialog Builder
new AlertDialog.Builder(this)
    .setView(dialogView)
    .show();

```

## 2. Alert Dialogs

**Concept:** A small window that prompts the user to make a decision or enter additional information. It does not fill the screen.

### A. Standard Alert Dialog (Builder Pattern)
Used for simple Yes/No confirmations.

**Lambda Style (Arrow Function):**
```java
new AlertDialog.Builder(this)
    .setTitle("Delete File?")
    .setMessage("This action cannot be undone.")
    .setIcon(R.drawable.ic_warning)
    .setCancelable(false) // User MUST click a button
    .setPositiveButton("Yes", (dialog, which) -> {
        // 'which' is the ID of the button clicked
        deleteFile();
    })
    .setNegativeButton("No", (dialog, which) -> {
        dialog.dismiss(); // Close dialog
    })
    .setNeutralButton("Archive", (dialog, which) -> {
        archiveFile();
    })
    .show(); // CRITICAL: Never forget .show()!

```

**Full Interface Style:**

```java
.setPositiveButton("Yes", new DialogInterface.OnClickListener() {
    @Override
    public void onClick(DialogInterface dialog, int which) {
        // Logic here
    }
})

```

### B. Custom Alert Dialog (Custom Layout)

Used when you need EditTexts, Images, or complex designs inside the popup.

**Steps:**

1. Create a layout file (e.g., `dialog_input.xml`).
2. Inflate it using `LayoutInflater`.
3. Set it using `.setView()`.

**Example:**

```java
// 1. Inflate the custom design
View dialogView = LayoutInflater.from(this).inflate(R.layout.dialog_input, null);

// 2. Find views inside that design (NOT findViewById(R.id...))
EditText inputField = dialogView.findViewById(R.id.et_name);

// 3. Build the Dialog
new AlertDialog.Builder(this)
    .setTitle("Enter Name")
    .setView(dialogView) // <--- The Magic Line
    .setPositiveButton("Save", (dialog, which) -> {
        String name = inputField.getText().toString();
        saveName(name);
    })
    .show();

```

---

## 3. Handlers (Background to Main Thread)

**Concept:** You cannot update the UI (TextView, Buttons) from a background thread. You use a `Handler` to move work from a background thread back to the Main UI thread.

### A. The Setup

**1. Full Interface Style (The Long Way):**

```java
Handler handler = new Handler(Looper.getMainLooper());

handler.post(new Runnable() {
    @Override
    public void run() {
        // Code here runs on the Main UI Thread
        textView.setText("Updated from background!");
    }
});

```

**2. Lambda Style (The Short Way):**

```java
Handler handler = new Handler(Looper.getMainLooper());

handler.post(() -> {
    // Code here runs on the Main UI Thread
    textView.setText("Updated from background!");
});

```

### B. Delayed Execution (Timer)

You can also use handlers to run code after a specific time (e.g., Splash Screen).

```java
// Run this code after 3000 milliseconds (3 seconds)
handler.postDelayed(() -> {
    openNextScreen();
}, 3000);

```

---

## 4. Animations (View Animations)

**Concept:** Animations are defined in XML (`res/anim`) and loaded using `AnimationUtils`.

### A. Common Animation Types (XML Attributes)

**1. Alpha (Fade In/Out)**

* `fromAlpha`: 0.0 (Invisible) -> 1.0 (Visible)
* `duration`: Time in ms.

**2. Translate (Move)**

* `fromXDelta`: Starting position.
* `toXDelta`: Ending position.
* `0%`: Current position. `100%`: Width of the view. `100%p`: Width of the screen (parent).

**3. Scale (Zoom)**

* `pivotX="50%"`, `pivotY="50%"`: Zooms from the center.

**4. Rotate (Spin)**

* `fromDegrees`: 0 -> `toDegrees`: 360.

### B. Loading & Running (Java)

```java
// 1. Load from XML
Animation fadeIn = AnimationUtils.loadAnimation(this, R.anim.fade_in);

// 2. (Optional) Set Attribute programmatically
fadeIn.setDuration(1000); 
fadeIn.setFillAfter(true); // Keeps the view in the final state after animation

// 3. Start
myImageView.startAnimation(fadeIn);

```

### C. Animation Sets (Multiple Animations)

You can combine animations to run Together or Sequentially.

**Option 1: Using XML `<set>` tag**

```xml
<set xmlns:android="...">
    <alpha ... />
    <scale ... />
</set>

```

## 5. Listeners Mastery

### A. Input Listeners (Text & Toggles)

#### 1. TextWatcher (EditText)
**Purpose:** Monitor every keystroke. Used for validation (password length) or live search.
**Note:** You **cannot** use a Lambda (Arrow function) here because the interface has 3 distinct methods. You must implement all of them.

**Example: Enable Button only if input > 5 characters**
```java
EditText passwordInput = findViewById(R.id.et_password);
Button loginBtn = findViewById(R.id.btn_login);
loginBtn.setEnabled(false); // Disable initially

passwordInput.addTextChangedListener(new TextWatcher() {
    @Override
    public void beforeTextChanged(CharSequence s, int start, int count, int after) {
        // Runs before the text changes (Rarely used)
    }

    @Override
    public void onTextChanged(CharSequence s, int start, int before, int count) {
        // Runs WHILE typing. Good for Search suggestions.
    }

    @Override
    public void afterTextChanged(Editable s) {
        // Runs AFTER typing is done. BEST for Validation.
        if (s.length() > 5) {
            loginBtn.setEnabled(true);
        } else {
            loginBtn.setError("Password too short!");
            loginBtn.setEnabled(false);
        }
    }
});

```

**Exam Tip:** Don't change the text *inside* `onTextChanged` (it creates an infinite loop). Use `afterTextChanged` for modifications.

---

#### 2. OnCheckedChangeListener (CheckBox, Switch, Radio)

**Purpose:** Detect when a toggle state changes.
**Lambda Supported:** Yes (Single method interface).

**Full Interface Way:**

```java
CheckBox termsBox = findViewById(R.id.cb_terms);

termsBox.setOnCheckedChangeListener(new CompoundButton.OnCheckedChangeListener() {
    @Override
    public void onCheckedChanged(CompoundButton buttonView, boolean isChecked) {
        if (isChecked) {
            // Box Ticked
        } else {
            // Box Unticked
        }
    }
});

```

**Lambda (Arrow) Way:**

```java
termsBox.setOnCheckedChangeListener((buttonView, isChecked) -> {
    // 'isChecked' is the boolean result
    if (isChecked) {
        Toast.makeText(this, "Accepted", Toast.LENGTH_SHORT).show();
    }
});

```

---

### B. Interaction Listeners (Click & Long Press)

#### 1. OnLongClickListener (The "Hold" Action)

**Purpose:** Detect when user holds down a view.
**Critical Rule:** The method returns a `boolean`.

* **Return `true`:** "I have handled this. Do NOT run the normal click after I let go." (Consumes event).
* **Return `false`:** "I saw it, but keep processing." (The normal `onClick` will fire immediately after you release).

**Example: Show Delete Dialog on Hold**

```java
Button delBtn = findViewById(R.id.btn_delete);

// Lambda Way
delBtn.setOnLongClickListener(v -> {
    showDeleteDialog(); 
    return true; // Important: Stops normal click from happening
});

```

---

### C. Animation Listeners (Recap)

**1. View Animation (XML Loaded)**

* **Lambda:** No (3 methods).
* **Tip:** Always use `onAnimationEnd` to set `visibility = GONE` if you are fading something out, otherwise it's still clickable but invisible.

```java
// Full Interface required
slideAnim.setAnimationListener(new Animation.AnimationListener() {
    @Override public void onAnimationStart(Animation a) {}
    @Override public void onAnimationEnd(Animation a) {
         view.setVisibility(View.GONE);
    }
    @Override public void onAnimationRepeat(Animation a) {}
});

```

# 6. Android Options Menu (The "3 Dots")

## 1. Where to write the Java Code?
**File:** `MainActivity.java`
**Location:** Inside the Class, but **STRICTLY OUTSIDE** the `onCreate` method.

### The Structure

```java
public class MainActivity extends AppCompatActivity {

    // 1. onCreate Method
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        
        // Normal setup code goes here...
    } 
    // <--- onCreate ENDS here. Do not write menu code inside!


    // ---------------------------------------------------------
    // 2. MENU METHODS (Siblings to onCreate)
    // ---------------------------------------------------------

    // Method A: INFLATE (Loads the XML to show the dots)
    @Override
    public boolean onCreateOptionsMenu(Menu menu) {
        // Syntax: getMenuInflater().inflate(R.menu.filename, menu);
        getMenuInflater().inflate(R.menu.my_menu_file, menu);
        return true;
    }

    // Method B: CLICK LISTENER (Handles the actions)
    @Override
    public boolean onOptionsItemSelected(@NonNull MenuItem item) {
        int id = item.getItemId(); // Get ID of clicked item

        if (id == R.id.action_settings) {
            // Handle Settings
            Toast.makeText(this, "Settings Clicked", Toast.LENGTH_SHORT).show();
            return true;
        } 
        else if (id == R.id.action_logout) {
            // Handle Logout
            return true;
        }

        return super.onOptionsItemSelected(item);
    }

}

```

---

## 2. The Menu XML File

**File Location:** `res/menu/my_menu_file.xml`
*(If `menu` folder doesn't exist: Right-click `res` -> New -> Directory -> name it "menu")*

```xml
<menu xmlns:android="[http://schemas.android.com/apk/res/android](http://schemas.android.com/apk/res/android)"
      xmlns:app="[http://schemas.android.com/apk/res-auto](http://schemas.android.com/apk/res-auto)">

    <item android:id="@+id/action_settings"
          android:title="Settings"
          app:showAsAction="never" /> <item android:id="@+id/action_logout"
          android:title="Logout"
          app:showAsAction="never" />

</menu>

```
