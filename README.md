This is a README.md file tailored for your Swimming Performance Dashboard based on the features and technical structure implemented in version 5 of your project.

🏊 Swimming Performance Dashboard
A comprehensive, client-side web application designed for competitive swimmers to track race history, analyze pacing strategy through an AI-driven coach, and monitor progress toward County Qualifying times.

✨ Key Features
📊 Multi-Dimensional Overview: High-level statistics including total race counts, event variety, and venue history.

📈 Progression Tracking: Dynamic charts to visualize time improvements over months or years, with the ability to compare current marks against first swims, latest swims, or previous personal bests.

🏆 Personal Best (PB) Gallery: A dedicated view of all current records, automatically categorized by stroke and course (Short Course vs. Long Course).

🤖 AI Pacing Coach:

Analyzes split data to provide actionable feedback.

Contextual Logic: Tailors advice differently for 100m sprints (e.g., praising "White-White" controlled pacing) versus 200m+ endurance events.

Visual Color Coding: Uses Green (fast/burst), White (steady), and Red (fade) chips to visually represent race efficiency relative to the race average.

📋 Detailed Result History: A filterable, sortable table containing every recorded swim, competition name, and specific lap splits.

🎯 County Qualifying Analysis: Automatically compares your current PBs against standard Qualifying and Consideration times for specific age groups and genders.

⚙️ Data Management
This dashboard is data-less by default; it does not store your personal information on a server. Instead, it utilizes your browser's LocalStorage to keep data secure and private on your own device.

How to Load Data
Click the ⚙️ Data button in the top header.

Upload your race history file (my_swims.json).

Upload your qualifying times file (county_qt.json).

Click 💾 Save to Browser to initialize the dashboard.

Data Formats
The application requires data in specific JSON formats.

Example my_swims.json entry:

JSON
{
  "course": "S",
  "event": "100 Free",
  "date": "2025-11-15",
  "competition": "County Qualifier 2025",
  "venue": "Watford",
  "time": "1:17.43",
  "splits": [37.28, 40.15]
}
📱 Mobile Usage
The dashboard is optimized for mobile viewing. For the best experience on iOS or Android:

Documents by Readdle: Use this app to open the HTML file locally with full JavaScript permissions.

GitHub Pages: Host the HTML file as a static site (e.g., index.html) to access it via a dedicated URL.

Add to Home Screen: Once opened in Safari or Chrome, use the "Add to Home Screen" feature to treat the dashboard like a native app.

🛠️ Technical Details
Frontend: Pure HTML5, CSS3, and Vanilla JavaScript.

Charts: Powered by Chart.js for responsive visualizations.

Storage: Browser localStorage API.

Validation: The Data Manager includes built-in JSON validation to prevent dashboard crashes from malformed files.

Note: This dashboard is a private tool. Always keep a backup of your .json data files, as clearing your browser cache may remove stored records.
