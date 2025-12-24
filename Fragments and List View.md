# FRAGMENTS

A Fragment is a reusable, modular section of an Activity’s UI.
Think of it as a "mini-Activity" that lives inside an Activity.

## Analogy:
If an Activity is a “full page,” then Fragments are “sections” of that page 
that can be swapped in or out dynamically


## WHY USE FRAGMENTS

- To reuse parts of UI in multiple activities.  
- To support different screen sizes 
  (e.g., tablets show multiple fragments side-by-side).  
- To manage dynamic UIs where one part changes while others stay the same.  
- To enable navigation without switching activities (via FragmentTransaction).

## FRAGMENT LIFECYCLE
A fragment has its own lifecycle — but it is tied to the activity’s lifecycle.

|Method              | When It Runs                          | Purpose |
|--|--|--|
|onAttach()| Fragment attached to its parent activity  | Initialize variables that depend on context|
|onCreate()          | Fragment object is created             | Initialize non-UI logic or data|
|onCreateView()      | Fragment UI is created                 | Inflate layout XML → returns a View|
onViewCreated()     | After view is created                  | Safe to access UI elements and bind adapters
onStart()           | Fragment becomes visible               | UI visible but not interactive
onResume()          | Fragment active & interactive          | Ready for user input
onPause()           | User navigates away partially          | Pause updates or animations
onStop()            | Fragment fully hidden                  | Stop ongoing tasks
onDestroyView()     | Fragment’s view destroyed              | Cleanup view-related resources
onDestroy()         | Fragment destroyed                     | Final cleanup
onDetach()          | Fragment detached from activity        | Fragment no longer has access to activity|



## HOW ADAPTERS FIT INTO FRAGMENTS

When displaying lists (ListView / RecyclerView) inside a Fragment,
an Adapter acts as a bridge between:

• Data source → (e.g., ArrayList<Passenger>)
• UI component → (e.g., ListView)

Example:
``` java
PassengerAdapter adapter = new PassengerAdapter(requireContext(), AppData.getPassengers());
listView.setAdapter(adapter);
```

## Step-by-Step Process:

1. PassengerAdapter → Custom class extending ArrayAdapter<Passenger>
2. AppData.getPassengers() → Provides data list
3. listView.setAdapter(adapter) → Binds data to UI
4. getView() (inside adapter) → Called for each row to create or modify list item views



# PASSING INFO AMONG FRAGMENTS

## 1) Activity-mediated communication using direct fragment references

its a good fit when:

- You have a **single Activity** managing multiple fragments, are **created at Activity startup**, have **fixed IDs**,  are **always alive** when you press the button.
- You want **synchronous, immediate communication** between fragments.
- You want to **validate or coordinate** data in the Activity (like checking both inputs).

This approach starts breaking down when:

1. **Fragments are added/replaced dynamically.**
    
    - For example, when you replace Fragment A with Fragment B using transactions, you may lose the direct reference (`findFragmentById()` returns null).
    - In that case, you’d need to pass data through **`setArguments(Bundle)`** before adding/replacing.
2. **You have deeply nested or decoupled fragments.**
    
    - The Activity becomes a “middleman” for too many things, leading to **tight coupling** and messy code.
3. **You want reusable fragments** (used in multiple activities).
    
    - This method hard-codes the Activity as the central hub, so fragments depend on it.

## Project Structure

We’ll have:

- **MainActivity.java** → hosts 3 fragments.
- **FragmentOne.java** → top-left fragment (with EditText 1).
- **FragmentTwo.java** → top-right fragment (with EditText 2).
- **FragmentThree.java** → bottom fragment (displays combined data).
- **activity_main.xml** → layout that arranges the fragments (2 on top, 1 below, and a button).

## 1. `activity_main.xml`

Here we define **3 fragments and a button**.

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity"
    android:padding="16dp">

    <!-- Top two fragments in a horizontal layout -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal">

        <fragment
            android:id="@+id/fragmentOne"
            android:name="com.example.myapp.FragmentOne"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:layout_height="match_parent" />

        <fragment
            android:id="@+id/fragmentTwo"
            android:name="com.example.myapp.FragmentTwo"
            android:layout_width="0dp"
            android:layout_weight="1"
            android:layout_height="match_parent" />
    </LinearLayout>

    <!-- Bottom fragment -->
    <fragment
        android:id="@+id/fragmentThree"
        android:name="com.example.myapp.FragmentThree"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />

    <!-- Button -->
    <Button
        android:id="@+id/btnSendData"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Send Data to Bottom Fragment" />
</LinearLayout>
```

## 2. `MainActivity.java`

Here we handle button click, **fetch data** from both top fragments, **bundle** it, and **pass it** to bottom fragment.

```java
package com.example.myapp;

import androidx.appcompat.app.AppCompatActivity;
import android.os.Bundle;
import android.view.View;
import android.widget.Button;
import android.widget.Toast;

public class MainActivity extends AppCompatActivity {

    FragmentOne fragmentOne;
    FragmentTwo fragmentTwo;
    FragmentThree fragmentThree;
    Button btnSendData;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        fragmentOne = (FragmentOne) getSupportFragmentManager().findFragmentById(R.id.fragmentOne);
        fragmentTwo = (FragmentTwo) getSupportFragmentManager().findFragmentById(R.id.fragmentTwo);
        fragmentThree = (FragmentThree) getSupportFragmentManager().findFragmentById(R.id.fragmentThree);
        btnSendData = findViewById(R.id.btnSendData);

