# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**WitchWayHosting** is the static website and data hosting repository for the Witch Way iOS application. It serves three primary purposes:

1. **Marketing Site**: Landing page at www.witchway.app showcasing the iOS app
2. **Support Hub**: FAQ, troubleshooting, and privacy documentation
3. **Data Source**: Hosts `LocalPlaces.json` with 480+ curated Salem, MA locations consumed by the iOS app

**Related Repository**: The iOS app code is located at `../Witch Way/` (sibling directory)

**Website URL**: www.witchway.app (GitHub Pages custom domain)

## Technology Stack

- **Hosting**: GitHub Pages (static site)
- **Frontend**: Vanilla HTML + CSS (no JavaScript, no build tools)
- **Data Format**: JSON (LocalPlaces.json)
- **Platform Integration**: Apple Universal Links and App Clips

This is intentionally minimal—no package managers, no dependencies, no backend. Pure static content delivery for maximum performance and privacy.

## File Structure

```
WitchWayHosting/
├── index.html              # Main landing page
├── download.html           # App Store redirect page
├── support.html            # FAQ & support documentation
├── site.css                # Shared stylesheet (purple/dark theme)
├── LocalPlaces.json        # 480+ curated Salem locations (164 KB)
├── CNAME                   # GitHub Pages domain: www.witchway.app
│
├── .well-known/
│   └── apple-app-site-association  # Universal links + App Clips config
│
├── Assets/                 # Source design files (119 MB)
│   ├── AppStore Screenshots/
│   ├── Banner/
│   ├── Cards/
│   ├── Clips/
│   └── Poster/
│
└── images/                 # Web-optimized images (79 MB)
    ├── icon.png / icon.dark.png
    ├── banner.jpg
    └── screenshots, clips, etc.
```

## Primary Workflows

Based on typical development work in this repository:

### 1. Updating Place Data (Most Frequent)

Editing `LocalPlaces.json` to add, remove, or modify locations.

**IMPORTANT RULES:**

#### Required Fields
Every place entry MUST include:
- `name` (string) - Place name
- `address` (string) - Full street address
- `coordinate` (object) - Latitude and longitude
  - `latitude` (number)
  - `longitude` (number)
- `categories` (array) - At least one valid category

#### Optional Fields
- `phoneNumber` (string) - Contact phone
- `placeId` (string) - Unique identifier (see ID rules below)
- `website` (string) - Full URL

#### Category Validation
Only use categories from the iOS app's taxonomy. Available categories (52 total):

**Food & Drink:**
- `diner`, `bar`, `pub`, `brewery`, `coffeeChain`, `coffeeLocals`
- `eastAsian`, `italian`, `latinAmerican`, `mediterranean`, `seafood`, `foodRestaurant`

**Facilities:**
- `parking`, `publicRestroom`, `customerRestroom`, `gasStation`, `carCharger`

**Shopping:**
- `clothingStore`, `liquorStore`, `market`, `shop`, `souvenierShop`, `paganStore`

**Services:**
- `atm`, `bank`, `pharmacy`, `hairSalon`, `nailSalon`, `barber`, `spa`, `tattooParlor`

**Cultural/Historical:**
- `attraction`, `historicalMarker`, `haunt`, `museum`, `theater`, `venue`, `psychicReading`

**Recreation:**
- `arcade`, `beach`, `park`, `marina`, `yogaStudio`

**Transportation:**
- `transport`

**Civic:**
- `fireStation`, `hospital`, `library`, `postOffice`, `school`, `university`, `placeOfWorship`

**Local Interest:**
- `cannabisDispensary`

**Category notes:**
- Multiple categories allowed per place
- Use specific categories over generic ones (e.g., `coffeeLocals` instead of just `foodRestaurant`)
- Category names are case-sensitive and must match exactly

#### ID Format & Uniqueness
- `placeId` format: Composite ID combining name and coordinates
- Pattern: `"<name>|<latitude>,<longitude>"` (e.g., `"Salem Witch Museum|42.5221,-70.8975"`)
- Each place must have a unique ID
- If `placeId` is omitted, the iOS app will generate one using this format

