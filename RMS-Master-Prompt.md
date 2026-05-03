# 🍽️ RMS — Restaurant Management System

---

# 🚀 THE MASTER PROMPT



---

## SYSTEM CONTEXT

You are a senior full-stack web developer. Build a complete, production-ready **Restaurant Management System (RMS)** as a **static multi-page web app** (plain HTML + CSS + Vanilla JS, no build tools, no frameworks, no npm) that can be hosted on **GitHub Pages** with zero server-side code.

The entire backend is powered by:
- **Firebase Auth (v10.12.2)** — authentication
- **Supabase (v2)** — Postgres database + Realtime websockets

Both SDKs are loaded via CDN. The user will manually add their Firebase config and Supabase credentials after receiving the completed files. Leave clearly marked placeholder comments for all credentials.

**OUTPUT**: A complete file tree with full code for every file. Zero placeholder logic — every button, modal, and feature must be fully functional.

---

## 🗂️ FILE STRUCTURE

```
rms/
├── index.html              ← Customer Order Page (public, no login required to browse)
├── kitchen.html            ← Kitchen Display / Order Management Dashboard
├── admin.html              ← Restaurant Admin Panel (owner/manager)
├── superadmin.html         ← Super Admin Panel (multi-branch control)
├── menu-manager.html       ← Menu & Item Management
├── staff.html              ← Staff Management
├── reports.html            ← Analytics & Reports
├── assets/
│   ├── style.css           ← Global shared CSS variables & base styles
│   ├── auth.js             ← Shared Firebase Auth module (imported by all pages)
│   ├── supabase-client.js  ← Shared Supabase client init
│   └── bell.mp3            ← Notification sound (placeholder, user provides)
└── README.md               ← Step-by-step setup guide for Firebase + Supabase
```

---

## 🎨 DESIGN SYSTEM

Use this exact color palette and typography across **all pages**:

```css
:root {
  --bg: #0f1a0f;
  --surface: #162016;
  --surface2: #1e2e1e;
  --border: #2a3d2a;
  --green-accent: #4caf50;
  --green-bright: #66bb6a;
  --green-dim: #2d5a2d;
  --gold: #ffc107;
  --gold-dim: #3a2f00;
  --red: #f44336;
  --red-dim: #3a0f0f;
  --blue: #42a5f5;
  --blue-dim: #0d2a3a;
  --text: #e8f5e9;
  --text-mid: #81c784;
  --text-dim: #4a6b4a;
  --pending: #ffc107;
  --preparing: #42a5f5;
  --done: #66bb6a;
  --rejected: #f44336;
}
```

**Fonts**: `Playfair Display` (headings/logo) + `DM Mono` (numbers/IDs) + `DM Sans` (body) — all via Google Fonts CDN.

**Customer Order Page** (`index.html`): Use a light warm theme:
```css
--cream: #f7f3ed; --warm-white: #fdfaf6; --gold: #c8a96e;
--green-dark: #1a2e1a; --green-mid: #2d4a2d; --green-accent: #4a7c4a;
```

All other pages use the **dark green theme** above.

---

## 🗄️ SUPABASE DATABASE SCHEMA

Create these tables (include the full SQL in `README.md`):

