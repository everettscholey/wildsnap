# WildSnap — Product Design Document

> *A real-world creature collector.* Snap photos of animals in the wild, identify them with AI, build your personal collection, and pin your sightings on a beautiful watercolor map.

---

## 1. Product Overview

**WildSnap** is a mobile app for animal lovers who want to document and celebrate the animals they encounter in a fun, beautiful way. Users photograph animals in the real world — either through the in-app camera or by uploading from their gallery — and the app identifies the species using AI. Each identified animal is added to a personal collection and pinned on a stylized watercolor world map.

### Core Value Proposition

- **Discover** — Learn what animals are around you with species-level AI identification
- **Collect** — Build a personal collection of every species you've encountered
- **Remember** — Revisit your sightings on a beautiful, personalized map

### Target Audience

Animal lovers of all experience levels who want to document wildlife sightings in a fun, visually delightful way. From casual hikers who spot a cool bird to dedicated wildlife enthusiasts building a life list.

---

## 2. Core Loop

```
┌─────────────────────────────────────────────────┐
│                                                 │
│   SNAP ──► IDENTIFY ──► COLLECT                 │
│                          │                      │
│                          ▼                      │
│                         MAP                     │
│                                                 │
└─────────────────────────────────────────────────┘
```

1. **Snap** — User takes a photo (in-app camera or gallery upload)
2. **Identify** — AI analyzes the photo and returns its best guess at species-level identification
3. **Collect** — The animal is added to the user's Collection (new entry or stacked sighting)
4. **Map** — A pin is dropped on the user's personal sightings map

---

## 3. Platform & Tech

| Attribute | Decision |
|-----------|----------|
| **Platforms** | iOS + Android (mobile-first) |
| **Framework** | React Native or Flutter (TBD) |
| **Photo storage** | Local-only for MVP; cloud sync on roadmap |
| **Offline support** | Yes — snap photos offline, queue for identification when back online |
| **Monetization** | Fully free for MVP; monetization strategy TBD |
| **Age restrictions** | TBD — revisit before launch |

---

## 4. Feature Specifications

### 4.1 Photo Capture

**In-App Camera**
- Full-screen viewfinder optimized for wildlife (fast shutter, tap-to-focus)
- Optional zoom controls
- Automatic GPS tagging of photo location
- Works offline — photos are stored locally and queued for ID

**Gallery Upload**
- Select existing photos from device gallery
- Extract GPS metadata from photo EXIF data if available
- Allow manual location tagging if no GPS data present

### 4.2 AI Identification

**Behavior**
- Sends photo to identification API
- Returns best-guess species-level identification
- Displays result on a clean result screen (no animation — simple and clear)
- Includes: common name, scientific name, and taxonomy group

**MVP Scope: Mammals & Birds**
- Initial AI model/API must support reliable species-level identification for mammals and birds
- Insects, reptiles, amphibians, and marine life planned for future expansion

**Offline Handling**
- Photos taken offline are stored in a local queue
- When connectivity is restored, queued photos are automatically sent for identification
- User receives notification when offline photos have been identified

**API Candidates**

| API | Coverage | Accuracy | Hosting | License | Notes |
|-----|----------|----------|---------|---------|-------|
| **Google SpeciesNet** | 2,000+ species (mammals, birds, some reptiles) | 94.5% species-level, 99.4% animal detection | Self-hosted (Python/PyPI) | Apache 2.0 (free, commercial OK) | Open-source. Combines MegaDetector + Species Classifier. Supports geofencing by country code. Trained on 65M+ camera trap images. |
| **Animal Detect API** | Same as SpeciesNet (hosted wrapper) | Same as SpeciesNet | Hosted REST API | Pay-per-use (contact for pricing) | Hosted version of SpeciesNet — avoids self-hosting overhead. 1.2-1.5s response time. 5MB image limit. |
| **Google Cloud Vision** | General object/animal detection | Good for broad detection, poor at species level | Hosted REST API | $1.50/1K images (1K/month free) | Not suitable for species-level ID. Useful as a pre-filter ("is this a photo of an animal?"). |
| **Merlin / eBird (Cornell Lab)** | 6,000+ bird species | Best-in-class for birds | No public API (app only) | N/A | ML models not available for third-party integration. eBird data API is available (free, API key required). |
| **iNaturalist Computer Vision** | 108,000+ taxa | Gold standard (community-validated) | **Private — not available for third-party apps** | N/A | CV model restricted to iNaturalist's own apps (website, iOS, Android, Seek). Not a viable integration path. |

