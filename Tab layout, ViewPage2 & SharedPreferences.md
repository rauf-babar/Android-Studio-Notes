
# 1. TabLayout

`TabLayout` is a **UI component** from the **Material Design library** that displays a series of **tabs** horizontally — typically used for **navigating between different views or fragments**.

For example:

```
| Home | Profile | Settings |
```

Each tab represents a different screen or section.
When the user clicks a tab, you show a new content area (like a fragment).

It is part of:

```gradle
com.google.android.material.tabs.TabLayout
```



## Requirements

To use `TabLayout`, you must include the **Material Components dependency** in your `build.gradle`:

```gradle
implementation 'com.google.android.material:material:1.12.0'
```

Then, in XML layout files, always use the full tag:

```xml
<com.google.android.material.tabs.TabLayout
    android:id="@+id/tabLayout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"/>
```

---

## Structure Overview

Typically, you use `TabLayout` **with**:

* `ViewPager` (old) or `ViewPager2` (new)
* `Fragment`s or custom layouts
* `TabItem` (optional helper tag for static tabs)

Typical structure:

```xml
<LinearLayout
    android:orientation="vertical"
    ...>

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout"
        ... />

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager"
        ... />

</LinearLayout>
```

---

## Creating multiple Tabs

### (a) `<com.google.android.material.tabs.TabLayout>`

This is the main container of all tabs.

### (b) `<com.google.android.material.tabs.TabItem>`

This represents a **single tab** (used only when tabs are static, not dynamic).

Example:

```xml
<com.google.android.material.tabs.TabLayout
    android:id="@+id/tabLayout"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <com.google.android.material.tabs.TabItem
        android:text="Home" />

    <com.google.android.material.tabs.TabItem
        android:text="Profile" />

    <com.google.android.material.tabs.TabItem
        android:text="Settings" />

</com.google.android.material.tabs.TabLayout>
```

---

##  Key Attributes of `TabLayout`

Let’s explore the **important attributes** you can use in XML.

| Attribute    | Description                                                                                     | Example  |
| ------------ | ------------------------- | -------------------------------------------- |
| `app:tabMode`               | Controls how tabs are displayed. <br>Options: `fixed` (equal width), `scrollable` (scrolls horizontally). | `app:tabMode="scrollable"`                   |
| `app:tabGravity`            | Controls tab alignment. <br>Options: `fill`, `center`.                                                    | `app:tabGravity="fill"`                      |
| `app:tabIndicatorColor`     | Sets the color of the underline (indicator).                                                              | `app:tabIndicatorColor="@color/blue"`        |
| `app:tabIndicatorHeight`    | Height of the underline indicator.                                                                        | `app:tabIndicatorHeight="3dp"`               |
| `app:tabTextColor`          | Default text color for unselected tabs.                                                                   | `app:tabTextColor="@color/gray"`             |
| `app:tabSelectedTextColor`  | Text color for selected tab.                                                                              | `app:tabSelectedTextColor="@color/black"`    |
| `app:tabIconTint`           | Tint for tab icons.                                                                                       | `app:tabIconTint="@color/white"`             |
| `app:tabBackground`         | Background drawable for each tab.                                                                         | `app:tabBackground="@drawable/tab_selector"` |
| `app:tabRippleColor`        | Ripple effect color when clicked.                                                                         | `app:tabRippleColor="@color/purple_200"`     |
| `app:tabIndicatorFullWidth` | Whether indicator spans full width or just text width.                                                    | `app:tabIndicatorFullWidth="false"`          |

## Adding Tabs Dynamically in Java

You can also create tabs **programmatically**:

```java
TabLayout tabLayout = findViewById(R.id.tabLayout);

tabLayout.addTab(tabLayout.newTab().setText("Home"));
tabLayout.addTab(tabLayout.newTab().setText("Profile"));
tabLayout.addTab(tabLayout.newTab().setText("Settings"));
```

To listen for tab changes:

```java
tabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
    @Override
    public void onTabSelected(TabLayout.Tab tab) {
        Toast.makeText(getApplicationContext(), "Selected: " + tab.getText(), Toast.LENGTH_SHORT).show();
    }

    @Override
    public void onTabUnselected(TabLayout.Tab tab) { }

    @Override
    public void onTabReselected(TabLayout.Tab tab) { }
});
```

---

## Linking with `ViewPager2`

