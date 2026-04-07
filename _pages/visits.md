---
layout: page
permalink: /visits/
title: Visits
description: Research is a journey through remarkable places. This map traces where my research has taken me.
nav: true
nav_order: 6
---

<link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

<style>
  #travel-map {
    height: 600px;
    width: 100%;
    border-radius: 12px;
    margin-top: 1rem;
  }

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
    scrollWheelZoom: false
  }).setView([20, 0], 2);

  L.tileLayer("https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png", {
    attribution: "&copy; OpenStreetMap contributors"
  }).addTo(map);

  const bounds = [];

  function getColor(category) {
    switch (category) {
      case "conference": return "blue";
      case "work": return "green";
      case "visit": return "orange";
      default: return "gray";
    }
  }

  // Keep track of how many times each exact coordinate pair appears.
  const coordCounts = {};
  const coordSeen = {};

  places.forEach((p) => {
    const key = `${p.lat},${p.lng}`;
    coordCounts[key] = (coordCounts[key] || 0) + 1;
  });

  function getJitteredLatLng(lat, lng, index, total) {
    if (total === 1) {
      return [lat, lng];
    }

    // Spread repeated visits in a small circle around the true location.
    const radius = 0.10; // degrees; small but visible at city scale
    const angle = (2 * Math.PI * index) / total;

    // Adjust longitude offset by latitude so the visual spacing is more balanced.
    const latOffset = radius * Math.sin(angle);
    const lngOffset = (radius * Math.cos(angle)) / Math.cos(lat * Math.PI / 180);

    return [lat + latOffset, lng + lngOffset];
  }

  places.forEach((p) => {
    const key = `${p.lat},${p.lng}`;
    const total = coordCounts[key];
    const index = coordSeen[key] || 0;
    coordSeen[key] = index + 1;

    const [plotLat, plotLng] = getJitteredLatLng(p.lat, p.lng, index, total);

    const marker = L.circleMarker([plotLat, plotLng], {
      radius: 6,
      color: getColor(p.category),
      fillColor: getColor(p.category),
      fillOpacity: 0.8,
      weight: 1
    }).addTo(map);

    marker.bindPopup(`
      <div class="popup-title">${p.title}</div>
      <div>${p.location}</div>
      <div class="popup-date">${p.date}</div>
      <div>${p.note}</div>
    `);

    bounds.push([plotLat, plotLng]);
  });

  if (bounds.length > 0) {
    map.fitBounds(bounds, { padding: [40, 40] });
  }
});
</script>