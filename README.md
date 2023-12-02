import datetime
import sqlite3

# Connect to the SQLite database
conn = sqlite3.connect("fitness_tracker.db")
cursor = conn.cursor()

# Create tables if not exists
cursor.execute("""
CREATE TABLE IF NOT EXISTS users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    age INTEGER
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS weight_logs (
    id INTEGER PRIMARY KEY,
    user_id INTEGER,
    weight REAL,
    goal TEXT,
    timestamp TEXT,
    FOREIGN KEY (user_id) REFERENCES users (id)
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS workout_logs (
    id INTEGER PRIMARY KEY,
    user_id INTEGER,
    workout TEXT,
    duration REAL,
    calories_burned INTEGER,
    timestamp TEXT,
    FOREIGN KEY (user_id) REFERENCES users (id)
)
""")

cursor.execute("""
CREATE TABLE IF NOT EXISTS meal_logs (
    id INTEGER PRIMARY KEY,
    user_id INTEGER,
    meal TEXT,
    calories INTEGER,
    timestamp TEXT,
    FOREIGN KEY (user_id) REFERENCES users (id)
)
""")

# Commit the changes and close the connection
conn.commit()
conn.close()

def log_weight(user_id, weight, goal):
    conn = sqlite3.connect("fitness_tracker.db")
    cursor = conn.cursor()

    timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')

    cursor.execute("""
    INSERT INTO weight_logs (user_id, weight, goal, timestamp)
    VALUES (?, ?, ?, ?)
    """, (user_id, weight, goal, timestamp))

    conn.commit()
    conn.close()

    print(f"Weight of {weight} kg logged with a goal to {goal} weight.")

def log_workout(user_id, workouts):
    conn = sqlite3.connect("fitness_tracker.db")
    cursor = conn.cursor()

    timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')

    for workout, duration, calories_burned in workouts:
        cursor.execute("""
        INSERT INTO workout_logs (user_id, workout, duration, calories_burned, timestamp)
        VALUES (?, ?, ?, ?, ?)
        """, (user_id, workout, duration, calories_burned, timestamp))

    conn.commit()
    conn.close()

    print("Workout log updated.")

def log_meal(user_id, meals):
    conn = sqlite3.connect("fitness_tracker.db")
    cursor = conn.cursor()

    timestamp = datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')

    for meal, calories in meals:
        cursor.execute("""
        INSERT INTO meal_logs (user_id, meal, calories, timestamp)
        VALUES (?, ?, ?, ?)
        """, (user_id, meal, calories, timestamp))

    conn.commit()
    conn.close()

    print("Meal log updated.")

def display_logs(user_id):
    conn = sqlite3.connect("fitness_tracker.db")
    cursor = conn.cursor()

    cursor.execute("SELECT * FROM weight_logs WHERE user_id = ? ORDER BY timestamp DESC LIMIT 1", (user_id,))
    weight_log = cursor.fetchone()

    cursor.execute("SELECT * FROM workout_logs WHERE user_id = ? ORDER BY timestamp DESC", (user_id,))
    workout_logs = cursor.fetchall()

    cursor.execute("SELECT * FROM meal_logs WHERE user_id = ? ORDER BY timestamp DESC", (user_id,))
    meal_logs = cursor.fetchall()

    conn.close()

    print("\nWeight Log:")
    if weight_log:
        print(f"{weight_log[2]} kg (Goal: {weight_log[3]})")

    print("\nWorkout Log:")
    for workout in workout_logs:
        print(f"{workout[3]} for {workout[4]} minutes, burning {workout[5]} calories.")

    print("\nMeal Log:")
    for meal in meal_logs:
        print(f"{meal[3]} ({meal[4]} calories)")