If you want each tab to show a **different fragment** (most common case):

### Step 1 — XML

```xml
<LinearLayout
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"/>
</LinearLayout>
```

### Step 2 — Java code

```java
TabLayout tabLayout = findViewById(R.id.tabLayout);
ViewPager2 viewPager = findViewById(R.id.viewPager);

ViewPagerAdapter adapter = new ViewPagerAdapter(this);
viewPager.setAdapter(adapter);

new TabLayoutMediator(tabLayout, viewPager,
    (tab, position) -> {
        switch (position) {
            case 0:
                tab.setText("Home");
                break;
            case 1:
                tab.setText("Profile");
                break;
            case 2:
                tab.setText("Settings");
                break;
        }
    }
).attach();
```

### Step 3 — Adapter Class

```java
public class ViewPagerAdapter extends FragmentStateAdapter {
    public ViewPagerAdapter(@NonNull FragmentActivity fa) {
        super(fa);
    }

    @NonNull
    @Override
    public Fragment createFragment(int position) {
        switch (position) {
            case 0: return new HomeFragment();
            case 1: return new ProfileFragment();
            case 2: return new SettingsFragment();
            default: return new HomeFragment();
        }
    }

    @Override
    public int getItemCount() {
        return 3;
    }
}
```

This allows swiping between tabs and auto-sync between `ViewPager2` and `TabLayout`.

## Related Concepts

| Concept                    | Explanation                                                                                |
| -------------------------- | ------------------------------------------------------------------------------------------ |
| **ViewPager / ViewPager2** | Component that allows swiping between pages (fragments or layouts). Used with `TabLayout`. |
| **Fragment**               | Each tab’s content is often a fragment (small portion of UI).                              |
| **TabLayoutMediator**      | A bridge between `TabLayout` and `ViewPager2` to keep them in sync.                        |
| **Material Components**    | `TabLayout` belongs to Material library, so it follows Material Design principles.         |
| **Custom Tabs**            | You can create custom tab views with icons, badges, or layouts.                            |

Example of custom tab:

```java
TabLayout.Tab tab = tabLayout.newTab();
tab.setCustomView(R.layout.custom_tab);
tabLayout.addTab(tab);
```


## Common Customizations

✅ Add icons:

```java
tabLayout.getTabAt(0).setIcon(R.drawable.ic_home);
```

✅ Add badges:

```java
tabLayout.getTabAt(1).getOrCreateBadge().setNumber(5);
```

✅ Change indicator style (XML):

```xml
app:tabIndicatorGravity="bottom"
app:tabIndicatorFullWidth="false"
app:tabIndicatorColor="@color/red"
```

---

## Common Mistakes to Avoid

| Mistake                                                 | Explanation                           |
| ------------------------------------------------------- | ------------------------------------- |
| Forgetting to use `TabLayoutMediator` with `ViewPager2` | Without it, tab selection won’t sync. |
| Using old `ViewPager` (deprecated)                      | Use `ViewPager2` for modern apps.     |
| Not adding Material dependency                          | The TabLayout class won’t be found.   |
| Mixing static `<TabItem>` with dynamic `addTab()`       | Choose one approach.                  |


# `ViewPager` / `ViewPager2`

Normally, `ViewPager2` is the **engine** that:

* Manages **which fragment** is visible right now.
* Automatically handles **swiping gestures** (left ↔ right).
* Synchronizes with `TabLayout` (through `TabLayoutMediator`).
* Automatically **destroys and restores** fragments efficiently.

So, when you swipe → it changes page.
When you tap a tab → it automatically scrolls to the matching page.


## What if we **don’t use `ViewPager`**?

Totally valid.
Without ViewPager, **TabLayout** is just a **visual bar** — a UI component that can show tabs and detect which one you clicked.
👉 But it **doesn’t know what content to display** on its own.

That means:

* You can still **listen for tab selection** events.
* You must **manually switch the content** (e.g., replace Fragments or change layouts) yourself.

✅ It will work — just **no automatic swiping** or automatic fragment handling.

## How to do it manually (without ViewPager)

Let’s say we have three fragments:

* `HomeFragment`
* `ProfileFragment`
* `SettingsFragment`

### Step 1 — XML

```xml
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <FrameLayout
        android:id="@+id/frameContainer"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```

### Step 2 — Java (manual fragment switching)

