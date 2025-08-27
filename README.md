# Market Sessions
This indicator displays the **major Forex market sessions** (Tokyo, London, New York, Sydney) directly on your MT5 chart.  
It supports **Daylight Saving Time (DST) adjustments**, customizable countdown to the next session,  
and session overlap display (e.g. *Tokyo + London → New York*).

---

## 📑 Table of Contents
- [Features](#features)
- [Installation](#installation)
- [Usage](#usage)
- [Inputs](#inputs)
- [Customization](#customization)
- [Script Code](#script-code)
- [Contributing](#contributing)
- [License](#license)

---

## ✨ Features
- Display active Forex sessions (Tokyo, London, New York, Sydney).
- Show overlaps (e.g. `Tokyo + London`).
- Countdown to the next session (`London → New York (150 min)`).
- Support for DST adjustments (per session).
- Customizable position, distance, and font styling.
- Alert types
  - 🔕 Disable
  - 🔔 Sound only
  - 📲 Push notification (MetaTrader mobile app)

---

## 🚀 Installation
1. Download the Script: Download the `Market Sessions.mq5` file from this repository.
2. Open MetaTrader 5
   - Launch MetaTrader 5.
   - Go to `File` -> `Open Data Folder`.
3. Place the Script
   - Navigate to `MQL5` -> `Indicators`.
   - Copy the `Market Sessions.mq5` file into the Indicators folder.
4. Refresh MetaTrader 5
   - Restart MetaTrader 5 or right-click in the Navigator window and select Refresh.

---

## 📖 Usage
1. **Load the Indicator**
   - Open **MetaTrader 5**.
   - In the *Navigator* window, find the `Market Session` indicator under `Indicators`.
   - Drag and drop the indicator onto the chart you want to monitor.

2. **Configure Settings**
   - Adjust **Countdown_Minutes** to decide how early the next session countdown should appear.
   - Choose **Alert_Type** (Disable, Sound, or Notification).
   - Set **DST options** (Sydney, London, New York) according to current daylight saving rules.
   - Customize **Text Position**, **X/Y Distance**, and **Font** settings for display.

3. **View Market Sessions**
   - The current active session will be displayed on the chart.
   - Overlaps will appear as combined sessions, e.g. `Tokyo + London → New York (285 min)`.
   - A countdown will show how many minutes until the next session begins.

4. **Alerts**
   - If alerts are enabled, you will receive either a sound alert or a push notification when a session changes.
   - Example alert
     ```
     Session Update  London  to  New York
     ```

---

## ⚙️ Inputs
### Settings
- **Countdown_Minutes** → Minutes before next session to show countdown.
- **Alert_Type** → Choose alert type (disable, sound, notification).

### DST Settings
- **DST_Sydney** → Add +1h shift.
- **DST_London** → Subtract -1h shift.
- **DST_NewYork** → Subtract -1h shift.

### Position
- **Text_Position** → Where the text will appear on the chart.
- **X_Distance / Y_Distance** → Offset from chart corner/center.

### Font
- **Font_Color** → Session text color.
- **Font_Size** → Session text size.

---

## 💻 Script Code
Below is the MQL5 code used to create the "Market Sessions" indicator
```mql5

```

---

## 🤝 Contributing
Contributions are welcome! If you have any improvements, bug fixes, or new features to suggest, please follow these steps

1. Fork the repository.
2. Create a new branch for your feature or fix.
3. Commit your changes with descriptive commit messages.
4. Push your branch to your forked repository.
4. Open a pull request to the main repository's main branch.
5. Please ensure your code follows the existing code style and includes comments where necessary.

Please ensure your code follows the existing code style and includes comments where necessary.

---

## 📜 License
This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.
