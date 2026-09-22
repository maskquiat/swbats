SWBATs
A digital sticky-note board for teachers. Post the day's objective (SWBAT), agenda, and homework where students can see them, without needing to claim wall or whiteboard space in a shared classroom.
Live site: https://swbats.mytimer.io
Built as a single self-contained index.html file. No build step, no framework, no npm install.
What it does
•	Repositionable notes. Drag notes by the top bar, resize from the bottom-right corner. Works with a mouse, trackpad, or finger on a tablet.
•	Note types. SWBAT, Agenda, Homework, Reminder, Checklist, and blank notes. Agenda and checklist items can be checked off live during class.
•	Multiple classes. Each period gets its own tab with its own notes and background. "Duplicate this class" copies a lesson to the next section.
•	Present mode. Hides all editing controls, goes full screen, and displays the class name and date for projecting.
•	Float window. On Chrome and Edge on a computer, opens a small always-on-top window so the objective stays visible over slides or video. (Not supported in Safari or on iPad — use Split View there.)
•	Minimize. Any note collapses to a slim bar showing its title and first line.
•	Phones. Notes become a stacked list of bars; tap one to expand it.
•	Accounts. Teachers sign in with email and password. Boards sync to every device they use. Signing out clears their boards from the browser, which matters on shared classroom computers.
•	Without an account. "Use without an account" saves boards in that browser only. Export and Import (under Class) move them between devices.
Note positions and text size scale with the screen, so a board arranged on a laptop looks the same on a projector or an iPad.
Tech
Piece	What it is
Front end	One HTML file: vanilla JS, no dependencies to install
Auth + storage	Supabase (@supabase/supabase-js loaded from jsDelivr)

Hosting	GitHub Pages
DNS	Porkbun (CNAME swbats → maskquiat.github.io)
Fonts	Atkinson Hyperlegible from Google Fonts, with system-font fallbacks
Local fallback: if the Supabase keys are blank, or the network is unavailable, the app runs entirely on localStorage and still works offline.
Setup
1. Supabase
Run supabase-setup.sql in SQL Editor → New query. It creates the boards table and the Row Level Security policies that keep each teacher's row private to them.
Then in Project Settings → API, copy the Project URL and the anon (publishable) key into the top of the script in index.html:
const SUPABASE_URL = "https://yourproject.supabase.co";
const SUPABASE_KEY = "your-anon-or-publishable-key";
The anon key is meant to be public and is safe here — Row Level Security is what protects the data. Never put the service_role or secret key in this file. It bypasses every policy.
In Authentication → URL Configuration, set Site URL to https://swbats.mytimer.io and add it to Redirect URLs, so confirmation and password-reset emails land on the right page.
2. Hosting
index.html sits at the repository root. Settings → Pages is set to deploy from main / root, with swbats.mytimer.io as the custom domain and Enforce HTTPS on.
To update the site, replace index.html on main. The deploy takes a minute or two.
Data model
One row per teacher. The whole board state — classes, notes, positions, colors, sizes — is a single JSON blob:
boards
  user_id     uuid   primary key, references auth.users
  data        jsonb  { v, active, boards: [ { id, name, bg, notes: [...] } ], updatedAt }
  updated_ms  bigint milliseconds, used to decide which copy is newer
  updated_at  timestamptz
Writes are debounced ~900ms. On sign-in, the newer of the local cache and the server copy wins. A brand-new account keeps whatever board is already on the device.
Privacy
The app stores teacher account emails and lesson content only. It is not for student data. Teachers should keep student names out of notes, since notes are meant to be projected to a whole class. Districts adopting this formally in New York may require a data privacy agreement under Ed Law 2-d; staying teacher-content-only keeps that straightforward.
Known limits
•	Some school filters block free shared hosts (*.netlify.app, *.github.io) under a "domain-sharing" category. This is why the site runs on a custom domain. If a district still blocks it, submit the domain to that filter's categorization tool or ask their IT to allowlist it — including the *.supabase.co address the app calls for sign-in.
•	The Float window uses the Document Picture-in-Picture API: Chrome and Edge on desktop only.
•	Supabase's built-in email sender is rate-limited and meant for testing. Connect custom SMTP before opening signups widely.
Roadmap ideas
•	Countdown timer note
•	View-only class link students can open on their own devices
•	Saved lesson templates and a weekly view
•	Copy one note to all classes at once