        btnSendData.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String text1 = fragmentOne.getInputText();
                String text2 = fragmentTwo.getInputText();

                if (text1.isEmpty() || text2.isEmpty()) {
                    Toast.makeText(MainActivity.this, "Please enter text in both fields!", Toast.LENGTH_SHORT).show();
                    return;
                }

                // Create bundle
                Bundle bundle = new Bundle();
                bundle.putString("data1", text1);
                bundle.putString("data2", text2);

                // Pass to FragmentThree
                fragmentThree.updateData(bundle);
            }
        });
    }
}
```

## 3. `FragmentOne.java`

```java
package com.example.myapp;

import android.os.Bundle;
import androidx.fragment.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.EditText;

public class FragmentOne extends Fragment {

    EditText editText;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_one, container, false);
        editText = view.findViewById(R.id.editText1);
        return view;
    }

    public String getInputText() {
        return editText.getText().toString().trim();
    }
}
```

## 4. `FragmentTwo.java`

```java
package com.example.myapp;

import android.os.Bundle;
import androidx.fragment.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.EditText;

public class FragmentTwo extends Fragment {

    EditText editText;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_two, container, false);
        editText = view.findViewById(R.id.editText2);
        return view;
    }

    public String getInputText() {
        return editText.getText().toString().trim();
    }
}
```

## 5. `FragmentThree.java`

```java
package com.example.myapp;

import android.os.Bundle;
import androidx.fragment.app.Fragment;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

public class FragmentThree extends Fragment {

    TextView textViewResult;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_three, container, false);
        textViewResult = view.findViewById(R.id.textViewResult);
        return view;
    }

    public void updateData(Bundle bundle) {
        String data1 = bundle.getString("data1");
        String data2 = bundle.getString("data2");

        textViewResult.setText("Received: " + data1 + " & " + data2);
    }
}
```

## 6. Fragment Layouts

### `fragment_one.xml`

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="8dp">

    <EditText
        android:id="@+id/editText1"
        android:hint="Enter first text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</FrameLayout>
```

### `fragment_two.xml`

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="8dp">

    <EditText
        android:id="@+id/editText2"
        android:hint="Enter second text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</FrameLayout>
```

### `fragment_three.xml`

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="8dp">

    <TextView
        android:id="@+id/textViewResult"
        android:text="Result will appear here"
        android:textSize="18sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</FrameLayout>
```

## 🧩 Flow Summary

1. **User enters text** in FragmentOne and FragmentTwo.
2. **Button click in MainActivity** collects data from both.
3. **Bundle** is created and sent to FragmentThree.
4. **FragmentThree updates its UI** to show both inputs.

## 2) Dynamic fragment management using `Bundle` arguments

- You **create fragment instances at runtime**,
- **attach data to them** using a `Bundle` (through `setArguments()`),
- and then **show/replace** them dynamically using a `FragmentTransaction`.

**Basic pattern:**

```java
FragmentX fragment = new FragmentX();
Bundle bundle = new Bundle();
bundle.putString("key", "value");
fragment.setArguments(bundle);

getSupportFragmentManager().beginTransaction()
    .replace(R.id.container, fragment)
    .commit();
```

In `FragmentX.java`:

```java
Bundle args = getArguments();
if (args != null) {
    String value = args.getString("key");
}
```

## When It’s Useful

✅ **When fragments are created dynamically**, e.g.:

- You show a detail screen based on user selection.
- You switch fragments via navigation or tabs.
- You add fragments one on top of another (stack behavior).

✅ **When fragments are not known in advance (not in XML)**.

✅ **When you want the data to survive configuration changes** (Android automatically saves fragment arguments).

## When It Fails or Not Ideal

|Situation|Why it fails  | Better Options |
|--|--| -- |
|You need two-way real-time updates (like chat or live counter)  | Bundle is one-time — not live |  Shared ViewModel or LiveData  |
|You pass non-serializable objects (like large custom classes)| Use repository/ViewModel | Bundle only supports primitives, Strings, Parcelable|  
|Fragment already exists and you call `setArguments()` again|Throws exception (only valid before attach)| Use fragment methods or ViewModel |

---

Let’s imagine a **MainActivity** with:

- An **EditText** for user input
- **Three buttons** to open Fragment 1, 2, or 3
- **Two control buttons**:
    
    - `Replace Fragment` (replaces current fragment)
    - `Add Fragment` (adds new fragment over the current one)

This gives you full control:

- Choose **which fragment** to show
- Choose whether to **replace** or **add** the new one
- Each fragment shows the **text** entered in the EditText

## Code Implementation

### activity_main.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <EditText
        android:id="@+id/inputField"
        android:hint="Enter text to send"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"/>

    <!-- Fragment Selection Buttons -->
    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <Button
            android:id="@+id/btnFragment1"
            android:text="Fragment 1"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <Button
            android:id="@+id/btnFragment2"
            android:text="Fragment 2"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <Button
            android:id="@+id/btnFragment3"
            android:text="Fragment 3"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
    </LinearLayout>

    <!-- Add vs Replace Control -->
    <LinearLayout
        android:orientation="horizontal"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        <Button
            android:id="@+id/btnReplace"
            android:text="Replace"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>

        <Button
            android:id="@+id/btnAdd"
            android:text="Add Over"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"/>
    </LinearLayout>

    <!-- Container for Fragments -->
    <FrameLayout
        android:id="@+id/fragmentContainer"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:background="#EEEEEE"/>
</LinearLayout>
```

### MainActivity.java

```java
public class MainActivity extends AppCompatActivity {

