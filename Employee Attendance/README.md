# 🧑‍💼 Employee Attendance Management System

A Python mini project built using **SQLite and Jupyter Notebook** to manage employee attendance efficiently.  
It allows organizations to track employee check-in and check-out times, calculate total working hours, and generate detailed attendance logs — all stored in a local SQLite database.

## 🔑 Key Features

- Add employee records with name and department
- Mark check-in and check-out with real-time timestamps
- Prevent double check-ins without checkout
- Automatically calculate total working hours per employee
- View complete attendance history using Pandas (tabular format)
- Lightweight and works offline with SQLite

## 🛠️ Built With
- Python 3
- SQLite (via `sqlite3`)
- Jupyter Notebook
- `pandas` (for tabular data)
- `datetime` & `pytz` (for timestamps)




PART 1: Connecting to the Database and Creating Tables

import sqlite3
from datetime import datetime
import pytz
import pandas as pd
🔍 Explanation:
Line	Meaning
import sqlite3	🧠 We need this to work with a SQLite database (a lightweight database stored in a .db file). It allows us to store and retrieve data like employee info, check-in/out logs.
from datetime import datetime	🕒 We use datetime to get the current time when someone checks in/out.
import pytz	🌏 pytz lets us set the timezone — like IST. By default, Python uses UTC (not helpful for India).
import pandas as pd	📊 Pandas helps us create table-like summaries (for reports or display of attendance).

Code:

conn = sqlite3.connect("attendance.db")
cursor = conn.cursor()
Line	Meaning
sqlite3.connect("attendance.db")	🔗 This connects to a file called attendance.db. If it doesn’t exist, it creates one.
cursor = conn.cursor()	✍️ A cursor is like a pen to write commands into the database — like inserting or fetching records.

Code:

cursor.execute("""
CREATE TABLE IF NOT EXISTS employee_info (
    emp_id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    department TEXT
)
""")
Part	Meaning
CREATE TABLE IF NOT EXISTS	✅ This command creates a table (if it doesn’t exist already).
employee_info	📋 Table name where we store employee details.
emp_id INTEGER PRIMARY KEY AUTOINCREMENT	🆔 Unique employee ID. It increases automatically (1, 2, 3...).
name TEXT NOT NULL	👤 Employee name. Must be filled (not null).
department TEXT	🏢 Optional department name.

Code:

cursor.execute("""
CREATE TABLE IF NOT EXISTS attendance_log (
    log_id INTEGER PRIMARY KEY AUTOINCREMENT,
    emp_id INTEGER,
    checkin DATETIME,
    checkout DATETIME,
    duration REAL,
    FOREIGN KEY(emp_id) REFERENCES employee_info(emp_id)
)
""")
conn.commit()
Part	Meaning
attendance_log	📊 Second table — stores check-in and check-out data.
log_id	Unique ID for each login session (auto increments).
emp_id	🔗 Connects to employee_info (employee who logged in).
checkin / checkout	🕐 Timestamps for check-in/out.
duration	⏱ Total hours calculated between check-in and checkout.
conn.commit()	💾 Saves everything we’ve written so far into the file (attendance.db).

✅ Up to this point:
You have:

Imported needed libraries

Connected to a DB file

Created two tables: employee info & attendance log

Visual explanation 



🧠 What happens when:
You connect to the database

You create tables

You insert data later on

It will visually explain the structure like:

+--------------------+
|  attendance.db     |  <-- SQLite database file
+--------------------+

        |
        v

Creates 2 tables inside it:

+-------------------------+         +---------------------------+
|    employee_info        |         |     attendance_log        |
+-------------------------+         +---------------------------+
| emp_id (PK)            |<------->| emp_id (FK from above)    |
| name                   |         | log_id (PK)               |
| department             |         | checkin                   |
+-------------------------+         | checkout                  |
                                    | duration                  |

✅ Created the database (attendance.db)

✅ Created tables: employee_info and attendance_log

🔄 Next Steps — Let’s break it down:

🔹 1. Add New Employee

def add_employee():
    name = input("Enter employee name: ")
    dept = input("Enter department: ")

    cursor.execute("""
        INSERT INTO employee_info (name, department)
        VALUES (?, ?)
    """, (name, dept))

    conn.commit()
    print(f"✅ {name} added successfully.")