**Recommendation:** **Google SpeciesNet** as the primary identification engine, either self-hosted or via the **Animal Detect API** for a managed solution. SpeciesNet offers strong species-level accuracy (94.5%) for mammals and birds, an open-source Apache 2.0 license, and geofencing support to improve predictions by region. **Google Cloud Vision** can serve as a lightweight pre-filter to confirm a photo contains an animal before running the heavier species classifier.

**Note on future insect expansion:** SpeciesNet has limited insect coverage. Adding insects in v1.1 will likely require a supplementary model — either a custom model fine-tuned on publicly available iNaturalist research-grade data, or open-source insect classifiers from Hugging Face/Kaggle.

**Species Data & Fun Facts**

| Source | Use | Pricing |
|--------|-----|---------|
| **Wikipedia REST API** | Fun facts, species descriptions, habitat info. Massive coverage, frequently updated. Simple endpoint: `/page/summary/{Species_Name}`. | Free, no API key required |
| **IUCN Red List API** | Conservation status (Endangered, Vulnerable, etc.). Authoritative extinction risk data. 172,600+ species assessed. | Free for non-commercial use (API key required). Commercial use requires IBAT (paid). |
| **Encyclopedia of Life (EOL) / TraitBank** | Structured trait data (body size, diet, habitat, lifespan, weight). Covers 1.9M+ species. | Free and open |
| **eBird API 2.0** | Bird-specific supplementary data: taxonomy, range, recent sightings, hotspots, seasonal occurrence. | Free with API key |

**Recommendation:** **Wikipedia REST API** as the primary source for fun facts and descriptions. **IUCN Red List** for conservation status badges (note: commercial use licensing needs review). **EOL TraitBank** for structured trait data (weight, lifespan, diet). **eBird API** for bird-specific range and sighting data.

### 4.3 Collection

**Overview**
A personal catalog of every species the user has encountered — browsable, sortable, and satisfying to fill.

**Entry Structure**
Each Collection entry contains:
- **Photo(s)** — All photos the user has taken of this species
- **Species info** — Common name, scientific name, fun facts
- **Sighting history** — Date and (fuzzed) location of each sighting
- **Sighting count** — "Spotted 5 times"
- **Nickname** — Optional user-assigned nickname per individual sighting

**Duplicate Handling**
- If the user photographs a species they've already collected, it **stacks** onto the existing Collection entry
- The entry updates with: new photo added, sighting count incremented, new map pin added
- Each individual sighting is also browsable in a sighting log

**Sorting & Organization**

| Sort Option | Description |
|-------------|-------------|
| **Taxonomy** | Grouped by category: Mammals, Birds (expandable to Reptiles, Insects, etc.) |
| **Alphabetical** | A-Z by common name |
| **Recent** | Most recently spotted first |
| **Most spotted** | Species with the highest sighting count first |

**Empty States**
- Unspotted categories show as locked/silhouetted cards to create a sense of discovery
- Encouraging copy: "Get out there and start snapping!"

### 4.4 Map

**Style**
- Stylized **watercolor** map tiles — hand-painted, organic aesthetic
- Earthy natural tones consistent with the app's overall visual identity
- Custom tile provider or styled tiles (e.g., Stamen Watercolor, Mapbox custom style)