    EditText inputField;
    Button btnFrag1, btnFrag2, btnFrag3, btnReplace, btnAdd;

    boolean useReplace = true;  // toggle add/replace mode

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        inputField = findViewById(R.id.inputField);
        btnFrag1 = findViewById(R.id.btnFragment1);
        btnFrag2 = findViewById(R.id.btnFragment2);
        btnFrag3 = findViewById(R.id.btnFragment3);
        btnReplace = findViewById(R.id.btnReplace);
        btnAdd = findViewById(R.id.btnAdd);

        btnReplace.setOnClickListener(v -> useReplace = true);
        btnAdd.setOnClickListener(v -> useReplace = false);

        btnFrag1.setOnClickListener(v -> showFragment(new FragmentOne()));
        btnFrag2.setOnClickListener(v -> showFragment(new FragmentTwo()));
        btnFrag3.setOnClickListener(v -> showFragment(new FragmentThree()));
    }

    private void showFragment(Fragment fragment) {
        String text = inputField.getText().toString().trim();

        if (text.isEmpty()) {
            Toast.makeText(this, "Enter some text first!", Toast.LENGTH_SHORT).show();
            return;
        }

        // Attach data to fragment
        Bundle bundle = new Bundle();
        bundle.putString("inputText", text);
        fragment.setArguments(bundle);

        FragmentTransaction ft = getSupportFragmentManager().beginTransaction();

        if (useReplace)
            ft.replace(R.id.fragmentContainer, fragment);
        else
            ft.add(R.id.fragmentContainer, fragment);

        ft.addToBackStack(null); // allow back navigation
        ft.commit();
    }
}
```

### fragment_layout.xml

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="20dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textViewFragment"
        android:textSize="18sp"
        android:textStyle="bold"
        android:textColor="#000"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content" />
</LinearLayout>
```

### FragmentOne.java (similar for others)

```java
public class FragmentOne extends Fragment {

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {

        View view = inflater.inflate(R.layout.fragment_layout, container, false);
        TextView textView = view.findViewById(R.id.textViewFragment);

        Bundle args = getArguments();
        if (args != null) {
            String text = args.getString("inputText");
            textView.setText("Fragment 1 received: " + text);
        }
        return view;
    }
}
```

Repeat for `FragmentTwo` and `FragmentThree`, only changing the label text.

---

## Conceptually What’s Going On

- Each time user presses a fragment button → Activity **creates** a new fragment instance.
- The Activity **packages the input text** inside a **Bundle**.
- The fragment receives it in `getArguments()`.
- The `useReplace` toggle controls whether:
    
    - The old fragment is **replaced** (previous destroyed), or
    - The new one is **added over** it (previous remains underneath, and user can pop back).

# 1. What is a ListView?

A **ListView** is a UI component used to display a **scrollable list of items** (like messages, contacts, settings, etc.).
Each row of the list is typically made from a layout (e.g., a TextView or a custom layout with image + text).

The key idea:
Instead of manually adding multiple `TextView`s or `Buttons`, we use a **ListView** that automatically creates, reuses, and manages views for large data sets efficiently.

## 2. What is an Adapter?

A **bridge** between your data and your ListView.   The adapter converts each item in your data source (array, list, cursor, etc.) into a **View** that ListView can display.

There are different types:

| Adapter | Description | Typical Use |
| -- | -- | -- |
|`ArrayAdapter`| Simplest one | binds arrays/lists of Strings or simple objects  Basic text lists |
|`SimpleAdapter`|Maps data from a list of maps to views|Custom data|
|`CursorAdapter`|Works with SQLite database cursors|Database-backed lists|
|`BaseAdapter`|Fully customizable|Complex custom lists|

## 3. The Connection Between ListView and Adapter

You create your adapter → set it on your ListView:

```java
listView.setAdapter(adapter);
```

Then Android will:

- Ask adapter: “How many items?”
- For each position, call `getView(position)` to create/recycle a row view.
- Display those efficiently with scrolling & recycling.

## 4. Why `ListFragment` instead of `Fragment`?

A **ListFragment** is a special subclass of Fragment **designed specifically for lists**.
It already includes a built-in ListView, so you don’t have to declare it in your XML.
You just set your adapter with:

```java
setListAdapter(adapter);
```

Then you get:

- Automatic click handling with `onListItemClick()`
- Lifecycle already managing the internal ListView
- Less boilerplate code

### Use `ListFragment` when:

- The fragment’s main content **is a list** (like contact list, email inbox, etc.)
- You want quick setup, minimal XML

### Use normal `Fragment` + `ListView` when:

- You need **other views** around your list (e.g., a search bar, buttons, etc.)
- You want **custom layout control**

## 5. Getting position and context

When a user clicks an item:

```java
@Override
public void onListItemClick(ListView l, View v, int position, long id) {
    String selected = (String) l.getItemAtPosition(position);
}
```

- **`position`** gives the index in your dataset.
- **`id`** is usually the database ID if data comes from a DB.
- **`getContext()`** or **`requireContext()`** gives the Activity context.

## 6. Lifecycle relevance

In fragments with lists, you should:

- **Initialize adapter** in `onCreateView()` or `onViewCreated()`
- **Attach listeners** (like item clicks) in `onViewCreated()`
- **Access context** safely — use `requireContext()` instead of `getContext()` if you’re sure fragment is attached.

If you need to communicate back to Activity (like item clicked → show detail), use:

- A callback interface, or
- A shared ViewModel (modern way)

## Chat layout Example