```java
TabLayout tabLayout = findViewById(R.id.tabLayout);

// Add tabs
tabLayout.addTab(tabLayout.newTab().setText("Home"));
tabLayout.addTab(tabLayout.newTab().setText("Profile"));
tabLayout.addTab(tabLayout.newTab().setText("Settings"));

// Show default fragment
getSupportFragmentManager().beginTransaction()
        .replace(R.id.frameContainer, new HomeFragment())
        .commit();

// Listen for tab changes
tabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
    @Override
    public void onTabSelected(TabLayout.Tab tab) {
        Fragment selected = null;
        switch (tab.getPosition()) {
            case 0:
                selected = new HomeFragment();
                break;
            case 1:
                selected = new ProfileFragment();
                break;
            case 2:
                selected = new SettingsFragment();
                break;
        }
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.frameContainer, selected)
                .commit();
    }

    @Override
    public void onTabUnselected(TabLayout.Tab tab) { }

    @Override
    public void onTabReselected(TabLayout.Tab tab) { }
});
```

✅ You can now click tabs and see fragments change.
❌ But there’s **no swipe gesture**.

---
# MINI WHATSAPP CHAT SYSTEM
We’ll make 3 fragments:

* **ChatsFragment**
* **StatusFragment**
* **CallsFragment**

and an **adapter** (`MainPagerAdapter`) plus the **MainActivity** with full explanations.



## 1️⃣ Fragments

Each fragment just displays some sample data.

### **ChatsFragment.java**

```java
package com.example.whatsappclone;

import android.os.Bundle;
import androidx.fragment.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

public class ChatsFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        // Inflate a simple layout with text
        TextView textView = new TextView(getContext());
        textView.setText("This is the CHATS fragment\nUnread chat messages shown here.");
        textView.setTextSize(18);
        textView.setPadding(40, 100, 40, 100);
        return textView;
    }
}
```



### **StatusFragment.java**

```java
package com.example.whatsappclone;

import android.os.Bundle;
import androidx.fragment.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

public class StatusFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        TextView textView = new TextView(getContext());
        textView.setText("This is the STATUS fragment\nFriends' statuses appear here.");
        textView.setTextSize(18);
        textView.setPadding(40, 100, 40, 100);
        return textView;
    }
}
```



### **CallsFragment.java**

```java
package com.example.whatsappclone;

import android.os.Bundle;
import androidx.fragment.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

public class CallsFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        TextView textView = new TextView(getContext());
        textView.setText("This is the CALLS fragment\nRecent call logs appear here.");
        textView.setTextSize(18);
        textView.setPadding(40, 100, 40, 100);
        return textView;
    }
}
```


## 2️⃣ ViewPager2 Adapter

### **MainPagerAdapter.java**

```java
package com.example.whatsappclone;

import androidx.annotation.NonNull;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentActivity;
import androidx.viewpager2.adapter.FragmentStateAdapter;

// This adapter tells ViewPager2 which fragment to show for each position
public class MainPagerAdapter extends FragmentStateAdapter {

    public MainPagerAdapter(@NonNull FragmentActivity fragmentActivity) {
        super(fragmentActivity);
    }

    @NonNull
    @Override
    public Fragment createFragment(int position) {
        switch (position) {
            case 0:
                return new ChatsFragment();
            case 1:
                return new StatusFragment();
            case 2:
                return new CallsFragment();
            default:
                return new ChatsFragment();
        }
    }

    @Override
    public int getItemCount() {
        return 3; // total number of tabs/fragments
    }
}
```



## 3️⃣ Main Activity

### **MainActivity.java**