def calculate_calories_balance(user_id):
    conn = sqlite3.connect("fitness_tracker.db")
    cursor = conn.cursor()

    cursor.execute("SELECT SUM(calories_burned) FROM workout_logs WHERE user_id = ?", (user_id,))
    total_calories_burned = cursor.fetchone()[0]

    cursor.execute("SELECT SUM(calories) FROM meal_logs WHERE user_id = ?", (user_id,))
    total_calories_consumed = cursor.fetchone()[0]

    conn.close()

    if total_calories_burned is None:
        total_calories_burned = 0

    if total_calories_consumed is None:
        total_calories_consumed = 0

    balance = total_calories_consumed - total_calories_burned
    print(f"Calories balance: {balance} calories")

    goal_cursor = conn.cursor()
    goal_cursor.execute("SELECT goal FROM weight_logs WHERE user_id = ? ORDER BY timestamp DESC LIMIT 1", (user_id,))
    weight_goal = goal_cursor.fetchone()

    if weight_goal:
        weight_goal = weight_goal[0]

        if "decrease" in weight_goal.lower() and balance < 0:
            print("You're doing great! Keep it up, you're on your way to achieving your weight loss goal.")
        elif "decrease" in weight_goal.lower() and balance >= 0:
            print("Stay consistent, you'll reach your weight loss goal with determination.")
        elif "increase" in weight_goal.lower() and balance > 0:
            print("Awesome job! You're making progress towards your weight gain goal.")
        elif "increase" in weight_goal.lower() and balance <= 0:
            print("Keep going, you'll reach your weight gain goal with persistence")
    else:
        print("No weight goal set.")

# Additional Functions
# ...

print("Welcome to the Fitness Tracker!")

# Ensure the user is in the database
user_name = input("What's your name? ")
user_age = int(input("How old are you? "))

conn = sqlite3.connect("fitness_tracker.db")
cursor = conn.cursor()

cursor.execute("SELECT * FROM users WHERE name = ? AND age = ?", (user_name, user_age))
user = cursor.fetchone()

if not user:
    cursor.execute("INSERT INTO users (name, age) VALUES (?, ?)", (user_name, user_age))
    conn.commit()

    cursor.execute("SELECT * FROM users WHERE name = ? AND age = ?", (user_name, user_age))
    user = cursor.fetchone()

conn.close()

user_id = user[0]

while True:
    print("\nFitness Tracker Menu:")
    print("1. Log Weight")
    print("2. Log Workout")
    print("3. Log Meal")
    print("4. Display Logs")
    print("5. Calculate Calories Balance")
    print("6. Exit")

    choice = input("Enter your choice (1/2/3/4/5/6): ")

    if choice == "1":
        weight = float(input("Enter your weight (in kilograms): "))
        goal = input("Do you want to increase or decrease your weight? (increase/decrease): ")
        log_weight(user_id, weight, goal)
    elif choice == "2":
        workouts = []
        num_workouts = int(input("How many workouts would you like to log? "))
        for i in range(num_workouts):
            workout_type = input(f"Enter workout {i+1} type: ")
            duration = float(input(f"Enter the duration (in minutes) for workout {i+1}: "))
            calories_burned = int(input(f"How many calories burned for workout {i+1}: "))
            workouts.append((workout_type, duration, calories_burned))
        log_workout(user_id, workouts)
    elif choice == "3":
        meals = []
        num_meals = int(input("How many meals would you like to log? "))
        for i in range(num_meals):
            meal_description = input(f"Enter meal {i+1} description: ")
            calories = int(input(f"Enter the calorie count for meal {i+1}: "))
            meals.append((meal_description, calories))
        log_meal(user_id, meals)
    elif choice == "4":
        display_logs(user_id)
    elif choice == "5":
        calculate_calories_balance(user_id)
    elif choice == "6":
        current_time = datetime.datetime.now()
        current_time = current_time.strftime('%Y-%m-%d %H:%M:%S')
        print(f"Goodbye {user_name}! Program exited at {current_time}")
        break
    else:
        print("Invalid choice. Please enter 1, 2, 3, 4, 5, or 6.")
