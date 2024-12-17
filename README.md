# PlaceTrace Architecture Documentation

## Overview

PlaceTrace is a geo-guessing game built with React, TypeScript, and MapillaryJS. Players are shown street-view locations and must guess their position on a map. This document provides a detailed breakdown of the project's architecture and components.

## Core Components

### 1. MapillaryViewer (`src/components/MapillaryViewer.tsx`)

The primary component for displaying street view imagery.

**Key Features:**
- Initializes and manages the MapillaryJS viewer
- Handles image loading and error states
- Integrates with game controls and timer
- Manages viewer configuration and cleanup

**Notable Implementation Details:**
```typescript
// Viewer initialization with optimized settings
const viewer = new Viewer({
  accessToken: MAPILLARY_ACCESS_TOKEN,
  container: containerRef.current,
  imageId,
  component: {
    cover: false,
    sequence: false,
    image: true,
    navigation: false,
  },
  transitionMode: 'instantaneous',
  renderMode: 'quality',
});
```

### 2. GuessMap (`src/components/GuessMap.tsx`)

Interactive map component for player guesses.

**Key Features:**
- Leaflet map integration
- Custom markers for guesses
- Expandable map interface
- Distance calculation visualization

**Notable Implementation:**
```typescript
// Custom marker implementation
const markerHtml = `
  <div class="relative">
    <div class="absolute -translate-x-1/2 -translate-y-full flex flex-col items-center">
      <div class="w-6 h-6 bg-red-500 rounded-full border-2 border-white shadow-lg">
        <!-- Marker content -->
      </div>
    </div>
  </div>
`;
```

### 3. ResultMap (`src/components/ResultMap.tsx`)

Displays the results after each guess.

**Key Features:**
- Shows actual vs guessed locations
- Calculates and displays distance
- Animated line between points
- Custom markers with animations

### 4. ScoreBoard (`src/components/ScoreBoard.tsx`)

Tracks and displays game progress.

**Key Features:**
- Current score display
- Round tracking
- Animated score updates
- Responsive design

### 5. Timer (`src/components/Timer.tsx`)

Countdown timer for each round.

**Key Features:**
- Configurable time limit
- Visual countdown
- Auto-submission on expiry
- Pause/resume functionality

## Game Logic

### Score Calculation (`src/utils/game.ts`)

The scoring system uses an exponential decay function:

```typescript
export const calculateScore = (actual, guessed): number => {
  const distance = calculateDistance(actual, guessed);
  return Math.round(Math.max(0, 5000 * Math.exp(-distance/2000)));
};
```

**Scoring Breakdown:**
- Perfect guess: 5000 points
- Points decrease exponentially with distance
- Minimum score: 0 points

### Distance Calculation

Uses the Haversine formula for accurate Earth-surface distances:

```typescript
export const calculateDistance = (actual, guessed): number => {
  const R = 6371; // Earth's radius in km
  // ... Haversine formula implementation
};
```

## State Management

### Game State Interface

```typescript
interface GameState {
  score: number;
  round: number;
  maxRounds: number;
  currentLocation: Location | null;
  guessedLocation: [number, number] | null;
  showResults: boolean;
  gameOver: boolean;
  timeExpired: boolean;
}
```

### Location Management

Locations are stored in `src/data/locations.ts`:

```typescript
export interface Location {
  id: string;        // Mapillary image ID
  lat: number;       // Latitude
  lng: number;       // Longitude
  name: string;      // Location name
}
```

## UI/UX Design

### Styling Architecture

The project uses Tailwind CSS with custom utilities:

1. **Base Styles** (`src/index.css`):
   - Custom animations
   - Map control styling
   - Marker animations

2. **Component-Specific Styles**:
   - Backdrop blur effects
   - Responsive layouts
   - Interactive elements

### Responsive Design

- Mobile-first approach
- Flexible layouts
- Touch-friendly controls
- Adaptive map sizing

## Performance Optimizations

1. **MapillaryJS Viewer:**
   - Optimized image loading
   - Component cleanup
   - Error recovery

2. **Map Components:**
   - Lazy marker creation
   - Efficient re-renders
   - Memory management

3. **State Management:**
   - Memoized calculations
   - Optimized re-renders
   - Efficient data structures

## Error Handling

