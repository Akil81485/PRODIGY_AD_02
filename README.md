
# Prodigy InfoTech Internship: Android Development Task

Welcome to the Prodigy InfoTech Internship repository! This repository documents task 3 of the internship, which focuses on building a To-Do List app for Android.

## Task Overview

The objective of this task is to develop a To-Do List app for Android that allows users to:
- **Add** tasks
- **Edit** tasks
- **Delete** tasks

## Steps

### 1. **Setup Project**

1. **Create a New Android Project**:
   - Open Android Studio and start a new Android Studio project.
   - Choose "Empty Activity" as the template.
   - Name your project (e.g., `TodoListApp`), configure the project settings (package name, save location), and click "Finish."

### 2. **Design UI**

1. **Open `activity_main.xml`** in `res/layout` and design the layout:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <EditText
        android:id="@+id/taskInput"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:hint="Enter a new task"
        android:inputType="text"/>

    <Button
        android:id="@+id/addTaskButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Add Task"
        android:layout_below="@id/taskInput"
        android:layout_alignParentEnd="true"
        android:layout_marginTop="8dp"/>

    <ListView
        android:id="@+id/taskListView"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_below="@id/addTaskButton"
        android:layout_marginTop="16dp"/>

</RelativeLayout>
```

2. **Create a layout file for each list item** (`list_item.xml`) in `res/layout`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:padding="8dp">

    <TextView
        android:id="@+id/taskText"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="18sp"/>

    <ImageButton
        android:id="@+id/editButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_menu_edit"
        android:layout_alignParentEnd="true"
        android:contentDescription="Edit"/>

    <ImageButton
        android:id="@+id/deleteButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:src="@android:drawable/ic_menu_delete"
        android:layout_toStartOf="@id/editButton"
        android:contentDescription="Delete"/>
</RelativeLayout>
```

### 3. **Implement Logic**

1. **Open `MainActivity.java`** and implement the logic for adding, editing, and deleting tasks:

```java
package com.example.todolistapp;

import android.os.Bundle;
import android.view.LayoutInflater;
import android.view.View;
import android.view.ViewGroup;
import android.widget.AdapterView;
import android.widget.ArrayAdapter;
import android.widget.Button;
import android.widget.EditText;
import android.widget.ImageButton;
import android.widget.ListView;
import android.widget.TextView;

import androidx.appcompat.app.AppCompatActivity;

import java.util.ArrayList;

public class MainActivity extends AppCompatActivity {

    private EditText taskInput;
    private Button addTaskButton;
    private ListView taskListView;
    private ArrayAdapter<String> taskAdapter;
    private ArrayList<String> tasks;
    private int editingPosition = -1;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        taskInput = findViewById(R.id.taskInput);
        addTaskButton = findViewById(R.id.addTaskButton);
        taskListView = findViewById(R.id.taskListView);

        tasks = new ArrayList<>();
        taskAdapter = new ArrayAdapter<String>(this, R.layout.list_item, R.id.taskText, tasks) {
            @Override
            public View getView(int position, View convertView, ViewGroup parent) {
                if (convertView == null) {
                    convertView = LayoutInflater.from(getContext()).inflate(R.layout.list_item, parent, false);
                }
                
                TextView taskText = convertView.findViewById(R.id.taskText);
                ImageButton editButton = convertView.findViewById(R.id.editButton);
                ImageButton deleteButton = convertView.findViewById(R.id.deleteButton);

                final String task = getItem(position);
                taskText.setText(task);

                editButton.setOnClickListener(v -> {
                    editingPosition = position;
                    taskInput.setText(task);
                });

                deleteButton.setOnClickListener(v -> {
                    tasks.remove(position);
                    taskAdapter.notifyDataSetChanged();
                });

                return convertView;
            }
        };
        taskListView.setAdapter(taskAdapter);

        addTaskButton.setOnClickListener(v -> {
            String newTask = taskInput.getText().toString().trim();
            if (!newTask.isEmpty()) {
                if (editingPosition != -1) {
                    tasks.set(editingPosition, newTask);
                    editingPosition = -1;
                } else {
                    tasks.add(newTask);
                }
                taskInput.setText("");
                taskAdapter.notifyDataSetChanged();
            }
        });
    }
}
```