```sql
-- BRANCHES (for multi-branch support)
CREATE TABLE branches (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  name text NOT NULL,
  address text,
  gstin text,
  phone text,
  city text,
  is_active boolean DEFAULT true,
  created_at timestamptz DEFAULT now()
);

-- STAFF / USERS
CREATE TABLE staff (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  firebase_uid text UNIQUE,
  email text NOT NULL,
  name text NOT NULL,
  role text NOT NULL CHECK (role IN ('superadmin','admin','manager','kitchen','waiter')),
  branch_id uuid REFERENCES branches(id),
  is_active boolean DEFAULT true,
  created_at timestamptz DEFAULT now()
);

-- MENU CATEGORIES
CREATE TABLE categories (
  id serial PRIMARY KEY,
  branch_id uuid REFERENCES branches(id),
  name text NOT NULL,
  emoji text DEFAULT '🍽️',
  sort_order int DEFAULT 0,
  is_active boolean DEFAULT true
);

-- MENU ITEMS
CREATE TABLE menu_items (
  id serial PRIMARY KEY,
  branch_id uuid REFERENCES branches(id),
  category_id int REFERENCES categories(id),
  name text NOT NULL,
  description text,
  emoji text DEFAULT '🍽️',
  price numeric(10,2) NOT NULL,
  is_available boolean DEFAULT true,
  is_veg boolean DEFAULT true,
  image_url text,
  sort_order int DEFAULT 0,
  created_at timestamptz DEFAULT now()
);

-- TABLES
CREATE TABLE restaurant_tables (
  id serial PRIMARY KEY,
  branch_id uuid REFERENCES branches(id),
  table_number text NOT NULL,
  seats int DEFAULT 4,
  is_occupied boolean DEFAULT false,
  qr_token text UNIQUE DEFAULT gen_random_uuid()::text
);

-- ORDERS
CREATE TABLE orders (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  order_id text UNIQUE NOT NULL,
  branch_id uuid REFERENCES branches(id),
  table_number text,
  customer_phone text,
  customer_name text,
  firebase_uid text,
  items jsonb NOT NULL,
  subtotal numeric(10,2) NOT NULL,
  cgst numeric(10,2) DEFAULT 0,
  sgst numeric(10,2) DEFAULT 0,
  total numeric(10,2) NOT NULL,
  status text NOT NULL DEFAULT 'pending' CHECK (status IN ('pending','preparing','done','rejected','cancelled')),
  payment_status text DEFAULT 'unpaid' CHECK (payment_status IN ('unpaid','paid')),
  payment_method text DEFAULT 'counter',
  note text,
  invoice_no text,
  created_at timestamptz DEFAULT now(),
  updated_at timestamptz DEFAULT now()
);

-- SETTINGS (per branch)
CREATE TABLE settings (
  id serial PRIMARY KEY,
  branch_id uuid REFERENCES branches(id),
  key text NOT NULL,
  value text,
  UNIQUE(branch_id, key)
);
-- Keys used: 'store_open' (true/false), 'gst_rate' ('5'/'12'/'18'), 'restaurant_name', 'address', 'gstin', 'phone'

-- REALTIME: enable on orders table
-- Run: ALTER PUBLICATION supabase_realtime ADD TABLE orders;
-- Run: ALTER PUBLICATION supabase_realtime ADD TABLE settings;
-- Run: ALTER PUBLICATION supabase_realtime ADD TABLE menu_items;
```

**Row Level Security**: For simplicity, disable RLS during development. Include instructions in README.md for enabling RLS in production.

---

## 📄 PAGE 1: `index.html` — Customer Order Page

### Features Required:

**Header:**
- Restaurant logo + name (pulled from `settings` table for current branch)
- 🔑 Login / Sign Up button → opens auth modal
- After login: shows avatar (first letter of name), username, Logout button
- 🛒 Cart button with item count badge
- 📋 My Orders button → opens order history drawer

**Hero Section:**
- Welcome heading with restaurant tagline
- If store is CLOSED: show a red banner "🔴 We are currently not accepting orders"
- Guest warning banner (bottom, fixed): "⚠️ Ordering as Guest — history won't be saved. Sign In to track orders." with Sign In button + dismiss

**Table & Phone Bar:**
- Text input: Table Number (with validation — must not be empty)
- Tel input: Mobile Number (10-digit Indian mobile, validated)
- Optional: Customer name field

**Category Tabs (horizontal scrollable):**
- Dynamically loaded from Supabase `categories` table
- "All" tab shown first
- Clicking a tab filters menu items
- Items marked `is_available = false` show "Sold Out" badge and are un-addable

**Menu Grid:**
- Cards with: emoji, item name, description, price (₹), Veg/Non-veg dot indicator
- "+ Add" button → adds to cart (inline quantity control after first add: −, count, +)
- Favourite ❤️ button (stored in localStorage per Firebase UID, or Supabase if logged in)
- Search bar: filter items by name
- Items unavailable are greyed with "Sold Out" label

**Cart Drawer (slides from right):**
- Lists all added items with quantity controls (−, count, +) and remove button
- Special instructions textarea
- Subtotal, CGST, SGST, Grand Total display
- "🌿 Place Order" button
  - Validates table number + phone before submitting
  - Inserts order into Supabase `orders` table
  - Sends to `BroadcastChannel('rms_orders')` as fallback
  - Shows order confirmation with Order ID
  - Clears cart after success

