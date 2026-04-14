# PetLog Screens

The app uses a simple layout: a fixed sidebar (220px wide) on the left with pet navigation, and a content area on the right. No auth, no routing library needed — use React state to track the current view.

---

## Sidebar

- **Header:** App name "PetLog" in H2 teal-600, 24px bottom padding
- **Pet list:** Each pet as a row: Pet Avatar (32px) + pet name (Body) + species in Small. Active pet has teal-50 background. Click to select pet.
- **Add pet button:** "Add pet" at bottom of sidebar (Ghost button with + icon). Opens the Add Pet modal.

---

## Main Content Area

When a pet is selected, show a tab bar with 4 tabs: **Overview**, **Vaccinations**, **Weight**, **Medications**.

---

### Overview Tab

**Pet header section:**
- Pet Avatar (80px) + Pet name (H1) + species and breed (Body gray-500) + age calculated from DOB (Small, e.g., "3 years, 2 months old")
- Right side: "Edit" button (Secondary) opens Edit Pet modal. "Delete" button (Ghost, red-500 text) opens delete confirmation modal.

**Stats row (3 cards in a row):**
1. **Next vaccination** — Show name and date of the earliest upcoming vaccination. Badge: green "Up to date" if none due in next 30 days, orange "Due soon" if within 30 days, red "Overdue" if past due. If no vaccinations at all, show "—".
2. **Last vet visit** — Date of the most recent vaccination or weight entry (whichever is more recent). Small text: relative date ("2 weeks ago"). If none, show "No visits recorded".
3. **Current weight** — Most recent weight entry value + unit. Small text: trend arrow ↑ or ↓ compared to previous entry. If only one entry, show "—" for trend. If none, show "No weight recorded".

**Recent activity section:**
- H3: "Recent activity"
- List of the last 5 events across all types (vaccinations, weight logs, medications), sorted by date desc.
- Each row: icon (syringe for vaccination, scale for weight, pill for medication) + description + date (Small mono).
- If no activity: show empty state with content from content.json key `empty_overview`.

---

### Vaccinations Tab

**Header row:** H2 "Vaccinations" (left) + "Add vaccination" button (Primary, right)

**Vaccination table:**
- Columns: Vaccine name, Date given (mono), Next due (mono + badge), Vet/clinic, Notes
- Next due badge: green "Up to date" if > 30 days away, orange "Due soon" if ≤ 30 days, red "Overdue" if past
- Sort by date given, newest first
- Click row: no action (data is visible in table)
- Each row has a delete icon button (Ghost, trash icon) on the far right — opens confirmation modal

**Empty state:** Use content.json key `empty_vaccinations`

**Add Vaccination modal:**
- Fields: Vaccine name (text, required), Date given (date, required), Next due date (date, optional), Vet/clinic (text, optional), Notes (textarea, optional)
- Footer: "Cancel" (Secondary) + "Add vaccination" (Primary)
- Validation: name and date required. Show inline errors.

---

### Weight Tab

**Header row:** H2 "Weight log" (left) + "Log weight" button (Primary, right)

**Weight chart:**
- Simple line chart (use Recharts) showing weight over time
- X axis: dates. Y axis: weight value with unit
- Teal-600 line, dots on data points
- Tooltip on hover: date + exact weight
- If fewer than 2 entries: show message "Add at least 2 entries to see the chart" instead of chart

**Weight table (below chart):**
- Columns: Date (mono), Weight (mono, bold), Change (green +X.X or red -X.X compared to previous, or "—" for first entry), Notes
- Sort by date, newest first
- Each row has delete icon button → confirmation modal

**Empty state:** Use content.json key `empty_weight`

**Log Weight modal:**
- Fields: Weight (number, required), Unit (select: "lb" or "kg", default to the pet's last used unit or "lb"), Date (date, default today, required), Notes (textarea, optional)
- Footer: "Cancel" (Secondary) + "Log weight" (Primary)

---

### Medications Tab

**Header row:** H2 "Medications" (left) + "Add medication" button (Primary, right)

**Medication cards (grid, 2 columns):**
Each medication as a Card:
- Medication name (H3)
- Dosage (Body mono)
- Frequency (Body, e.g., "Twice daily")
- Start date (Small mono)
- End date (Small mono, or badge "Ongoing" in green if no end date)
- Status badge: green "Active" if no end date or end date in future, gray "Completed" if end date is past
- Notes (Small gray-500, max 2 lines, truncated)
- Delete button (Ghost, trash icon) in top-right corner of card → confirmation modal

**Empty state:** Use content.json key `empty_medications`

**Add Medication modal:**
- Fields: Medication name (text, required), Dosage (text, required, e.g., "10mg"), Frequency (select: "Once daily", "Twice daily", "Three times daily", "Weekly", "Monthly", "As needed" — required), Start date (date, required, default today), End date (date, optional), Notes (textarea, optional)
- Footer: "Cancel" (Secondary) + "Add medication" (Primary)

---

## Add Pet Modal

Triggered from sidebar "Add pet" button.
- Fields: Name (text, required), Species (select: "Dog", "Cat", "Bird", "Rabbit", "Fish", "Reptile", "Other" — required), Breed (text, optional), Date of birth (date, optional), Color/markings (text, optional)
- Footer: "Cancel" (Secondary) + "Add pet" (Primary)
- After adding: pet appears in sidebar and is auto-selected

---

## Edit Pet Modal

Same fields as Add Pet, pre-populated with current values.
- Footer: "Cancel" (Secondary) + "Save changes" (Primary)

---

## Delete Confirmation Modal

Used for deleting pets, vaccinations, weight entries, and medications.
- H2: content.json key `delete_confirm_title`
- Body: content.json key `delete_confirm_{type}` where type is pet/vaccination/weight/medication
- Footer: "Cancel" (Secondary) + "Delete" (Destructive)

---

## No Pet Selected State

When the app loads (no pet selected yet), the main content area shows:
- Empty state using content.json key `empty_no_pet_selected`