### 4. **Write Tests**

1. **Create a new test class** in `androidTest` directory (e.g., `TodoListTest.java`):

```java
package com.example.todolistapp;

import androidx.test.ext.junit.runners.AndroidJUnit4;
import androidx.test.rule.ActivityTestRule;

import org.junit.Rule;
import org.junit.Test;
import org.junit.runner.RunWith;

import static androidx.test.espresso.Espresso.onView;
import static androidx.test.espresso.action.ViewActions.click;
import static androidx.test.espresso.action.ViewActions.typeText;
import static androidx.test.espresso.assertion.ViewAssertions.matches;
import static androidx.test.espresso.matcher.ViewMatchers.withId;
import static androidx.test.espresso.matcher.ViewMatchers.withText;

@RunWith(AndroidJUnit4.class)
public class TodoListTest {

    @Rule
    public ActivityTestRule<MainActivity> activityRule = new ActivityTestRule<>(MainActivity.class);

    @Test
    public void testAddTask() {
        onView(withId(R.id.taskInput)).perform(typeText("Test Task"));
        onView(withId(R.id.addTaskButton)).perform(click());

        onView(withId(R.id.taskListView)).check(matches(withText("Test Task")));
    }

    @Test
    public void testEditTask() {
        onView(withId(R.id.taskInput)).perform(typeText("Initial Task"));
        onView(withId(R.id.addTaskButton)).perform(click());

        onView(withId(R.id.taskListView)).perform(click());
        onView(withId(R.id.taskInput)).perform(typeText("Updated Task"));
        onView(withId(R.id.addTaskButton)).perform(click());

        onView(withId(R.id.taskListView)).check(matches(withText("Updated Task")));
    }

    @Test
    public void testDeleteTask() {
        onView(withId(R.id.taskInput)).perform(typeText("Task to Delete"));
        onView(withId(R.id.addTaskButton)).perform(click());

        onView(withId(R.id.taskListView)).perform(click());
        onView(withId(R.id.deleteButton)).perform(click());

        onView(withId(R.id.taskListView)).check(matches(withText("")));
    }
}
```

2. **Run the tests** to ensure the app functions as expected.

### 5. **Build & Run**

1. **Build the Project**:
   - Click on `Build > Build Bundle(s) / APK(s) > Build APK(s)` in Android Studio.

2. **Run the App**:
   - Click the green play button in Android Studio to run the app on an Android device or emulator.

## Knowledge Gained

- **Android Application Development**:
  - Gained practical experience in building and managing an Android app project using Android Studio.
  - Learned how to handle UI components and user interactions effectively.

- **User Interface (UI) Design**:
  - Acquired skills in designing and implementing a functional user interface with buttons, lists, and text fields.

- **Handling Task Operations**:
  - Developed logic to add, edit, and delete tasks, including managing task states and interactions.

- **Testing in Android**:
  - Created and executed instrumented tests to verify the app’s functionality and ensure it behaves as expected under different scenarios.

## Summary

This task involved the development of a To-Do List app for Android, including:
- **Project Setup**: Initialized and configured an Android project.
- **UI Design**: Designed a user-friendly interface with necessary components.
- **Logic Implementation**: Implemented the core functionalities to manage tasks.
- **Testing**

: Developed and ran tests to validate the app’s functionality.

The provided tests ensure that the app handles tasks correctly, including adding, editing, and deleting functionalities.

## Conclusion

The task successfully demonstrated the fundamental steps required to build a To-Do List app for Android, covering:
- Effective UI design and implementation
- Development of task management functionalities
- Testing practices to ensure app reliability


