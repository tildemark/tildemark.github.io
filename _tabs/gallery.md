---
layout: page
title: Gallery
permalink: /gallery/
icon: fas fa-camera-retro
order: 4
---

<style>
/* 1. Force all images to be the same size */
.gallery-img {
  width: 100%;
  height: 250px; /* Adjust height as you like */
  object-fit: cover; /* This prevents squashing/stretching */
  object-position: center;
}

/* 2. simple fade animation */
.fade-in {
  animation: fadeIn 0.5s;
}
@keyframes fadeIn {
  0% { opacity: 0; }
  100% { opacity: 1; }
}

/* 3. Button active state color */
.filter-btn.active {
  background-color: var(--btn-btn-primary-bg) !important;
  color: #fff !important;
  border-color: var(--btn-btn-primary-bg) !important;
}
</style>

<div class="filter-buttons mb-4 text-center">
  <button class="btn btn-outline-primary filter-btn active" data-filter="all">All</button>
  
  {% assign categories = site.data.gallery | map: 'category' | uniq %}
  {% for cat in categories %}
    <button class="btn btn-outline-primary filter-btn" data-filter="{{ cat | slugify }}">
      {{ cat }}
    </button>
  {% endfor %}
</div>

<div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4">
  {% for photo in site.data.gallery %}
  <div class="col gallery-item {{ photo.category | slugify }}">
    <div class="card h-100 border-0 shadow-sm">
      
      <a href="{{ photo.image }}" class="popup-trigger">
        <img src="{{ photo.image }}" class="card-img-top rounded-top gallery-img" alt="{{ photo.title }}">
      </a>
      
      <div class="card-body d-flex flex-column">
        <h5 class="card-title h6 fw-bold">{{ photo.title }}</h5>
        
        {% if photo.description %}
          <p class="card-text small text-muted mb-2">{{ photo.description }}</p>
        {% endif %}

        {% if photo.details %}
        <div class="bg-light p-2 rounded small mb-2 border mt-auto">
          <ul class="list-unstyled mb-0" style="font-size: 0.85rem;">
            {% if photo.details.imo %}
              <li><strong>IMO:</strong> <a href="https://www.marinetraffic.com/en/ais/details/ships/imo:{{ photo.details.imo }}" target="_blank">{{ photo.details.imo }}</a></li>
            {% endif %}
            {% if photo.details.reg %}
              <li><strong>Reg:</strong> <a href="https://www.flightradar24.com/data/aircraft/{{ photo.details.reg }}" target="_blank">{{ photo.details.reg }}</a></li>
            {% endif %}
            {% if photo.details.location %}
              <li><strong>Loc:</strong> {{ photo.details.location }}</li>
            {% endif %}
          </ul>
        </div>
        {% endif %}
        
        {% if photo.tags %}
        <div class="mt-2">
          {% for tag in photo.tags %}
            <span class="badge bg-secondary text-light fw-normal" style="font-size: 0.7rem;">{{ tag }}</span>
          {% endfor %}
        </div>
        {% endif %}

      </div>
    </div>
  </div>
  {% endfor %}
</div>

<script>
document.addEventListener("DOMContentLoaded", function() {
  const buttons = document.querySelectorAll('.filter-btn');
  const items = document.querySelectorAll('.gallery-item');

  buttons.forEach(button => {
    button.addEventListener('click', () => {
      // 1. Get the filter value from the clicked button
      const filterValue = button.getAttribute('data-filter');

      // 2. Remove 'active' class from all buttons, add to clicked
      buttons.forEach(btn => btn.classList.remove('active'));
      button.classList.add('active');

      // 3. Loop through gallery items
      items.forEach(item => {
        // If filter is 'all' OR item has the class matching the filter
        if (filterValue === 'all' || item.classList.contains(filterValue)) {
          item.classList.remove('d-none');
          item.classList.add('fade-in');
        } else {
          item.classList.add('d-none');
          item.classList.remove('fade-in');
        }
      });
    });
  });
});
</script>
