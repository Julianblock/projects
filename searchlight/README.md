# Searchlight, Nevada

A browser flight game over Searchlight, the old gold-mining town on US-95 between Las Vegas and Laughlin, built from real survey data.

**Role:** Sole developer and designer  
**Type:** Browser game (desktop and mobile)  
**Period:** 2026  
**Status:** Playable at [julianblock.github.io/Searchlight-game](https://julianblock.github.io/Searchlight-game/)  
**Source:** [github.com/Julianblock/Searchlight-game](https://github.com/Julianblock/Searchlight-game)

<p align="center">
  <img src="./hawk-joshua-trees.jpg" alt="A red-tailed hawk gliding low over a Joshua tree forest at golden hour" width="800"/>
</p>

## Overview

You fly as one of sixteen desert birds over Searchlight and 400 km of the Mojave around it: the Joshua tree forests, the Highland Range, Lake Mohave in the canyon below, and the Laughlin casino strip down the Colorado River. The terrain, land cover, roads, buildings and plants all come from public survey data.

The desert changes with the season and time of day: wildflowers and Joshua tree blooms in spring, white-hot summer afternoons, monsoon storms and rare winter snow. At night the neon of Terrible's, the McDonald's sign and the gas station canopies glow on the highway, with the Las Vegas sky-glow and the Luxor beam on the northern horizon. You can swap the bird for a pickup truck at any time.

## What I built

- The whole game: rendering engine, data pipeline, flight model, truck, wildlife, sound, UI and touch controls
- A Node pipeline that turns USGS elevation and imagery, USDA land cover and OpenStreetMap into compact runtime assets, including a separate high-detail region for Laughlin
- Quadtree terrain that streams nested resolution levels and follows the curvature of the Earth
- A physically based sky with Rayleigh and Mie scattering, sun and moon positions for the real date and place, the Milky Way and volumetric clouds
- About half a million desert plants detected from aerial imagery: Joshua trees, yuccas, creosote, cholla, desert willow and more
- Desert washes derived from the terrain with a flow-accumulation model, so vegetation and runoff follow the real drainage
- Hand-built landmarks from photographs: Terrible's Roadhouse, the El Rey Motel, the Duplex Mine headframe and the Laughlin casinos
- A weather system with dust storms, monsoon thunderstorms, rain in the washes and snow on the Joshua trees
- Sixteen birds with researched plumage and separate flight models, a drivable truck with a first-person cockpit, and twelve fish species in Lake Mohave
- About thirty hidden discoveries with lore cards, procedural ambient sound, touch controls and continuous deployment to GitHub Pages

## Technical highlights

### Real data, baked offline

`npm run data` fetches AWS Terrain Tiles (USGS 3DEP), USGS National Map imagery, the USDA Cropland Data Layer and OpenStreetMap, and bakes them into packed height textures, imagery mosaics, a land cover map and a road and building index. Searchlight gets a 22 km high-detail region, the surrounding desert a 92 km one, and the far ranges a 400 km one. Laughlin and Bullhead City get their own 13 km region at higher imagery zoom.

### Washes from the terrain

The desert's character comes from its washes. The pipeline fills pits in the elevation model with a priority flood, routes every cell to its steepest downhill neighbor, and accumulates upstream area. Channels that drain enough desert and sit below their surroundings are painted into the land cover as sandy washes, which decides where desert willow and catclaw grow and where rain runs after a storm.

### Terrain from 64 m to the horizon

The terrain is a quadtree of instanced patches, from a 524 km root down to 64 m tiles near the camera, and the vertex shader bends the ground to the Earth's curvature so distant ranges sink below the horizon.

### A sky computed for the real date

Sun and moon positions come from astronomy for Searchlight's latitude and longitude, and the sky shader integrates Rayleigh and Mie scattering along the view ray. The same model drives desert haze on the terrain. At night the dark-sky Milky Way, the moon phase and the glow of Las Vegas are all in their real positions.

### Running on a phone

The engine tracks frame time and adjusts render resolution to hold a steady frame rate. Touch devices get a lighter rendering path, a floating thumbstick and on-screen buttons, and the game can be added to the home screen as an app.

## Architecture

```mermaid
flowchart LR
  subgraph Sources["Public data (build time)"]
    USGS["USGS 3DEP elevation<br/>and National Map imagery"]
    CDL["USDA land cover"]
    OSM["OpenStreetMap<br/>roads, buildings, landmarks"]
  end

  subgraph Pipeline["Node data pipeline"]
    BAKE["Height, imagery and<br/>land cover mosaics"]
    WASH["Wash detection<br/>flow accumulation"]
    DETECT["Plant detection<br/>from imagery"]
  end

  subgraph Assets["Static assets"]
    TEX["Packed height and<br/>imagery textures"]
    PLANTS["trees.bin"]
    JSON["Road and<br/>building index"]
  end

  subgraph Game["Browser game (three.js, WebGL)"]
    TERRAIN["Quadtree terrain<br/>with Earth curvature"]
    SKY["Atmospheric sky<br/>sun, moon, Milky Way, weather"]
    VEG["Instanced Joshua trees<br/>and desert plants"]
    TOWN["Main Street and Laughlin<br/>hand-built landmarks"]
    LIFE["Birds, wildlife, fish,<br/>traffic"]
    PLAYER["Flight model and truck"]
    AUDIO["Procedural audio"]
  end

  USGS --> BAKE
  CDL --> BAKE
  BAKE --> WASH
  BAKE --> DETECT
  OSM --> JSON
  BAKE --> TEX
  WASH -->|"wash land cover"| TEX
  DETECT --> PLANTS
  TEX --> TERRAIN
  PLANTS --> VEG
  JSON --> TOWN
  SKY --> TERRAIN
  PLAYER --> TERRAIN
  PLAYER --> AUDIO
  LIFE --> AUDIO
  Game -->|"GitHub Actions"| PAGES["GitHub Pages"]
```

## Stack

| Layer | Technology |
|---|---|
| Rendering | three.js, WebGL, custom GLSL shaders |
| Language and build | TypeScript, Vite |
| Data pipeline | Node.js, sharp, geotiff, proj4 |
| Data sources | USGS 3DEP and National Map, USDA Cropland Data Layer, OpenStreetMap |
| Audio | Web Audio API, procedural synthesis |
| Hosting | GitHub Pages, deployed by GitHub Actions |

## Screenshots

### Searchlight at night

<p align="center">
  <img src="./town-at-night.jpg" alt="Searchlight's lights at night under a starry sky, with the Las Vegas glow and the Luxor beam on the horizon" width="800"/>
</p>

The Las Vegas sky-glow and the Luxor beam sit on the northern horizon, where they really are.

### Duplex Hill at sunset

<p align="center">
  <img src="./overlook-sunset.jpg" alt="Looking over Searchlight from Duplex Hill as the sun sets behind the Highland Range" width="800"/>
</p>

### Main Street after dark

<p align="center">
  <img src="./main-street-night.jpg" alt="Searchlight's Main Street and US-95 lit up at night" width="800"/>
</p>

### The Colorado below Davis Dam

<p align="center">
  <img src="./colorado-river.jpg" alt="A bird flying low over the Colorado River through the canyon at dusk" width="800"/>
</p>

### Joshua Tree Highway

<p align="center">
  <img src="./joshua-tree-highway.jpg" alt="Flying above the Joshua Tree Highway heading west into the low sun" width="800"/>
</p>
