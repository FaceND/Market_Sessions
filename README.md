# Market Sessions
This indicator displays the **major Forex market sessions** (Tokyo, London, New York, Sydney) directly on your MT5 chart.  
It supports **Daylight Saving Time (DST) adjustments**, customizable countdown to the next session,  
and session overlap display (e.g. *Tokyo  +  London  →  New York*).

---

## 📑 Table of Contents
- [Features](#-features)
- [Installation](#-installation)
- [Usage](#-usage)
- [Inputs](#-inputs)
- [Script Code](#-script-code)
- [Contributing](#-contributing)
- [License](#-license)

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

## 📝 Inputs

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
//+------------------------------------------------------------------+
//|                                              Market Sessions.mq5 |
//|                                           Copyright 2025, FaceND |
//|                        https://github.com/FaceND/Market_Sessions |
//+------------------------------------------------------------------+
#property copyright "Copyright 2025, FaceND"
#property link      "https://github.com/FaceND/Market_Sessions"
#property version   "1.5"

#property description "An indicator that displays major Forex market sessions:"
#property description "Tokyo, London, New York, and Sydney with optional DST adjustments."
#property description ""
#property description "It shows overlaps (e.g. Tokyo +  London), countdown to the next session,"
#property description "and allows customizing position, font, and colors on the chart."
#property description "Alerts can be triggered when sessions change, via sound or push notifications."

#property indicator_chart_window
#property indicator_plots 0
#property strict

enum ENUM_POSITION
{
 POSITION_CENTER_UPPER = 4,  // Upper chart center
 POSITION_CENTER_LOWER = 5,  // Lower chart center
 POSITION_LEFT_UPPER   = 0,  // Upper left corner
 POSITION_LEFT_LOWER   = 1,  // Lower left corner
 POSITION_RIGHT_LOWER  = 2,  // Upper right corner
 POSITION_RIGHT_UPPER  = 3   // Upper right corner
};

enum ENUM_ALERT
{
 ALERT_DISABLE, // 🔕  Disable
 ALERT_SOUND,   // 🔔  Sound only
 ALERT_NOTI,    // 📲  Push notification
};

input group "SETTINGS"
input int            Countdown_Minutes  = 60;           // Countdown next session (minutes)
input ENUM_ALERT         Alert_Type     = ALERT_SOUND;  // Alert type

input group "DST SETTINGS"
input bool               DST_Sydney     = false;        // Sydney  DST   ->  UTC shift +1
input bool               DST_London     = true;         // London  DST   ->  UTC shift -1
input bool               DST_NewYork    = true;         // New York  DST ->  UTC shift -1

input group "POSITION"
input ENUM_POSITION      Text_Position  = POSITION_CENTER_UPPER; // Position on the chart
input int                Y_Distance     = 4;            // Y distance
input int                X_Distance     = 3;            // X distance

input group "FONT"
input color              Font_Color     = clrWhite;    // Color
input int                Font_Size      = 10;          // Size

#define OBJ_NAME "Session_Now"

// ---- Data structures ----
struct Session 
{
 string name;       // Session name
 int    startHour;  // Start hours UTC
 int    endHour;    // End hours UTC
 int    shift;      // Shift DST time
};

Session sessions[4];
string SessionText = "";

string LastActive = "";
int LastMinute = -1;

int text_width;
int approxChar_width;

bool isCenter = true;
//+------------------------------------------------------------------+
//| Custom indicator initialization function                         |
//+------------------------------------------------------------------+
int OnInit()
  {
   /* -------------------------------
   ==================================
   +--------------------------------+
   |         Market Sessions        |
   |       (Base hours in UTC)      |
   |------------+---------+---------|
   |   Session  |  Start  |   End   |
   |------------+---------+---------|
   |   Sydney   |   21    |   06    |
   |   Tokyo    |   00    |   09    |
   |   London   |   08    |   17    |
   |   NewYork  |   13    |   22    |
   +------------+---------+---------+
   ==================================
   --------------------------------*/
   sessions[0].name      = "Sydney";
   sessions[0].startHour = 21;
   sessions[0].endHour   = 6;

   sessions[1].name      = "Tokyo";
   sessions[1].startHour = 0;
   sessions[1].endHour   = 9;

   sessions[2].name      = "London";
   sessions[2].startHour = 8;
   sessions[2].endHour   = 17;

   sessions[3].name      = "New York";
   sessions[3].startHour = 13;
   sessions[3].endHour   = 22;
   //--------------------------------

   SetDstShift();
   approxChar_width = (int)(Font_Size / 2);

   EventSetTimer(1);
   return(INIT_SUCCEEDED);
  }
//+------------------------------------------------------------------+
//| Custom indicator deinitialization function                       |
//+------------------------------------------------------------------+
void OnDeinit(const int reason)
  {
   EventKillTimer();
   ObjectDelete(0, OBJ_NAME);
  }
//+------------------------------------------------------------------+
//| Custom indicator iteration function                              |
//+------------------------------------------------------------------+
int OnCalculate(const int           rates_total,
                const int       prev_calculated,
                const datetime          &time[],
                const double            &open[],
                const double            &high[],
                const double             &low[],
                const double           &close[],
                const long       &tick_volume[],
                const long            &volume[],
                const int             &spread[])
  {
   return rates_total;
  }
//+------------------------------------------------------------------+
//| Custom indicator alert trigger function                          |
//+------------------------------------------------------------------+
bool OnAlert(const string message)
  {
   bool success = true;

   const datetime publish_time = TimeLocal();
   string timeStr = TimeToString(publish_time, TIME_MINUTES);

   //--- Session change Alert
   if(Alert_Type != ALERT_DISABLE)
     {
      if(Alert_Type == ALERT_NOTI)
        {
         string content = StringFormat(
            "📢 Session Update  %s"
            "\n"
            "\n🌍 %s"
            "\n"
            "\nSent via MetaTrader 5 Notification System",
            timeStr,
            message
         );
         success = SendNotification(content);
        }
      Print("Session Update  ", message);
      PlaySound("request.wav");
     }
   return success;
  }
//+------------------------------------------------------------------+
//| Custom indicator timer function                                  |
//+------------------------------------------------------------------+
void OnTimer()
  {
   datetime now = TimeGMT();
   MqlDateTime tm;
   TimeToStruct(now,tm);

   //--- Only run once when the minute changes (00 seconds)
   if(tm.min != LastMinute)
     {
      LastMinute = tm.min;
      string activeLine = UpdateSession(tm);

      ObjectSetLabel(OBJ_NAME, SessionText);

      if(LastActive != "" && StringFind(LastActive, activeLine) == -1)
        {
         string message = StringFormat("\"%s\"  to  \"%s\"", LastActive, activeLine);
         OnAlert(message);
        }
      LastActive = activeLine;
     }
   if(isCenter)
     {
      ObjectAdjustLabel(OBJ_NAME);
     }
  }
//+------------------------------------------------------------------+
//| Configures and sets text for a label object                      |
//+------------------------------------------------------------------+
bool ObjectSetLabel(const string obj_name, const string text)
  {
   if(ObjectFind(0, obj_name) == -1)
     {
      if(!ObjectCreate(0, obj_name, OBJ_LABEL, 0, 0, 0))
        {
         Print("Error creating label: ", obj_name,
               " Error code: ", GetLastError());
         return false;
        }
      else
        {
         int pos = GetPosition(Text_Position);
         ObjectSetInteger(0, obj_name, OBJPROP_CORNER,            pos);
         ObjectSetInteger(0, obj_name, OBJPROP_FONTSIZE,    Font_Size);
         ObjectSetInteger(0, obj_name, OBJPROP_COLOR,      Font_Color);
         ObjectSetInteger(0, obj_name, OBJPROP_SELECTABLE,      false);
         ObjectSetInteger(0, obj_name, OBJPROP_SELECTED,        false);
         ObjectSetInteger(0, obj_name, OBJPROP_HIDDEN,           true);
         ObjectSetInteger(0, obj_name, OBJPROP_YDISTANCE,  Y_Distance);
         if(!isCenter)
           {
            ObjectSetInteger(0, obj_name, OBJPROP_XDISTANCE,  X_Distance);
           }
        }
     }
   ObjectSetString(0, obj_name, OBJPROP_TEXT, text);
   return true;
  }
//+------------------------------------------------------------------+
//| Adjust the horizontal center position of the label object        |
//+------------------------------------------------------------------+
void ObjectAdjustLabel(string obj_name)
  {
   //+---------------------------------------------------------------+
   const int chart_width = (int)ChartGetInteger(0,
                            CHART_WIDTH_IN_PIXELS, 0);
   //+---------------------------------------------------------------+
   int x_offset = (chart_width - text_width) / 2;
   ObjectSetInteger(0, obj_name, OBJPROP_XDISTANCE, x_offset);

   ChartRedraw();
  }
//+------------------------------------------------------------------+
//| Determine text position index and center alignment               |
//+------------------------------------------------------------------+
int GetPosition(ENUM_POSITION position = 1)
  {
   if(Text_Position == 4)
     {
      isCenter = true;
      return 0;
     }
   if(Text_Position == 5)
     {
      isCenter = true;
      return 1;
     }
   isCenter = false;
   return position;
  }
//+------------------------------------------------------------------+
//| Find and update the current and next trading session             |
//+------------------------------------------------------------------+
string UpdateSession(const MqlDateTime &tm)
  {
   int hour = tm.hour, minute = tm.min, nowMin = hour*60 + minute;

   string active    = "";
   string next      = ""; 
   int    minToNext = 10000;

   for(int i = 0; i < 4; i++)
     {
      int shift = sessions[i].shift;
      int start = NormalizeHour(sessions[i].startHour + shift);
      int end   = NormalizeHour(sessions[i].endHour + shift);

      bool isActive = false;

      if(start < end)
        {
         isActive = (hour >= start && hour < end);
        }
      else
        {
         isActive = (hour >= start || hour < end);
        }

      if(isActive)
        {
         if(active != "")
           {
            active += "  +  ";
           }
         active += sessions[i].name;
        }

      int startMin = start * 60;
      int diff = (startMin - nowMin + 1440) % 1440;

      if(diff > 0 && diff < minToNext)
        {
         minToNext = diff;
         next = sessions[i].name;
        }
     }

   if(active == "") active = "Closed";

   if(minToNext <= Countdown_Minutes && Countdown_Minutes != 0)
     {
      SessionText = active +"  →  "+ next +"  ("+
                     IntegerToString(minToNext) +" min)";
     }
   else
     {
      SessionText = active;
     }

   if(isCenter)
     {
      text_width = StringLen(SessionText) * approxChar_width;
     }
   return active;
  }
//+------------------------------------------------------------------+
//| Set each session’s DST shift                                     |
//+------------------------------------------------------------------+
void SetDstShift()
  {
   for(int i = 0; i < 4; i++)
     {
      sessions[i].shift = GetDstShift(i);
     }
  }
//+------------------------------------------------------------------+
//| Normalize any hour into the range                                |
//+------------------------------------------------------------------+
int NormalizeHour(const int hour)
  {
   int range = hour % 24;
   if(range < 0) range += 24;

   return range;
  }
//+------------------------------------------------------------------+
//| Return DST hour adjustment for the given session                 |
//+------------------------------------------------------------------+
int GetDstShift(const int index)
  {
   if(index == 0) return (DST_Sydney  ? +1 : 0);
   if(index == 2) return (DST_London  ? -1 : 0);
   if(index == 3) return (DST_NewYork ? -1 : 0);

   return 0;
  }
//+------------------------------------------------------------------+
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
