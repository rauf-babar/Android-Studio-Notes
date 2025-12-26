# Firebase Realtime Database

## 1. What is Firebase?

Firebase is a Backend-as-a-Service (BaaS) by Google. It provides a cloud-hosted infrastructure for developers to build apps without managing servers. It includes tools for authentication, databases, file storage, and hosting.

## 2. Realtime Database vs. Firestore

* **Realtime Database:** An older, legacy NoSQL database that stores data as one giant **JSON Tree**. It is optimized for low-latency synchronization (e.g., chat apps).
* **Cloud Firestore:** A newer NoSQL database that stores data in **Documents and Collections**. It is better for complex queries, high scalability, and structured data.

We will cover **Realtime Database** for now
---

## 3. Firebase Authentication 

### Create User (Signup)

Used to register a new user using email and password.

```java
mAuth.createUserWithEmailAndPassword(email, password)
    .addOnSuccessListener(new OnSuccessListener<AuthResult>() {
        @Override
        public void onSuccess(AuthResult authResult) {
            // User created and logged in automatically
        }
    })
    .addOnFailureListener(new OnFailureListener() {
        @Override
        public void onFailure(@NonNull Exception e) {
            // Signup failed
        }
    });


mAuth.createUserWithEmailAndPassword(email, password)
    .addOnSuccessListener(authResult -> {
        // User created successfully
    })
    .addOnFailureListener(e -> {
        // Signup failed
    });

```

### Login (Sign In)

Used to authenticate an existing user.

```java
mAuth.signInWithEmailAndPassword(email, password)
    .addOnSuccessListener(new OnSuccessListener<AuthResult>() {
        @Override
        public void onSuccess(AuthResult authResult) {
            // Login successful
        }
    })
    .addOnFailureListener(new OnFailureListener() {
        @Override
        public void onFailure(@NonNull Exception e) {
            // Login failed
        }
    });

```

### Password Reset (Forgot Password)

Sends an automated email to the user to reset their password via a secure link.

```java
mAuth.sendPasswordResetEmail(email)
    .addOnSuccessListener(new OnSuccessListener<Void>() {
        @Override
        public void onSuccess(Void unused) {
            // Email sent
        }
    })
    .addOnFailureListener(new OnFailureListener() {
        @Override
        public void onFailure(@NonNull Exception e) {
            // Error sending email
        }
    });

```

### Logout (Session End)

Only clears the local session on the device. The account remains in Firebase.

```java
mAuth.signOut();

```

### Signout (Delete User Account)

Permanently deletes the currently logged-in user's record from Firebase.

```java
mAuth.getCurrentUser().delete()
    .addOnSuccessListener(new OnSuccessListener<Void>() {
        @Override
        public void onSuccess(Void unused) {
            // Account deleted
        }
    })
    .addOnFailureListener(new OnFailureListener() {
        @Override
        public void onFailure(@NonNull Exception e) {
            // Error deleting account
        }
    });

```

---

## 4. Implementation Example

### MainActivity.java

```java
public class MainActivity extends AppCompatActivity {
    EditText etEmail, etPass;
    Button btnLogin, btnSignup, btnReset;
    FirebaseAuth mAuth;
    FirebaseUser user;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        mAuth = FirebaseAuth.getInstance();
        user = mAuth.getCurrentUser();

        // Session check: If user exists, go to Profile
        if (user != null) {
            startActivity(new Intent(this, ProfileActivity.class));
            finish();
        }

        btnLogin.setOnClickListener(v -> {
            mAuth.signInWithEmailAndPassword(etEmail.getText().toString(), etPass.getText().toString())
                .addOnSuccessListener(authResult -> {
                    startActivity(new Intent(MainActivity.this, ProfileActivity.class));
                })
                .addOnFailureListener(e -> Toast.makeText(this, e.getMessage(), Toast.LENGTH_SHORT).show());
        });

        btnSignup.setOnClickListener(v -> {
            mAuth.createUserWithEmailAndPassword(etEmail.getText().toString(), etPass.getText().toString())
                .addOnSuccessListener(authResult -> {
                    startActivity(new Intent(MainActivity.this, ProfileActivity.class));
                })
                .addOnFailureListener(e -> Toast.makeText(this, e.getMessage(), Toast.LENGTH_SHORT).show());
        });

        btnReset.setOnClickListener(v -> {
            mAuth.sendPasswordResetEmail(etEmail.getText().toString())
                .addOnSuccessListener(unused -> Toast.makeText(this, "Check Email", Toast.LENGTH_SHORT).show());
        });
    }
}

```