1. **Viewer Initialization:**
   - Retry mechanism
   - Graceful fallbacks
   - User feedback

2. **Network Issues:**
   - Loading states
   - Error messages
   - Recovery options

## Future Enhancements

1. **Planned Features:**
   - Multiplayer support
   - Custom location sets
   - Achievement system
   - Social sharing

2. **Technical Improvements:**
   - PWA support
   - Offline capabilities
   - Performance monitoring

## Development Guidelines

1. **Code Style:**
   - TypeScript strict mode
   - React best practices
   - Component composition

2. **Testing:**
   - Unit tests for utilities
   - Component testing
   - Integration tests

3. **Performance:**
   - Bundle optimization
   - Image optimization
   - Caching strategies



# PlaceTrace Code Organization Documentation

## Directory Structure

The project follows a modular organization pattern with clear separation of concerns:

```
src/
├── data/        # Data constants and static content
├── types/       # TypeScript type definitions
└── utils/       # Utility functions and helpers
```

## 1. Data Layer (`src/data/`)

### locations.ts

This file contains the game's location database with carefully curated points of interest.

```typescript
export const LOCATIONS: Location[] = [
  {
    id: '279515028402745',
    lat: 57.053029733333,
    lng: -3.0346660333333,
    name: 'Cairngorms National Park, Scotland'
  },
  // ... more locations
];
```

**Key Aspects:**
- Each location is precisely geocoded
- Mapillary IDs are verified and active
- Diverse global coverage
- Balanced difficulty distribution
- Memorable landmarks and locations

**Usage Example:**
```typescript
import { LOCATIONS } from '../data/locations';
const randomLocation = LOCATIONS[Math.floor(Math.random() * LOCATIONS.length)];
```

## 2. Type Definitions (`src/types/`)

### index.ts

Central type definitions file that ensures type safety across the application.

```typescript
export interface Location {
  id: string;        // Mapillary image ID
  lat: number;       // Latitude
  lng: number;       // Longitude
  name: string;      // Location description
}

export interface GameState {
  score: number;     // Current game score
  round: number;     // Current round number
  maxRounds: number; // Total rounds in game
  currentLocation: Location | null;
  guessedLocation: [number, number] | null;
  showResults: boolean;
  gameOver: boolean;
  timeExpired: boolean;
}

export interface ScoreBoardProps {
  score: number;
  round: number;
  maxRounds: number;
}
```

**Type System Benefits:**
1. **Location Interface:**
   - Ensures data consistency
   - Prevents invalid coordinates
   - Maintains Mapillary ID format
   - Enforces location naming

2. **GameState Interface:**
   - Tracks complete game lifecycle
   - Manages round progression
   - Handles game completion states
   - Coordinates timing events

3. **Component Props:**
   - Enforces proper prop passing
   - Documents component requirements
   - Enables IDE autocompletion
   - Prevents runtime errors

## 3. Utility Functions (`src/utils/`)

### game.ts

Core game mechanics and calculations.

```typescript
// 1. Distance Calculation
export const calculateDistance = (
  actual: { lat: number; lng: number },
  guessed: [number, number]
): number => {
  const R = 6371; // Earth's radius in kilometers
  const φ1 = actual.lat * Math.PI / 180;
  const φ2 = guessed[0] * Math.PI / 180;
  const Δφ = (guessed[0] - actual.lat) * Math.PI / 180;
  const Δλ = (guessed[1] - actual.lng) * Math.PI / 180;

  // Haversine formula for great-circle distance
  const a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
           Math.cos(φ1) * Math.cos(φ2) *
           Math.sin(Δλ/2) * Math.sin(Δλ/2);
  
  const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
  return Math.round(R * c);
};

// 2. Score Calculation
export const calculateScore = (
  actual: { lat: number; lng: number },
  guessed: [number, number]
): number => {
  const distance = calculateDistance(actual, guessed);
  return Math.round(Math.max(0, 5000 * Math.exp(-distance/2000)));
};

// 3. Array Randomization
export const shuffleArray = <T>(array: T[]): T[] => {
  const newArray = [...array];
  for (let i = newArray.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [newArray[i], newArray[j]] = [newArray[j], newArray[i]];
  }
  return newArray;
};

// 4. Location Selection
export const getRandomLocation = (
  locations: any[],
  usedLocations: Set<string>
) => {
  const availableLocations = locations.filter(
    loc => !usedLocations.has(loc.id)
  );
  if (availableLocations.length === 0) {
    return locations[Math.floor(Math.random() * locations.length)];
  }
  return availableLocations[Math.floor(Math.random() * availableLocations.length)];
};
```