- Left fragment (`PeopleFragment`): shows a list of people.
- Right fragment (`ChatFragment`): shows the chat area.
- When you click a person in the list, data is sent to the activity using an **interface**, and the activity then updates the `ChatFragment`.

## HOW IT WORKS

1. **`PeopleFragment`** uses a **built-in ListView + ArrayAdapter**.
    
    - It defines an interface `OnPersonSelectedListener`.
    - The Activity implements this interface.
    - When a name is clicked → fragment calls the interface method.
2. **`MainActivity`** implements the interface.
    
    - When notified, it finds the `ChatFragment` and tells it which person was selected.
3. **`ChatFragment`** updates its TextView to show that person’s name.

## activity_main.xml

We’ll use a simple horizontal layout for both fragments.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <fragment
        android:id="@+id/peopleFragment"
        android:name="com.example.chatdemo.PeopleFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="1" />

    <fragment
        android:id="@+id/chatFragment"
        android:name="com.example.chatdemo.ChatFragment"
        android:layout_width="0dp"
        android:layout_height="match_parent"
        android:layout_weight="2" />
</LinearLayout>
```

## fragment_people.xml

Note: Built-in ListView uses id **`@android:id/list`**, not a custom one.

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@android:id/list"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />

</LinearLayout>
```

## fragment_chat.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:padding="20dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/chatText"
        android:text="Select a person to chat"
        android:textSize="18sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />

</LinearLayout>
```

## PeopleFragment.java

```java
package com.example.chatdemo;

import android.content.Context;
import android.os.Bundle;
import android.view.View;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import android.widget.Toast;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.ListFragment;

// Using ListFragment simplifies setup — no need to define onCreateView
public class PeopleFragment extends ListFragment {

    // Define interface for communication
    interface OnPersonSelectedListener {
        void onPersonSelected(String name);
    }

    private OnPersonSelectedListener listener;
    private String[] people = {"Ali", "Sara", "Rauf", "John", "Mina"};

    @Override
    public void onAttach(@NonNull Context context) {
        super.onAttach(context);
        // Polymorphism: activity must implement interface
        if (context instanceof OnPersonSelectedListener) {
            listener = (OnPersonSelectedListener) context;
        } else {
            throw new ClassCastException(context.toString() +
                    " must implement OnPersonSelectedListener");
        }
    }

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        // Built-in adapter for simple text list
        ArrayAdapter<String> adapter = new ArrayAdapter<>(
                requireContext(),
                android.R.layout.simple_list_item_1,
                people
        );

        setListAdapter(adapter);
    }

    @Override
    public void onListItemClick(@NonNull ListView l, @NonNull View v, int position, long id) {
        super.onListItemClick(l, v, position, id);
        String selected = (String) l.getItemAtPosition(position);
        Toast.makeText(requireContext(), "Clicked: " + selected, Toast.LENGTH_SHORT).show();

        // Inform the activity via interface callback
        if (listener != null) listener.onPersonSelected(selected);
    }
}
```

## ChatFragment.java

```java
package com.example.chatdemo;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;

import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.Fragment;

public class ChatFragment extends Fragment {

    private TextView chatText;

    @Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater,
                             @Nullable ViewGroup container,
                             @Nullable Bundle savedInstanceState) {
        View view = inflater.inflate(R.layout.fragment_chat, container, false);
        chatText = view.findViewById(R.id.chatText);
        return view;
    }

    // Public method that can be called by activity
    public void updateChat(String name) {
        chatText.setText("Chatting with " + name);
    }
}
```

## MainActivity.java

```java
package com.example.chatdemo;

import android.os.Bundle;

import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.FragmentManager;

