# Flight Booking System Documentation

## Table of Contents
- [Overview](#overview)
- [Components](#components)
  - [Flight Details Section](#flight-details-section)
  - [Flight Search Modal](#flight-search-modal)
  - [Searched Flight List](#searched-flight-list)
  - [Flight Selection](#flight-selection)
- [Filtering and Sorting](#filtering-and-sorting)
  - [Sorting Options](#sorting-options)
  - [Filter Options](#filter-options)
- [Modify Search Flights](#modify-search-flights)
- [Data Flow](#data-flow)
- [State Management](#state-management)

---

## Overview

This flight booking system allows users to search, filter, and select flights from multiple sources (TBO and Google SERP API). The system supports multi-route trips with comprehensive filtering options and maintains state through local storage for persistence across page reloads.

---

## Components

### Flight Details Section

#### Initial State (Before Adding Flights)
- Displays a search icon that opens the flight search modal
- Shows empty flight details waiting for user input

#### With Selected Flights
- Displays sections for each route with timestamps
- Shows action icons for each section:
  - **Yellow icon**: Data is being fetched
  - **Search icon**: Search for flights
  - **Checkbox icon**: Select flights (disabled if no API response data available)
  - **Hand icon**: Manually add a flight (for custom bookings when customer wants specific flight not available in system)
  - **Trash icon**: Delete the route

---

### Flight Search Modal

#### Passenger Configuration
- **Number of adults**: Fixed (inherited from form, cannot be changed)
- **Number of children and infants**: Can be modified
- Changes update local `formData` and `localFetchingFlightsData` states
- On submit, updates global/context `formData` and `fetchingFlightsData` states

#### Search Behavior
- Only searches for routes without selected flights (Selected Flights are empty)
- Creates or updates route configurations
- Stores results in `fetchingFlightData` context

#### Modal Features
- Route management (add/remove flights)
- Real-time status indicators
- Simultaneous updates to `fetchingFlightData` and local storage
- Additional flight details configuration (Total Flight Time, Luggage Weight, Seat Day, LCC or FSC, Stopovers, Airline Options)

---

### Searched Flight List

#### Navigation Elements

**Route Tabs**
- Display all available routes at the top of the screen
- Allow switching between different flight segments

**Back Button**
- Returns user to the main form

**Continue/Submit Button**
- **Continue**: Shown for non-final routes → advances to next route
- **Submit**: Shown for final route → saves all data (selected flights, modified formData) to context/global `fetchingFlightsData`, `formData`, and Local Storage

#### State Persistence
Refresh behavior depends on last saved state:
- **If not submitted**: Removes unsaved flight selections and modifications
- **If submitted**: Restores last saved state from local storage

#### Flight Cards

**Interaction**
- Clicking a card selects that flight
- Selected flight details appear in "Selected Flights" section (right sidebar)

**Flight Data Sources**

##### TBO Flights
- Return journeys show both outbound and return legs in single card
- Includes luggage details
- Contains `total_flight_time` field
- Full service information immediately available

##### Google API Flights
- Shows only outbound leg per card
- Requires manual luggage verification before booking
- Luggage details must be checked at later stage
- Additional step required for return flight selection

---

### Flight Selection

#### TBO Flights
- Direct selection on single click
- All information immediately available

#### Google Flights
1. Click opens "Select Return Flight" modal
2. User selects return flight from available options
3. After return flight selection, user chooses booking platform
4. User edits baggage details and updates prices
5. Submit creates Google flight card similar to TBO format with updated details

---

## Filtering and Sorting

### Sorting Options

#### Cheapest
- Compares using `final_price?.total` field
- Works consistently for both TBO and Google flights

#### Fastest
**TBO Flights**
- Uses `total_flight_time` field directly

**Google Flights**
- Primary sort: Outbound duration
- Secondary sort: TBO flights positioned relatively according to their `outbound + return` flight time
- Special handling since Google flights only have one leg

---

### Filter Options

#### Round Trip Total Budget
- Range slider with min and max values
- Dynamically set according to min and max prices in current flight list
- Filters flights within selected price range

#### Stops

**Outbound Section**
- Applies to all flights
- Returns flights with **exact** number of stops as selected
- Options: Direct, 1 Stop, etc.

**Return Section**
- Applies **only** to flights with return data (TBO flights)
- When applied, filters flights with return data based on exact number of stops
- Flights without return data are excluded when return filter is active

#### Departure Time

**Outbound Section**
- Time range slider (12 AM - 12 AM / 0-24 hours)
- Filters all flights based on departure time

**Return Section**
- Time range slider (12 AM - 12 AM / 0-24 hours)
- Applies **only** to flights with return flight data
- Filters based on return flight departure time

#### Journey Hours

**Outbound Section**
- Range slider (0h - 48h)
- Filters based on total journey duration for outbound leg

**Return Section**
- Range slider (0h - 48h)
- Filters based on total journey duration for return leg
- Only applies to flights with return data

#### Airlines
- Multi-select checkbox list
- Shows all available airlines
- Passes flight if **either** outbound **or** return flight airline matches any selected option
- Examples: AirAsia, Indonesia AirAsia, Malaysia Airlines, Philippine Airlines, Scoot, Singapore Airlines, Vietjet, Vietnam Airlines

#### Minimum Required Luggage
- Dropdown selection (0 Kg, 7 Kg, 15 Kg, 20 Kg, 25 Kg, etc.)
- Passes flights where **sum of all check-in luggage** for all passengers for **any leg** (outbound or return) is **greater than or equal to** selected value
- Ensures minimum baggage capacity requirements are met

#### Type of Airlines
- Dropdown selection: FSC / LCC / Both
- **FSC**: Full Service Carrier
- **LCC**: Low Cost Carrier
- Filters based on airline type for any leg (outbound or return)

#### Start Day

**Outbound Flight Start Day**
- Dropdown selection
- Filters flights based on when outbound flight starts relative to selected date
- Options typically include: Same Day, Next Day, Both

**Return Flight Start Day**
- Dropdown selection
- Filters flights based on when return flight starts relative to selected date
- Only applies to flights with return data

---

## Modify Search Flights

### Behavior
- Uses the same component as "Search Flight" modal
- **Key Difference**: Upon submission, saves to temporary states (not context directly)
- Temporary states are declared in the Searched Flight List component
- Data is **not** saved to local storage from this modal

### Workflow
1. User clicks "Modify Search" from Searched Flight List
2. Modal opens with current search parameters
3. User modifies search criteria
4. On submit, updates temporary states in Searched Flight List component
5. User must submit from Searched Flight List to update context states and local storage
6. This allows users to preview changes before committing them

---

## Data Flow

### Data Sources
1. **Data Contexts**: Contains `fetchingFlightData` and `formData`
2. **Local Storage**: Persists data between page reloads
3. **Component States**: Temporary states for intermediate operations

### formData Structure

#### flightDetails Field
Array containing flight details for selected routes

**Empty Route (No Flight Selected)**
```javascript
{
  flightDetails: [
    {
      // Route 1 metadata - no flight selected
    },
    {
      // Route 2 metadata - no flight selected
    }
  ]
}
```

**Route with Selected Flight**
```javascript
{
  flightDetails: [
    {
      id: "4020085726173",
      hasLatestVersionRef: null,
      // Selected flight data
      0: {
        subPrice: 115748,
        supply_cost_incr: 115748,
        totalPrice: 115748,
        returnFlight: true,
        provider: 'TBO',
        // Additional flight details...
      },
      // Passenger information
      1: [
        {
          name: 'Singapore Airlines',
          date: '24/12/2025, 11:58:00',
          // More details...
        }
      ],
      length: 3
    }
  ]
}
```

### fetchingFlightsData Structure

```javascript
{
  0: {
    data: [/* Array of flight objects */],
    error: null,
    error2: "Invalid flight data at index 0",
    loading: false,
    locallyFetched: false,
    payload: {
      isReturnFlight: true,
      origin: 'BOM',
      destination: 'DPS',
      dateRange: Array(2),
      adults: 2,
      // Additional payload data...
    },
    selectedAirlines: ['Singapore Airlines'],
    selectedFlight: [
      {
        subPrice: 115748,
        supply_cost_incr: 115748,
        totalPrice: 115748,
        returnFlight: true,
        provider: 'TBO',
        // Flight details...
      },
      // Passenger data...
    ]
  },
  length: 1
}
```

#### Key Fields
- **data**: Array of available flights for this route
- **error**: Error message if fetch failed
- **loading**: Boolean indicating fetch status
- **payload**: Search parameters used for this route
- **selectedAirlines**: Array of airline names selected for filtering
- **selectedFlight**: Currently selected flight data for this route

---

## State Management

### Component State Flow

#### 1. Search Flight Modal
- Creates **local states** for `formData` and `fetchingFlightsData`
- On adding new route:
  - Adds metadata to local `formData.flightDetails`
  - Adds metadata to local `fetchingFlightDetails`
- **Submit button**: Updates context states from local states

#### 2. Searched Flight List States

**Primary States**
- `selectedFlights`: Currently selected flights across all routes
- `selectedFlightIndex`: Index of selected flight in current sorted/filtered list
- `temporaryFetchingFlightsData`: Initialized from context's `fetchingFlightData`

**selectedFlightIndex Purpose**
- Updates when sort option changes
- Ensures correct flight displays as selected after re-sorting
- Maintains selection consistency across filter/sort operations

#### 3. Modify Search Flow

When user clicks "Modify Search":
1. Passes `selectedFlights` and `selectedFlightIndex` to modal
2. If modifying route with selected flight:
   - Removes selected flight data from local `formData`
   - Removes from local `fetchingFlightsData`
   - Removes from `selectedFlights`
   - Updates `selectedFlightIndex`
3. Context states remain **unchanged** (only temporary states update)
4. Changes only affect temporary state in Searched Flight List

#### 4. Submit from Searched Flight List
- Temporary states update context states
- Data saved to local storage
- User redirected to main form
- Changes now persisted across application

### State Persistence Strategy

**Local Storage**
- Stores context `formData` and `fetchingFlightData`
- Enables recovery after page refresh
- Updated only on final submit (not during modify operations)

**Context/Global State**
- Single source of truth for application
- Updated from local states on submit
- Shared across all components

**Temporary States**
- Used during modification operations
- Allows preview before committing
- Discarded if user cancels or refreshes without submitting

---

## Best Practices

### For Users
1. Always submit from Searched Flight List to save changes
2. Use "Modify Search" to experiment with different search parameters
3. Check luggage details carefully for Google flights before booking
4. Verify all route selections before final submit

### For Developers
1. Maintain separation between temporary and global states
2. Always update local storage when updating context states
3. Handle both TBO and Google flight formats in all components
4. Validate flight data before allowing selection
5. Ensure filter logic accounts for flights with/without return data
6. Test state persistence across page reloads

---

## Error Handling

### Common Scenarios
- **Invalid flight data**: Check `error` and `error2` fields in `fetchingFlightsData`
- **Missing return data**: Filters automatically exclude when return filters applied
- **State mismatch**: Refresh restores last submitted state from local storage
- **API failures**: Loading states and error messages guide user

### Recovery
- Unsaved changes lost on refresh (by design)
- Local storage provides fallback for submitted data
- Context state rebuilds from local storage on app initialization

---

## Summary

This flight booking system provides a robust, multi-step process for searching and selecting flights:

1. **Search**: Configure routes and passenger counts
2. **Browse**: View filtered and sorted flight options
3. **Select**: Choose flights for each route segment
4. **Modify**: Adjust search parameters as needed
5. **Submit**: Persist selections to global state and storage

The dual-state system (temporary + global) ensures users can experiment safely while maintaining data integrity through explicit save points.