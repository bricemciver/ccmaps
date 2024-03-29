<template>
  <div id="map"></div>
</template>

<script setup lang="ts">
import 'leaflet/dist/leaflet.css'
import markerIcon from 'leaflet/dist/images/marker-icon.png'
import markerIcon2x from 'leaflet/dist/images/marker-icon-2x.png'
import markerShadow from 'leaflet/dist/images/marker-shadow.png'
import placeData from '../assets/photos.json'

import { onMounted } from 'vue'
import { map, tileLayer, latLng, marker, icon, geoJSON, latLngBounds } from 'leaflet'

import type { GeoJsonObject, Feature, Geometry, Point } from 'geojson'
import type { LocationEvent, ErrorEvent, Layer } from 'leaflet'

onMounted(() => {
  // Code to be executed when the component is mounted
  const leafletMap = map('map').fitWorld()

  const onEachFeature = (feature: Feature<Geometry, any>, layer: Layer) => {
    const popupContent = `<div>
      <img src="${feature.properties.url}" alt="${feature.properties.subject}" width="100%">
      <h3>${feature.properties.subject}</h3>
      <p>${feature.properties.description}</p>
    </div>`

    layer.bindPopup(popupContent)
  }

  const tiles = tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
    maxZoom: 19,
    attribution: '© OpenStreetMap'
  })

  const theIcon = icon({
    iconUrl: markerIcon,
    iconRetinaUrl: markerIcon2x,
    shadowUrl: markerShadow,
    iconSize: [25, 41],
    iconAnchor: [12, 41],
    popupAnchor: [1, -34],
    tooltipAnchor: [16, -28],
    shadowSize: [41, 41]
  })

  const places = geoJSON(placeData as GeoJsonObject, {
    onEachFeature: onEachFeature,
    pointToLayer: (feature, latlng) => marker(latlng, { icon: theIcon }),
    filter: (feature) =>
      (feature.geometry as Point).coordinates[0] !== 0 &&
      (feature.geometry as Point).coordinates[1] !== 0
  })

  const onLocationError = (e: ErrorEvent) => {
    alert(e.message)
  }

  const onLocationFound = (e: LocationEvent) => {
    // find three closest layers in placeData
    const sortedPlaces = placeData.features.sort((a, b) => {
      const aCoords = a.geometry.coordinates
      const bCoords = b.geometry.coordinates
      const aLatLng = latLng(aCoords[1], aCoords[0])
      const bLatLng = latLng(bCoords[1], bCoords[0])
      return e.latlng.distanceTo(aLatLng) - e.latlng.distanceTo(bLatLng)
    })
    // create bounds including first three points
    let bounds = latLngBounds([e.latlng])
    for (let idx = 0; idx < 3; idx++) {
      const tmpLatLng = latLng(
        sortedPlaces[idx].geometry.coordinates[1],
        sortedPlaces[idx].geometry.coordinates[0]
      )
      bounds = bounds.extend(tmpLatLng)
    }
    // fit map to bounds
    leafletMap.fitBounds(bounds)
  }

  leafletMap.addLayer(tiles)
  leafletMap.addLayer(places)

  leafletMap.on('locationerror', onLocationError)
  leafletMap.on('locationfound', onLocationFound)

  leafletMap.locate({ setView: true })
})
</script>

<style scoped>
#map {
  height: 100%;
  width: 100vw;
}
</style>
