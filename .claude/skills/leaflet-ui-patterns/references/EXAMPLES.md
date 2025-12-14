# Leaflet UI Pattern Examples

## Type Definitions

```typescript
// types/geo.ts
export interface GeoPoint {
  id: string;
  lat: number;
  lng: number;
  properties?: Record<string, any>;
}

export interface GeoPolyline {
  id: string;
  points: [number, number][]; // [lat, lng][]
  properties?: Record<string, any>;
}

export interface MapViewProps {
  center: [number, number];
  zoom: number;
  children?: React.ReactNode;
  onMoveEnd?: (bounds: L.LatLngBounds) => void;
  onZoomEnd?: (zoom: number) => void;
}

export interface MarkersLayerProps {
  map: L.Map;
  points: GeoPoint[];
  selectedId?: string;
  onMarkerClick?: (point: GeoPoint) => void;
}

export interface PolylinesLayerProps {
  map: L.Map;
  polylines: GeoPolyline[];
  color?: string;
  weight?: number;
}
```

## MapView Component

```typescript
// components/MapView.tsx
import { useEffect, useRef, useCallback } from 'react';
import L from 'leaflet';
import 'leaflet/dist/leaflet.css';
import type { MapViewProps } from '../types/geo';

export const MapView: React.FC<MapViewProps> = ({ 
  center, 
  zoom, 
  children,
  onMoveEnd,
  onZoomEnd
}) => {
  const mapRef = useRef<L.Map | null>(null);
  const containerRef = useRef<HTMLDivElement>(null);

  // Initialize map once
  useEffect(() => {
    if (!containerRef.current || mapRef.current) return;

    const map = L.map(containerRef.current, {
      zoomControl: false, // Add custom controls separately
    }).setView(center, zoom);

    // Add tile layer
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '© OpenStreetMap contributors',
      maxZoom: 19,
    }).addTo(map);

    // Add zoom control to top-right
    L.control.zoom({ position: 'topright' }).addTo(map);

    mapRef.current = map;

    return () => {
      map.remove();
      mapRef.current = null;
    };
  }, []); // Empty deps - only initialize once

  // Handle resize
  useEffect(() => {
    const map = mapRef.current;
    if (!map) return;

    const handleResize = () => {
      map.invalidateSize();
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  // Update center/zoom when props change
  useEffect(() => {
    const map = mapRef.current;
    if (!map) return;

    const currentCenter = map.getCenter();
    const currentZoom = map.getZoom();

    // Only update if actually different (avoid unnecessary animations)
    if (currentCenter.lat !== center[0] || currentCenter.lng !== center[1] || currentZoom !== zoom) {
      map.setView(center, zoom);
    }
  }, [center, zoom]);

  // Attach move/zoom event handlers
  useEffect(() => {
    const map = mapRef.current;
    if (!map) return;

    const handleMoveEnd = () => {
      onMoveEnd?.(map.getBounds());
    };

    const handleZoomEnd = () => {
      onZoomEnd?.(map.getZoom());
    };

    if (onMoveEnd) map.on('moveend', handleMoveEnd);
    if (onZoomEnd) map.on('zoomend', handleZoomEnd);

    return () => {
      if (onMoveEnd) map.off('moveend', handleMoveEnd);
      if (onZoomEnd) map.off('zoomend', handleZoomEnd);
    };
  }, [onMoveEnd, onZoomEnd]);

  return (
    <div 
      ref={containerRef} 
      style={{ width: '100%', height: '100%' }}
      data-testid="map-container"
    >
      {mapRef.current && children}
    </div>
  );
};
```

## MarkersLayer Component