**Utility Functions Breakdown:**

1. **calculateDistance:**
   - Uses Haversine formula for accuracy
   - Accounts for Earth's curvature
   - Returns distance in kilometers
   - Handles coordinate wrapping

2. **calculateScore:**
   - Exponential scoring system
   - Maximum score: 5000 points
   - Decreases with distance
   - Minimum score: 0 points

3. **shuffleArray:**
   - Fisher-Yates shuffle algorithm
   - Type-safe implementation
   - Maintains array immutability
   - Ensures random distribution

4. **getRandomLocation:**
   - Prevents location repetition
   - Handles exhausted locations
   - Maintains game variety
   - Type-safe selection

## Best Practices Implementation

1. **Single Responsibility Principle:**
   - Each file has a specific purpose
   - Functions are focused and pure
   - Clear separation of concerns
   - Modular organization

2. **Type Safety:**
   - Comprehensive type definitions
   - Generic type implementations
   - Strict null checks
   - Interface segregation

3. **Code Reusability:**
   - Utility function exports
   - Shared type definitions
   - Consistent data structures
   - DRY principle adherence

4. **Performance Considerations:**
   - Efficient algorithms
   - Optimized calculations
   - Memoization-ready
   - Resource-conscious

## Usage Guidelines

1. **Adding New Locations:**
   ```typescript
   // In locations.ts
   export const LOCATIONS: Location[] = [
     ...existingLocations,
     {
       id: 'new-mapillary-id',
       lat: 0.0,
       lng: 0.0,
       name: 'New Location Name'
     }
   ];
   ```

2. **Extending Types:**
   ```typescript
   // In types/index.ts
   export interface ExtendedGameState extends GameState {
     newFeature: FeatureType;
   }
   ```

3. **Adding Utility Functions:**
   ```typescript
   // In utils/game.ts
   export const newUtilityFunction = (params: ParamType): ReturnType => {
     // Implementation
   };
   ```

This organization ensures maintainability, scalability, and code quality while following TypeScript and React best practices.



# PlaceTrace Core Files Documentation

## Core Application Files

### 1. App.tsx

The main application component that orchestrates the entire game flow.

```typescript
// Key Components
const App: React.FC = () => {
  const [gameState, setGameState] = useState<GameState>(() => ({
    score: 0,
    round: 1,
    maxRounds: 5,
    currentLocation: shuffleArray(LOCATIONS)[0],
    guessedLocation: null,
    showResults: false,
    gameOver: false,
    timeExpired: false,
    locations: shuffleArray(LOCATIONS)
  }));
```

**Key Features:**
1. **State Management:**
   - Centralized game state
   - Round progression
   - Score tracking
   - Location management

2. **Game Flow Handlers:**
   ```typescript
   const handleGuess = useCallback((coords: [number, number]) => {
     // Processes player guesses
     // Calculates scores
     // Updates game state
   });

   const handleTimeUp = useCallback(() => {
     // Manages round time expiration
     // Forces round completion
   });

   const nextRound = useCallback(() => {
     // Advances to next round
     // Updates locations
     // Resets round state
   });
   ```

3. **UI Components:**
   - MapillaryViewer for street view
   - GuessMap for player input
   - ScoreBoard for game progress
   - ResultMap for round results

4. **Game Over Handling:**
   ```typescript
   {gameState.gameOver && (
     <div className="absolute inset-0 bg-black/80 backdrop-blur-md">
       // Final score display
       // Restart options
       // Share functionality
     </div>
   )}
   ```

### 2. config.ts

Configuration management and environment variables.

```typescript
export const MAPILLARY_ACCESS_TOKEN = import.meta.env.VITE_MAPILLARY_ACCESS_TOKEN;
```

**Key Aspects:**
- Environment variable management
- API key security
- Configuration centralization
- Type-safe exports

### 3. index.css

Global styles and Tailwind CSS configuration.

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom Animations */
@keyframes marker-ping {
  75%, 100% {
    transform: scale(2);
    opacity: 0;
  }
}

