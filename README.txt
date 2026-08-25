================================================================================
35 SQUADRON FLEET MANAGEMENT
index.html + README.txt
================================================================================

WHAT THIS IS

One HTML file containing three pages:

  1. Fleet Forecast   inspection and component life tracking, replaces the
                      Excel and PDF forecast sheet
  2. Fleet Ops        live flight line status board for the crew
  3. History          charts, tables and archived operations days

Everything runs in the browser. There is nothing to install and nothing to
compile. The only outside pieces are loaded from the internet when the page
opens: Firebase (for sharing between devices), jsPDF (PDF export) and SheetJS
(Excel export).

The aircraft list is stored in the app, not written into the code. Add or
retire a tail number and both boards update at once.


================================================================================
PART 1  PUT IT ONLINE
================================================================================

The app works fine with no Firebase at all, but it then saves to one browser
only. Everyone would see their own separate board. Follow Part 2 as well if the
crew needs a shared board.

IMPORTANT: Firebase only works when the page is served over http or https.
Double clicking index.html on your desktop opens it as file:// and Firebase
will refuse to connect. Use GitHub Pages or any other web host.

Option A, GitHub Pages, free
----------------------------
 1. Create a free account at github.com if you do not have one.
 2. Go to github.com/new. Name the repository, for example fleet-board.
    Choose Public or Private. Do not add a README. Click Create repository.
 3. On the next screen click "uploading an existing file".
 4. Drag index.html and README.txt into the box. Click Commit changes.
 5. Go to Settings, then Pages in the left menu.
 6. Under Build and deployment set Source to "Deploy from a branch",
    Branch to "main" and the folder to "/ (root)". Click Save.
 7. Wait about a minute, then reload the Pages screen. It shows the live
    address, which looks like:
        https://YOUR-USERNAME.github.io/fleet-board/
 8. Open that address on any device. Bookmark it on the shared screen.

To update the app later, open index.html in the repository, click the pencil
icon, paste the new version, and commit. Pages redeploys by itself.

Option B, any other static host
-------------------------------
Upload index.html to any web space that serves over https. Netlify Drop,
Cloudflare Pages and a normal web server all work the same way.

SECURITY NOTE
A GitHub Pages site is public even when the repository is private, unless you
pay for private Pages. Anyone who has the address can open the board. If that
is not acceptable for this data, host it inside the unit network instead, or
put the file behind a login on your own server.


================================================================================
PART 2  FIREBASE SETUP, FOR SHARED LIVE BOARDS
================================================================================

This is what makes the same board appear on the shared screen and on every
phone at once, updating within about a second.

Step 1  Create the project
 1. Go to console.firebase.google.com and sign in with a Google account.
 2. Click "Create a project". Name it, for example fleet35. Google Analytics
    is not needed, switch it off.

Step 2  Create the Realtime Database
 1. In the left menu open Build, then Realtime Database.
 2. Click "Create Database".
 3. Choose a location near you, for example Singapore.
 4. Choose "Start in test mode". Click Enable.

    Test mode means anyone who knows the database address can read and write
    it, and it stops working after 30 days. Read Part 3 before real data goes
    in, and set the rules there instead.

Step 3  Turn on anonymous sign in
 1. In the left menu open Build, then Authentication.
 2. Click "Get started".
 3. Open the "Sign-in method" tab.
 4. Click "Anonymous", switch it on, and Save.

    This is what lets the crew open the link and be signed in automatically,
    with no username and no password to hand out.

Step 4  Get the settings for the app
 1. Click the gear icon at the top left, then Project settings.
 2. Scroll to "Your apps" and click the web icon, which looks like </>.
 3. Give it any nickname. Do not tick Firebase Hosting. Click Register app.
 4. Firebase shows a block of settings that looks like this:

        const firebaseConfig = {
          apiKey: "AIzaSy...",
          authDomain: "fleet35.firebaseapp.com",
          databaseURL: "https://fleet35-default-rtdb.firebaseio.com",
          projectId: "fleet35",
          ...
        };

    Copy it. If databaseURL is missing from that block, open Realtime Database
    again and copy the address shown at the top of the Data tab.

Step 5  Put the settings into the app
 Either of these works.

 A. Edit the file. Open index.html in Notepad or any text editor. Near the top
    of the script, search for PASTE_API_KEY. Replace the four placeholder
    values with your own. Save and upload the file again.

 B. Do it from the app. Open the hosted page, click Settings in the top right,
    paste the whole config block into the Firebase config box, and click
    "Save config and reload". This stores it in that one browser, so it has to
    be repeated on each device. Option A is better for a shared link.

Step 6  Check it worked
 Open the page. The badge in the top bar should read "Live sync: connected"
 with a pulsing green dot. Open the same address on a phone, press a button,
 and watch the shared screen change.

 If it says "not configured", the settings did not take.
 If it says "permission denied", the database rules are blocking it, see below.