**Pins**
- Each sighting places a pin on the map
- Pins are **species-colored** or use a small circular thumbnail of the user's photo
- Tapping a pin reveals a **mini-card**:
  - Species common name
  - Date of sighting
  - Nickname (if assigned)
  - Small photo thumbnail
  - Tap to expand to full Collection entry

**Location Privacy**
- GPS coordinates are **fuzzed** to a general area (~1km radius) for storage
- Exact coordinates are never stored or transmitted
- Important foundation for future social/sharing features

**Scope**
- MVP: Personal sightings map only
- Roadmap: Community/global sightings map with social features

---

## 5. Visual Design & Art Direction

### 5.1 Design Philosophy

**"Nature Journal"** — The app should feel like a beautiful, hand-crafted field journal. Watercolor textures, organic shapes, and hand-drawn elements throughout. Clean and uncluttered, but with warmth and personality.

### 5.2 Color Palette

**Foundation**
- White/off-white backgrounds as the default canvas
- Watercolor accent textures on cards, headers, and dividers

**Primary Palette (Earthy/Natural)**
| Role | Color | Usage |
|------|-------|-------|
| Forest Green | `#4A7C59` | Primary actions, headers, navigation |
| Sky Blue | `#6B9BC3` | Secondary actions, bird-related accents |
| Warm Brown | `#8B6F47` | Mammal-related accents, earthy elements |
| Soft Cream | `#F5F0E8` | Background tints, card backgrounds |
| Charcoal | `#3A3A3A` | Primary text |

**Biome Accent Colors (Mini-Card Borders)**
| Biome | Color | Example Species |
|-------|-------|-----------------|
| Forest | Deep green | Deer, fox, woodpecker |
| Grassland | Golden wheat | Bison, meadowlark |
| Wetland | Teal blue | Heron, otter |
| Mountain | Slate grey | Mountain goat, eagle |
| Urban | Warm terracotta | Pigeon, raccoon, squirrel |
| Coastal | Sandy gold | Seagull, seal |

### 5.3 Typography

- **Headings:** Rounded, friendly sans-serif (e.g., Nunito, Quicksand)
- **Body:** Clean, readable sans-serif (e.g., Inter, Source Sans Pro)
- **Scientific names:** Italicized body font

### 5.4 Iconography & Illustration

- Hand-drawn / watercolor-style icons for navigation and categories
- Silhouetted animal outlines for unspotted Collection entries
- Subtle paper/canvas texture overlays on card backgrounds
- No heavy drop shadows — use soft watercolor washes for depth

### 5.5 Key UI Elements

**Snap Button**
- Large, centered, circular capture button
- Watercolor-textured ring around it
- Subtle pulse animation when camera is active

**Collection Cards**
- Rounded corners, off-white card with subtle paper texture
- Watercolor biome-accent stripe along the top or left edge
- Species photo as hero image
- Clean typography for species name and sighting count

**Map Pins**
- Small circular photo thumbnail with biome-colored border ring
- Cluster pins when zoomed out (number badge showing count)

**Result Screen (Post-Identification)**
- Clean white card with species photo
- Common name (large), scientific name (small, italic)
- "Added to your Collection!" confirmation
- Button to view entry or nickname the sighting

---

## 6. Information Architecture

```
WildSnap
├── Camera (Default/Home)
│   ├── Viewfinder
│   ├── Capture button
│   ├── Gallery upload button
│   └── Flash / zoom controls
│
├── Collection
│   ├── Sort/filter bar (Taxonomy | A-Z | Recent | Most Spotted)
│   ├── Category sections (Mammals, Birds, ...)
│   │   └── Species cards (grid view)
│   └── Species Detail
│       ├── Photo gallery (all user photos of this species)
│       ├── Species info & fun facts
│       ├── Sighting log (dates, locations, nicknames)
│       └── "View on Map" link
│
├── Map
│   ├── Watercolor map with sighting pins
│   ├── Pin mini-cards (tap to reveal)
│   ├── Filter by species/category
│   └── Zoom clustering
│
└── Profile
    ├── Total species count
    ├── Total sightings count
    ├── Category breakdown (Mammals: 12, Birds: 8, ...)
    ├── Offline queue status
    └── Settings
        ├── Location permissions
        ├── Notification preferences
        └── About / Credits
```