/* Map Customization */
.leaflet-container {
  font-family: inherit;
  background-color: #f0f8ff;
}

/* Control Styling */
.leaflet-control-zoom {
  margin: 12px !important;
  border: none !important;
  box-shadow: 0 2px 4px rgba(0,0,0,0.2) !important;
}
```

**Style Categories:**
1. **Tailwind Integration:**
   - Base styles
   - Component utilities
   - Custom extensions

2. **Custom Animations:**
   - Marker effects
   - UI transitions
   - Loading states

3. **Map Customization:**
   - Control styling
   - Interactive elements
   - Responsive design

### 4. main.tsx

Application entry point and React initialization.

```typescript
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';
import App from './App.tsx';
import './index.css';

createRoot(document.getElementById('root')!).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

**Key Features:**
1. **React 18 Setup:**
   - Strict Mode enabled
   - Concurrent features
   - Development tools

2. **Entry Point:**
   - Application mounting
   - Style imports
   - Root configuration

### 5. types.ts

Core type definitions and interfaces.

```typescript
export interface Location {
  id: string;
  lat: number;
  lng: number;
  name: string;
}

export interface GameState {
  score: number;
  round: number;
  maxRounds: number;
  currentLocation: Location | null;
  guessedLocation: [number, number] | null;
  showResults: boolean;
  gameOver: boolean;
  timeExpired: boolean;
}
```

**Type System:**
1. **Location Interface:**
   - Geographic coordinates
   - Mapillary integration
   - Location metadata

2. **GameState Interface:**
   - Game progression
   - Player interactions
   - Round management
   - UI state control

## Best Practices Implementation

### 1. Component Organization
- Clear separation of concerns
- Modular structure
- Reusable components
- Consistent patterns

### 2. State Management
- Centralized game state
- Immutable updates
- Callback memoization
- Type-safe actions

### 3. Performance Optimization
- Efficient re-renders
- Memoized callbacks
- Lazy loading
- Resource management

### 4. Error Handling
- Graceful fallbacks
- User feedback
- Type safety
- Recovery mechanisms

## Development Guidelines

### 1. File Structure
```
src/
├── App.tsx           # Main application component
├── config.ts         # Configuration and environment
├── index.css         # Global styles
├── main.tsx         # Application entry
└── types.ts         # Core type definitions
```

### 2. Code Style
- Consistent formatting
- Clear naming conventions
- Comprehensive comments
- Type safety

### 3. State Updates
```typescript
// Correct way to update state
setGameState(prev => ({
  ...prev,
  score: prev.score + newScore,
  round: prev.round + 1
}));
```

### 4. Component Composition
```typescript
// Prefer composition over inheritance
const GameLayout: React.FC = () => (
  <div className="relative w-screen h-screen">
    <MapillaryViewer />
    <GuessMap />
    <Controls />
  </div>
);
```

## Maintenance and Updates

### 1. Adding Features
- Update types first
- Maintain backwards compatibility
- Document changes
- Test thoroughly

### 2. Configuration Changes
```typescript
// Add new environment variables
export const NEW_CONFIG = import.meta.env.VITE_NEW_CONFIG;
```

### 3. Style Updates
```css
/* Add new custom styles */
@layer components {
  .new-component {
    @apply /* tailwind classes */;
  }
}
```

This documentation provides a comprehensive overview of the core files in the PlaceTrace project, their purposes, and best practices for maintenance and development.

# PlaceTrace UI Components Documentation

## Interactive Components

### 1. Achievements Component (`Achievements.tsx`)

A modal component that displays player achievements and progress.

```typescript
interface Achievement {
  id: string;
  name: string;
  description: string;
  icon: string;
  unlocked: boolean;
  progress: number;
  target: number;
}
```

**Key Features:**
- Modal-based display
- Progress tracking
- Animated unlocks
- Achievement categories

**Implementation Details:**
```typescript
const Achievements: React.FC<AchievementsProps> = ({ achievements }) => {
  const [isOpen, setIsOpen] = useState(false);
  
  // Modal trigger button
  <button className="w-12 h-12 bg-white/10 backdrop-blur-sm rounded-full">
    <Medal className="w-6 h-6" />
  </button>

  // Achievement cards with progress bars
  {achievements.map((achievement) => (
    <div className={`p-4 rounded-lg ${
      achievement.unlocked ? 'bg-green-500/20' : 'bg-white/5'
    }`}>
    // Achievement content
  </div>
  ))}
```