### ProfileActivity.java

```java
public class ProfileActivity extends AppCompatActivity {
    Button btnLogout, btnDelete;
    FirebaseAuth mAuth;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_profile);

        mAuth = FirebaseAuth.getInstance();

        btnLogout.setOnClickListener(v -> {
            mAuth.signOut();
            startActivity(new Intent(this, MainActivity.class));
            finish();
        });

        btnDelete.setOnClickListener(v -> {
            mAuth.getCurrentUser().delete().addOnSuccessListener(unused -> {
                startActivity(new Intent(this, MainActivity.class));
                finish();
            });
        });
    }
}

```

---

## 5. Professional Auth Repository

In professional development, logic is separated into a "Helper" or "Repository" class to keep activities clean.

### AuthRepository.java

```java
public class AuthRepository {
    private FirebaseAuth mAuth;

    public AuthRepository() {
        this.mAuth = FirebaseAuth.getInstance();
    }

    public void login(String email, String pass, OnSuccessListener<AuthResult> success, OnFailureListener failure) {
        mAuth.signInWithEmailAndPassword(email, pass).addOnSuccessListener(success).addOnFailureListener(failure);
    }

    public void signup(String email, String pass, OnSuccessListener<AuthResult> success, OnFailureListener failure) {
        mAuth.createUserWithEmailAndPassword(email, pass).addOnSuccessListener(success).addOnFailureListener(failure);
    }

    public void resetPassword(String email, OnSuccessListener<Void> success, OnFailureListener failure) {
        mAuth.sendPasswordResetEmail(email).addOnSuccessListener(success).addOnFailureListener(failure);
    }

    public void logout() {
        mAuth.signOut();
    }

    public void deleteAccount(OnSuccessListener<Void> success, OnFailureListener failure) {
        if (mAuth.getCurrentUser() != null) {
            mAuth.getCurrentUser().delete().addOnSuccessListener(success).addOnFailureListener(failure);
        }
    }
}

```

### How to use in Activity/Fragment

1. Initialize the repo: `AuthRepository repo = new AuthRepository();`
2. Call the function and pass the listeners:

```java
repo.login(email, pass, 
    authResult -> { /* move to dashboard */ }, 
    e -> { /* show error */ }
);

```

-----

# Firebase Realtime Database & RecyclerView Mastery

## Part 1: Core Concepts & Data Structure

### 1. `push()` vs `setValue()`

In Firebase, every data point needs a "Path".

**`setValue()`**: You define the path (Key).
* *Usage:* When you have a unique ID (like a Student Roll Number or User UID).
* *Risk:* If the ID exists, it overwrites the data.


**`push()`**: Firebase defines the path.
* *Usage:* When you are adding items to a list (like Chat messages or multiple Courses) and don't care about the specific ID.
* *Mechanism:* It generates a unique, timestamp-based ID (e.g., `-Nxyz123...`).



### 2. The Truth About `.getKey()`

When you write:

```java
String key = databaseRef.push().getKey();

```

* **Does it hit the server?** **No.**
* **How it works:** This generation happens locally on your phone instantly. It is mathematically guaranteed to be unique. This allows your app to "create" data even while offline, syncing it later.

### 3. Handling Relationships (NoSQL Style)

Firebase is **not** a Relational Database (SQL). You do not have "Foreign Keys" or "Joins".

* **Scenario:** Linking a **Teacher** to a **Course**.
* **Strategy:** Store the *ID* of the related entity.
* **Bad:** Storing the entire Teacher object inside the Course node (Data duplication).
* **Good:** Storing `teacherId` inside the Course node.
* *Fetching:* When you load the Course, you read the `teacherId`, then make a second quick call to the "Teachers" node to get that teacher's name.



---

## Part 2: The CRUD Operations Guide

*All examples assume `ref` points to `FirebaseDatabase.getInstance().getReference("Courses")`.*

### 1. CREATE (Insert Data)

**A. Adding a Whole Object (e.g., New Course)**

```java
String id = ref.push().getKey(); // 1. Generate ID locally
Course newCourse = new Course(id, "Android Dev", "CS-502");

ref.child(id).setValue(newCourse) // 2. Send to Cloud
    .addOnSuccessListener(unused -> Log.d("FB", "Success"))
    .addOnFailureListener(e -> Log.e("FB", "Failed: " + e.getMessage()));

```

**B. Adding a Single Field (e.g., Adding 'isActive' flag to existing course)**

```java
ref.child(courseId).child("isActive").setValue(true);

```

