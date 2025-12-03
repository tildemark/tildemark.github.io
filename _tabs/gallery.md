---
layout: page
title: Gallery
permalink: /gallery/
icon: fas fa-camera-retro
order: 4
---

<div class="filter-buttons mb-4 text-center">
  <button class="btn btn-outline-primary active" onclick="filterGallery('all')">All</button>
  {% assign categories = site.data.gallery | map: 'category' | uniq %}
  {% for cat in categories %}
    <button class="btn btn-outline-primary" onclick="filterGallery('{{ cat | slugify }}')">
      {{ cat }}
    </button>
  {% endfor %}
</div>

<div class="row row-cols-1 row-cols-md-2 row-cols-lg-3 g-4" id="gallery-container">
  {% for photo in site.data.gallery %}
  <div class="col gallery-item {{ photo.category | slugify }}">
    <div class="card h-100 border-0 shadow-sm">
      
      <a href="{{ photo.image }}" class="popup-trigger">
        <img src="{{ photo.image }}" class="card-img-top rounded-top" alt="{{ photo.title }}" style="object-fit: cover; height: 250px;">
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
              <li><strong>IMO:</strong> <a href="https://www.marinetraffic.com/en/ais/details/ships/imo:{{ photo.details.imo }}" target="_blank" class="text-decoration-none">{{ photo.details.imo }} <i class="fas fa-external-link-alt fa-xs"></i></a></li>
            {% endif %}
            
            {% if photo.details.flag %}
              <li><strong>Flag:</strong> {{ photo.details.flag }}</li>
            {% endif %}

            {% if photo.details.reg %}
              <li><strong>Reg:</strong> <a href="https://www.flightradar24.com/data/aircraft/{{ photo.details.reg }}" target="_blank" class="text-decoration-none">{{ photo.details.reg }} <i class="fas fa-external-link-alt fa-xs"></i></a></li>
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
function filterGallery(category) {
  const items = document.getElementsByClassName('gallery-item');
  const buttons = document.querySelectorAll('.filter-buttons button');

  // Manage Active Button Styling
  buttons.forEach(btn => {
    btn.classList.remove('active');
    if (btn.innerText.toLowerCase() === category || (category === 'all' && btn.innerText === 'All')) {
      btn.classList.add('active');
    }
  });

  // Show/Hide Images
  for (let i = 0; i < items.length; i++) {
    const item = items[i];
    if (category === 'all') {
      item.classList.remove('d-none'); 
      item.classList.add('animate__animated', 'animate__fadeIn'); 
    } else {
      if (item.classList.contains(category)) {
        item.classList.remove('d-none');
        item.classList.add('animate__animated', 'animate__fadeIn');
      } else {
        item.classList.add('d-none');
      }
    }
  }
}
</script>

<style>
.filter-buttons .btn.active {
  background-color: var(--btn-btn-primary-bg);
  color: #fff;
}
</style>