#### Example Entry
```json
{
  "name": "Salem Witch Museum",
  "address": "19 Washington Square N, Salem, MA 01970",
  "coordinate": {
    "latitude": 42.5221,
    "longitude": -70.8975
  },
  "categories": ["museum", "attraction"],
  "phoneNumber": "+1 (978) 744-1692",
  "placeId": "Salem Witch Museum|42.5221,-70.8975",
  "website": "https://www.salemwitchmuseum.com"
}
```

#### Coordinate Guidelines
- Salem, MA center: ~42.5195°N, 70.8967°W
- The iOS app validates places are within ~1.6km radius of Salem center
- Use 4 decimal places for precision (≈11 meters accuracy)
- Coordinates must use standard WGS84 format

#### JSON Formatting
- Use 2-space indentation
- Maintain alphabetical order of fields within each entry (not required but preferred for consistency)
- No trailing commas
- Validate JSON syntax before committing

### 2. Managing Assets & Images

Adding or updating screenshots, banners, and visual assets.

**Directories:**
- `/Assets/` - Source files (Sketch, Figma exports, high-res images)
- `/images/` - Web-optimized versions for deployment

**Guidelines:**
- Keep web images optimized (use JPEG for photos, PNG for graphics)
- Maintain consistent naming: `screenshot-1.jpg`, `screenshot-2.jpg`, etc.
- Update both source files and web versions
- Test image loading on slow connections

### 3. Website Content Updates

Modifying HTML pages or CSS styling.

**Key Pages:**
- `index.html` - Main landing page with app description and download links
- `support.html` - FAQ and troubleshooting (update when new issues arise)
- `download.html` - Simple redirect to App Store (rarely needs changes)

**Design System:**
- Brand color: `#7d4cc7` (purple)
- Dark theme with radial gradient background
- Responsive design using CSS Grid and Flexbox
- System fonts: `-apple-system`, `Segoe UI`, sans-serif
- Glassmorphic card design with shadows

## Apple Platform Integration

### Universal Links Configuration

**File**: `.well-known/apple-app-site-association`

This file enables:
1. **Universal Links**: Direct linking from web to iOS app
2. **App Clips**: Lightweight app experiences at select landmarks

**Configuration:**
- Team ID: `G9LCFVC8QK`
- App Bundle: `com.icalvin.Witch-Way`
- App Clip Bundle: `com.icalvin.Witch-Way.Clip`

**Important:**
- Must be served over HTTPS (GitHub Pages handles this)
- No file extension (already correct)
- Content-Type should be `application/json` (set via `.gitattributes`)
- Changes require 24-48 hours to propagate through Apple's CDN

### Deep Linking

The iOS app handles deep links via URL scheme:
- Format: `witchway://[slug or query]`
- Examples:
  - `witchway://food` → Show food category
  - `witchway://coffee-locals` → Show local coffee shops
  - `witchway://salem-witch-museum` → Search for specific place

## iOS App Integration

### How the iOS App Uses This Repository

1. **Data Consumption**:
   - Downloads `LocalPlaces.json` from www.witchway.app/LocalPlaces.json
   - Caches locally for offline use
   - Checks for updates every 6 hours (throttled)
   - Falls back to bundled copy if network unavailable

2. **Place Model**:
   - iOS app uses `LocalPlace` struct (Codable, Equatable, Sendable)
   - Direct mapping from JSON to Swift model
   - Validates coordinates are within Salem area

3. **Category System**:
   - iOS app defines taxonomy in `LocalFilter.swift`
   - Each category has icon, color, and URL slug
   - Deep linking uses category slugs

4. **Development Workflow**:
   - iOS app has Debug-only place editor for testing
   - Changes in editor write to local override file
   - This repository is source of truth for production data

### Data Update Flow