### 2. Compass Component (`Compass.tsx`)

A dynamic compass that shows the current viewing direction.

**Key Features:**
- Real-time bearing updates
- Smooth animations
- Visual direction indicator
- Degree display

**Implementation Details:**
```typescript
const Compass: React.FC<CompassProps> = ({ bearing }) => {
  const [normalizedBearing, setNormalizedBearing] = useState(0);

  // Normalize bearing to 0-360 range
  useEffect(() => {
    const normalized = ((bearing % 360) + 360) % 360;
    setNormalizedBearing(normalized);
  }, [bearing]);

  return (
    <div className="absolute top-3 left-1/2 -translate-x-1/2 z-10">
      <div className="bg-white/10 backdrop-blur-sm rounded-full">
        // Compass visualization
        <div className="w-32 h-1 bg-white/20 rounded-full relative">
          // Direction indicator
        </div>
      </div>
    </div>
  );
};
```

### 3. Controls Component (`Controls.tsx`)

Game control buttons for settings and actions.

**Key Features:**
- Settings access
- Flag placement
- Floating button layout
- Hover effects

**Implementation Details:**
```typescript
const Controls: React.FC = () => {
  return (
    <div className="absolute bottom-4 left-4 flex flex-col gap-2">
      <button className="w-12 h-12 bg-white/10 backdrop-blur-sm rounded-full">
        <Settings className="w-6 h-6" />
      </button>
      <button className="w-12 h-12 bg-white/10 backdrop-blur-sm rounded-full">
        <Flag className="w-6 h-6" />
      </button>
    </div>
  );
};
```

### 4. DirectionIndicator Component (`DirectionIndicator.tsx`)

Shows cardinal directions and bearing angles.

**Key Features:**
- Cardinal direction display (N, NE, E, etc.)
- Rotating compass needle
- Bearing angle display
- Smooth transitions

**Implementation Details:**
```typescript
const DirectionIndicator: React.FC<DirectionIndicatorProps> = ({ bearing }) => {
  const directions = ['N', 'NE', 'E', 'SE', 'S', 'SW', 'W', 'NW'];
  const index = Math.round(((bearing % 360) / 45)) % 8;
  const direction = directions[index];

  return (
    <div className="absolute top-3 left-1/2 -translate-x-1/2 z-10">
      <div className="bg-white/10 backdrop-blur-sm rounded-lg">
        <Navigation 
          className="w-5 h-5 text-white" 
          style={{ transform: `rotate(${bearing}deg)` }}
        />
        <span className="text-white font-medium">{direction}</span>
      </div>
    </div>
  );
};
```

### 5. GuessMap Component (`GuessMap.tsx`)

Interactive map for player location guesses.

**Key Features:**
- Leaflet map integration
- Custom markers
- Expandable interface
- Distance calculation

**Implementation Details:**
```typescript
const GuessMap: React.FC<GuessMapProps> = ({
  onGuess,
  guessedLocation,
  actualLocation,
  bearing
}) => {
  const mapRef = useRef<L.Map | null>(null);
  const [tempMarker, setTempMarker] = useState<[number, number] | null>(null);
  const [isExpanded, setIsExpanded] = useState(false);

  // Map initialization
  useEffect(() => {
    if (!mapRef.current) {
      const map = L.map('map').setView([20, 0], 2);
      // Map setup and event handlers
    }
  }, []);

  // Custom marker implementation
  const markerHtml = `
    <div class="relative">
      <div class="absolute -translate-x-1/2 -translate-y-full">
        // Marker content
      </div>
    </div>
  `;
};
```

## Component Integration

### 1. Layout Organization
```typescript
<div className="relative w-screen h-screen">
  <MapillaryViewer />
  <Compass bearing={currentBearing} />
  <DirectionIndicator bearing={currentBearing} />
  <GuessMap onGuess={handleGuess} />
  <Controls />
</div>
```

### 2. State Management
```typescript
// Parent component manages state
const [bearing, setBearing] = useState(0);
const [isGuessing, setIsGuessing] = useState(false);

// Pass state to children
<Compass bearing={bearing} />
<DirectionIndicator bearing={bearing} />
```