**Order History Drawer:**
- Shows all past orders for logged-in user (queried by `firebase_uid`)
- Shows: Order ID, date/time, items, total, status badge (Pending/Preparing/Done/Rejected)
- Real-time status updates via Supabase Realtime subscription
- Guest users see "Login to view order history" message

**Auth Modal:**
- Login tab + Sign Up tab
- Email/Password fields
- Google Sign-In button (with Google icon SVG)
- Error messages in plain English (no Firebase error codes)
- After login, modal closes and UI updates

**Bill Print (hidden, print-only):**
- Restaurant name, address, GSTIN
- Invoice number, Order ID, Table, Phone, Date/Time
- Items table: name, qty, unit price, subtotal
- Subtotal, CGST @ 2.5%, SGST @ 2.5%, Grand Total
- "Payment at counter · Thank you 🌿" footer

---

## 📄 PAGE 2: `kitchen.html` — Kitchen Display Dashboard

**Auth Gate:** Full-screen login overlay. Only staff with role `kitchen`, `manager`, or `admin` can access. Email + password only. Email must be verified. Forgot password link included.

### Features Required:

**Header:**
- Logo + "Kitchen Dashboard" subtitle
- 🟢 Live dot with "Live" / "Reconnecting…" / "Local only" status
- Real-time clock (HH:MM:SS)
- 🔇 Audio enable button (browsers block autoplay — user must click once to enable bell sound)
- Volume slider (0–100%) for bell sound
- 🟢/🔴 Store Open/Closed toggle button (updates `settings` table, reflected on order page instantly)
- 📥 Export Excel button
- 📊 Daily Summary button
- 🔔 Enable Desktop Notifications button
- Logged-in email display + ⏏ Sign Out button

**Stats Bar (horizontal scroll):**
- Pending (yellow dot) — count
- Preparing (blue dot) — count
- Done (green dot) — count
- Rejected (red dot) — count
- Today's Revenue (green dot) — ₹ total of all `done` orders today

**New Order Alert Banner:**
- Appears when a new order arrives: "🔔 New order received! Scroll down to see it."
- Pulses 3 times then stays visible until dismissed

**Menu Availability Toggles:**
- Each menu item shown as a toggle chip
- Toggle OFF = mark item as `is_available = false` in Supabase → instantly reflected on order page
- Changes are real-time via Supabase Realtime

**Search Bar:**
- Filter displayed orders by table number or phone number

**Filter Tabs:**
- All Orders | 🟡 Pending | 🔵 Preparing | 🟢 Done | 🔴 Rejected

**Orders Grid:**
- Auto-fill grid layout (min 300px per card)
- Each card has left border colored by status
- Card Header: Order ID (monospace), Table Number, arrival time, status badge
- Card Body: List of items (emoji + name + quantity chip)
- Special note shown if present (in gold color)
- Card Footer: Total (₹, green), Action Buttons

**Order Card Action Buttons (context-sensitive):**
- **Pending**: [✅ Accept → Preparing] [❌ Reject]
- **Preparing**: [✓ Mark Done] [❌ Reject]
- **Done**: [🖨️ Print Slip] [💳 Mark Paid] [✏️ Edit]
- **Rejected**: [🔄 Restore → Pending]
- All status changes update Supabase instantly

**Edit Order Modal:**
- Opened from "Done" card or any card via edit button
- Shows current items with quantity controls + remove button
- Dropdown to add new items from full menu
- "Save Changes" → updates `items` and `total` in Supabase

**Print Slip:**
- Same format as customer bill
- Triggered by "Print Slip" button
- Uses `window.print()` with `@media print` hiding everything except slip

**Daily Summary Modal:**
- Total Orders today
- Total Revenue today
- Completed count
- Rejected count
- Pending count
- Average Order Value
- Top 5 most-ordered items (name + count)
- "Export Today's Excel" button inside modal

**Export to Excel:**
- Date range picker modal: From Date, To Date
- Quick presets: Today, Yesterday, Last 7 Days, This Month, All Time
- Fetches from Supabase by date range
- Generates `.xlsx` file with columns: Order ID, Date, Time, Table, Phone, Items, Subtotal, CGST, SGST, Total, Status, Payment
- Uses SheetJS (xlsx@0.18.5 CDN)