🧠 Explanation:
input() takes user input.

SQL INSERT INTO adds new row to employee_info table.

? is a placeholder to prevent SQL injection.

conn.commit() saves it to the DB permanently.

🔹 2. Mark Check-in

from datetime import datetime

def mark_checkin():
    emp_id = int(input("Enter the employee id for checkin: "))
    
    # Check if already checked in without checkout
    cursor.execute("""
        SELECT * FROM attendance_log
        WHERE emp_id = ? AND checkout IS NULL
        ORDER BY checkin DESC LIMIT 1
    """, (emp_id,))
    
    existing = cursor.fetchone()

    if existing:
        print("⚠️ Employee has already checked in without checkout!")
        return

    checkin_time = datetime.now().strftime("%Y-%m-%d %I:%M:%S %p")  # 12-hr format

    cursor.execute("""
        INSERT INTO attendance_log (emp_id, checkin)
        VALUES (?, ?)
    """, (emp_id, checkin_time))

    conn.commit()
    print(f"✅ Checked in at {checkin_time} for employee {emp_id}")
🧠 Explanation:
datetime.now().strftime("%I:%M:%S %p") → 12-hour format with AM/PM.

Prevents multiple check-ins without checkout using the SQL condition checkout IS NULL.

🔹 3. Mark Checkout

def mark_checkout():
    emp_id = int(input("Enter the employee id for checkout: "))

    cursor.execute("""
        SELECT log_id, checkin FROM attendance_log
        WHERE emp_id = ? AND checkout IS NULL
        ORDER BY checkin DESC LIMIT 1
    """, (emp_id,))

    result = cursor.fetchone()

    if not result:
        print("⚠️ No active check-in found!")
        return

    log_id, checkin_str = result
    checkout_time = datetime.now().strftime("%Y-%m-%d %I:%M:%S %p")

    # Calculate work duration
    checkin = datetime.strptime(checkin_str, "%Y-%m-%d %I:%M:%S %p")
    checkout = datetime.strptime(checkout_time, "%Y-%m-%d %I:%M:%S %p")
    duration = round((checkout - checkin).total_seconds() / 3600, 2)

    # Update the checkout and duration
    cursor.execute("""
        UPDATE attendance_log
        SET checkout = ?, duration = ?
        WHERE log_id = ?
    """, (checkout_time, duration, log_id))

    conn.commit()
    print(f"✅ Checkout at {checkout_time}. Total hours: {duration}")

           
🔹 4. View All Attendance Logs

import pandas as pd

def view_logs():
    query = """
    SELECT al.log_id, ei.emp_id, ei.name, ei.department, al.checkin, al.checkout, al.duration
    FROM attendance_log al
    JOIN employee_info ei ON al.emp_id = ei.emp_id
    ORDER BY al.checkin DESC
    """
    
    df = pd.read_sql_query(query, conn)
    if df.empty:
        print("📭 No records yet.")
    else:
        print("📋 Attendance Logs:")
        display(df)

                 
🔹 5. Update Work Duration (Manually fix for past check-in/outs)

                 
def update_work_duration():
    cursor.execute("""
        SELECT log_id, checkin, checkout FROM attendance_log
        WHERE checkout IS NOT NULL AND duration IS NULL
    """)
    rows = cursor.fetchall()

    for log_id, checkin_str, checkout_str in rows:
        if isinstance(checkin_str, str):
            checkin = datetime.strptime(checkin_str, "%Y-%m-%d %I:%M:%S %p")
        else:
            checkin = checkin_str

        if isinstance(checkout_str, str):
            checkout = datetime.strptime(checkout_str, "%Y-%m-%d %I:%M:%S %p")
        else:
            checkout = checkout_str

        duration = round((checkout - checkin).total_seconds() / 3600, 2)

        cursor.execute("""
            UPDATE attendance_log SET duration = ? WHERE log_id = ?
        """, (duration, log_id))

    conn.commit()
    print("🛠️ Duration updated where needed.")
                
🧭 6. Menu to Run Program

