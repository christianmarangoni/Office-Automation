```markdown
## Creating Tasks from Specific Emails in Gmail

This guide outlines how to automatically create tasks in Google Tasks from emails received in Gmail that match specific criteria.  This is achieved using a combination of Gmail filters and Google Apps Script.

**1. Create a Gmail Filter:**

*   In Gmail, click the search options dropdown (the small down arrow in the search bar).
*   In the "From" field, enter `{address}`.
*   You can add other criteria if needed (e.g., subject keywords).
*   Click "Create filter" (or "Create filter with this search").
*   Check the box "Apply the label:" and choose or create a new label (e.g., "{label_name}"). This will help identify the emails quickly. You could also choose "Mark as read" if you don't want these to clutter your inbox.
*   Crucially, also check "Also apply filter to matching conversations." if you want the filter to apply to existing emails.
*   Click "Create filter."

**2. Create a Google Apps Script:**

*   Go to script.google.com (you'll need a Google account).
*   Create a new project.

**3. Write the Apps Script Code:**

```javascript
function createTaskFromEmail() {
  // Get the label you created in Gmail.
  var label = GmailApp.getUserLabelByName("{label_name}"); // Replace with your label name

  // Get the threads with the label.
  var threads = label.getThreads();

  // Iterate through the threads (most recent first).
  for (var i = 0; i < threads.length; i++) {
    var thread = threads[i];
    var messages = thread.getMessages();

    // Iterate through the messages in the thread (newest first).
    for (var j = messages.length - 1; j >= 0; j--) {
      var message = messages[j];

      //Check if message has been processed already to avoid creating duplicate tasks.
      if (message.isStarred()) {
        continue; //Skip if already processed
      }

      var subject = message.getSubject();
      var body = message.getBody();

      // Create a Google Tasks list (or use an existing one).
      var taskList = Tasks.getDefaultTaskList(); // Or Tasks.createTaskList("{task_list_name}");

      // Create the task.
      Tasks.insert(taskList.getId(), { title: subject, notes: body }); //Use subject as title and body as notes

      // Mark the email as starred so that it is not processed again.
      message.star();
    }
  }
}

// Set up a time-driven trigger to run the script periodically.
function createTrigger() {
  // Trigger every 5 minutes (adjust as needed).
  ScriptApp.newTrigger("createTaskFromEmail")
      .timeBased()
      .everyMinutes(5)
      .create();
}
```

**4. Authorize the Script:**

*   Click the "Run" button (the play icon). You'll be asked to authorize the script to access your Gmail and Tasks. Review the permissions carefully and grant them. This will run the `createTaskFromEmail` function the first time and you may get an error. This is expected.
*   Click the "Triggers" icon (the clock icon) in the left sidebar.
*   Click "+ Add Trigger".
*   Choose the `createTaskFromEmail` function.
*   Set the event source to "Time-driven".
*   Choose a time interval (e.g., "Minute timer" every 5 minutes). This is how often the script will check for new emails.
*   Save the trigger.

**5. Test:**

*   Send a test email to the specified address.
*   Wait for the trigger to fire (up to 5 minutes).
*   Check your Google Tasks list. The new task should be there.

**Explanation of the Code:**

*   `GmailApp.getUserLabelByName()` gets the Gmail label.
*   `label.getThreads()` gets the email threads with that label.
*   The script iterates through the threads and messages within each thread (newest first).
*   `message.getSubject()` and `message.getBody()` extract the email content.
*   `Tasks.getDefaultTaskList()` gets your default task list. You can create a new one if you prefer.
*   `Tasks.insert()` creates the task using the subject as the title and the body as notes.
*   `message.star()` marks the email as starred, so it's not processed again. This is a simple method to avoid duplicates.
*   The `createTrigger()` function sets up a time-driven trigger, so the script runs automatically.

**Key Improvements and Considerations:**

*   **Duplicate Prevention:** The code now marks processed emails as starred. This prevents the script from creating duplicate tasks if it runs multiple times and the same email is still present.
*   **Trigger Frequency:** Adjust the trigger frequency (e.g., every 5 minutes, every hour) based on how quickly you need the tasks to be created. Shorter intervals use more script execution time.
*   **Error Handling:** Consider adding error handling (e.g., `try...catch` blocks) to the script to catch potential issues (like the Tasks API being unavailable) and prevent the script from failing silently.
*   **Task List Selection:** The code uses `Tasks.getDefaultTaskList()`. You can modify this to use a specific task list by its ID if needed. You can create a new task list and use that one.
*   **Email Content Formatting:** The email body is added as notes. You may need to clean or format the body text if you need to extract specific information.

This approach provides a robust and automated way to create tasks from specific emails in Gmail. Remember to replace `{label_name}`, `{address}`, and `{task_list_name}` with your actual values.
```