public class MainActivity extends AppCompatActivity
        implements PeopleFragment.OnPersonSelectedListener {

    private ChatFragment chatFragment;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Get ChatFragment using FragmentManager
        FragmentManager fm = getSupportFragmentManager();
        chatFragment = (ChatFragment) fm.findFragmentById(R.id.chatFragment);
    }

    @Override
    public void onPersonSelected(String name) {
        // Interface polymorphism in action
        chatFragment.updateChat(name);
    }
	
	
	// another way by getting view of fragment rather than calling public function
	public void onNameSelected(String name) {
        FragmentManager fm = getSupportFragmentManager();

        ChatFragment chatFragment = (ChatFragment) fm.findFragmentById(R.id.fragChat);

        if (chatFragment != null && chatFragment.getView() != null) {
            // Direct access to TextView inside fragment
            TextView tv = chatFragment.getView().findViewById(R.id.tv_chatInfo);
            tv.setText("Chatting with: " + name);
        }
    }
}
```

## FLOW SUMMARY

1. `PeopleFragment` (ListFragment) inflates automatically with its built-in ListView (`@android:id/list`).
2. `onAttach()` runs → attaches interface listener to Activity.
3. `onActivityCreated()` sets adapter for the list.
4. User clicks an item → `onListItemClick()` is triggered.
5. `listener.onPersonSelected(name)` calls the method in `MainActivity`.
6. `MainActivity` then calls `chatFragment.updateChat(name)` to change text.
7. UI updates.

## VIEW ACCESS IN FRAGMENTS

> When a fragment has its own XML, you always access views **via the inflated `View`**, not with `findViewById()` directly on the activity.

Example:

```java
View view = inflater.inflate(R.layout.fragment_chat, container, false);
TextView tv = view.findViewById(R.id.chatText);
```

If you use `getView()` later (after onCreateView), then:

```java
TextView tv = getView().findViewById(R.id.chatText);
```

**Never use** `getActivity().findViewById()` unless you want a view from the **activity’s layout**, not the fragment’s.

---

## TEACHER AND STUDENTS INFO WITH MULTIPLE VIEW LIST

### 1️⃣ Why each fragment needs its own adapter

Each fragment has its **own lifecycle**, **layout**, and **data source**.
So when you use a `ListView` in both fragments (say, Students and Teachers), each fragment:

- manages its own `ListView`
- sets its own adapter
- handles its own `onListItemClick()` or `OnItemClickListener`.

That’s why we must create **two adapter instances** — even if they’re both simple `ArrayAdapter<String>` types.

### 2️⃣ How fragments talk to each other

Fragments **can’t directly talk to each other** — they must go **through the Activity** that hosts them.

So, when an item in one fragment is clicked:

- That fragment calls a **method of an interface**, implemented by the Activity.
- The Activity then decides what to do (e.g., show another fragment or update its view).

### 3️⃣ Two ways to update another fragment’s UI

|Approach|How It Works|When to Use|
|--|--|--|
|**A: Directly access fragment’s view**|Activity finds the fragment using `getSupportFragmentManager().findFragmentById()` and then `findViewById()` on it.|When the fragment’s layout is simple (e.g., just a TextView) and you only need to set text or color.
|**B: Call a function of the target fragment**|Activity finds the fragment, then calls a method (like `updateInfo(String text)`) inside it.|When the update is more complex (e.g., involves logic or multiple UI components).

We’ll use **both** methods in this example — one for the Students list, and one for the Teachers list.

## 🏗 SCENARIO OVERVIEW

- **FragmentA** → Students list
- **FragmentB** → Teachers list
- **FragmentInfo** → shows selected person’s info in full screen
- **MainActivity** → acts as communication bridge

We’ll use:

- **Built-in ListFragment**
- **Built-in ArrayAdapter**
- **Interface callbacks**
- **Fragment Transactions** to show the info fragment dynamically.

## 📂 FILES

### activity_main.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">

    <!-- Top part: Two fragments side by side -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:orientation="horizontal">

        <fragment
            android:id="@+id/fragStudents"
            android:name="com.example.school.FragmentStudents"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />

        <fragment
            android:id="@+id/fragTeachers"
            android:name="com.example.school.FragmentTeachers"
            android:layout_width="0dp"
            android:layout_height="match_parent"
            android:layout_weight="1" />
    </LinearLayout>

    <!-- Bottom hidden fragment for info (appears when clicked) -->
    <FrameLayout
        android:id="@+id/fragInfoContainer"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1" />
</LinearLayout>
```

### FragmentStudents.java

```java
package com.example.school;

import android.content.Context;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import androidx.fragment.app.ListFragment;

public class FragmentStudents extends ListFragment {

    // Interface reference
    OnPersonSelected listener;

    public interface OnPersonSelected {
        void onStudentSelected(String name);
    }

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        if (context instanceof OnPersonSelected)
            listener = (OnPersonSelected) context;
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        String[] students = {"Ali", "Sara", "Hina", "Zain"};
        ArrayAdapter<String> adapter = new ArrayAdapter<>(
                getActivity(),
                android.R.layout.simple_list_item_1,
                students
        );
        setListAdapter(adapter);
    }

    @Override
    public void onListItemClick(ListView l, android.view.View v, int position, long id) {
        String name = (String) getListView().getItemAtPosition(position);
        if (listener != null)
            listener.onStudentSelected(name);
    }
}
```

### FragmentTeachers.java

```java
package com.example.school;

import android.content.Context;
import android.os.Bundle;
import android.widget.ArrayAdapter;
import android.widget.ListView;
import androidx.fragment.app.ListFragment;

public class FragmentTeachers extends ListFragment {

    OnPersonSelected listener;

    public interface OnPersonSelected {
        void onTeacherSelected(String name);
    }

    @Override
    public void onAttach(Context context) {
        super.onAttach(context);
        if (context instanceof OnPersonSelected)
            listener = (OnPersonSelected) context;
    }

    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        String[] teachers = {"Mr. Khan", "Ms. Fatima", "Sir Aslam"};
        ArrayAdapter<String> adapter = new ArrayAdapter<>(
                getActivity(),
                android.R.layout.simple_list_item_1,
                teachers
        );
        setListAdapter(adapter);
    }

    @Override
    public void onListItemClick(ListView l, android.view.View v, int position, long id) {
        String name = (String) getListView().getItemAtPosition(position);
        if (listener != null)
            listener.onTeacherSelected(name);
    }
}
```

### FragmentInfo.java

```java
package com.example.school;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.TextView;
import androidx.fragment.app.Fragment;

public class FragmentInfo extends Fragment {

    TextView tvInfo;

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container,
                             Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.fragment_info, container, false);
        tvInfo = v.findViewById(R.id.tv_info);
        return v;
    }

    // Method used when activity calls directly (Approach B)
    public void updateInfo(String text) {
        if (tvInfo != null)
            tvInfo.setText(text);
    }
}
```

### fragment_info.xml

```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/tv_info"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:textSize="22sp"
        android:text="Select a person" />
</FrameLayout>
```

### MainActivity.java

