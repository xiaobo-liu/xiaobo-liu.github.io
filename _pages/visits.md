---
layout: page
permalink: /visits/
title: Visits
description: Map of research visits.
nav: true
nav_order: 6
---

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.css" />
<link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster@1.5.3/dist/MarkerCluster.Default.css" />

<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
<script src="https://unpkg.com/leaflet.markercluster@1.5.3/dist/leaflet.markercluster.js"></script>

<style>
  #travel-map {
    height: 600px;
    width: 100%;
    border-radius: 12px;
    margin-top: 1rem;
  }

  /* popup styling (matches al-folio typography better) */
  .leaflet-popup-content {
    font-size: 0.95rem;
    line-height: 1.4;
  }

  .popup-title {
    font-weight: 600;
    margin-bottom: 4px;
  }

  .popup-date {
    color: #888;
    font-size: 0.85rem;
  }
</style>

<div id="travel-map"></div>

<script>
document.addEventListener("DOMContentLoaded", function () {

  const places = [
    {% for p in site.data.travel_locations %}
    {
      title: {{ p.title | jsonify }},
      location: {{ p.location | jsonify }},
      lat: {{ p.lat }},
      lng: {{ p.lng }},
      date: {{ p.date | jsonify }},
      note: {{ p.note | default: "" | jsonify }},
      category: {{ p.category | default: "other" | jsonify }}
    }{% unless forloop.last %},{% endunless %}
    {% endfor %}
  ];

  const map = L.map("travel-map", {
    scrollWheelZoom: false   // nicer UX inside a page
  }).setView([20, 0], 2);

  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution: "&copy; OpenStreetMap contributors"
  }).addTo(map);

  const markers = L.markerClusterGroup();
  const bounds = [];

  function getColor(category) {
    switch (category) {
      case "conference": return "blue";
      case "work": return "green";
      case "travel": return "orange";
      default: return "gray";
    }
  }

  places.forEach((p) => {

    const marker = L.circleMarker([p.lat, p.lng], {
      radius: 6,
      color: getColor(p.category),
      fillOpacity: 0.8
    });

    marker.bindPopup(`
      <div class="popup-title">${p.title}</div>
      <div>${p.location}</div>
      <div class="popup-date">${p.date}</div>
      <div>${p.note}</div>
    `);

    markers.addLayer(marker);
    bounds.push([p.lat, p.lng]);
  });

  map.addLayer(markers);

  if (bounds.length > 0) {
    map.fitBounds(bounds, { padding: [40, 40] });
  }
});
</script>