### 3. Event Handling
```typescript
const handleGuess = (coords: [number, number]) => {
  setGuessedLocation(coords);
  calculateScore(coords);
  showResults();
};
```

## Styling Patterns

### 1. Common UI Elements
```typescript
// Backdrop blur effect
className="bg-white/10 backdrop-blur-sm"

// Button styling
className="w-12 h-12 rounded-full hover:bg-white/20 transition-colors"

// Container positioning
className="absolute top-3 left-1/2 -translate-x-1/2 z-10"
```

### 2. Animations
```css
// Marker ping animation
@keyframes marker-ping {
  75%, 100% {
    transform: scale(2);
    opacity: 0;
  }
}

// Transition properties
transition: transform 0.3s ease-out;
```

## Best Practices

### 1. Component Design
- Single responsibility
- Prop type validation
- Memoization where needed
- Clean event handling

### 2. Performance
- Efficient re-renders
- Proper cleanup
- Optimized calculations
- Resource management

### 3. Accessibility
- ARIA labels
- Keyboard navigation
- Color contrast
- Focus management

### 4. Error Handling
- Graceful fallbacks
- Loading states
- Error boundaries
- User feedback

This documentation provides a comprehensive overview of the UI components in PlaceTrace, their implementation details, and best practices for development and maintenance.


# PlaceTrace UI Components Documentation - Part 2

## Core Game Components

### 1. MapillaryViewer Component (`MapillaryViewer.tsx`)

The primary component for displaying street view imagery using MapillaryJS.

**Key Features:**
- Street view initialization and management
- Error handling and recovery
- Loading states
- Component lifecycle management

**Implementation Details:**
```typescript
interface MapillaryViewerProps {
  imageId: string;
  round: number;
  maxRounds: number;
  onTimeUp?: () => void;
}

const MapillaryViewer: React.FC<MapillaryViewerProps> = ({
  imageId,
  round,
  maxRounds,
  onTimeUp
}) => {
  const containerRef = useRef<HTMLDivElement>(null);
  const viewerRef = useRef<Viewer | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);
```

**Viewer Initialization:**
```typescript
const initViewer = async () => {
  // Configuration for optimal performance
  viewer = new Viewer({
    accessToken: MAPILLARY_ACCESS_TOKEN,
    container: containerRef.current,
    imageId,
    component: {
      cover: false,
      sequence: false,
      image: true,
      navigation: false,
    },
    transitionMode: 'instantaneous',
    renderMode: 'quality',
  });
```

**Error Recovery:**
```typescript
try {
  await new Promise((resolve, reject) => {
    const timeout = setTimeout(() => {
      reject(new Error('Image load timeout'));
    }, 10000);

    viewer!.on('image', () => {
      clearTimeout(timeout);
      setIsLoading(false);
      resolve(true);
    });
  });
} catch (error) {
  // Retry mechanism
  if (initializationAttempts.current < 3) {
    await initViewer();
  } else {
    setError('Failed to initialize street view');
  }
}
```

### 2. ResultMap Component (`ResultMap.tsx`)

Displays the results after each guess, showing actual vs guessed locations.

**Key Features:**
- Custom markers for actual and guessed locations
- Distance line visualization
- Animated transitions
- Map bounds management

**Implementation Details:**
```typescript
interface ResultMapProps {
  guessedLocation: [number, number];
  actualLocation: [number, number];
  timeExpired?: boolean;
}

const ResultMap: React.FC<ResultMapProps> = ({
  guessedLocation,
  actualLocation,
  timeExpired
}) => {
  const mapRef = useRef<L.Map | null>(null);
```

**Marker Creation:**
```typescript
// Guess marker
guessMarkerRef.current = L.marker(guessedLocation, {
  icon: L.divIcon({
    className: 'guess-marker',
    html: `
      <div class="relative">
        <div class="absolute -top-3 -left-3 w-6 h-6 bg-red-500 rounded-full border-2 border-white shadow-lg">
          <div class="absolute inset-0 bg-red-500 rounded-full animate-ping opacity-75"></div>
        </div>
      </div>
    `,
  })
}).addTo(map);

// Distance line
lineRef.current = L.polyline(
  [guessedLocation, actualLocation],
  {
    color: '#6366f1',
    weight: 3,
    opacity: 0.8,
    dashArray: '10, 10'
  }
).addTo(map);
```