```java
package com.example.school;

import androidx.appcompat.app.AppCompatActivity;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentTransaction;
import android.os.Bundle;
import android.widget.TextView;

public class MainActivity extends AppCompatActivity
        implements FragmentStudents.OnPersonSelected,
                   FragmentTeachers.OnPersonSelected {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    // When a student is clicked → show info via FUNCTION CALL (Approach B)
    @Override
    public void onStudentSelected(String name) {
        FragmentInfo infoFrag = new FragmentInfo();
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.fragInfoContainer, infoFrag)
                .addToBackStack(null)
                .commit();

        // Delay until fragment attached
        getSupportFragmentManager().executePendingTransactions();

        infoFrag.updateInfo("Student selected: " + name);
    }

    // When a teacher is clicked → show info via DIRECT VIEW ACCESS (Approach A)
    @Override
    public void onTeacherSelected(String name) {
        FragmentInfo infoFrag = new FragmentInfo();
        getSupportFragmentManager().beginTransaction()
                .replace(R.id.fragInfoContainer, infoFrag)
                .addToBackStack(null)
                .commit();

        getSupportFragmentManager().executePendingTransactions();

        // Access TextView directly from fragment’s view
        TextView tv = infoFrag.getView().findViewById(R.id.tv_info);
        tv.setText("Teacher selected: " + name);
    }
}
```

## 🔄 FLOW SUMMARY

1️⃣ User clicks a **student name** → `FragmentStudents` → calls `listener.onStudentSelected()`.
2️⃣ `MainActivity` receives that callback.
3️⃣ It loads `FragmentInfo` and **calls `updateInfo()`** to set text.
4️⃣ User clicks a **teacher name** → `FragmentTeachers` → calls `listener.onTeacherSelected()`.
5️⃣ `MainActivity` again loads `FragmentInfo` but now **accesses its TextView directly**.

Both end up showing info, but via **different communication styles**.

## ⚙️ When to Use Each

|Style|Good for|Avoid when|
|--|--|--|
|**Direct view access**|Quick UI updates (e.g., one TextView)|Fragment may not be fully created → can cause `NullPointerException|
|**Function call**|Safer and cleaner for logic/UI both|Requires exposing public methods|

# 1️⃣ Why Do We Need a Custom ListView?

The **built-in adapters** (`ArrayAdapter`, `SimpleAdapter`, `SimpleCursorAdapter`) are **limited**.
They work well only when:

- Each list item has a **single text line** (or maybe text + small image).
- The data is simple, like an array of strings.

But in most real apps (chat lists, contact lists, news feeds, etc.), each row needs:

- Multiple text fields (e.g., name + message + time)
- Images (e.g., profile picture)
- Buttons, CheckBoxes, or progress indicators
- Custom fonts, colors, alignment, etc.

In those cases, we make our **own layout for each row** and our **own Adapter class** to populate it.

## 2️⃣ How Is It Different From Built-in?

|Feature|Built-in `ArrayAdapter`|Custom Adapter|
|--|--|--|
|**Layout per row**|Predefined (`simple_list_item_1`, etc.)|Your own XML layout (`custom_row.xml`)
|**Data Source**|Usually just `List` | Can be list of objects (`List`)|
|**Binding Data**|Automatic (adapter handles it)|You manually set values to views
|**Performance**|Good for simple lists|You must handle optimization manually (like ViewHolder pattern)|
|**Flexibility**|Very limited|Fully customizable layout, fonts, colors, and logic

## 3️⃣ What Must You Create Yourself in a Custom ListView Setup

Here are the **new components** you must build manually:

|Component|Purpose|
|--|--|
|**Custom model class (POJO)**|Represents one row’s data (e.g., `Student` object).
|**Custom layout XML for list row**|Defines how one row looks (e.g., image + 2 text lines).
|**Custom Adapter class (extends BaseAdapter or ArrayAdapter)**|Binds model data → custom row layout views.|
|**ViewHolder pattern**|(Optional but important) For performance — reuses old views.|
|**ListView in fragment/activity**|Same as before — but you attach your custom adapter instead of built-in one.

## 4️⃣ Common Adapter Base Classes You Can Extend

1. **`ArrayAdapter<T>`** → easiest when your data is a list of objects.
    You can override `getView()` to control layout.
2. **`BaseAdapter`** → most flexible, used when you need full control or data isn’t a simple list.
3. **`RecyclerView.Adapter`** → modern, advanced replacement for ListView (we’ll come to it later).

## 5️⃣ Structure Comparison (Built-in vs Custom)

### Built-in (simple)

```java
ArrayAdapter<String> adapter = new ArrayAdapter<>(
    this,
    android.R.layout.simple_list_item_1,
    namesArray
);
listView.setAdapter(adapter);
```

✅ No extra layout, adapter, or model class.

### Custom (complex)

1. Create a **model class:**

```java
class Student {
    String name;
    String roll;
    int imageRes;
    // constructor + getters
}
```

2. Create **custom_row.xml**:

```xml
<LinearLayout ...>
    <ImageView android:id="@+id/img_profile" .../>
    <TextView android:id="@+id/tv_name" .../>
    <TextView android:id="@+id/tv_roll" .../>