```typescript
// components/MarkersLayer.tsx
import { useEffect, useMemo } from 'react';
import L from 'leaflet';
import type { MarkersLayerProps, GeoPoint } from '../types/geo';
import { isValidCoordinate } from '../utils/validation';

export const MarkersLayer: React.FC<MarkersLayerProps> = ({
  map,
  points,
  selectedId,
  onMarkerClick
}) => {
  // Create layer group once
  const markerGroup = useMemo(() => L.layerGroup(), []);
  
  // Store marker refs for selection highlighting
  const markersRef = useRef<Map<string, L.Marker>>(new Map());

  // Add layer group to map
  useEffect(() => {
    markerGroup.addTo(map);
    return () => {
      markerGroup.remove();
    };
  }, [map, markerGroup]);

  // Update markers when points change
  useEffect(() => {
    markerGroup.clearLayers();
    markersRef.current.clear();

    points.forEach(point => {
      // Validate coordinates
      if (!isValidCoordinate(point.lat, point.lng)) {
        console.warn(`Invalid coordinates for point ${point.id}:`, point);
        return;
      }

      const marker = L.marker([point.lat, point.lng], {
        icon: L.icon({
          iconUrl: '/marker-icon.png',
          iconSize: [25, 41],
          iconAnchor: [12, 41],
        }),
      });

      // Add click handler
      if (onMarkerClick) {
        marker.on('click', () => onMarkerClick(point));
      }

      // Add popup if properties exist
      if (point.properties) {
        const popupContent = Object.entries(point.properties)
          .map(([key, value]) => `<strong>${key}:</strong> ${value}`)
          .join('<br/>');
        marker.bindPopup(popupContent);
      }

      marker.addTo(markerGroup);
      markersRef.current.set(point.id, marker);
    });
  }, [points, onMarkerClick, markerGroup]);

  // Handle selection highlighting
  useEffect(() => {
    markersRef.current.forEach((marker, id) => {
      const icon = marker.getIcon() as L.Icon;
      if (id === selectedId) {
        // Highlight selected marker
        marker.setIcon(L.icon({
          ...icon.options,
          className: 'selected-marker',
        }));
      } else {
        // Reset non-selected markers
        marker.setIcon(L.icon({
          ...icon.options,
          className: '',
        }));
      }
    });
  }, [selectedId]);

  return null;
};
```

## MarkersLayer with Clustering

```typescript
// components/MarkersLayerClustered.tsx
import { useEffect, useMemo } from 'react';
import L from 'leaflet';
import 'leaflet.markercluster';
import type { MarkersLayerProps } from '../types/geo';
import { isValidCoordinate } from '../utils/validation';

export const MarkersLayerClustered: React.FC<MarkersLayerProps> = ({
  map,
  points,
  onMarkerClick
}) => {
  // Create cluster group once
  const clusterGroup = useMemo(() => 
    L.markerClusterGroup({
      maxClusterRadius: 50,
      spiderfyOnMaxZoom: true,
      showCoverageOnHover: false,
      zoomToBoundsOnClick: true,
    }), 
    []
  );

  // Add cluster group to map
  useEffect(() => {
    clusterGroup.addTo(map);
    return () => {
      clusterGroup.remove();
    };
  }, [map, clusterGroup]);

  // Update markers when points change
  useEffect(() => {
    clusterGroup.clearLayers();

    const markers = points
      .filter(point => isValidCoordinate(point.lat, point.lng))
      .map(point => {
        const marker = L.marker([point.lat, point.lng]);
        
        if (onMarkerClick) {
          marker.on('click', () => onMarkerClick(point));
        }

        return marker;
      });

    clusterGroup.addLayers(markers);
  }, [points, onMarkerClick, clusterGroup]);

  return null;
};
```

## PolylinesLayer Component

```typescript
// components/PolylinesLayer.tsx
import { useEffect, useMemo } from 'react';
import L from 'leaflet';
import type { PolylinesLayerProps } from '../types/geo';
import { isValidCoordinate } from '../utils/validation';

export const PolylinesLayer: React.FC<PolylinesLayerProps> = ({
  map,
  polylines,
  color = '#3388ff',
  weight = 3
}) => {
  const polylineGroup = useMemo(() => L.layerGroup(), []);

  useEffect(() => {
    polylineGroup.addTo(map);
    return () => {
      polylineGroup.remove();
    };
  }, [map, polylineGroup]);

  useEffect(() => {
    polylineGroup.clearLayers();

    polylines.forEach(polyline => {
      // Validate all points
      const validPoints = polyline.points.filter(([lat, lng]) => 
        isValidCoordinate(lat, lng)
      );

      if (validPoints.length < 2) {
        console.warn(`Polyline ${polyline.id} has insufficient valid points`);
        return;
      }

      const line = L.polyline(validPoints, {
        color,
        weight,
        opacity: 0.7,
      });

      line.addTo(polylineGroup);
    });
  }, [polylines, color, weight, polylineGroup]);

  return null;
};
```

