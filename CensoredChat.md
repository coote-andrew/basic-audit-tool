# ME
I want to write a super quick python script that opens a csv - shows me the text of a note side by side (from a column) then waits for a 1,0 or ? then shows me the next note.
If I ever type in a 1, then that "CSN" is positive, don't show me any more of that CSN's notes, then show me the next. If 0 or ?, then continue on with that CSN, 
if eventually all notes are reviewed and all are 0, then mark as 0, if any are 1, then mark as 1, if all are 0 and at least one ? then mark as ?

save this into a separate CSV, and ensure even if I quit the python program I never see marked csns again.
We should review these by total_flag count descending

Relevant columns are "TOTAL_FLAGS","PAT_ENC_CSN_ID","NOTE_ID","NOTE_TEXT"
output should be saved to a separate csv which keeps csn, total_flags and manual review

Preferably this should run from a juypyter notebook

---

# ChatGPT
Here's a simple interactive script you can run in a Jupyter notebook to review clinical notes as described.

This script does the following:

Loads your main CSV of notes.

Loads (or creates) a CSV of your manual reviews (manual_review.csv).

Groups notes by CSN and presents each note one at a time.

Allows you to input 1, 0, or ? to classify the CSN.

Automatically handles skipping already-reviewed CSNs and saves progress.

✅ Code to paste into a Jupyter Notebook:
```
python
Copy
Edit
import pandas as pd
import os

#=== CONFIGURATION ===
INPUT_CSV = "notes.csv"
REVIEW_OUTPUT_CSV = "manual_review.csv"
RELEVANT_COLUMNS = ["TOTAL_FLAGS", "PAT_ENC_CSN_ID", "NOTE_ID", "NOTE_TEXT"]

#=== Load data ===
df = pd.read_csv(INPUT_CSV, usecols=RELEVANT_COLUMNS)
df = df.sort_values(by="TOTAL_FLAGS", ascending=False)

Load or initialise manual review file
if os.path.exists(REVIEW_OUTPUT_CSV):
    reviewed_df = pd.read_csv(REVIEW_OUTPUT_CSV)
    reviewed_csns = set(reviewed_df["PAT_ENC_CSN_ID"])
else:
    reviewed_df = pd.DataFrame(columns=["PAT_ENC_CSN_ID", "TOTAL_FLAGS", "MANUAL_REVIEW"])
    reviewed_csns = set()

#=== Review Loop ===
csn_groups = df.groupby("PAT_ENC_CSN_ID")

for csn, group in csn_groups:
    if csn in reviewed_csns:
        continue

    decision = None
    has_question = False
    for _, row in group.iterrows():
        print(f"\n--- NOTE ID: {row['NOTE_ID']} | TOTAL_FLAGS: {row['TOTAL_FLAGS']} ---")
        print(row['NOTE_TEXT'])
        
        while True:
            user_input = input("Label this note (1 = positive, 0 = negative, ? = unsure): ").strip()
            if user_input in ["1", "0", "?"]:
                break
            print("Invalid input. Please type 1, 0, or ?.")

        if user_input == "1":
            decision = "1"
            break  # Stop reviewing this CSN
        elif user_input == "?":
            has_question = True
        # If "0", just continue

    # Finalise decision for this CSN
    if decision != "1":
        if has_question:
            decision = "?"
        else:
            decision = "0"

    # Save decision
    total_flags = group["TOTAL_FLAGS"].iloc[0]
    reviewed_df = pd.concat([reviewed_df, pd.DataFrame([{
        "PAT_ENC_CSN_ID": csn,
        "TOTAL_FLAGS": total_flags,
        "MANUAL_REVIEW": decision
    }])], ignore_index=True)

    # Save progress after each CSN
    reviewed_df.to_csv(REVIEW_OUTPUT_CSV, index=False)
    print(f"✓ Saved CSN {csn} as {decision}")

print("✅ All done!")
```
🔄 How it works:
You see one note at a time.

If you type 1, that CSN is marked positive and no more notes for that CSN will be shown.

