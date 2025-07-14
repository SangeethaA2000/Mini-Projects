# 📘 Student Mark Management System

This is a beginner-friendly Python mini project to manage student marks using SQLite and Jupyter Notebook.  
It allows users to add students, update marks, calculate total and average, view all records, and delete data — all stored locally in a lightweight SQL database.


## ✅ Features

- Add student name and 3 subject marks
- View all students with ID, marks, total, and average
- Update marks of existing students
- Automatically calculate total and average marks
- Delete student records
- Simple CLI-style menu in notebook
- Built using Python + SQLite (`sqlite3`)


## 🛠️ Technologies Used

- Python 3
- SQLite3 (no server required)
- Jupyter Notebook / VS Code (for development)


### Functional Breakdown:
1. **Add Student**
   - Inputs name and 3 subject marks.
   - Saves to SQLite database.

2. **View Students**
   - Displays all records with ID, Name, Marks, Total, and Average.

3. **Update Marks**
   - Allows updating marks of an existing student using name.

4. **Delete Student**
   - Removes a student from the database.

5. **Update Total & Average**
   - Automatically calculates total and average for each student.



```python
conn = sqlite3.connect("mark_database.db")
# Connect to local SQLite DB file (creates if not exists)

cursor = conn.cursor()
# Cursor object lets us execute SQL commands

cursor.execute("DROP TABLE IF EXISTS stu_tab")
# Drop table if it already exists to avoid duplicates

cursor.execute("""
CREATE TABLE IF NOT EXISTS stu_tab (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    stu_name TEXT NOT NULL,
    s_sub1 INT,
    s_sub2 INT,
    s_sub3 INT,
    s_total INT,
    average REAL
)
""")
# Create the student table with all required fields




