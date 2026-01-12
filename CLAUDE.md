# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a client-side inventory counting web application built for warehouse/facility inventory management. It runs entirely in the browser with no backend server, using localStorage for data persistence. The app is designed for physical inventory counts where users scan/enter item information and submit count data.

## Key Architecture Patterns

### Data Storage Architecture

The application uses **browser localStorage** for all data persistence with two distinct storage keys:

1. **`inventoryAppData`** - Stores the master list of inventory items (imported from CSV)
   - Contains item metadata: Part #, Description, Category, Vendor, etc.
   - Does NOT include location or quantity (those are count-specific)
   - Modified in: `js/app.js`

2. **`inventoryCounts`** - Stores individual count submissions
   - Each entry is a complete snapshot: item metadata + Tag #, Location, UOM, Quantity, Notes, Timestamp
   - Append-only structure (new counts are pushed to array)
   - Modified in: `js/app.js` (when submitting counts) and `js/tracking.js` (when editing/deleting)

3. **`userName`** - Stores the user's name for count attribution

4. **`sessionStorage.authenticated`** - Simple client-side auth flag

### Page Architecture

**Three-page structure:**

- **`login.html`** → `js/auth.js`
  - Simple password check (hardcoded, client-side only - not production secure)
  - Sets `sessionStorage.authenticated = 'true'` on success

- **`index.html`** → `js/app.js` (Main counting page)
  - Import master item list from CSV (uses PapaParse library)
  - Filter/search items by Category, Part #, Description, Vendor, etc.
  - Submit counts for items (Tag #, Location, UOM, Quantity, Notes required)
  - Add new items via modal (creates both master item entry + initial count entry)
  - Export master list to Excel (uses SheetJS/xlsx library)

- **`tracking.html`** → `js/tracking.js` (Count history/log)
  - View all submitted counts from `inventoryCounts`
  - Filter by Tag #, Category, Part #, Description
  - Edit/delete individual count entries
  - Export counts to Excel

### Configuration Pattern

**`js/config.js`** contains application-wide dropdown data:

- `UOM_LIST` - Units of measurement (EA, BOX, LB, KG, etc.)
- `CATEGORY_LIST` - Inventory categories (product types, storage locations)
- `LOCATION_LIST` - Physical warehouse locations

These arrays are imported globally and used to populate `<select>` dropdowns throughout the application.

### Master CSV Format

The `master_item_list.csv` file has the following structure:
```
Category,Part #,Description,Vendor Item #,Vendor,Location,UOM,Quantity,Notes
```

- When imported, Location and Quantity are stripped before storing in `inventoryAppData`
- These fields are count-specific, not item-specific

## Common Development Tasks

### Running the Application

This is a static web application with no build step. To run:

```bash
# Serve the directory with any static file server, e.g.:
python -m http.server 8000
# or
npx serve .
```

Then navigate to `http://localhost:8000/login.html`

### Modifying Validation Requirements

**Tag # Requirement Toggle:**

Per `README.txt`, to make Tag # optional:

1. In `js/app.js` around line 247-250: Remove Tag # validation block
2. In `js/app.js` around line 439-445: Remove `"tagNo"` from `requiredFields` array
3. Update error messages to exclude "Tag #" from required field lists

### Adding New Dropdown Options

Modify the relevant array in `js/config.js`:
- `UOM_LIST` for units of measurement
- `CATEGORY_LIST` for categories
- `LOCATION_LIST` for physical locations

## Important Implementation Notes

### Data Duplication Pattern

When a count is submitted, the full item data is duplicated into the count entry. This means:
- Count entries are self-contained snapshots
- If master item data changes, existing counts are unaffected
- Item description/category at time of count is preserved

### Why Modal Creates Two Entries

When adding a new item via the modal (`js/app.js` lines 479-508):
1. A display-only item (no Location/Quantity) is added to `inventoryAppData`
2. A complete count entry (with all fields including Location/Quantity) is added to `inventoryCounts`

This allows immediate count submission while also making the item available for future counts.

### Real-time Filtering Behavior

On `index.html`, the table shows NO items until at least one filter field has a value. This is intentional to avoid rendering potentially thousands of items on load (see `js/app.js` lines 165-168).

### Authentication Model

The current authentication is **client-side only** using a hardcoded password. The code includes warnings that this is NOT production-ready (see `js/auth.js` lines 1-3). Any security improvements should implement server-side authentication.

## File Structure

```
/
├── index.html              # Main counting interface
├── login.html              # Login page
├── tracking.html           # Count history/log viewer
├── master_item_list.csv    # Master inventory data (imported by user)
├── css/
│   ├── style.css          # Main app styles
│   └── login.css          # Login page styles
└── js/
    ├── app.js             # Main counting logic (index.html)
    ├── tracking.js        # Count history logic (tracking.html)
    ├── auth.js            # Login logic (login.html)
    ├── config.js          # Dropdown data (UOM, Categories, Locations)
    └── lib/
        ├── papaparse.min.js   # CSV parsing library
        └── xlsx.full.min.js   # Excel export library
```

## External Dependencies

- **PapaParse** - CSV import functionality
- **SheetJS (xlsx.full.min.js)** - Excel export functionality
- Both libraries are vendored in `js/lib/`