If you type 0 or ?, it continues until all notes are reviewed.

If there’s at least one ? and no 1, it’s marked ?.

If all 0, it’s marked 0.

It saves your progress after every CSN so you can resume any time.

Let me know if you want it to:

Hide long notes until you press enter.

Filter out short notes or specific keywords.

Track how many you’ve done vs how many left.

Happy reviewing 👀

---
# ME

two improvements, don't show discharge summary (which is IP_NOTE_TYPE_C of 5 unless all other notes have been read)
can we highlight any phrases in the note - "PHRASE 1","PHRASE 2","PHRASE 3"

Here's my current code
```
import pandas as pd
import os


INPUT_CSV = "1 row per note.csv"
REVIEW_OUTPUT_CSV = "manual_review.csv"
ALL_COLUMNS = ["TOTAL_FLAGS", "PAT_ENC_CSN_ID", "NOTE_ID","IP_NOTE_TYPE_C","CRT_DTTM", "NOTE_TEXT"]
RELEVANT_COLUMNS = ["TOTAL_FLAGS", "PAT_ENC_CSN_ID", "NOTE_ID","NOTE_TEXT"]


df = pd.read_csv(INPUT_CSV, header=None)
df.columns = ALL_COLUMNS
df = df[RELEVANT_COLUMNS].copy()
df = df.sort_values(by="TOTAL_FLAGS", ascending=False)


if os.path.exists(REVIEW_OUTPUT_CSV):
    reviewed_df = pd.read_csv(REVIEW_OUTPUT_CSV)
    reviewed_csns = set(reviewed_df["PAT_ENC_CSN_ID"])
else:
    reviewed_df = pd.DataFrame(columns=["PAT_ENC_CSN_ID", "TOTAL_FLAGS", "MANUAL_REVIEW"])
    reviewed_csns = set()

csn_groups = df.groupby("PAT_ENC_CSN_ID")

for csn, group in csn_groups:
    if csn in reviewed_csns:
        continue

    decision = None
    has_question = False
    for _, row in group.iterrows():
        print(f"\n--- NOTE ID: {row['NOTE_ID']} | TOTAL_FLAGS: {row['TOTAL_FLAGS']} ---")
        print(row['NOTE_TEXT'])
        
        while True:
            user_input = input("Label this note (1 = positive, 0 = negative, ? = unsure): ").strip()
            if user_input in ["1", "0", "?"]:
                break
            print("Invalid input. Please type 1, 0, or ?.")

        if user_input == "1":
            decision = "1"
            break  # Stop reviewing this CSN
        elif user_input == "?":
            has_question = True
        # If "0", just continue

    # Finalise decision for this CSN
    if decision != "1":
        if has_question:
            decision = "?"
        else:
            decision = "0"

    # Save decision
    total_flags = group["TOTAL_FLAGS"].iloc[0]
    reviewed_df = pd.concat([reviewed_df, pd.DataFrame([{
        "PAT_ENC_CSN_ID": csn,
        "TOTAL_FLAGS": total_flags,
        "MANUAL_REVIEW": decision
    }])], ignore_index=True)

    # Save progress after each CSN
    reviewed_df.to_csv(REVIEW_OUTPUT_CSV, index=False)
    print(f"✓ Saved CSN {csn} as {decision}")

print("✅ All done!")
```

---

# ChatGPT
Great start — here's a cleaned-up and enhanced version of your code with the two requested improvements:

✅ Improvements added:
Don't show discharge summaries (IP_NOTE_TYPE_C == 5) until all other notes are exhausted.

Highlight keywords ("PHRASE 1","PHRASE 2","PHRASE 3") in bold yellow using ANSI escape codes (for most Jupyter notebooks or terminals).