</LinearLayout>
```

3. Create a **CustomAdapter.java**:

```java
class CustomAdapter extends ArrayAdapter<Student> {
    ...
    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        // Inflate custom_row.xml
        // Bind Student data to each View
    }
}
```

4. Attach adapter:

```java
listView.setAdapter(new CustomAdapter(this, studentList));
```

## 6️⃣ Important Concept: `getView()` and View Recycling

Every row in a ListView is drawn using the adapter’s `getView()` method.

```java
@Override
public View getView(int position, View convertView, ViewGroup parent)
```

- **`position`** → index of current item
- **`convertView`** → old (reusable) view; if `null`, you must inflate new one
- **`parent`** → the ListView itself

This mechanism prevents creating a new view for every scroll, improving speed.

✅ **ViewHolder pattern** stores references to views inside a row so they can be reused efficiently.

---

## ---- Key Lifecycle Hooks in Fragments

If ListView is inside a Fragment:

- Inflate layout in `onCreateView()`
- Setup adapter in `onViewCreated()`
- Use `getActivity()` or `requireContext()` for context
- Clean up heavy references in `onDestroyView()`

## 1️⃣ CUSTOMIZE**D** PASSENGER INFO

- A **Passenger class** → holds data for one passenger (name, phone, preference, etc.)
- An **Application class** → global holder for all passenger data (so all fragments can access it)
- A **List Fragment (Fragment A)** → shows a list of passengers with their name + preference icon
- A **Details Fragment (Fragment B)** → shows full info when one passenger is clicked
- A **Custom Adapter** → handles custom layout, binds `Passenger` data to views

## 2️⃣ HOW DATA FLOWS

```
AppData (Application)
     ↓
PassengerListFragment  →  CustomAdapter  →  custom_row.xml
     ↓ (on item click)
PassengerDetailFragment (shows info of selected passenger)
```

## 3️⃣ WHY `listView.setAdapter()` IS USED

Because the `ListView` object needs to **know which adapter will supply data**.
You can’t just call `setAdapter()` _without specifying an object_ — `listView` is an instance of a `ListView` class, and you call its method to set the adapter:

```java
listView.setAdapter(adapter);
```

Here:

- `listView` = your specific ListView in the layout.
- `setAdapter()` = ListView’s method.
- `adapter` = your custom adapter instance.

If you call just `setAdapter()` alone, it doesn’t know which ListView to apply to — that’s why you always use `listView.setAdapter(adapter)`. else implment to list fragment rather than just fragment

---

## 4️⃣ CODE STRUCTURE

We’ll build 6 files:

1. `Passenger.java` – model class
2. `AppData.java` – Application class (global data)
3. `fragment_passenger_list.xml` – layout with ListView
4. `row_passenger.xml` – layout for each row in list
5. `PassengerAdapter.java` – custom adapter class
6. `PassengerListFragment.java` – fragment with list
7. `PassengerDetailFragment.java` – fragment with details
8. `MainActivity.java` – hosts both fragments

### 1. `Passenger.java`

```java
public class Passenger {
    private String name;
    private String phone;
    private String preference; // "Bus" or "Plane"

    public Passenger(String name, String phone, String preference) {
        this.name = name;
        this.phone = phone;
        this.preference = preference;
    }

    public String getName() { return name; }
    public String getPhone() { return phone; }
    public String getPreference() { return preference; }
}
```

### 2. `AppData.java`

```java
import android.app.Application;
import java.util.ArrayList;

public class AppData extends Application {
    private static ArrayList<Passenger> passengers = new ArrayList<>();

    @Override
    public void onCreate() {
        super.onCreate();
        passengers.add(new Passenger("Alice", "1234567890", "Bus"));
        passengers.add(new Passenger("Bob", "9876543210", "Plane"));
        passengers.add(new Passenger("Charlie", "1122334455", "Bus"));
    }

    public static ArrayList<Passenger> getPassengers() {
        return passengers;
    }
}
```

> Remember to declare this Application class in your `AndroidManifest.xml`
> `<application android:name=".AppData" ... >`

### 3. `row_passenger.xml`

Custom layout for each row.

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal"
    android:padding="8dp"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <TextView
        android:id="@+id/tvName"
        android:textSize="18sp"
        android:layout_weight="1"
        android:layout_width="0dp"
        android:layout_height="wrap_content"/>

    <ImageView
        android:id="@+id/imgPreference"
        android:layout_width="40dp"
        android:layout_height="40dp"
        android:contentDescription="Preference icon" />
</LinearLayout>
```

### 4. `PassengerAdapter.java`

```java
import android.content.Context;
import android.view.*;
import android.widget.*;
import java.util.List;

public class PassengerAdapter extends ArrayAdapter<Passenger> {
    private LayoutInflater inflater;

    public PassengerAdapter(Context context, List<Passenger> passengers) {
        super(context, 0, passengers);
        inflater = LayoutInflater.from(context);
    }

    @Override
    public View getView(int position, View convertView, ViewGroup parent) {
        // Reuse view if possible
        if (convertView == null) {
            convertView = inflater.inflate(R.layout.row_passenger, parent, false);
        }

        Passenger p = getItem(position);
        TextView tvName = convertView.findViewById(R.id.tvName);
        ImageView imgPref = convertView.findViewById(R.id.imgPreference);

        tvName.setText(p.getName());

        if (p.getPreference().equalsIgnoreCase("Bus")) {
            imgPref.setImageResource(R.drawable.ic_bus);
        } else {
            imgPref.setImageResource(R.drawable.ic_plane);
        }

        return convertView;
    }
}
```

### 5. `fragment_passenger_list.xml`

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <ListView
        android:id="@+id/listPassengers"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</FrameLayout>
```

### 6. `PassengerListFragment.java`

##### --- Extend to List Fragment

```java
import android.os.Bundle;
import android.widget.ListView;
import androidx.annotation.NonNull;
import androidx.annotation.Nullable;
import androidx.fragment.app.ListFragment;
import androidx.fragment.app.FragmentTransaction;

public class PassengerListFragment extends ListFragment {

    @Override
    public void onActivityCreated(@Nullable Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);