### 2. READ (Fetch Data)

**A. Realtime Stream (ValueEventListener)**
*Used for keeping UI in sync. Triggers every time data changes.*

```java
ref.child(courseId).addValueEventListener(new ValueEventListener() {
    @Override
    public void onDataChange(DataSnapshot snapshot) {
        if (snapshot.exists()) {
            Course c = snapshot.getValue(Course.class);
            // Update UI
        }
    }
    @Override
    public void onCancelled(DatabaseError error) { }
});

```

**B. One-Time Fetch (get)**
*Used for checking a value once (e.g., during login or calculation).*

```java
ref.child(courseId).get().addOnCompleteListener(task -> {
    if (task.isSuccessful()) {
        Course c = task.getResult().getValue(Course.class);
    }
});

```

### 3. UPDATE (Modify Data)

**A. Full Overwrite (`setValue`)**
*Warning: This replaces the entire node. If you `setValue` on a course but forget to include the `courseCode`, the code will be deleted.*

**B. Specific Field Update (`updateChildren`) - The Professional Way**
*Merges new data with existing data.*

```java
HashMap<String, Object> updates = new HashMap<>();
updates.put("courseName", "Advanced Android"); // Change Name
updates.put("creditHours", 4);                 // Change Credits

ref.child(courseId).updateChildren(updates)
    .addOnSuccessListener(unused -> Log.d("FB", "Updated"))
    .addOnFailureListener(e -> Log.e("FB", "Error: " + e.getMessage()));

```

### 4. DELETE (Remove Data)

**A. Delete Whole Object**

```java
ref.child(courseId).removeValue()
    .addOnSuccessListener(unused -> Log.d("FB", "Deleted"));

```

**B. Delete Single Field**
*Setting a value to null deletes it.*

```java
ref.child(courseId).child("courseName").removeValue();

ref.child(courseId).child("courseName").setValue(null);

```

---

## Part 3: FirebaseRecyclerAdapter Explained

### Why use it over Standard RecyclerView?

1. **Lifecycle Aware:** It stops listening when the activity stops (saving battery/data).
2. **Real-Time Sync:** It handles the `ChildEventListener` logic internally. If User A adds a course, User B sees it appear instantly without "Pull to Refresh."
3. **Code Reduction:** Removes the need for a local `ArrayList` and manual `notifyDataSetChanged()` calls.

### Key Components

1. **Query:** Defines *what* data to fetch (e.g., `ref.child("Courses")` or `ref.orderByChild("name")`).
2. **FirebaseRecyclerOptions:** The configuration object that binds the Query to your Model Class (`Course.class`).

---

## Part 4: Complete Implementation Example

*Scenario: A "Course Manager" app where you can view lists, add new courses, delete them, and edit their names.*

### 1. Model Class (`Course.java`)

```java
public class Course {
    private String id;
    private String name;
    private String code;

    // 1. Empty Constructor (MANDATORY for Firebase)
    public Course() {}

    // 2. Main Constructor
    public Course(String id, String name, String code) {
        this.id = id;
        this.name = name;
        this.code = code;
    }

    // 3. Getters and Setters
    public String getId() { return id; }
    public void setId(String id) { this.id = id; }
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    public String getCode() { return code; }
    public void setCode(String code) { this.code = code; }
}

```

### 2. XML Row Layout (`item_course.xml`)

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="16dp"
    android:gravity="center_vertical">

    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:orientation="vertical">

        <TextView android:id="@+id/tvName"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="18sp"
            android:textStyle="bold"/>

        <TextView android:id="@+id/tvCode"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textSize="14sp"
            android:textColor="#757575"/>
    </LinearLayout>

    <ImageView android:id="@+id/btnEdit"
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:src="@android:drawable/ic_menu_edit"
        android:layout_marginEnd="10dp"/>

    <ImageView android:id="@+id/btnDelete"
        android:layout_width="30dp"
        android:layout_height="30dp"
        android:src="@android:drawable/ic_menu_delete"/>
</LinearLayout>

```

### 3. The Adapter (`CourseAdapter.java`)

- NO GETITEMCOUNT() FUNCTION

```java
public class CourseAdapter extends FirebaseRecyclerAdapter<Course, CourseAdapter.CourseViewHolder> {
    
    Context context;

    public CourseAdapter(@NonNull FirebaseRecyclerOptions<Course> options, Context context) {
        super(options);
        this.context = context;
    }