💡 Paste this into your Jupyter notebook:
```
python
Copy
Edit
import pandas as pd
import os
import re

# === CONFIGURATION ===
INPUT_CSV = "1 row per note.csv"
REVIEW_OUTPUT_CSV = "manual_review.csv"
ALL_COLUMNS = ["TOTAL_FLAGS", "PAT_ENC_CSN_ID", "NOTE_ID", "IP_NOTE_TYPE_C", "CRT_DTTM", "NOTE_TEXT"]
HIGHLIGHT_TERMS = ["PHRASE 1","PHRASE 2","PHRASE 3"]

# === Load data ===
df = pd.read_csv(INPUT_CSV, header=None)
df.columns = ALL_COLUMNS
df = df.copy()
df = df.sort_values(by="TOTAL_FLAGS", ascending=False)

RELEVANT_COLUMNS = ["TOTAL_FLAGS", "PAT_ENC_CSN_ID", "NOTE_ID", "IP_NOTE_TYPE_C", "NOTE_TEXT"]
df = df[RELEVANT_COLUMNS]

# Load or initialise manual review file
if os.path.exists(REVIEW_OUTPUT_CSV):
    reviewed_df = pd.read_csv(REVIEW_OUTPUT_CSV)
    reviewed_csns = set(reviewed_df["PAT_ENC_CSN_ID"])
else:
    reviewed_df = pd.DataFrame(columns=["PAT_ENC_CSN_ID", "TOTAL_FLAGS", "MANUAL_REVIEW"])
    reviewed_csns = set()

# Highlighting function
def highlight_text(text):
    for term in HIGHLIGHT_TERMS:
        pattern = re.compile(rf'\b({term})\b', flags=re.IGNORECASE)
        text = pattern.sub(r'\033[1;33m\1\033[0m', text)
    return text

# === Review Loop ===
csn_groups = df.groupby("PAT_ENC_CSN_ID")

for csn, group in csn_groups:
    if csn in reviewed_csns:
        continue

    decision = None
    has_question = False

    # Prioritise non-discharge notes first (IP_NOTE_TYPE_C != 5)
    non_discharge_notes = group[group["IP_NOTE_TYPE_C"] != 5]
    discharge_notes = group[group["IP_NOTE_TYPE_C"] == 5]
    combined_notes = pd.concat([non_discharge_notes, discharge_notes], ignore_index=True)

    for _, row in combined_notes.iterrows():
        print(f"\n--- NOTE ID: {row['NOTE_ID']} | TOTAL_FLAGS: {row['TOTAL_FLAGS']} | TYPE: {row['IP_NOTE_TYPE_C']} ---")
        print(highlight_text(row['NOTE_TEXT']))

        while True:
            user_input = input("Label this note (1 = positive, 0 = negative, ? = unsure): ").strip()
            if user_input in ["1", "0", "?"]:
                break
            print("Invalid input. Please type 1, 0, or ?.")

        if user_input == "1":
            decision = "1"
            break  # Stop reviewing this CSN
        elif user_input == "?":
            has_question = True
        # If "0", continue

    # Finalise decision for this CSN
    if decision != "1":
        if has_question:
            decision = "?"
        else:
            decision = "0"

    # Save decision
    total_flags = group["TOTAL_FLAGS"].iloc[0]
    reviewed_df = pd.concat([reviewed_df, pd.DataFrame([{
        "PAT_ENC_CSN_ID": csn,
        "TOTAL_FLAGS": total_flags,
        "MANUAL_REVIEW": decision
    }])], ignore_index=True)

    # Save progress after each CSN
    reviewed_df.to_csv(REVIEW_OUTPUT_CSV, index=False)
    print(f"✓ Saved CSN {csn} as {decision}")

print("✅ All done!")
```
🟡 Notes:
You’ll see the keywords in yellow bold (provided your notebook supports ANSI escape codes).

Discharge summaries (type 5) are shown only after you finish all other notes for a CSN.

You can expand the HIGHLIGHT_TERMS list any time.

Let me know if you want to add:

A progress bar?

A flag for long notes to preview before showing all?

Timestamps for when each CSN was reviewed?

Happy tagging 🧠

---

# Me

the sort function isn't working
I've tried adding something but it hasn't worked


