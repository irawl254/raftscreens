RAFT FRONT DESK

This is a static website with a sign-in screen and exactly four staff screens:
Home, Client lookup, Dashboard, and Add or edit a record.

HOSTING
Upload the contents of public/ to any HTTPS static website host.
Set the publish directory to public. There is no build step and no server.
Hash navigation works without rewrite rules.

LOCAL PREVIEW
From this folder, run:
  python3 -m http.server 4173 --directory public
Then visit http://localhost:4173 in a browser.
Use HTTP locally and HTTPS for hosting; do not open index.html as a file.

SUPABASE
public/config.js already contains the Project URL and publishable key supplied
in this conversation. These are public browser credentials, not administrator
credentials. No service-role key is used.

The application calls supabase.auth.signInWithPassword({ email, password }).
Supabase Auth persists the session and refreshes the access token. Only this
Auth session is persisted in browser storage. RAFT records are kept in memory
and are written only to your existing Supabase database. Signing out clears the
workspace and removes the local Supabase session.

Existing Supabase prerequisites:
- The raft schema must be exposed through the Supabase Data API.
- The existing RAFT role grants must already be in place.
- The existing raft_access_token_hook must be enabled in Supabase Auth settings.
- Staff users must already exist with the appropriate staff_role metadata.
The application reads raft_admin, raft_scheduler or raft_counselor directly
from the role claim in the session access token. A user whose token does not
have one of these roles sees a staff-access message and triggers no data reads.

No supplied SQL was executed. There are no migrations, schema changes, RPCs,
extra tables, extra databases, delete operations, or sample data in the site.
All queries name their columns explicitly. Data is paginated in batches of 500
so counts and lists are not truncated at the usual API response limit.

SCREENS AND PERMISSIONS
Every staff screen has all four navigation links, the staff role label, and
Sign out. Navigation is in-page, without adding more screens.

Client lookup searches names, IDs, email and phone. It shows contact details,
referral and reminder details, linked appointments, and each family membership
with the roles in that specific family. Clinical notes and assignments are
omitted for Schedulers.

Dashboard calculates totals from the records returned by the existing tables:
clients, appointments, families, attendance, reminders, counselors, employees,
rooms and memberships. Attendance counts each scheduledfor row separately and
excludes rows where kept is null. Only Admins see average hourly wage.

The Add or edit screen defaults to clients. It also supports appointments and
linking clients to appointments through Client attendance & assignment.
Admins can view and edit all table types, including employee wages.
Schedulers can create/edit people, clients, rooms, appointments and attendance.
Counselors can update only appointment notes and scheduledfor assignment/kept.
Database permissions remain authoritative for every action.

For a new client, choose an existing non-client person or enter new contact
information. This preserves the overlapping person subclasses in your schema.
There is no transaction RPC in the supplied schema, so saving a new person and
client requires two requests. If the client save fails, the screen reports the
saved person ID. Retrying the same form reuses that contact record. If you leave
or reload after a partial save, select the existing person when adding the client.
Updating client contact information and client details likewise uses two
requests and reports partial success if the second request fails.

Creating an appointment saves its appointment row. Then choose Client attendance
& assignment to link each participating client. A family appointment may have
several such links. No record can be deleted from this website.

The SQL stores AppointmentTime as a timestamp without a timezone. Appointment
forms and displays therefore use the browser's local wall-clock time, without
inventing a timezone column or converting to UTC. Use the practice timezone
on front-desk devices for consistent scheduling.

DATABASE LIMITATIONS
There is no appointment status/cancellation field, client birth date, billing
record, or conflict-prevention constraint in the supplied tables. The site does
not invent these fields or imply that appointment clashes are prevented.

VERIFICATION
- JavaScript syntax check passed.
- Automated Chromium checks passed for all three roles using isolated test
  fixtures: no pre-login reads, bad-password message, session restoration,
  explicit allowed columns, role-specific forms, safe update payloads,
  multi-family roles, dashboard calculations, composite-key updates,
  partial-save retry, sign-out, and responsive layout.
- The provided project's Supabase Auth settings endpoint responded HTTP 200
  with the supplied publishable key.
- Authenticated reads/writes against your actual database require a staff
  account and were not tested. No real patient or employee records were changed.
- Built-in Sites publication was unavailable in this workspace.

DEPENDENCIES
A pinned Supabase JS 2.49.8 browser bundle is included in public/supabase.js.
Typography requests DM Sans and Manrope from Google Fonts, with local sans-serif
fallbacks. There are no other runtime services beyond Supabase and those fonts.