================================================================================
PART 3  DATABASE RULES
================================================================================

In the Firebase console open Realtime Database, then the Rules tab.

Test mode, which expires
------------------------
This is what Firebase gives you at the start. It works, but it expires and it
is wide open until then.

  {
    "rules": {
      ".read": "now < 1790000000000",
      ".write": "now < 1790000000000"
    }
  }

Signed in only, recommended
---------------------------
Because the app signs everyone in anonymously, this rule keeps out anything
that is not the app itself, and never expires:

  {
    "rules": {
      "fleet35": {
        ".read": "auth != null",
        ".write": "auth != null"
      }
    }
  }

Paste it and click Publish. Anonymous sign in must be switched on, Part 2
Step 3, or nobody will get in.

Be aware this still lets anyone who finds the page address read and write the
board, because anonymous sign in is automatic. It stops random scanners, not a
person with the link. For tighter control, switch Authentication to Email and
Password, create one account per crew member, and change auth != null to a
list of allowed user IDs. That needs a small code change to add a login box.


================================================================================
PART 4  FIRST RUN CHECKLIST
================================================================================

 1. Open the page. The nine aircraft (703, 704, 711, 712, 713, 714, 716, 719,
    722) are created automatically the first time.

 2. The Initial Fleet Data Setup screen opens by itself.
    - Pick a tail number tab at the top.
    - Fill in the current figures from your existing forecast sheet.
      Hours are typed as HH:MM, for example 96:30. Typing 96 also works and
      means 96:00.
      Leave a box empty for anything that does not apply. It shows as "--".
    - Engine S/N status note is the "(2nd O/H)" style text under the serial.
    - In the Eng OH/Rtd status note, type Rtd for a retired engine. The
      countdown then stops for that aircraft.
    - "Save and next" moves to the next aircraft.
    - "Copy from previous aircraft" fills the form from the tail before it,
      so you only change what differs. It never copies an engine serial.
    - "Add custom field" adds any item not on the standard list. It applies to
      every aircraft and can be hour based, landing based or date based.
 3. When all nine are entered, click "Finish setup".

 4. Fleet Ops needs nothing set up. Every aircraft starts in No Activity.
    Press "Mark Ready" on the ones that are serviceable and the board is live.

 5. Try one full cycle on the Ops page before real flying:
    Mark Ready, Take Off, Landed, Serviceable, Start Ground Preparation,
    watch it count down and turn green by itself.

 6. On the Forecast page, put the day's FLG hours and landings in, then press
    "Save today's log". A summary appears first, showing exactly what will
    change. Nothing is written until you confirm.


================================================================================
PART 5  HOW IT WORKS DAY TO DAY
================================================================================

FLEET OPS, the flight line board
--------------------------------
Five states only: Ready for Flying, In Air, Under Ground Preparation,
Under Maintenance, No Activity.

  Take Off        only from Ready. Records the time, counts the sortie, and
                  shows the WhatsApp "taxied out" message.
  Landed          asks the landing time, then whether the aircraft is
                  serviceable, then Ground Prep or straight to Ready.
                  If you close this dialog halfway the aircraft simply stays
                  In Air, so a sortie is never half recorded.
  Maintenance     one click, always available, no duration is ever asked.
                  It stays until you release it.
  Mark Ready      works from Ground Prep, Maintenance and No Activity.
                  Never from In Air.
  Start Prep      from Under Maintenance, asks the minutes, default 30.
  +5 / -5 Min     adjusts a running ground preparation countdown.
  Reset           back to No Activity after a confirmation. Today's sortie
                  count and the log are kept, and Mark Ready always brings the
                  aircraft back.

Turnaround is measured from landing to ready on the serviceable path. When an
aircraft goes unserviceable, that period is counted as maintenance time
instead, and it is shown in its own column, so a long repair does not distort
the average turnaround figure.

The status tiles at the top list the tail numbers as small coloured chips.
Click a chip to jump to that aircraft's card.

WHATSAPP MESSAGES
After Take Off, and after the landing questions are answered, a preview opens
with the message already written. Edit it if you want, then press
"Send to WhatsApp". That opens the WhatsApp compose screen in a new tab and
you choose the group yourself. A static web page cannot post into a group on
its own. 703 and 704 are written as FT-7 BGI, every other tail as F-7 BGI, and
that comes from the registry, so a new aircraft can be set either way.

FLEET FORECAST, the inspection sheet
------------------------------------
"Save today's log" is the only thing that moves the forecast forward. It never
runs on a timer and never writes a future date.

  hour items      AC 100, AC 300, Eng 25, Eng 50, Eng 100 go down by the FLG
                  hours. Red is the last 10 percent of each item's own
                  interval, so AC 300 turns red below 30:00 and Eng 25 below
                  02:30. Nothing is hard coded.
  Total Eng Hrs   goes up by the FLG hours.
  Eng OH/Rtd      goes down. Red below 30:00, OVERDUE at zero.
  Turbo cooler    goes down. Red below 05:00. "Reset" puts it back to 25:00
                  and records the engine hours it was charged at.
  landing items   wheel bearing red below 5, brake disk red below 10.
  idle days       zero on a flying day, plus one on a non flying day, based on
                  the FLG entry and not the calendar.
  Last FLG        moves only on a flying day.
  G/R dates       never change by themselves.
  date items      red inside 30 days, dark red once past.