```
# Load or initialise manual review file
if os.path.exists(REVIEW_OUTPUT_CSV):
    reviewed_df = pd.read_csv(REVIEW_OUTPUT_CSV)
    reviewed_csns = set(reviewed_df["PAT_ENC_CSN_ID"])
else:
    reviewed_df = pd.DataFrame(columns=["PAT_ENC_CSN_ID", "TOTAL_FLAGS", "MANUAL_REVIEW"])
    reviewed_csns = set()

# Highlighting function
def highlight_text(text):
    for term in HIGHLIGHT_TERMS:
        pattern = re.compile(rf'\b({term})\b', flags=re.IGNORECASE)
        text = pattern.sub(r'\033[1;33m\1\033[0m', text)
    return text

# === Review Loop ===
csn_groups = df.groupby("PAT_ENC_CSN_ID").sort_values(by="TOTAL_FLAGS", ascending=False)

for csn, group in csn_groups:
    if csn in reviewed_csns:
        continue

    decision = None
    has_question = False

    # Prioritise non-discharge notes first (IP_NOTE_TYPE_C != 5)
    non_discharge_notes = group[group["IP_NOTE_TYPE_C"] != 5]
    discharge_notes = group[group["IP_NOTE_TYPE_C"] == 5]
    combined_notes = pd.concat([non_discharge_notes, discharge_notes], ignore_index=True)

    for _, row in combined_notes.iterrows():
        print(f"\n--- NOTE ID: {row['NOTE_ID']} | TOTAL_FLAGS: {row['TOTAL_FLAGS']} | TYPE: {row['IP_NOTE_TYPE_C']} ---")
        print(highlight_text(row['NOTE_TEXT']))

        while True:
            user_input = input("Label this note (1 = positive, 0 = negative, ? = unsure): ").strip()
            if user_input in ["1", "0", "?"]:
                break
            print("Invalid input. Please type 1, 0, or ?.")

        if user_input == "1":
            decision = "1"
            break  # Stop reviewing this CSN
        elif user_input == "?":
            has_question = True
        # If "0", continue

    # Finalise decision for this CSN
    if decision != "1":
        if has_question:
            decision = "?"
        else:
            decision = "0"

    # Save decision
    total_flags = group["TOTAL_FLAGS"].iloc[0]
    reviewed_df = pd.concat([reviewed_df, pd.DataFrame([{
        "PAT_ENC_CSN_ID": csn,
        "TOTAL_FLAGS": total_flags,
        "MANUAL_REVIEW": decision
    }])], ignore_index=True)

    # Save progress after each CSN
    reviewed_df.to_csv(REVIEW_OUTPUT_CSV, index=False)
    print(f"✓ Saved CSN {csn} as {decision}")

print("✅ All done!")
```

---
# ChatGPT

You're nearly there — the issue is that groupby(...).sort_values(...) won't work because groupby returns a DataFrameGroupBy object, which doesn't have a sort_values method.

Instead, you need to sort the original DataFrame first, then group it — like this:

✅ Fix the groupby sorting with this line:
```
csn_groups = df.sort_values(by="TOTAL_FLAGS", ascending=False).groupby("PAT_ENC_CSN_ID")
```
🔧 Replace your current review loop block with:
```
# === Review Loop ===
csn_groups = df.sort_values(by="TOTAL_FLAGS", ascending=False).groupby("PAT_ENC_CSN_ID")
```
This ensures that CSNs with more flags are reviewed first. The groupby will preserve the first appearance of each CSN, which, after sorting, will be the one with the highest TOTAL_FLAGS.

Bonus: In case CSNs have different TOTAL_FLAGS for some reason
You can take the max flag per CSN and use that for sorting before grouping:

```
df['MAX_FLAGS'] = df.groupby("PAT_ENC_CSN_ID")["TOTAL_FLAGS"].transform("max")
csn_groups = df.sort_values(by="MAX_FLAGS", ascending=False).groupby("PAT_ENC_CSN_ID")
```
Let me know if you want to persist that MAX_FLAGS into the review file too.

You're close — just needed to sort before grouping.
