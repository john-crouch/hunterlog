---
name: leaflet-ui-patterns
description: Best practices for implementing Leaflet maps in React 18 + TypeScript applications, particularly for desktop embedded webviews. Use when building or refactoring map UI with Leaflet, working with geo data visualization, implementing interactive maps with markers/polylines/layers, or optimizing map performance for large datasets.
---

# Leaflet UI Patterns

## Primary Goals

- Keep map interactions stable and predictable
- Maintain performance with large datasets
- Ensure map UI works well inside desktop embedded webviews

## Hard Rules

**Isolation**: Keep Leaflet-specific code isolated from general UI components

**Typing**: Use TypeScript types for all geo data objects and map-related props

**Stability**: Avoid re-creating map layers on every render - stabilize references and effects

**Performance**: Avoid unnecessary re-renders of map children, especially markers and polylines

## Component Structure

**MapView Component**: Owns map initialization and core layers
- Initialize map once in useEffect
- Store map instance in ref
- Clean up on unmount

**Layer Components**: Separate components for each layer type
- `MarkersLayer`, `PolylinesLayer`, `HeatLayer`, etc.
- Each manages its own layer group
- Add/remove from map in effects

**Non-Map UI**: Keep filters, search, details panels outside map component
- Communicate via props and callbacks
- Maintain state outside Leaflet objects when feasible

## Performance Patterns

**Large Datasets**:
- Use clustering (MarkerClusterGroup) for 100+ markers
- Consider canvas rendering for 1000+ simple markers
- Lazy load or virtualize off-screen features

**Memoization**:
- Use `useMemo` for derived geo arrays
- Use `useCallback` for stable event handlers
- Avoid expensive computations in render - precompute and cache

**Event Handling**:
- Throttle or debounce move, zoom, and search handlers
- Avoid attaching listeners to individual markers when possible
- Use event delegation on layer groups

## Interaction Patterns

**Consistency**:
- Marker click → select
- Popup → show details
- Hover → highlight or tooltip

**State Management**:
- Prefer controlled UI state in React
- Store selection, filters, view state outside Leaflet
- Sync Leaflet view with React state, not vice versa

**Accessibility**:
- Ensure popups don't trap focus
- Support keyboard navigation
- Provide text alternatives for visual-only interactions

## Data Correctness

**Coordinate Validation**:
- Always use `[lat, lng]` order (Leaflet convention)
- Validate: lat ∈ [-90, 90], lng ∈ [-180, 180]
- Handle invalid coords with fallback UI, not silent failure

**Error Handling**:
- Gracefully handle missing or malformed geo data
- Log validation failures for debugging
- Show clear user feedback when data issues occur

## Desktop Embed Considerations

**Responsive Sizing**:
- Use percentage-based dimensions, not fixed pixels
- Listen for window resize events
- Call `map.invalidateSize()` when container resizes

**Performance**:
- Make tile layers configurable (allow disabling for low-end hardware)
- Reduce animation complexity if needed
- Monitor GPU usage in embedded webview

**Container Assumptions**:
- Never assume full-screen availability
- Respect parent container sizing
- Test at various viewport sizes

## Implementation Checklist

**Before Writing Code**:
- [ ] Define TypeScript interfaces for all geo data models
- [ ] Plan layer hierarchy and interaction patterns
- [ ] Identify performance bottlenecks for expected dataset sizes

**During Development**:
- [ ] Initialize map in useEffect with cleanup
- [ ] Store map instance and layer groups in refs
- [ ] Validate all coordinates before rendering
- [ ] Memoize derived geo data and callbacks
- [ ] Test resize behavior

**Before Finalizing**:
- [ ] Verify map loads reliably
- [ ] Test pan, zoom, layer toggle, selection
- [ ] Validate performance with realistic data (1000+ features)
- [ ] Check no console errors or warnings
- [ ] Confirm resize works correctly
- [ ] Test in target embedded webview environment

## Common Pitfalls

**Map Instance Management**:
- ❌ Creating new map instance on every render
- ✅ Create once in useEffect, store in ref

**Event Listeners**:
- ❌ Not removing listeners in cleanup
- ✅ Always clean up in useEffect return function

**State Synchronization**:
- ❌ Mutating Leaflet state directly
- ✅ Keep state in React, sync to Leaflet

**Coordinate Validation**:
- ❌ Rendering invalid coordinates silently
- ✅ Validate and show clear error UI

**Performance**:
- ❌ Creating markers in render loop
- ✅ Use useMemo and stable layer groups

## Code Examples

See `references/EXAMPLES.md` for complete code examples including:
- MapView component implementation
- MarkersLayer with clustering
- Type definitions for geo data
- Validation utilities
- Performance optimization patterns