"Fill from Fleet Ops" copies today's Ops flight time and sortie count into the
input boxes so you do not retype them. It only fills the boxes. You still
check the figures and press save.

Send for inspection offers exactly two categories. Aircraft covers AC 100 and
AC 300, Engine covers Eng 25, Eng 50 and Eng 100. Releasing asks for the new
remaining hours and saves them as a new baseline, which is recorded separately
from a manual override.

Every value carries a label showing where it came from: initial, calculated,
manual override, new inspection baseline, or reset after maintenance. Nothing
is ever overwritten silently and the previous value is always kept.

"Generate forecast PDF" builds the landscape sheet with items down the side and
aircraft across the top, a DT field, and the Prepared by / Rank / Date line.
It reads the live figures at the moment you press it. Print is there as well
if you want paper straight from the browser.

REPORTS AND ARCHIVES
End of day report shows the KPIs, the aircraft table and the full event log,
and exports to CSV, PDF or Excel. The Excel file has three sheets: Summary,
Aircraft Stats and Event Log.

The day's figures are archived automatically. The check runs on every page
load as well as on the clock, so a day is still archived correctly even if the
laptop was shut at midnight. Live statuses are never reset by the archive,
because flying carries on across midnight. Only the daily counters go back to
zero. Past days are browsable, and two days can be compared side by side, from
the History button on the Ops page or from the History and Analysis page.


================================================================================
PART 6  WHERE THE DATA LIVES
================================================================================

Everything sits under one key in the Realtime Database, called fleet35:

  registry     the aircraft list, one entry per tail number
  forecast     the live forecast values for each aircraft
  logs         one record per aircraft per day, dated YYYY-MM-DD_tail
  inspections  sent, released, and the baseline given on release
  overrides    every manual change, with the previous value
  ops          the live flight line state and today's counters
  events       the operations log
  archives     one closed day per entry
  custom       any custom forecast fields you added

Hours are stored as whole minutes. 5790 means 96:30. This is deliberate: it
keeps HH:MM arithmetic exact over hundreds of subtractions instead of drifting
by seconds. The CSV exports give both forms.


================================================================================
PART 7  WHAT WAS TESTED, AND WHAT WAS NOT
================================================================================

Checked automatically before delivery, 80 separate checks, all passing:
  HH:MM parsing and formatting, including bad input
  every warning threshold, hour based, landing based and date based
  the daily roll forward, idle day rules, Last FLG rules, N/A handling
  manual and baseline labels surviving later daily logs
  a retired engine freezing its countdown
  the full Ops cycle: ready, take off, land serviceable to ready, land
    serviceable to ground prep, automatic flip to ready, land unserviceable,
    release from maintenance, maintenance from any state, reset, recovery
  turnaround counted only on the serviceable path
  WhatsApp prefix rule for 703 and 704 against the rest
  adding an aircraft, duplicate refused, retiring one with history kept
  the midnight archive and counter reset
  the analysis totals, idle run detection and all three chart types
  the forecast sheet matrix used by the PDF

Not tested, because it needs a real browser and a real Firebase project:
  the actual click through in a browser window
  live sync between two devices
  the PDF, Excel and print output as they appear on screen and on paper
  the WhatsApp compose screen opening on a phone

Please run through Part 4 step 5 and 6 once before using it on a flying day.


================================================================================
PART 8  IF SOMETHING GOES WRONG
================================================================================

"Live sync: not configured"
  The settings in Part 2 Step 5 were not saved, or a placeholder is still
  there. Check for PASTE_ inside index.html.

"Live sync: permission denied"
  The database rules are refusing the app. See Part 3. Also check that
  Anonymous sign in is switched on.

"Anonymous sign in failed"
  Firebase, Authentication, Sign-in method, Anonymous, switch on, Save.

Nothing syncs and the badge never turns green
  The page is probably open as file:// from the desktop. Firebase needs http
  or https. Use the hosted address.

The PDF or Excel button does nothing
  Those libraries load from the internet. Check the connection and reload.

The board looks empty after moving to a new device
  If you were in local mode, the data was in the old browser only. Local data
  is not uploaded when Firebase is switched on later.

An aircraft is stuck In Air
  Press Landed and answer the questions, or press Maintenance, which is
  allowed from any state and closes the flight leg.

A day was logged twice on the Forecast page
  It cannot be. Once a date is logged for an aircraft, that row is locked for
  that date. To log a day you missed, change the log date to that day.

================================================================================