        PassengerAdapter adapter = new PassengerAdapter(requireContext(), AppData.getPassengers());
        setListAdapter(adapter); // no need to find ListView manually
    }

    @Override
    public void onListItemClick(@NonNull ListView l, @NonNull android.view.View v, int position, long id) {
        super.onListItemClick(l, v, position, id);

        Passenger selected = AppData.getPassengers().get(position);

        Bundle bundle = new Bundle();
        bundle.putString("name", selected.getName());
        bundle.putString("phone", selected.getPhone());
        bundle.putString("preference", selected.getPreference());

        PassengerDetailFragment frag = new PassengerDetailFragment();
        frag.setArguments(bundle);

        FragmentTransaction ft = requireActivity().getSupportFragmentManager().beginTransaction();
        ft.replace(R.id.container, frag);
        ft.addToBackStack(null);
        ft.commit();
    }
}
```

---

##### --- Extend to Fragment

```java
import android.os.Bundle;
import android.view.*;
import android.widget.*;
import androidx.fragment.app.Fragment;
import androidx.fragment.app.FragmentTransaction;

public class PassengerListFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View v = inflater.inflate(R.layout.fragment_passenger_list, container, false);
        ListView listView = v.findViewById(R.id.listPassengers);

        PassengerAdapter adapter = new PassengerAdapter(requireContext(), AppData.getPassengers());
        listView.setAdapter(adapter);

        listView.setOnItemClickListener((parent, view, position, id) -> {
            Passenger selected = AppData.getPassengers().get(position);

            Bundle bundle = new Bundle();
            bundle.putString("name", selected.getName());
            bundle.putString("phone", selected.getPhone());
            bundle.putString("preference", selected.getPreference());

            PassengerDetailFragment frag = new PassengerDetailFragment();
            frag.setArguments(bundle);

            FragmentTransaction ft = requireActivity().getSupportFragmentManager().beginTransaction();
            ft.replace(R.id.container, frag);
            ft.addToBackStack(null);
            ft.commit();
        });

        return v;
    }
}
```

### 7. `PassengerDetailFragment.java`

```java
public class PassengerDetailFragment extends Fragment {

    @Override
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        // Only inflate and return
        return inflater.inflate(R.layout.fragment_passenger_detail, container, false);
    }

    @Override
    public void onViewCreated(@NonNull View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);

        TextView tvDetails = view.findViewById(R.id.tvDetails);
        Bundle args = getArguments();

        if (args != null) {
            String name = args.getString("name");
            String phone = args.getString("phone");
            String pref = args.getString("preference");

            tvDetails.setText("Name: " + name + "\nPhone: " + phone + "\nPreference: " + pref);
        }
    }
}
```

### 8. `fragment_passenger_detail.xml`

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:padding="16dp"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/tvDetails"
        android:textSize="18sp"
        android:layout_width="match_parent"
        android:layout_height="wrap_content" />
</FrameLayout>
```

### 9. `MainActivity.java`

```java
import android.os.Bundle;
import androidx.appcompat.app.AppCompatActivity;

public class MainActivity extends AppCompatActivity {
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        if (savedInstanceState == null) {
            getSupportFragmentManager()
                .beginTransaction()
                .add(R.id.container, new PassengerListFragment())
                .commit();
        }
    }
}
```

### 10. `activity_main.xml`

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" />
```

## 5️⃣ FLOW SUMMARY

1. App starts → `MainActivity` loads `PassengerListFragment`.
2. `PassengerListFragment` → gets data from `AppData` → populates ListView using `PassengerAdapter`.
3. User clicks on an item → passes passenger info via `Bundle` → opens `PassengerDetailFragment`.
4. `PassengerDetailFragment` → reads bundle → displays full info.

-----
| View Type                | Listener                       | Method                                                                 | Important Params                      | Meaning                        |
| ------------------------ | ------------------------------ | ---------------------------------------------------------------------- | ------------------------------------- | ------------------------------ |
| **Button / ImageButton** | `setOnClickListener()`         | `onClick(View v)`                                                      | `v` = the clicked view                | Triggered when clicked         |
| **ImageView**            | `setOnClickListener()`         | `onClick(View v)`                                                      | same                                  | Used for clickable images      |
| **EditText**             | `setOnFocusChangeListener()`   | `onFocusChange(View v, boolean hasFocus)`                              | `hasFocus` = gained/lost focus        | Detect when editing ends       |
| **CheckBox / Switch**    | `setOnCheckedChangeListener()` | `onCheckedChanged(CompoundButton buttonView, boolean isChecked)`       | `isChecked` = current state           | Detect selection toggle        |
| **RadioGroup**           | `setOnCheckedChangeListener()` | `onCheckedChanged(RadioGroup group, int checkedId)`                    | `checkedId` = selected RadioButton id | Detect gender/option selection |
| **SeekBar**              | `setOnSeekBarChangeListener()` | `onProgressChanged(SeekBar s, int progress, boolean fromUser)`         | progress = new value                  | Detect slider change           |
| **ListView**             | `setOnItemClickListener()`     | `onItemClick(AdapterView<?> parent, View view, int position, long id)` | position = clicked item               | Detect which row was clicked   |
| **Spinner**              | `setOnItemSelectedListener()`  | `onItemSelected(AdapterView<?> parent, View view, int pos, long id)`   | pos = selected index                  | Detect selected item           |
| **View (general)**       | `setOnLongClickListener()`     | `onLongClick(View v)`                                                  | v = clicked view                      | For long-press actions         |