```java
package com.example.whatsappclone;

import androidx.annotation.NonNull;
import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import com.google.android.material.tabs.TabLayout;
import com.google.android.material.tabs.TabLayoutMediator;
import androidx.viewpager2.widget.ViewPager2;
import com.google.android.material.badge.BadgeDrawable;

public class MainActivity extends AppCompatActivity {

    private TabLayout tabLayout;
    private ViewPager2 viewPager;

    // Example counts (can be dynamic)
    private int unreadChats = 5;
    private int missedCalls = 2;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tabLayout = findViewById(R.id.tabLayout);
        viewPager = findViewById(R.id.viewPager);

        // Attach adapter to ViewPager2
        MainPagerAdapter adapter = new MainPagerAdapter(this);
        viewPager.setAdapter(adapter);

        // Add tabs manually (optional)
        tabLayout.addTab(tabLayout.newTab().setText("CHATS"));
        tabLayout.addTab(tabLayout.newTab().setText("STATUS"));
        tabLayout.addTab(tabLayout.newTab().setText("CALLS"));

        // Add badge directly (method 1)
        tabLayout.getTabAt(0).getOrCreateBadge().setNumber(unreadChats);

        // Add badge manually by creating a BadgeDrawable (method 2)
        BadgeDrawable callBadge = tabLayout.getTabAt(2).getOrCreateBadge();
        callBadge.setVisible(true);
        callBadge.setNumber(missedCalls);

        // Link TabLayout and ViewPager2 together using TabLayoutMediator
        new TabLayoutMediator(tabLayout, viewPager, (tab, position) -> {
            switch (position) {
                case 0:
                    tab.setText("CHATS");
                    break;
                case 1:
                    tab.setText("STATUS");
                    break;
                case 2:
                    tab.setText("CALLS");
                    break;
            }
        }).attach();

        // 1️⃣ If you want to control switching through ViewPager2
        //    e.g. user swipes → change tab automatically
        viewPager.registerOnPageChangeCallback(new ViewPager2.OnPageChangeCallback() {
            @Override
            public void onPageSelected(int position) {
                super.onPageSelected(position);

                // Example: remove badge when tab is viewed
                if (position == 0 && tabLayout.getTabAt(0).getBadge() != null)
                    tabLayout.getTabAt(0).removeBadge();
                if (position == 2 && tabLayout.getTabAt(2).getBadge() != null)
                    tabLayout.getTabAt(2).removeBadge();
            }
        });

        // 2️⃣ If you want to control switching through TabLayout listeners
        //    (use this if you want to handle tab clicks manually)
        tabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
            @Override
            public void onTabSelected(@NonNull TabLayout.Tab tab) {
                // When tab clicked, update ViewPager page
                viewPager.setCurrentItem(tab.getPosition());

                // Remove badge when that tab is opened
                if (tab.getBadge() != null)
                    tab.removeBadge();
            }

            @Override
            public void onTabUnselected(@NonNull TabLayout.Tab tab) {}

            @Override
            public void onTabReselected(@NonNull TabLayout.Tab tab) {}
        });
    }
}
```



## 4️⃣ Layout File

### **activity_main.xml**

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <!-- Top Tab Layout -->
    <com.google.android.material.tabs.TabLayout
        android:id="@+id/tabLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:tabGravity="fill"
        app:tabMode="fixed" />

    <!-- ViewPager2 for swipe navigation -->
    <androidx.viewpager2.widget.ViewPager2
        android:id="@+id/viewPager"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

---


# **1️⃣ SharedPreferences**

`SharedPreferences` is a **lightweight key–value storage** mechanism built into Android.
It lets you **store small pieces of data persistently** — that is, data that remains available **even after the app is closed or restarted**.

You can think of it like a small dictionary or JSON file saved inside your app’s private storage.

**Examples of what you might store:**

* User login state (e.g., logged in or not)
* Username, email, or profile info
* App settings (dark mode, language, sound on/off)
* High score or last played level in a game

---

## **2️⃣ How SharedPreferences Works Internally**

Internally, each `SharedPreferences` file is stored as an **XML file** in:

```
/data/data/<your_package_name>/shared_prefs/
```

Example:

```
/data/data/com.example.myapp/shared_prefs/user_prefs.xml
```

It stores data like:

```xml
<map>
    <string name="username">Abdul</string>
    <int name="score" value="95" />
    <boolean name="logged_in" value="true" />
</map>
```

---

## **3️⃣ Getting SharedPreferences**

You can get a `SharedPreferences` instance in **two main ways**:

### **(a) Activity Context**

```java
SharedPreferences prefs = getSharedPreferences("MyPrefs", MODE_PRIVATE);
```

* `"MyPrefs"` → file name (you can give any name)
* `MODE_PRIVATE` → file is accessible only by your app (default mode)

### **(b) Application-wide default preferences**

```java
SharedPreferences prefs = PreferenceManager.getDefaultSharedPreferences(this);
```

* This is a **global preference file** for the whole app### **(a) Using `getSharedPreferences()`**

``` java
SharedPreferences  prefs  = getSharedPreferences("UserPrefs", MODE_PRIVATE);
prefs.edit().putString("username", "Abdul").apply();
```

