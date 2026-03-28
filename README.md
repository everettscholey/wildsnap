# WildSnap

A real-world creature collector. Snap photos of animals in the wild, identify them with AI, build your personal collection, and pin your sightings on a beautiful watercolor map.

## What is WildSnap?

WildSnap is a mobile app for animal lovers who want to document and celebrate the animals they encounter. Photograph wildlife in the real world — through the in-app camera or by uploading from your gallery — and the app identifies the species using AI. Each identified animal is added to your personal collection and pinned on a stylized watercolor world map.

### Core Loop

**Snap** → **Identify** → **Collect** → **Map**

1. **Snap** — Take a photo of an animal (in-app camera or gallery upload)
2. **Identify** — AI analyzes your photo and identifies the species
3. **Collect** — The animal is added to your Collection
4. **Map** — A pin is dropped on your personal sightings map

## Features

- **AI Species Identification** — Species-level identification for mammals and birds, powered by Google SpeciesNet
- **Collection** — A personal catalog of every species you've encountered, with photos, fun facts, sighting history, and optional nicknames
- **Watercolor Sightings Map** — Pin your sightings on a beautiful, hand-painted-style world map with biome-colored mini-cards
- **Offline Support** — Snap photos without cell service; they'll be identified when you're back online
- **Sort & Browse** — Organize your collection by taxonomy, alphabetical, recent, or most spotted

## Tech Stack

- **Platform:** iOS + Android (React Native)
- **AI Engine:** Google SpeciesNet (Apache 2.0, 94.5% species-level accuracy)
- **Species Data:** Wikipedia REST API, IUCN Red List, EOL TraitBank, eBird API
- **Map:** Watercolor-styled tiles (Stamen/Mapbox)
- **Storage:** Local-first; cloud sync on roadmap

## Design

See [DESIGN.md](DESIGN.md) for the full product design document, including visual direction, information architecture, user flows, and roadmap.

## Roadmap

- **v1.0** — Photo capture, AI identification (mammals & birds), collection, watercolor map, offline queue
- **v1.1** — Expanded species (reptiles, amphibians, insects)
- **v1.2** — Social & community features
- **v1.3** — Gamification (badges, streaks, regional tracking)
- **v1.4** — Cloud sync & web companion app

## License

TBD