```
[Edit LocalPlaces.json]
    ↓
[Commit & Push to GitHub]
    ↓
[GitHub Pages serves updated JSON]
    ↓
[iOS app fetches update (6-hour throttle)]
    ↓
[App displays new/updated places]
```

## Git & Deployment

### Branch Strategy
- **main**: Production branch (deploys automatically to GitHub Pages)
- All commits to `main` are live immediately

### Commit Patterns
Recent commits show typical work:
- "Remove seasonal bathrooms" - Updating temporary locations
- "Add new assets" - Design file updates
- "Update phillips library category" - Place data corrections
- "Add new bathroom" - New location additions

### Deployment
- GitHub Pages builds automatically on push to `main`
- No build process required (static files)
- Changes are live within 1-2 minutes
- DNS via CNAME to www.witchway.app

## Common Tasks

### Adding a New Place

1. Open `LocalPlaces.json`
2. Find the appropriate insertion point (file is roughly organized by category/area)
3. Add new entry with all required fields:
   ```json
   {
     "name": "New Place Name",
     "address": "123 Main St, Salem, MA 01970",
     "coordinate": {
       "latitude": 42.5195,
       "longitude": -70.8967
     },
     "categories": ["category1", "category2"],
     "phoneNumber": "+1 (978) 555-0123",
     "website": "https://example.com"
   }
   ```
4. Validate JSON syntax (use `jq . LocalPlaces.json` or JSON linter)
5. Commit and push

### Removing a Place

1. Open `LocalPlaces.json`
2. Search for place by name or address
3. Delete entire entry (including surrounding commas)
4. Validate JSON syntax
5. Commit with descriptive message (e.g., "Remove [Place Name] - closed permanently")

### Updating Place Categories

1. Open `LocalPlaces.json`
2. Find place entry
3. Update `categories` array with valid category names
4. Ensure at least one category remains
5. Validate and commit

### Adding Seasonal Locations

Common for temporary bathrooms, seasonal vendors, etc.

1. Add place with descriptive name indicating season (e.g., "Public Restroom (Seasonal)")
2. Add note in commit message about removal date
3. Set calendar reminder to remove when season ends
4. Search git history for "Remove seasonal" to see pattern

### Validating JSON

Before committing changes to `LocalPlaces.json`:

```bash
# Check JSON syntax
jq . LocalPlaces.json > /dev/null && echo "Valid JSON" || echo "Invalid JSON"

# Pretty-print and save (maintains formatting)
jq . LocalPlaces.json > temp.json && mv temp.json LocalPlaces.json

# Count total places
jq '. | length' LocalPlaces.json

# List all unique categories
jq '[.[].categories[]] | unique' LocalPlaces.json

# Find places missing required fields
jq '.[] | select(.name == null or .address == null or .coordinate == null)' LocalPlaces.json
```

## Design Guidelines

### Color Palette
- Primary: `#7d4cc7` (brand purple)
- Background: Radial gradient from dark to purple
- Text: White with varying opacity for hierarchy
- Cards: `rgba(255, 255, 255, 0.05)` with backdrop blur

### Typography
- Headings: System font, bold, responsive sizing with `clamp()`
- Body: System font, regular weight
- Scale: Mobile-first, fluid typography

### Responsive Breakpoints
- Mobile: < 768px (single column)
- Tablet: 768px - 1024px (two columns)
- Desktop: > 1024px (three columns for cards)

## Support & Contact

- **Owner**: iCalvin, LLC
- **Support Email**: support@witchway.app
- **App Store**: Search "Witch Way" or use download link on website
- **iOS Requirements**: iOS 17+

## Important Notes

- This repository is PUBLIC (required for GitHub Pages)
- Be mindful of any sensitive information (though there shouldn't be any)
- The iOS app is PRIVATE and located in sibling directory
- Keep LocalPlaces.json under 500KB for optimal mobile loading
- Current file size: 164KB (480 places) - room to grow
- No PII or user data stored—privacy-focused by design
- Salem-centric focus: Don't add places outside ~1.6km radius
- Seasonal locations are common—use clear naming and commit messages