**Distance Display:**
```typescript
// Midpoint marker for distance
const midLat = (guessedLocation[0] + actualLocation[0]) / 2;
const midLng = (guessedLocation[1] + actualLocation[1]) / 2;
const distance = calculateDistance(
  { lat: actualLocation[0], lng: actualLocation[1] },
  guessedLocation
);

distanceMarkerRef.current = L.marker([midLat, midLng], {
  icon: L.divIcon({
    html: `
      <div class="px-3 py-1.5 bg-white/90 backdrop-blur-sm rounded-full shadow-lg">
        <span class="text-sm font-semibold">${distance.toLocaleString()} km</span>
      </div>
    `,
  })
}).addTo(map);
```

### 3. RoundCounter Component (`RoundCounter.tsx`)

Displays the current round progress.

**Key Features:**
- Round tracking
- Visual progress indication
- Responsive design
- Clean animation transitions

**Implementation Details:**
```typescript
interface RoundCounterProps {
  round: number;
  maxRounds: number;
}

const RoundCounter: React.FC<RoundCounterProps> = ({ round, maxRounds }) => {
  return (
    <div className="absolute top-4 right-4 bg-white/10 backdrop-blur-sm text-white px-4 py-2 rounded-lg shadow-lg">
      <div className="text-sm font-medium opacity-75">ROUND</div>
      <div className="text-xl font-bold">{round}/{maxRounds}</div>
    </div>
  );
};
```

### 4. ScoreBoard Component (`ScoreBoard.tsx`)

Displays the current game score and round information.

**Key Features:**
- Score display
- Round tracking
- Animated score updates
- Trophy icon integration

**Implementation Details:**
```typescript
interface ScoreBoardProps {
  score: number;
  round: number;
  maxRounds: number;
}

const ScoreBoard: React.FC<ScoreBoardProps> = ({
  score,
  round,
  maxRounds
}) => {
  return (
    <div className="flex items-center gap-4">
      <div className="bg-white/10 backdrop-blur-md rounded-lg shadow-lg px-4 py-2 flex items-center gap-2">
        <Trophy className="w-4 h-4 text-yellow-400" />
        <span className="text-base font-bold text-white">
          {score.toLocaleString()} points
        </span>
      </div>
      <div className="bg-white/10 backdrop-blur-md rounded-lg shadow-lg px-4 py-2 text-white text-sm">
        Round {round} of {maxRounds}
      </div>
    </div>
  );
};
```

## Component Integration Best Practices

### 1. State Management
```typescript
// In parent component
const [gameState, setGameState] = useState<GameState>({
  score: 0,
  round: 1,
  maxRounds: 5,
  // ... other state properties
});

// Pass required props to components
<ScoreBoard
  score={gameState.score}
  round={gameState.round}
  maxRounds={gameState.maxRounds}
/>
```

### 2. Error Boundaries
```typescript
class GameErrorBoundary extends React.Component {
  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // Log error to service
    // Show fallback UI
  }
}
```

### 3. Performance Optimization
```typescript
// Memoize components when needed
const MemoizedScoreBoard = React.memo(ScoreBoard);

// Use callbacks for event handlers
const handleTimeUp = useCallback(() => {
  // Handle time expiration
}, []);
```

### 4. Accessibility Features
```typescript
// Add ARIA labels and roles
<button
  aria-label="Start new round"
  role="button"
  className="..."
>
  Next Round
</button>
```

## Styling Guidelines

### 1. Common Classes
```typescript
// Backdrop blur container
const backdropClass = "bg-white/10 backdrop-blur-sm";

// Button styles
const buttonClass = "rounded-lg shadow-lg transition-colors";

// Text styles
const headingClass = "text-xl font-bold text-white";
```

### 2. Animation Classes
```css
.score-update {
  animation: score-pop 0.3s ease-out;
}

@keyframes score-pop {
  0% { transform: scale(1); }
  50% { transform: scale(1.1); }
  100% { transform: scale(1); }
}
```

This documentation provides a comprehensive overview of the core game components, their implementation details, and best practices for integration and styling.
