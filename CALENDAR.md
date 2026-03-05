FullCalendar works really nicely with React, and the setup is fairly straightforward.

1. **Installing the required packages**
2. **Creating a basic calendar component**
3. **Adding dummy events**
4. **Explaining the key pieces**

---

# 1. Install FullCalendar Packages

Run this in your project:

```bash
npm install @fullcalendar/react
npm install @fullcalendar/daygrid
npm install @fullcalendar/timegrid
npm install @fullcalendar/interaction
```

These plugins give you:

| Package                     | Purpose             |
| --------------------------- | ------------------- |
| `@fullcalendar/react`       | React wrapper       |
| `@fullcalendar/daygrid`     | Month calendar view |
| `@fullcalendar/timegrid`    | Week/day views      |
| `@fullcalendar/interaction` | Clicking, dragging  |

---

# 2. Create a Calendar Component

Example file:

```
src/components/Calendar.jsx
```

```javascript
import FullCalendar from "@fullcalendar/react";
import dayGridPlugin from "@fullcalendar/daygrid";
import timeGridPlugin from "@fullcalendar/timegrid";
import interactionPlugin from "@fullcalendar/interaction";

export default function Calendar() {
  const events = [
    {
      title: "Leg Day",
      start: "2026-03-07",
    },
    {
      title: "Chest Workout",
      start: "2026-03-09T10:30:00",
      end: "2026-03-09T12:00:00",
    },
    {
      title: "Run 5 Miles",
      start: "2026-03-10T07:00:00",
    },
    {
      title: "Yoga",
      start: "2026-03-12",
      end: "2026-03-13",
    },
  ];

  return (
    <div style={{ padding: "20px" }}>
      <h2>Workout Schedule</h2>

      <FullCalendar
        plugins={[dayGridPlugin, timeGridPlugin, interactionPlugin]}
        initialView="dayGridMonth"
        events={events}
        editable={true}
        selectable={true}
        headerToolbar={{
          left: "prev,next today",
          center: "title",
          right: "dayGridMonth,timeGridWeek,timeGridDay",
        }}
      />
    </div>
  );
}
```

---

# 3. Use the Calendar in Your App

Example `App.jsx`:

```javascript
import Calendar from "./components/Calendar";

function App() {
  return (
    <div>
      <Calendar />
    </div>
  );
}

export default App;
```

---

# 4. What the Important Props Do

### plugins

Registers calendar functionality.

```javascript
plugins={[dayGridPlugin, timeGridPlugin, interactionPlugin]}
```

---

### initialView

Controls the starting view.

```javascript
initialView="dayGridMonth"
```

Options:

```
dayGridMonth
timeGridWeek
timeGridDay
```

---

### events

Accepts an array of event objects.

```javascript
const events = [
  {
    title: "Leg Day",
    start: "2026-03-07",
  },
];
```

Event structure:

```
{
  title: string,
  start: ISO date,
  end?: ISO date
}
```

---

### selectable

Allows clicking or selecting time ranges.

```javascript
selectable={true}
```

---

### editable

Allows dragging events around.

```javascript
editable={true}
```

---

# 5. Example Event With More Fields

FullCalendar supports more options:

```javascript
{
  title: "Back Workout",
  start: "2026-03-11T09:00:00",
  end: "2026-03-11T10:30:00",
  backgroundColor: "green",
  borderColor: "darkgreen"
}
```

---

# 6. If You Want to Handle Clicks

Example:

```javascript
function handleEventClick(info) {
  alert("Event: " + info.event.title);
}
```

Add to calendar:

```javascript
<FullCalendar
  eventClick={handleEventClick}
/>
```

---

# 7. What It Will Look Like

You'll get:

✔ Month view
✔ Week view
✔ Day view
✔ Clickable events
✔ Drag and drop

All working out of the box.

---

# 8. Finalize

We will **load events from an API**, like:

```javascript
api.get("/workouts?start=DATE&end=DATE")
```

Then pass them into `events`.

Perfect for the **Django backend** you mentioned you're teaching with.

---

💡 Next steps:

* **Click day → add workout**
* **Drag workouts to different days**
* **Save to backend (Django)**