**Real-Time Sync:**
- Supabase Realtime subscription on `orders` table for `INSERT` and `UPDATE`
- New INSERT → plays bell sound, shows toast notification, shows new order banner, adds card to grid
- UPDATE → updates card status badge and buttons in-place
- BroadcastChannel fallback for same-browser demo
- Heartbeat check every 30s — reconnects if Supabase connection drops
- `visibilitychange` listener — reconnects when tab becomes visible again

**Toast Notifications:**
- Slide-in from bottom-right
- Shows emoji, title, message
- New orders have gold left border
- Auto-dismiss after 5 seconds

**Desktop Push Notifications:**
- "Enable Alerts" button requests `Notification.permission`
- New orders trigger `new Notification("New Order", { body: "Table X — ₹Y" })`

---

## 📄 PAGE 3: `admin.html` — Restaurant Admin Panel

**Auth Gate:** Only `admin` and `manager` roles can access. Email + password + email verification required.

### Features Required:

**Sidebar Navigation:**
- Dashboard Overview
- Orders (today's live view, same as kitchen but read-only)
- Menu Manager (link to `menu-manager.html`)
- Tables Manager
- Staff Manager (link to `staff.html`)
- Reports (link to `reports.html`)
- Settings
- Sign Out

**Dashboard Overview:**
- Today's Revenue card (₹)
- Today's Orders count card
- Pending Orders count card
- Average Order Value card
- Revenue chart: last 7 days bar chart (built with pure SVG or Canvas — no chart library)
- Top 5 menu items today (list)
- Recent 10 orders table

**Orders View (Live):**
- Same filter tabs as kitchen (All, Pending, Preparing, Done, Rejected)
- Table view: Order ID, Table, Phone, Items summary, Total, Status, Time, Actions
- Can change order status
- Can print slip
- Can mark as paid
- Export button (date range)

**Tables Manager:**
- Grid showing all tables for this branch
- Each table: number, seats, occupied/free status
- Add new table modal
- Edit table (number, seats)
- Delete table (with confirmation)
- QR code display per table (using a CDN QR library like `qrcode.js`) — encodes URL: `index.html?table=05&branch=BRANCH_ID`

**Settings Panel:**
- Restaurant Name (editable)
- Address (editable)
- GSTIN (editable)
- Phone (editable)
- GST Rate selector: 0% / 5% / 12% / 18%
- Store Open / Closed toggle
- "Save Settings" → updates `settings` table in Supabase

---

## 📄 PAGE 4: `menu-manager.html` — Menu & Item Management

**Auth Gate:** `admin`, `manager` roles.

### Features Required:

**Category Management:**
- List of all categories (drag-to-reorder via mouse drag — no library)
- Add Category: name + emoji picker (grid of common food emojis)
- Edit Category: name + emoji
- Toggle Active/Inactive
- Delete (only if no menu items reference it)

**Menu Item Management:**
- Filter by category
- Search by name
- Grid/List view toggle
- Each item card: emoji, name, category, price, veg/non-veg dot, available toggle, edit/delete buttons

**Add / Edit Item Modal:**
- Fields: Name, Category (dropdown), Description, Price (₹), Emoji (picker), Veg/Non-Veg radio, Available toggle, Sort Order
- Validation: name required, price > 0
- Save → insert/update in `menu_items` Supabase table

**Bulk Actions:**
- Select All / Deselect All checkboxes
- Bulk Toggle Available / Unavailable
- Bulk Delete (with confirmation)

**Import/Export:**
- Export menu to Excel (SheetJS)
- Import from Excel: column mapping guide shown, validates and inserts items

---

## 📄 PAGE 5: `staff.html` — Staff Management

**Auth Gate:** `admin` role only.

### Features Required:

**Staff List:**
- Table: Name, Email, Role badge, Branch, Active status, Actions
- Search by name or email
- Filter by role

**Add Staff Modal:**
- Fields: Full Name, Email, Role (dropdown: manager/kitchen/waiter), Branch (dropdown)
- Note shown: "Staff must sign up with this email via Firebase Auth. Their role is set here."
- Creates record in `staff` Supabase table with `is_active: true`

**Edit Staff:**
- Change role
- Change branch
- Toggle active/inactive

**Delete Staff:**
- Confirmation modal
- Deletes from `staff` table (does NOT delete Firebase Auth account — note shown)

**Role Permission Matrix:**
- Visual table showing what each role can access:

| Feature | SuperAdmin | Admin | Manager | Kitchen | Waiter |
|---|---|---|---|---|---|
| Super Admin Panel | ✅ | ❌ | ❌ | ❌ | ❌ |
| Admin Panel | ✅ | ✅ | ❌ | ❌ | ❌ |
| Menu Manager | ✅ | ✅ | ✅ | ❌ | ❌ |
| Kitchen Dashboard | ✅ | ✅ | ✅ | ✅ | ❌ |
| Staff Manager | ✅ | ✅ | ❌ | ❌ | ❌ |
| Reports | ✅ | ✅ | ✅ | ❌ | ❌ |
| Multi-Branch | ✅ | ❌ | ❌ | ❌ | ❌ |

---

## 📄 PAGE 6: `reports.html` — Analytics & Reports

**Auth Gate:** `admin`, `manager` roles.

### Features Required:

**Date Range Selector:**
- From/To date pickers
- Quick presets: Today, Yesterday, Last 7 Days, Last 30 Days, This Month, Custom

**Summary Cards:**
- Total Revenue (period)
- Total Orders (period)
- Completed Orders
- Rejected Orders
- Average Order Value
- Busiest Hour (hour with most orders)

**Charts (pure SVG — no library):**
- Revenue by Day: bar chart
- Orders by Status: donut/pie chart
- Orders by Hour: bar chart (0–23h)
- Top 10 Items: horizontal bar chart

**Detailed Orders Table:**
- Paginated (20 per page)
- Columns: Invoice No, Order ID, Date, Time, Table, Phone, Items, Subtotal, CGST, SGST, Total, Status, Payment
- Sort by any column
- Search/filter

**Export:**
- Export filtered data to Excel (SheetJS)
- Export to CSV
- Print report (print-friendly stylesheet)

---

## 📄 PAGE 7: `superadmin.html` — Multi-Branch Super Admin Panel

**Auth Gate:** `superadmin` role ONLY. Any other role sees "Access Denied" message. This page is the gated multi-branch feature.

### Features Required:

**Branch Management:**
- Cards for each branch: Name, City, Address, Active status, Order count today, Revenue today
- "Add Branch" modal:
  - Fields: Branch Name, City, Address, GSTIN, Phone
  - Creates record in `branches` table
- Edit Branch: all fields editable
- Toggle branch Active/Inactive (inactive branches cannot receive orders)
- Delete Branch: only if no orders in last 30 days (warning shown otherwise)

**Cross-Branch Dashboard:**
- Combined stats across ALL branches:
  - Total Revenue today (all branches)
  - Total Orders today (all branches)
  - Best performing branch (highest revenue today)
  - Slowest branch (lowest revenue today)
- Branch comparison bar chart (revenue, orders) — pure SVG
- Per-branch table: Name, Today Revenue, Today Orders, Pending, Done, Rejected

**Cross-Branch Menu Sync:**
- Select a "source branch" menu
- "Copy Menu to Branch" → duplicates all categories + items to another branch
- Warning: overwrites target branch menu

**Staff Overview (All Branches):**
- Full staff list across all branches with branch label
- Can reassign staff to different branch
- Can change any staff's role

**Global Settings:**
- Enable / Disable Multi-Branch Mode toggle
  - When DISABLED: all other pages see only the default single branch
  - When ENABLED: all pages show branch selector at top
- Default Branch selector

**Activity Log:**
- Last 50 events across all branches:
  - Order placed (branch, table, total)
  - Order status changed (who changed it, new status)
  - Menu item toggled (available/unavailable)
  - Staff added/removed
  - Branch created/modified
- Stored in Supabase `activity_log` table (include in schema)

---

## 🔐 AUTH SYSTEM (Shared Logic in `auth.js`)

### Role-Based Access Control:

On every protected page, after Firebase `onAuthStateChanged` confirms a logged-in, verified user:
1. Query `staff` table WHERE `firebase_uid = user.uid AND is_active = true`
2. Get their `role` and `branch_id`
3. Store in `window.currentStaff = { role, branch_id, name, email }`
4. If no staff record found → show "Access Denied — contact your administrator"
5. If role insufficient for this page → show "Access Denied — insufficient permissions"

### Firebase Auth Flows:

**Customer page (`index.html`):**
- Email/Password login
- Email/Password signup (with display name)
- Google Sign-In popup
- Guest browsing allowed (no login required to see menu or place order)
- `signOut` button

**Staff pages (kitchen, admin, menu-manager, staff, reports, superadmin):**
- Email/Password only (no Google, no signup — staff accounts created by admin)
- Email verification required
- Forgot password → `sendPasswordResetEmail`
- "Resend verification email" button on verify screen
- "I've Verified" button → reloads auth state

---

## 📦 PAYMENT SYSTEM

**Manual Counter Payment ONLY.**
- No online payment gateway
- Every order shows: `payment_method: 'counter'`, `payment_status: 'unpaid'`
- Kitchen/Admin can click "💳 Mark as Paid" on any completed order
- This sets `payment_status: 'paid'` in Supabase
- Bill/invoice clearly states: "Payment at counter"
- No UPI, no Razorpay, no Stripe — completely excluded

---

## 🖨️ BILL / INVOICE FORMAT

Consistent across all pages that print:

```
[Restaurant Name]
[Address]
GSTIN: [GSTIN]
─────────────────────────
Invoice No: INV-2026-0001
Order ID: ORD-1234
Table No: 05
Mobile: 9876543210
Date: 03 May 2026
Time: 14:32:10
Status: Done
─────────────────────────
[emoji] Item Name    ×2    ₹240
[emoji] Item Name    ×1    ₹90
─────────────────────────
Subtotal:           ₹330
CGST @ 2.5%:         ₹8.25
SGST @ 2.5%:         ₹8.25
─────────────────────────
Grand Total:        ₹346.50
─────────────────────────
Payment at counter · Thank you 🌿
```

- Invoice numbers: `INV-YYYY-NNNN` (auto-incremented per day per branch)
- GST rate configurable in settings (default 5% split as 2.5% CGST + 2.5% SGST)
- `@media print` hides all UI, shows only the bill div
- `@page { size: auto; margin: 14mm 12mm; }` for proper thermal/A4 printing

---

## ⚡ REAL-TIME FEATURES

### Supabase Realtime Subscriptions:

**`index.html` (Customer Page):**
```js
// Subscribe to own orders for live status tracking
supabaseClient
  .channel('my-orders')
  .on('postgres_changes', {
    event: 'UPDATE', schema: 'public', table: 'orders',
    filter: `firebase_uid=eq.${user.uid}`
  }, (payload) => {
    updateOrderStatusInHistory(payload.new);
  })
  .subscribe();
```

**`kitchen.html`:**
```js
supabaseClient
  .channel('kitchen-orders')
  .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'orders',
      filter: `branch_id=eq.${currentBranchId}` }, handleNewOrder)
  .on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'orders',
      filter: `branch_id=eq.${currentBranchId}` }, handleOrderUpdate)
  .on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'settings',
      filter: `branch_id=eq.${currentBranchId}` }, handleSettingsUpdate)
  .subscribe();
```

**`index.html` — Store Open/Closed sync:**
```js
// Subscribe to settings changes to show/hide closed banner in real-time
supabaseClient
  .channel('store-status')
  .on('postgres_changes', { event: '*', schema: 'public', table: 'settings',
      filter: `key=eq.store_open` }, (payload) => {
    updateStoreBanner(payload.new.value === 'true');
  })
  .subscribe();
```

**Menu Availability sync:**
```js
// On index.html: subscribe to menu_items updates to show/hide Sold Out instantly
supabaseClient
  .channel('menu-availability')
  .on('postgres_changes', { event: 'UPDATE', schema: 'public', table: 'menu_items' },
    (payload) => {
      updateItemAvailability(payload.new.id, payload.new.is_available);
    })
  .subscribe();
```

### BroadcastChannel (Same-Browser Fallback):
```js
const bc = new BroadcastChannel('rms_orders');
// Sender (index.html after placing order):
bc.postMessage(orderObject);
// Receiver (kitchen.html):
bc.onmessage = (e) => receiveOrder(e.data);
```

### Connection Health:
- Heartbeat: `setInterval` every 30s pings Supabase with a lightweight query
- If ping fails: show offline indicator, attempt reconnect
- `visibilitychange`: reconnect when browser tab becomes active

---

## 🏗️ MULTI-BRANCH ARCHITECTURE

### Branch Selection:
- When Multi-Branch Mode is ENABLED (set in superadmin settings):
  - All staff pages show a branch selector dropdown at top
  - Selector populates from `branches` table WHERE `is_active = true`
  - Selected branch stored in `localStorage` as `rms_active_branch`
  - All queries filter by `branch_id`
- When DISABLED:
  - Single default branch used
  - No branch selector shown

### Branch Context in Customer Page:
- URL parameter: `index.html?branch=BRANCH_UUID`
- If no branch param, use default branch from settings
- Can also be encoded in QR code per table

---

## 🔔 NOTIFICATION SYSTEM

**In-App Toasts (all staff pages):**
- Slide in from bottom-right
- Auto-dismiss after 5s
- Types: success (green), error (red), warning (gold), info (blue)
- Queue management: max 4 toasts at once

**Browser Audio:**
- `bell.mp3` loaded via `<audio id="bell" preload="auto">`
- Must be unlocked by user click (show "🔇 Click to enable sound" banner until clicked)
- Volume controlled by range slider
- Volume saved in `localStorage`

**Desktop Push Notifications:**
- Request `Notification.permission` on "Enable Alerts" button click
- New orders: `new Notification(restaurantName, { body: "Table X — ₹Y — N items" })`
- Status changes: notify if tab is hidden

---

## 📱 RESPONSIVE DESIGN

**Breakpoints:**
- Mobile: `< 480px`
- Tablet: `480px – 768px`
- Desktop: `> 768px`

**Mobile-specific:**
- Cart and history as bottom drawers (slides up)
- Sidebar nav collapses to hamburger menu on mobile
- Orders grid: 1 column on mobile, 2 on tablet, 3+ on desktop
- Stats bar: horizontal scroll (hide scrollbar visually)
- Filter tabs: horizontal scroll

---

## 🛡️ VALIDATION & ERROR HANDLING

**Order Placement:**
- Table number: required, max 4 chars
- Phone: required, exactly 10 digits, starts with 6-9
- Cart: must have at least 1 item
- Store must be open (check settings before submitting)

**Auth Errors — Map to friendly messages:**
```js
const errorMap = {
  'auth/user-not-found': 'No account found with this email.',
  'auth/wrong-password': 'Incorrect password.',
  'auth/invalid-credential': 'Invalid email or password.',
  'auth/email-already-in-use': 'This email is already registered.',
  'auth/weak-password': 'Password must be at least 6 characters.',
  'auth/invalid-email': 'Please enter a valid email address.',
  'auth/too-many-requests': 'Too many attempts. Please wait and try again.',
  'auth/network-request-failed': 'Network error. Check your internet connection.',
  'auth/popup-closed-by-user': 'Google sign-in was cancelled.',
};
```

**Supabase Errors:**
- Show user-friendly toast (not raw error message)
- Log full error to console for debugging
- On connection failure: show offline banner, attempt reconnect

---

## 📝 README.md CONTENT

The README must include these sections:

### 1. Firebase Setup
```
Step 1: Create Firebase Project at console.firebase.google.com
Step 2: Enable Authentication → Email/Password AND Google providers
Step 3: Add your domain to Authorized Domains (Settings → Authentication → Authorized domains)
Step 4: Copy config object from Project Settings → General → Your apps

WHERE TO PASTE:
In EVERY html file, find this comment:
  // 🔥 PASTE YOUR FIREBASE CONFIG HERE
Replace the entire firebaseConfig object with your values.
```

### 2. Supabase Setup
```
Step 1: Create project at supabase.com
Step 2: Go to SQL Editor → run the schema SQL from this README
Step 3: Go to Project Settings → API → copy Project URL and anon public key

WHERE TO PASTE:
In EVERY html file, find:
  const SUPABASE_URL = 'YOUR_SUPABASE_URL';
  const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
Replace with your values.

Step 4: Enable Realtime
  Go to Database → Replication → enable for tables: orders, settings, menu_items

Step 5: Insert your first branch
  INSERT INTO branches (name, city, address, gstin, phone)
  VALUES ('My Restaurant', 'Mumbai', '123 Main St', '27AABCX1234Y1Z5', '9876543210');

Step 6: Insert superadmin staff record
  First sign up via kitchen.html with your email + verify it.
  Then in Supabase: INSERT INTO staff (firebase_uid, email, name, role)
  VALUES ('YOUR_FIREBASE_UID', 'you@email.com', 'Your Name', 'superadmin');
  (Find your Firebase UID in Firebase Console → Authentication → Users)
```

### 3. GitHub Pages Deployment
```
Step 1: Create a GitHub repository (public)
Step 2: Upload all files maintaining the folder structure
Step 3: Go to Settings → Pages → Source: Deploy from branch → main → / (root)
Step 4: Your site will be at: https://yourusername.github.io/your-repo-name/
Step 5: Add this URL to Firebase Authorized Domains
```

### 4. First-Time Setup After Deploy
```
1. Open index.html — browse the menu (no auth needed)
2. Open kitchen.html — log in with your admin email
3. Open admin.html — add menu categories and items
4. Open superadmin.html — configure branches if needed
5. Share index.html URL with customers / print QR codes
```

---

## ⚠️ CRITICAL IMPLEMENTATION RULES

1. **NO build tools** — all CDN only. No webpack, no vite, no npm.
2. **NO frameworks** — Vanilla JS only. No React, Vue, Angular.
3. **Every `<script type="module">` for Firebase** must expose all helpers to `window.*` for access from regular `<script>` tags.
4. **SUPABASE_URL and SUPABASE_ANON_KEY** must be at the very top of each file's main `<script>` block with a clear comment: `// ← REPLACE WITH YOUR SUPABASE CREDENTIALS`
5. **Firebase config** must have a clear comment: `// ← REPLACE WITH YOUR FIREBASE CONFIG`
6. **`appInit` pattern**: Firebase auth module calls `window.appInit()` only after a verified, authorized user is confirmed. All Supabase initialization and UI rendering happens inside `appInit`.
7. **No `<form>` tags** — use `<div>` + button `onclick` handlers everywhere.
8. **GST calculation**: Always on subtotal. Default 5% = 2.5% CGST + 2.5% SGST. Rate from settings.
9. **Order IDs**: Format `ORD-XXXXXX` where X is last 6 chars of UUID. Example: `ORD-A3F9C2`.
10. **Invoice numbers**: Format `INV-YYYY-NNNN` auto-incremented daily. Store last invoice number in `settings` table.
11. **All monetary values**: Use `toFixed(2)` for display, store as `numeric(10,2)` in Supabase.
12. **Date/time**: Always use Indian locale — `new Date().toLocaleString('en-IN', { timeZone: 'Asia/Kolkata' })`.
13. **Animations**: `slideIn` keyframe for new cards, toast `toastIn` for notifications, pulse for live dot.
14. **Print styles**: `@media print { body > * { display: none !important; } #printSlip { display: block !important; } }` — only the slip/bill is visible when printing.
15. **Excel export**: Use SheetJS `XLSX.utils.json_to_sheet` + `XLSX.writeFile`. CDN: `https://cdn.jsdelivr.net/npm/xlsx@0.18.5/dist/xlsx.full.min.js`
16. **QR codes for tables**: Use `https://cdn.jsdelivr.net/npm/qrcodejs@1.0.0/qrcode.min.js`
17. **All modals**: Use `position:fixed; inset:0; z-index:500+; background:rgba(0,0,0,0.75)` overlay with centered content card.
18. **Accessibility**: All interactive elements have `cursor:pointer`. Buttons have `:hover` states. Inputs have `:focus` border highlight.

---

## 🎯 FINAL DELIVERABLE

Produce the complete code for all 7 HTML files + `assets/style.css` + `README.md`. Each file must be self-contained and immediately deployable by uploading to GitHub. The only manual step the user performs is pasting their Firebase config and Supabase credentials where the comments indicate.

**Start with `index.html`, then `kitchen.html`, then `admin.html`, then `superadmin.html`, then `menu-manager.html`, then `staff.html`, then `reports.html`, then `assets/style.css`, then `README.md`.**

Write complete, production-ready code with no TODOs, no placeholders in logic, and no "implement this yourself" comments anywhere except the clearly marked credential sections.