File created → `UserPrefs.xml`

If you later do:

``` java
SharedPreferences  prefs2  = getSharedPreferences("SettingsPrefs", MODE_PRIVATE);
prefs2.edit().putBoolean("dark_mode", true).apply();
``` 

Now there are **two separate files:**

`UserPrefs.xml
SettingsPrefs.xml` 

Each file is **independent** — one doesn’t know about the other.

----------

### **(b) Using `PreferenceManager.getDefaultSharedPreferences(this)`**

``` java
SharedPreferences  prefs  = PreferenceManager.getDefaultSharedPreferences(this);
prefs.edit().putString("username", "Abdul").apply();
prefs.edit().putBoolean("dark_mode", true).apply();
``` 

Now both values go inside the **same single file**:

`<package_name>_preferences.xml` 

So everything is stored globally — all settings in one place..



## **4️⃣ Modes (Old versions concept)**

| Mode                   | Description                                        |
| ---------------------- | -------------------------------------------------- |
| `MODE_PRIVATE`         | Default; file only accessible by your app.         |
| `MODE_APPEND`          | Append new data to existing (rarely used).         |
| `MODE_WORLD_READABLE`  | Deprecated – used to allow other apps to read it.  |
| `MODE_WORLD_WRITEABLE` | Deprecated – used to allow other apps to write it. |

**Modern Android (post API 24)** only uses `MODE_PRIVATE`.

---

## **5️⃣ Storing Data**

You **can’t write directly** into SharedPreferences —
you must use an **Editor** object.

### Example:

```java
SharedPreferences prefs = getSharedPreferences("MyPrefs", MODE_PRIVATE);
SharedPreferences.Editor editor = prefs.edit();

// Store values
editor.putString("username", "Abdul");
editor.putInt("score", 90);
editor.putBoolean("logged_in", true);

// Commit the changes
editor.apply();  // or editor.commit();
```

**Difference:**

| Method     | Description                                               |
| ---------- | --------------------------------------------------------- |
| `apply()`  | Asynchronous (faster, no return value). Recommended.      |
| `commit()` | Synchronous (returns boolean success). Slower, blocks UI. |

---

## **6️⃣ Retrieving Data**

```java
SharedPreferences prefs = getSharedPreferences("MyPrefs", MODE_PRIVATE);

String user = prefs.getString("username", "Guest"); // default = "Guest"
int score = prefs.getInt("score", 0);               // default = 0
boolean isLogged = prefs.getBoolean("logged_in", false);
```

If the key doesn’t exist, the **default value** is returned.

---

## **7️⃣ Removing or Clearing Data**

```java
SharedPreferences prefs = getSharedPreferences("MyPrefs", MODE_PRIVATE);
SharedPreferences.Editor editor = prefs.edit();

editor.remove("username"); // removes specific key
editor.clear();             // clears all data
editor.apply();
```
### Rmoving File from phone

``` java 
boolean  deleted  = getApplicationContext().deleteSharedPreferences("MyPrefs"); if (deleted) {
    Log.d("Prefs", "SharedPreferences file deleted successfully!");
} else {
    Log.d("Prefs", "Failed to delete SharedPreferences file!");
}
``` 

Explanation:

-   `"MyPrefs"` is the same name you passed to `getSharedPreferences("MyPrefs", MODE_PRIVATE)`.
    
-   This deletes the **entire XML file** (`MyPrefs.xml`) from the `shared_prefs` folder.
    
-   After this, the next time you call `getSharedPreferences("MyPrefs", ...)`, a **new empty file** will be created automatically.
    

- ✅ Deletes entire file  
- ✅ Safe, official method  
- ⚠️ Works only on Android **7.0+**
---

## **8️⃣ Checking if Key Exists**

```java
if (prefs.contains("username")) {
    // Key exists
}
```


# **`EncryptedSharedPreferences` (For Sensitive Data)**

Introduced in AndroidX Security library.

```java
SharedPreferences securePrefs = EncryptedSharedPreferences.create(
        "SecurePrefs",
        MasterKeys.getOrCreate(MasterKeys.AES256_GCM_SPEC),
        context,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
);
```
-----
## Login & logout Example


## 🔹 Project Overview

We’ll have **two activities**:

