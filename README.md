import streamlit as st
import json
from datetime import date, timedelta
from pathlib import Path

DATA_FILE = Path("goals.json")

def load_data():
    if DATA_FILE.exists():
        return json.loads(DATA_FILE.read_text())
    return {"goals": [], "streak": 0}

def save_data(data):
    DATA_FILE.write_text(json.dumps(data, indent=2))

data = load_data()

st.title("Daily Goal Tracker")

goal_date = st.date_input("Select a day", value=date.today())
goal_text = st.text_input("Enter your small goal")

if st.button("Save Goal"):
    data["goals"].append({
        "date": str(goal_date),
        "goal": goal_text,
        "status": "Pending"
    })
    save_data(data)
    st.success("Goal saved!")

st.subheader("Goals")

updated = False
for item in data["goals"]:
    st.write(f"**{item['date']}** - {item['goal']} ({item['status']})")

    if item["status"] == "Pending":
        col1, col2 = st.columns(2)

        with col1:
            if st.button(f"Complete {item['date']} {item['goal']}"):
                item["status"] = "Completed"
                data["streak"] += 1
                updated = True

        with col2:
            if st.button(f"Postpone {item['date']} {item['goal']}"):
                item["status"] = "Postponed"

                next_day = (
                    date.fromisoformat(item["date"]) + timedelta(days=1)
                )

                data["goals"].append({
                    "date": str(next_day),
                    "goal": item["goal"],
                    "status": "Pending"
                })
                updated = True

if updated:
    save_data(data)
    st.rerun()

completed = sum(1 for g in data["goals"] if g["status"] == "Completed")
postponed = sum(1 for g in data["goals"] if g["status"] == "Postponed")
pending = sum(1 for g in data["goals"] if g["status"] == "Pending")

st.divider()
st.write(f"Completed Goals: {completed}")
st.write(f"Pending Goals: {pending}")
st.write(f"Postponed Goals: {postponed}")
st.write(f"Current Streak: {data['streak']} days")

st.subheader("Calculator")
num1 = st.number_input("Number 1", value=0.0)
num2 = st.number_input("Number 2", value=0.0)

if st.button("Add"):
    st.write("Result:", num1 + num2)