    @Override
    protected void onBindViewHolder(@NonNull CourseViewHolder holder, int position, @NonNull Course model) {
        // 1. Set Data
        holder.tvName.setText(model.getName());
        holder.tvCode.setText(model.getCode());

        // 2. Handle Delete
        holder.btnDelete.setOnClickListener(v -> {
            // "getRef(position)" gives the DatabaseReference to this specific node
            getRef(position).removeValue()
                .addOnFailureListener(e -> Toast.makeText(context, "Delete Failed", Toast.LENGTH_SHORT).show());
        });

        // 3. Handle Update (Opens Dialog)
        holder.btnEdit.setOnClickListener(v -> showUpdateDialog(model));
    }

    private void showUpdateDialog(Course model) {
        EditText etName = new EditText(context);
        etName.setText(model.getName());

        new AlertDialog.Builder(context)
            .setTitle("Update Course Name")
            .setView(etName)
            .setPositiveButton("Update", (dialog, which) -> {
                String newName = etName.getText().toString();
                
                // Using updateChildren for specific field update
                HashMap<String, Object> map = new HashMap<>();
                map.put("name", newName);

                FirebaseDatabase.getInstance().getReference("Courses")
                    .child(model.getId())
                    .updateChildren(map)
                    .addOnFailureListener(e -> Toast.makeText(context, e.getMessage(), Toast.LENGTH_SHORT).show());
            })
            .setNegativeButton("Cancel", null)
            .show();
    }

    @NonNull
    @Override
    public CourseViewHolder onCreateViewHolder(@NonNull ViewGroup parent, int viewType) {
        // Standard Inflation
        View view = LayoutInflater.from(parent.getContext()).inflate(R.layout.item_course, parent, false);
        return new CourseViewHolder(view);
    }

    class CourseViewHolder extends RecyclerView.ViewHolder {
        TextView tvName, tvCode;
        ImageView btnDelete, btnEdit;

        public CourseViewHolder(@NonNull View itemView) {
            super(itemView);
            tvName = itemView.findViewById(R.id.tvName);
            tvCode = itemView.findViewById(R.id.tvCode);
            btnDelete = itemView.findViewById(R.id.btnDelete);
            btnEdit = itemView.findViewById(R.id.btnEdit);
        }
    }
}

```

### 4. The Main Activity (`MainActivity.java`)

```java
public class MainActivity extends AppCompatActivity {

    RecyclerView recyclerView;
    CourseAdapter adapter;
    DatabaseReference baseRef;
    FloatingActionButton fabAdd;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // 1. Initialize Firebase
        baseRef = FirebaseDatabase.getInstance().getReference("Courses");

        // 2. Setup UI
        recyclerView = findViewById(R.id.recyclerView);
        fabAdd = findViewById(R.id.fabAdd);
        
        recyclerView.setLayoutManager(new LinearLayoutManager(this));

        // 3. Configure Adapter Options
        FirebaseRecyclerOptions<Course> options =
                new FirebaseRecyclerOptions.Builder<Course>()
                        .setQuery(baseRef, Course.class)
                        .build();

        adapter = new CourseAdapter(options, this);
        recyclerView.setAdapter(adapter);

        // 4. Add New Course Logic
        fabAdd.setOnClickListener(v -> showAddDialog());
    }

    private void showAddDialog() {
        // Inflating with NULL because it's a dialog (No parent yet)
        View view = LayoutInflater.from(this).inflate(R.layout.dialog_add_course, null);
        EditText etName = view.findViewById(R.id.etName);
        EditText etCode = view.findViewById(R.id.etCode);

        new AlertDialog.Builder(this)
            .setTitle("Add New Course")
            .setView(view)
            .setPositiveButton("Save", (dialog, which) -> {
                String name = etName.getText().toString();
                String code = etCode.getText().toString();

                // PUSH operation
                String id = baseRef.push().getKey(); 
                Course newCourse = new Course(id, name, code);

                baseRef.child(id).setValue(newCourse)
                    .addOnSuccessListener(unused -> Toast.makeText(this, "Added", Toast.LENGTH_SHORT).show())
                    .addOnFailureListener(e -> Toast.makeText(this, "Error: " + e.getMessage(), Toast.LENGTH_SHORT).show());
            })
            .setNegativeButton("Cancel", null)
            .show();
    }

    // MANDATORY: Adapter Lifecycle Methods
    @Override
    protected void onStart() {
        super.onStart();
        adapter.startListening(); // Begin sync
    }

    @Override
    protected void onStop() {
        super.onStop();
        adapter.stopListening(); // Stop sync to save battery
    }
}

```