## Validation Utilities

```typescript
// utils/validation.ts

export function isValidCoordinate(lat: number, lng: number): boolean {
  return (
    typeof lat === 'number' &&
    typeof lng === 'number' &&
    !isNaN(lat) &&
    !isNaN(lng) &&
    lat >= -90 &&
    lat <= 90 &&
    lng >= -180 &&
    lng <= 180
  );
}

export function validateGeoPoint(point: unknown): point is GeoPoint {
  if (!point || typeof point !== 'object') return false;
  
  const p = point as any;
  
  return (
    typeof p.id === 'string' &&
    isValidCoordinate(p.lat, p.lng)
  );
}

export function sanitizeGeoPoints(points: unknown[]): GeoPoint[] {
  return points.filter(validateGeoPoint);
}
```

## Complete Usage Example

```typescript
// App.tsx
import { useState, useCallback } from 'react';
import { MapView } from './components/MapView';
import { MarkersLayerClustered } from './components/MarkersLayerClustered';
import { PolylinesLayer } from './components/PolylinesLayer';
import type { GeoPoint } from './types/geo';

export default function App() {
  const [center, setCenter] = useState<[number, number]>([37.7749, -122.4194]);
  const [zoom, setZoom] = useState(12);
  const [selectedId, setSelectedId] = useState<string>();
  
  // Mock data
  const points: GeoPoint[] = [
    { id: '1', lat: 37.7749, lng: -122.4194, properties: { name: 'Point 1' } },
    { id: '2', lat: 37.7849, lng: -122.4094, properties: { name: 'Point 2' } },
    // ... more points
  ];

  const handleMarkerClick = useCallback((point: GeoPoint) => {
    setSelectedId(point.id);
    setCenter([point.lat, point.lng]);
  }, []);

  const handleMoveEnd = useCallback((bounds: L.LatLngBounds) => {
    console.log('Map moved to:', bounds);
  }, []);

  return (
    <div style={{ height: '100vh', display: 'flex' }}>
      <div style={{ flex: 1 }}>
        <MapView
          center={center}
          zoom={zoom}
          onMoveEnd={handleMoveEnd}
          onZoomEnd={setZoom}
        >
          <MarkersLayerClustered
            map={mapRef.current!}
            points={points}
            selectedId={selectedId}
            onMarkerClick={handleMarkerClick}
          />
        </MapView>
      </div>
      
      <div style={{ width: 300, padding: 16 }}>
        <h3>Selected Point</h3>
        {selectedId && (
          <div>
            {points.find(p => p.id === selectedId)?.properties?.name}
          </div>
        )}
      </div>
    </div>
  );
}
```

## Performance Optimization Patterns

### Memoizing Filtered Data

```typescript
const visiblePoints = useMemo(() => {
  return points.filter(point => {
    // Expensive filtering logic
    return meetsFilterCriteria(point, filters);
  });
}, [points, filters]);
```

### Debouncing Search

```typescript
import { useMemo } from 'react';
import debounce from 'lodash/debounce';

const debouncedSearch = useMemo(
  () => debounce((searchTerm: string) => {
    // Perform search
    performSearch(searchTerm);
  }, 300),
  []
);
```

### Throttling Map Events

```typescript
import throttle from 'lodash/throttle';

const handleMoveEnd = useMemo(
  () => throttle(() => {
    // Handle expensive operation
    updateVisibleData(map.getBounds());
  }, 500),
  []
);
```