---

## 7. Navigation

**Bottom Tab Bar (4 tabs)**

| Tab | Icon | Screen |
|-----|------|--------|
| Camera | Camera icon | Photo capture (default home screen) |
| Collection | Book/grid icon | Collection browser |
| Map | Map pin icon | Sightings map |
| Profile | User icon | Stats & settings |

The camera is the default tab — the app opens ready to snap.

---

## 8. User Flows

### 8.1 First-Time User

1. App opens → brief onboarding (2-3 screens explaining Snap → Identify → Collect)
2. Permission prompts: Camera, Location, Photo Library
3. Lands on Camera tab — ready to snap
4. Empty Collection and Map with encouraging empty states

### 8.2 New Sighting (Online)

1. User opens Camera tab
2. Frames animal → taps Snap button (or uploads from gallery)
3. Loading state: "Identifying..."
4. Result screen: Species name, photo, "Added to your Collection!"
5. Option to nickname the sighting
6. Collection entry created/updated + map pin dropped

### 8.3 New Sighting (Offline)

1. User snaps photo
2. App displays: "Photo saved! We'll identify it when you're back online."
3. Photo enters offline queue (visible in Profile)
4. On reconnection: auto-identification runs
5. Push notification: "We identified your photo — it's a Red-tailed Hawk!"

### 8.4 Browsing the Collection

1. User taps Collection tab
2. Grid of species cards (collected ones show photo, uncollected show silhouette)
3. Sort/filter to browse by taxonomy or alphabetical
4. Tap a card → Species detail with photos, facts, sighting history
5. "View on Map" links to map filtered to that species

### 8.5 Exploring the Map

1. User taps Map tab
2. Watercolor map loads with all sighting pins
3. Pinch to zoom — pins cluster when zoomed out
4. Tap a pin → mini-card with species, date, nickname
5. Tap mini-card → full Collection entry

---

## 9. Roadmap

### MVP (v1.0)
- Photo capture (camera + gallery)
- AI identification (mammals & birds)
- Collection with sorting
- Personal sightings map (watercolor style)
- Offline photo queue
- Nicknames for sightings
- Local photo storage

### v1.1 — Expanded Species
- Add reptiles & amphibians
- Add insects (supplementary model needed — SpeciesNet has limited insect coverage)
- Improved ID accuracy with model updates

### v1.2 — Social & Community
- User profiles (public/private)
- Community sightings map (global)
- Follow other users
- Share sightings to social media

### v1.3 — Gamification
- Badges & achievements ("First Bird!", "10 Mammals", "7-Day Streak")
- Regional completion tracking ("12 of 47 species in your area")
- Seasonal challenges
- Leaderboards

### v1.4 — Cloud & Sync
- Cloud photo backup
- Cross-device sync
- Web companion app

---

## 10. Open Questions

| # | Question | Status |
|---|----------|--------|
| 1 | Cross-platform framework: React Native vs Flutter? | TBD |
| 2 | Map tile provider for watercolor style? (Stamen, Mapbox, custom) | TBD |
| 3 | SpeciesNet self-hosted vs Animal Detect API? (cost/latency tradeoff) | Needs evaluation |
| 4 | Age restrictions / COPPA compliance requirements? | TBD |
| 5 | Exact GPS fuzzing algorithm (random offset vs grid snap)? | TBD |
| 6 | Push notification strategy for offline queue results? | TBD |
| 7 | IUCN Red List commercial licensing — IBAT required? | Needs review |

---

*Document version: 1.2*
*Last updated: 2026-03-28*