1.  **LoginActivity** —
    
    -   User enters name and password
        
    -   On login, we store:
        
        -   `"username"`
            
        -   `"password"`
            
        -   `"isLoggedIn"` = `true`
            
    -   Then navigate to `MainActivity`.
        
2.  **MainActivity** —
    
    -   Checks if user is logged in on startup.
        
    -   If **not logged in**, redirect back to `LoginActivity`.
        
    -   If **logged in**, show user info and a **Logout button**.
        
    -   On logout → remove preferences completely.
        



## 🔹 File 1 — LoginActivity.java

```java
package com.example.sharedprefdemo;

import android.content.Context;
import android.content.Intent;
import android.content.SharedPreferences;
import android.os.Bundle;
import android.widget.Button;
import android.widget.EditText;
import android.widget.Toast;

import androidx.appcompat.app.AppCompatActivity;

public class LoginActivity extends AppCompatActivity {

    EditText etName, etPassword;
    Button btnLogin;

    // File name for local storage
    private static final String PREF_FILE = "UserPrefs";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);

        etName = findViewById(R.id.etName);
        etPassword = findViewById(R.id.etPassword);
        btnLogin = findViewById(R.id.btnLogin);

        // Step 1: Check if already logged in
        SharedPreferences prefs = getSharedPreferences(PREF_FILE, Context.MODE_PRIVATE);
        boolean isLoggedIn = prefs.getBoolean("isLoggedIn", false);

        if (isLoggedIn) {
            // User already logged in → skip login
            Intent intent = new Intent(LoginActivity.this, MainActivity.class);
            startActivity(intent);
            finish(); // close login screen
            return;
        }

        // Step 2: Handle login button
        btnLogin.setOnClickListener(v -> {
            String name = etName.getText().toString().trim();
            String password = etPassword.getText().toString().trim();

            if (name.isEmpty() || password.isEmpty()) {
                Toast.makeText(this, "Enter name and password", Toast.LENGTH_SHORT).show();
                return;
            }

            // Save data in SharedPreferences
            SharedPreferences.Editor editor = prefs.edit();
            editor.putString("username", name);
            editor.putString("password", password);
            editor.putBoolean("isLoggedIn", true);
            editor.apply(); // save asynchronously

            Toast.makeText(this, "Login Successful", Toast.LENGTH_SHORT).show();

            // Move to main activity
            Intent intent = new Intent(LoginActivity.this, MainActivity.class);
            startActivity(intent);
            finish();
        });
    }
}

```

### Explanation:

-   Uses `getSharedPreferences("UserPrefs", MODE_PRIVATE)` — private file per app.
    
-   Saves three fields:
    
    -   `username`
        
    -   `password`
        
    -   `isLoggedIn`
        
-   If already logged in, directly navigates to `MainActivity`.
    

----------

## 🔹 File 2 — MainActivity.java

```java
package com.example.sharedprefdemo;

import android.content.Context;
import android.content.Intent;
import android.content.SharedPreferences;
import android.os.Bundle;
import android.widget.Button;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {

    TextView tvWelcome;
    Button btnLogout;
    private static final String PREF_FILE = "UserPrefs";

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        tvWelcome = findViewById(R.id.tvWelcome);
        btnLogout = findViewById(R.id.btnLogout);

        SharedPreferences prefs = getSharedPreferences(PREF_FILE, Context.MODE_PRIVATE);

        // Step 1: Verify login status
        boolean isLoggedIn = prefs.getBoolean("isLoggedIn", false);

        if (!isLoggedIn) {
            // Not logged in → go back to login screen
            Intent intent = new Intent(MainActivity.this, LoginActivity.class);
            startActivity(intent);
            finish();
            return;
        }

        // Step 2: Retrieve stored user info
        String name = prefs.getString("username", "Unknown User");
        String password = prefs.getString("password", "N/A"); // only for demo

        tvWelcome.setText("Welcome, " + name + "\nPassword: " + password);

        // Step 3: Handle logout
        btnLogout.setOnClickListener(v -> {

            // Option 1: Clear all preferences (keeps empty file)
            // prefs.edit().clear().apply();

            // Option 2: Fully delete SharedPreferences file (recommended for full logout)
            getApplicationContext().deleteSharedPreferences(PREF_FILE);

            // Redirect back to login
            Intent intent = new Intent(MainActivity.this, LoginActivity.class);
            startActivity(intent);
            finish();
        });
    }
}

```

----------