while True:
    print("\n📘 MENU:")
    print("1. Add Employee")
    print("2. Check-In")
    print("3. Check-Out")
    print("4. View Logs")
    print("5. Update Work Duration")
    print("6. Exit")

    choice = input("Enter your choice: ")

    if choice == '1':
        add_employee()
    elif choice == '2':
        mark_checkin()
    elif choice == '3':
        mark_checkout()
    elif choice == '4':
        view_logs()
    elif choice == '5':
        update_work_duration()
    elif choice == '6':
        print("👋 Exiting...")
        break
    else:
        print("❌ Invalid choice.")
          

Add 1–2 employees.

Mark check-in and check-out.

View logs in table format.


✅ 1. Add Employee Function

def add_employee():
This defines a new function named add_employee. Functions are reusable blocks of code.


    name = input("Enter employee name: ")
This takes the employee's name from the user and stores it in the variable name.


    dept = input("Enter department: ")
This takes the department name and stores it in dept.

    cursor.execute("""
        INSERT INTO employee_info (name, department)
        VALUES (?, ?)
    """, (name, dept))
This line inserts the collected data (name, dept) into the employee_info table using cursor.execute().

The ? placeholders are used for parameterized SQL queries (to prevent SQL injection).

The second argument (name, dept) supplies the values to replace the placeholders.


    conn.commit()
This saves the changes permanently into the database.


    print(f"✅ {name} added successfully.")
A message is shown confirming the addition of the employee.

✅ 2. Check-in Function

def mark_checkin():
A function to log the time when an employee checks in.


    emp_id = int(input("Enter the employee id for checkin: "))
Takes employee ID as input and converts it to an integer.

    cursor.execute("""
        SELECT * FROM attendance_log
        WHERE emp_id = ? AND checkout IS NULL
        ORDER BY checkin DESC LIMIT 1
    """, (emp_id,))
This checks if the employee has already checked in but not checked out yet.

It gets the most recent check-in (due to ORDER BY checkin DESC LIMIT 1).


    existing = cursor.fetchone()
fetchone() retrieves the first result (if any). If nothing is found, it returns None.


    if existing:
        print("⚠️ Employee has already checked in without checkout!")
        return
If there's already a check-in with no checkout, the function stops.


    checkin_time = datetime.now().strftime("%Y-%m-%d %I:%M:%S %p")
Gets the current time in 12-hour format with AM/PM (e.g., 06:30:00 PM).


    cursor.execute("""
        INSERT INTO attendance_log (emp_id, checkin)
        VALUES (?, ?)
    """, (emp_id, checkin_time))
Inserts a new check-in record into the attendance_log.


    conn.commit()
Saves the check-in to the database.


    print(f"✅ Checked in at {checkin_time} for employee {emp_id}")
Shows confirmation message.

✅ 3. Checkout Function

def mark_checkout():
Function to mark the checkout time of an employee.


    emp_id = int(input("Enter the employee id for checkout: "))
Takes employee ID as input.


    cursor.execute("""
        SELECT log_id, checkin FROM attendance_log
        WHERE emp_id = ? AND checkout IS NULL
        ORDER BY checkin DESC LIMIT 1
    """, (emp_id,))
Retrieves the latest check-in record where no checkout is recorded.
t
    result = cursor.fetchone()
Stores that one result row.

    if not result:
        print("⚠️ No active check-in found!")
        return
If there’s no pending check-in, it exits the function.

    log_id, checkin_str = result
Unpacks the result into log_id and checkin_str.


    checkout_time = datetime.now().strftime("%Y-%m-%d %I:%M:%S %p")
Gets current time for checkout in 12-hour format.


    checkin = datetime.strptime(checkin_str, "%Y-%m-%d %I:%M:%S %p")
    checkout = datetime.strptime(checkout_time, "%Y-%m-%d %I:%M:%S %p")
Converts the check-in and checkout time strings back to datetime objects.

    duration = round((checkout - checkin).total_seconds() / 3600, 2)
Calculates total working hours and rounds it to 2 decimal places.
t
    cursor.execute("""
        UPDATE attendance_log
        SET checkout = ?, duration = ?
        WHERE log_id = ?
    """, (checkout_time, duration, log_id))
Updates the attendance_log with checkout time and duration.


    conn.commit()
Saves the changes.

t
    print(f"✅ Checkout at {checkout_time}. Total hours: {duration}")
Displays success message with hours worked.

✅ 4. View Attendance Logs (with pandas)

import pandas as pd
Imports the pandas library to show data as a table.


