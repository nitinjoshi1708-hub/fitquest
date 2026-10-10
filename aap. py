import streamlit as st
import sqlite3
import pandas as pd
from datetime import date, timedelta

# -------------------------------
# PAGE CONFIGURATION
# -------------------------------

st.set_page_config(
    page_title="FitQuest",
    page_icon="🔥",
    layout="centered"
)

DB_NAME = "fitquest.db"

# -------------------------------
# DATABASE SETUP
# -------------------------------

def get_connection():
    return sqlite3.connect(DB_NAME)

def setup_database():
    with get_connection() as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS activities (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                activity TEXT NOT NULL,
                quantity REAL NOT NULL,
                unit TEXT NOT NULL,
                points INTEGER NOT NULL,
                activity_date TEXT NOT NULL
            )
        """)

setup_database()

# -------------------------------
# EXERCISE POINTS
# -------------------------------

EXERCISES = {
    "Push-ups": {
        "unit": "reps",
        "points": 1
    },
    "Pull-ups": {
        "unit": "reps",
        "points": 1
    },
    "Squats": {
        "unit": "reps",
        "points": 1
    },
    "Plank": {
        "unit": "seconds",
        "points": 0.1
    },
    "Running": {
        "unit": "km",
        "points": 10
    },
    "Walking": {
        "unit": "km",
        "points": 5
    }
}

# -------------------------------
# FETCH ACTIVITY HISTORY
# -------------------------------

def get_activities():
    with get_connection() as conn:
        return pd.read_sql_query(
            "SELECT * FROM activities ORDER BY id DESC",
            conn
        )

# -------------------------------
# ADD ACTIVITY
# -------------------------------

def add_activity(activity, quantity):
    exercise = EXERCISES[activity]

    points = round(
        quantity * exercise["points"]
    )

    with get_connection() as conn:
        conn.execute("""
            INSERT INTO activities
            (activity, quantity, unit, points, activity_date)
            VALUES (?, ?, ?, ?, ?)
        """, (
            activity,
            quantity,
            exercise["unit"],
            points,
            date.today().isoformat()
        ))

# -------------------------------
# CALCULATE STREAK
# -------------------------------

def calculate_streaks(df):
    if df.empty:
        return 0, 0

    active_dates = set(df["activity_date"])

    today = date.today()
    yesterday = today - timedelta(days=1)

    # Current streak:
    # Today's activity OR yesterday's activity
    if today.isoformat() in active_dates:
        current_day = today
    elif yesterday.isoformat() in active_dates:
        current_day = yesterday
    else:
        current_day = None

    current_streak = 0

    if current_day:
        while current_day.isoformat() in active_dates:
            current_streak += 1
            current_day -= timedelta(days=1)

    # Longest streak
    sorted_dates = sorted(
        date.fromisoformat(d)
        for d in active_dates
    )

    longest_streak = 0
    running_streak = 0
    previous_day = None

    for activity_day in sorted_dates:
        if (
            previous_day is not None
            and activity_day == previous_day + timedelta(days=1)
        ):
            running_streak += 1
        else:
            running_streak = 1

        longest_streak = max(
            longest_streak,
            running_streak
        )

        previous_day = activity_day

    return current_streak, longest_streak

# -------------------------------
# APP HEADER
# -------------------------------

st.title("🔥 FitQuest")
st.caption(
    "Build consistency. Track your workouts. Earn points."
)

st.divider()

# -------------------------------
# DASHBOARD
# -------------------------------

df = get_activities()

total_points = int(df["points"].sum()) if not df.empty else 0

today_str = date.today().isoformat()

today_df = (
    df[df["activity_date"] == today_str]
    if not df.empty
    else df
)

today_points = (
    int(today_df["points"].sum())
    if not today_df.empty
    else 0
)

total_workouts = len(df)

current_streak, longest_streak = calculate_streaks(df)

col1, col2 = st.columns(2)

col1.metric(
    "🏆 Total Points",
    total_points
)

col2.metric(
    "⭐ Today's Points",
    today_points
)

col3, col4 = st.columns(2)

col3.metric(
    "💪 Activities Logged",
    total_workouts
)

col4.metric(
    "🔥 Current Streak",
    f"{current_streak} days"
)

st.caption(
    f"Longest streak: {longest_streak} days"
)

st.divider()

# -------------------------------
# ADD NEW ACTIVITY
# -------------------------------

st.header("🏋️ Log Your Activity")

selected_exercise = st.selectbox(
    "Choose an exercise",
    list(EXERCISES.keys())
)

exercise_details = EXERCISES[selected_exercise]

unit = exercise_details["unit"]
rate = exercise_details["points"]

st.info(
    f"Points: {rate:g} point(s) per {unit[:-1] if unit.endswith('s') else unit}"
)

quantity = st.number_input(
    f"Enter quantity ({unit})",
    min_value=0.0,
    max_value=10000.0,
    value=10.0,
    step=1.0
)

estimated_points = round(quantity * rate)

st.write(f"### You will earn {estimated_points} points")

if st.button("✅ Save Activity", use_container_width=True):

    if quantity <= 0:
        st.error("Please enter a quantity greater than zero.")

    elif unit in ("reps", "seconds") and not quantity.is_integer():
        st.error("Please enter a whole number for reps or seconds.")

    else:
        add_activity(
            selected_exercise,
            quantity
        )

        st.success(
            f"Great work! You earned {estimated_points} points!"
        )

        st.rerun()

st.divider()

# -------------------------------
# STREAK MESSAGE
# -------------------------------

st.header("🔥 Your Consistency")

if current_streak == 0:
    st.write(
        "Start today and begin building your streak!"
    )
elif current_streak < 7:
    st.write(
        f"Awesome! You have a {current_streak}-day streak. "
        "Keep going!"
    )
else:
    st.balloons()
    st.success(
        f"Incredible! You have maintained a "
        f"{current_streak}-day streak!"
    )

# -------------------------------
# ACTIVITY HISTORY
# -------------------------------

st.divider()

st.header("📅 Activity History")

if df.empty:
    st.info(
        "No activities recorded yet. "
        "Complete your first workout to get started!"
    )

else:
    history = df.copy()

    history = history.rename(columns={
        "activity_date": "Date",
        "activity": "Activity",
        "quantity": "Quantity",
        "unit": "Unit",
        "points": "Points"
    })

    history = history[
        ["Date", "Activity", "Quantity", "Unit", "Points"]
    ]

    st.dataframe(
        history,
        use_container_width=True,
        hide_index=True
    )

    csv_data = history.to_csv(index=False).encode("utf-8")

    st.download_button(
        "⬇️ Download Activity History",
        data=csv_data,
        file_name="fitquest_history.csv",
        mime="text/csv"
    )

st.divider()

st.caption(
    "Remember: consistency matters more than points. "
    "Rest and recovery are part of training too."
)
