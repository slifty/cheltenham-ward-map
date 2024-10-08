<template>
	<div id="map"></div>
</template>

<script lang="ts">
import 'leaflet/dist/leaflet.css'
import '@geoapify/leaflet-address-search-plugin/dist/L.Control.GeoapifyAddressSearch.min.css'
import type GeoJSON from 'geojson'
import { defineComponent } from 'vue'
import L from 'leaflet'
import '@geoapify/leaflet-address-search-plugin'
import polylabel from 'polylabel'
import { wardGeoJson } from '../data/wardGeoJson'

interface WardFeatureData {
	properties: {
		districtId: string,
	},
}

function isWardFeatureData(obj: unknown): obj is WardFeatureData {
	return ((obj as WardFeatureData).properties !== undefined
		&& (obj as WardFeatureData).properties.districtId !== undefined)
}

function addAttribution(map: L.Map) : void {
	L.tileLayer('https://api.maptiler.com/maps/streets/{z}/{x}/{y}.png?key=eglpIpyresS4hgKGBVvz', {
		tileSize: 512,
		zoomOffset: -1,
		minZoom: 1,
		attribution: '\u003ca href="https://www.maptiler.com/copyright/" target="_blank"\u003e\u0026copy; MapTiler\u003c/a\u003e \u003ca href="https://www.openstreetmap.org/copyright" target="_blank"\u003e\u0026copy; OpenStreetMap contributors\u003c/a\u003e',
		crossOrigin: true,
	}).addTo(map)
}

function addWardLabel(map: L.Map, wardFeatureData: GeoJSON.GeoJsonObject) {
	L.geoJSON(wardFeatureData, {
		onEachFeature: (feature: GeoJSON.Feature<GeoJSON.Polygon>) => {
			const centroid = polylabel(feature.geometry.coordinates)
			const marker = L.marker(
				[centroid[1], centroid[0]],
				{ opacity: 0.00 },
			)
			if (isWardFeatureData(feature)) {
				marker.bindTooltip(
					feature.properties.districtId,
					{
						permanent: true,
						direction: 'center',
						className: 'precinctLabel',
						offset: [-25, 25],
					},
				)
			}
			marker.addTo(map)
		},
	})
}

function addWardPolygon(map: L.Map, wardFeatureData: GeoJSON.GeoJsonObject) {
	const wardFeatureGeoJson = L.geoJSON(wardFeatureData)
	wardFeatureGeoJson.addTo(map)
}

function addAddressSearch(map: L.Map) {
	// eslint-disable-next-line @typescript-eslint/no-explicit-any
	const addressSearchControl = (L.control as any).addressSearch(
		'6db172c824474328b92769853b10c2a8', // there's no backend so... I guess we're just gonna publish this lol.
		{
			position: 'topright',
			resultCallback: (address: { lat: number, lon: number }) => {
				const newMarker = L.circleMarker(
					[
						address.lat,
						address.lon,
					],
					{
						radius: 5,
						color: 'red',
					},
				)
				newMarker.addTo(map)
			},
		},
	)
	map.addControl(addressSearchControl)
}

export default defineComponent({
	name: 'WardMap',
	props: {},
	methods: {
		setupLeafletMap(): void {
			const map = L.map('map').setView([0, 0], 1)
			const wardLeafletGeoJson = L.geoJSON(wardGeoJson)
			addAttribution(map)
			map.fitBounds(wardLeafletGeoJson.getBounds())
			wardGeoJson.features.forEach((wardFeatureData: GeoJSON.GeoJsonObject) => {
				addWardPolygon(map, wardFeatureData)
				addWardLabel(map, wardFeatureData)
			})
			addAddressSearch(map)
		},
	},
	mounted() : void {
		this.setupLeafletMap()
	},
})
</script>

<style scoped>
#map {
	height:100%;
	width: 100%;
}
</style>
<style>
#map div.precinctLabel {
	padding: 2px 5px;
	border-radius: 0px;
	font-weight: bold;
}
</style>