def view_logs():
Function to view all logs.


    query = """
    SELECT al.log_id, ei.emp_id, ei.name, ei.department, al.checkin, al.checkout, al.duration
    FROM attendance_log al
    JOIN employee_info ei ON al.emp_id = ei.emp_id
    ORDER BY al.checkin DESC
    """
SQL query to join both tables and show detailed records.


    df = pd.read_sql_query(query, conn)
Executes the SQL and loads the result into a DataFrame (df).


    if df.empty:
        print("📭 No records yet.")
If no data exists, it notifies the user.

    else:
        print("📋 Attendance Logs:")
        display(df)
Otherwise, displays the logs in a table format.

✅ 5. Update Duration (For Old Records)

def update_work_duration():
Updates duration in logs where checkout was done but duration wasn’t calculated.

    cursor.execute("""
        SELECT log_id, checkin, checkout FROM attendance_log
        WHERE checkout IS NOT NULL AND duration IS NULL
    """)
Gets only those rows where checkout is done but duration is not calculated yet.


    rows = cursor.fetchall()
Fetches all such rows.

    for log_id, checkin_str, checkout_str in rows:
Loops through each row.


        checkin = datetime.strptime(checkin_str, "%Y-%m-%d %I:%M:%S %p")
        checkout = datetime.strptime(checkout_str, "%Y-%m-%d %I:%M:%S %p")
Converts strings into datetime format.


        duration = round((checkout - checkin).total_seconds() / 3600, 2)
Calculates hours worked.


        cursor.execute("""
            UPDATE attendance_log SET duration = ? WHERE log_id = ?
        """, (duration, log_id))
Updates the duration field.


    conn.commit()
    print("🛠️ Duration updated where needed.")
Commits and prints success message.

✅ 6. MENU SYSTEM

while True:
Infinite loop for menu system.


    print("\n📘 MENU:")
    print("1. Add Employee")
    print("2. Check-In")
    print("3. Check-Out")
    print("4. View Logs")
    print("5. Update Work Duration")
    print("6. Exit")
Shows menu options.


    choice = input("Enter your choice: ")
Takes the user’s selection.

    if choice == '1':
        add_employee()
Calls respective function based on input.


    elif choice == '6':
        print("👋 Exiting...")
        break
Exits the program.


    else:
        print("❌ Invalid choice.")
Handles invalid inputs.

✅ Summary:
This code covers:

Database creation

Adding employees

Check-in/out logic

Time duration calculation

Validation

Clean UI with a menu



You said:
Sometimes we are giving strftime and sometimes strptime y


ChatGPT said:
Great question! 🙌
Understanding the difference between strftime() and strptime() is super important — and once you get it, you’ll never forget it!

✅ strftime() vs strptime()
Function	Purpose	Mnemonic Tip
strftime()	"Format time" — Converts datetime ➡ string	String From Time
strptime()	"Parse time" — Converts string ➡ datetime	String Parse Time

📌 When to use each:
🔷 1. strftime() — Format the output
Use this when you want to display a datetime object in a readable format (like 12-hour with AM/PM).

Example:
python
Copy
Edit
now = datetime.now()
formatted = now.strftime("%d-%b-%Y %I:%M:%S %p")
print(formatted)
🟢 Output:

Copy
Edit
14-Jul-2025 08:10:15 PM
🔷 2. strptime() — Parse from user/input/file
Use this when you have a string like "14-Jul-2025 08:10:15 PM" and you want to convert it into a datetime object to do math with it (like subtracting check-in from checkout).

Example:
python
Copy
Edit
dt_string = "14-Jul-2025 08:10:15 PM"
dt_object = datetime.strptime(dt_string, "%d-%b-%Y %I:%M:%S %p")
print(dt_object)
🟢 Output:

yaml
Copy
Edit
2025-07-14 20:10:15
💡 Real-Life Project Example from Your Code:
✅ When checking in:
You use strftime() to store the time as string:

python
Copy
Edit
checkin_time = datetime.now().strftime("%Y-%m-%d %I:%M:%S %p")  # string
✅ When calculating duration:
You use strptime() to convert it back to datetime for math:

python
Copy
Edit
checkin = datetime.strptime(checkin_str, "%Y-%m-%d %I:%M:%S %p")
    
🔁 Why not just use datetime always?
Databases store and return values in text format (str). So:

When saving: Convert to string → strftime()

