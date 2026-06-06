<script>
  import { onMount } from 'svelte';
  import blueprints from './data/blueprints.json';

  // State
  let searchQuery = '';
  let selectedBlueprintKey = null;
  let selectedTab = 'visual'; // 'visual' or 'json'
  let activeFilter = 'all';
  let showMobileDetail = false;

  const categories = ['all', 'Core', 'Marketplace', 'Real Estate', 'Directory', 'Fundraising', 'Booking', 'Content'];

  // Type badge styling helper
  function getFieldTypeBadge(type) {
    const t = type.toLowerCase();
    if (t === 'number') return 'badge-number';
    if (t === 'date') return 'badge-date';
    if (t === 'text') return 'badge-text';
    return 'badge-generic';
  }

  // Type label helper
  function getFieldTypeLabel(type) {
    const t = type.toLowerCase();
    if (t === 'number') return 'Číslo';
    if (t === 'date') return 'Dátum';
    if (t === 'text') return 'Text';
    return type;
  }

  $: filteredBlueprints = blueprints.filter(bp => {
    const matchesSearch = bp.name.toLowerCase().includes(searchQuery.toLowerCase()) ||
                          bp.description.toLowerCase().includes(searchQuery.toLowerCase()) ||
                          JSON.stringify(bp.data).toLowerCase().includes(searchQuery.toLowerCase());
    
    const matchesFilter = activeFilter === 'all' || bp.category === activeFilter;
    
    return matchesSearch && matchesFilter;
  });

  $: selectedBlueprint = blueprints.find(bp => bp.name === selectedBlueprintKey);

  onMount(() => {
    if (blueprints.length > 0) {
      selectedBlueprintKey = blueprints[0].name;
    }
  });

  function selectBlueprint(key) {
    selectedBlueprintKey = key;
    showMobileDetail = true;
  }
</script>

<main class="main-wrapper">
  <!-- Header -->
  <header class="navbar">
    <div class="navbar-container">
      <div class="logo-group">
        <div class="logo-box">
          <img src="/logo.svg" alt="JetEngine Logo" class="logo-svg" />
        </div>
        <div>
          <h1 class="brand-title">JetEngine Master System</h1>
          <p class="brand-subtitle">PWA Architecture Explorer</p>
        </div>
      </div>

      <div class="badge-group">
        <span class="status-badge offline-badge">
          <span class="status-dot"></span> Offline Ready
        </span>
        <span class="status-badge version-badge">
          PWA v2.0.0
        </span>
      </div>
    </div>
  </header>

  <!-- Content Grid -->
  <div class="content-grid {showMobileDetail ? 'mobile-show-detail' : 'mobile-show-sidebar'}">
    
    <!-- Sidebar -->
    <aside class="sidebar">
      
      <!-- Search Box -->
      <div class="search-panel">
        <h2 class="section-tag">Vyhľadávanie & filtre</h2>
        
        <div class="search-input-wrapper">
          <input 
            type="text" 
            placeholder="Hľadať blueprint, entitu, meta field..." 
            bind:value={searchQuery}
            class="search-input"
          />
          <span class="search-icon">🔍</span>
        </div>

        <div class="filter-chips">
          {#each categories as category}
            <button 
              on:click={() => activeFilter = category}
              class="filter-chip {activeFilter === category ? 'active' : ''}"
            >
              {category === 'all' ? 'Všetko' : category}
            </button>
          {/each}
        </div>
      </div>

      <!-- Blueprint list -->
      <div class="blueprint-list">
        {#each filteredBlueprints as bp}
          <button 
            on:click={() => selectBlueprint(bp.name)}
            class="blueprint-card {selectedBlueprintKey === bp.name ? 'active' : ''}"
          >
            <div class="card-inner">
              <div class="card-icon" style="background: {bp.gradient}">
                {bp.icon}
              </div>
              <div class="card-info">
                <div class="card-header-row">
                  <h3 class="card-title">{bp.name}</h3>
                  <span class="card-cat-badge">{bp.category}</span>
                </div>
                <p class="card-desc">{bp.description}</p>
                <div class="card-stats">
                  <span>📦 {bp.data.cpt?.length || 0} CPT</span>
                  <span>⚙️ {bp.data.cct?.length || 0} CCT</span>
                  {#if bp.data.relations?.length}
                    <span>🔗 {bp.data.relations.length} Relácií</span>
                  {/if}
                </div>
              </div>
            </div>
          </button>
        {:else}
          <div class="no-results">
            <p>Nenašli sa žiadne blueprinty</p>
          </div>
        {/each}
      </div>
    </aside>

    <!-- Detail Area -->
    <section id="blueprint-detail" class="detail-container">
      {#if selectedBlueprint}
        <!-- Back button for mobile view -->
        <button class="mobile-back-btn" on:click={() => showMobileDetail = false}>
          ← Späť na zoznam
        </button>

        <div class="detail-panel">
          
          <!-- Detail Header -->
          <div class="detail-header">
            <div class="detail-header-main">
              <div class="detail-icon" style="background: {selectedBlueprint.gradient}">
                {selectedBlueprint.icon}
              </div>
              <div>
                <h2 class="detail-title">{selectedBlueprint.name}</h2>
                <p class="detail-subtitle">{selectedBlueprint.description}</p>
              </div>
            </div>

            <!-- Tab selector -->
            <div class="tab-selector">
              <button 
                on:click={() => selectedTab = 'visual'}
                class="tab-btn {selectedTab === 'visual' ? 'active' : ''}"
              >
                Vizuálna Schéma
              </button>
              <button 
                on:click={() => selectedTab = 'json'}
                class="tab-btn {selectedTab === 'json' ? 'active' : ''}"
              >
                Raw JSON
              </button>
            </div>
          </div>

          <!-- Tab Content: Visual ER-Diagram -->
          {#if selectedTab === 'visual'}
            
            <!-- Settings panel (if present) -->
            {#if selectedBlueprint.data.settings && Object.keys(selectedBlueprint.data.settings).length > 0}
              <div class="info-block">
                <h4 class="block-title">Globálne nastavenia (Settings)</h4>
                <div class="settings-grid">
                  {#each Object.entries(selectedBlueprint.data.settings) as [key, val]}
                    <div class="setting-item">
                      <span class="setting-key">{key}</span>
                      <span class="setting-val">{val}</span>
                    </div>
                  {/each}
                </div>
              </div>
            {/if}

            <!-- UI home sections wireframe preview -->
            {#if selectedBlueprint.data.ui && selectedBlueprint.data.ui.home}
              <div class="wireframe-block">
                <h4 class="block-title">Home Page Wireframe (UI)</h4>
                <p class="wireframe-desc">Vykreslená logická štruktúra domovskej stránky:</p>
                <div class="wireframe-screen">
                  <div class="wireframe-header">
                    <div class="wireframe-dot"></div>
                    <div class="wireframe-dot"></div>
                    <div class="wireframe-dot"></div>
                  </div>
                  <div class="wireframe-body">
                    {#each selectedBlueprint.data.ui.home as section}
                      <div class="wireframe-section-card">
                        <span class="wireframe-section-icon">⚡</span>
                        <span class="wireframe-section-name">{section} section</span>
                      </div>
                    {/each}
                  </div>
                </div>
              </div>
            {/if}

            <!-- Entities Grid -->
            <div class="entities-block">
              <h4 class="block-title">Databázová Architektúra</h4>
              
              <div class="entities-grid">
                
                <!-- CPT Entities -->
                {#if selectedBlueprint.data.cpt && selectedBlueprint.data.cpt.length > 0}
                  {#each selectedBlueprint.data.cpt as cpt}
                    <div class="entity-card border-cpt">
                      <div class="entity-header">
                        <div class="entity-type-group">
                          <span class="entity-type-badge badge-cpt">CPT</span>
                          <span class="entity-name">{cpt.name}</span>
                        </div>
                      </div>

                      {#if cpt.fields && cpt.fields.length > 0}
                        <div class="fields-section">
                          <span class="fields-header">Custom Fields</span>
                          <div class="fields-grid-rows">
                            {#each cpt.fields as field}
                              <div class="field-row">
                                <span class="field-name-text">{field.name}</span>
                                <span class="field-type-badge {getFieldTypeBadge(field.type)}">
                                  {getFieldTypeLabel(field.type)}
                                </span>
                              </div>
                            {/each}
                          </div>
                        </div>
                      {:else}
                        <p class="no-fields-text">Žiadne custom meta polia.</p>
                      {/if}
                    </div>
                  {/each}
                {/if}

                <!-- CCT Entities -->
                {#if selectedBlueprint.data.cct && selectedBlueprint.data.cct.length > 0}
                  {#each selectedBlueprint.data.cct as cct}
                    <div class="entity-card border-cct">
                      <div class="entity-header">
                        <div class="entity-type-group">
                          <span class="entity-type-badge badge-cct">CCT</span>
                          <span class="entity-name">{cct.name}</span>
                        </div>
                      </div>

                      {#if cct.fields && cct.fields.length > 0}
                        <div class="fields-section">
                          <span class="fields-header">Custom Fields</span>
                          <div class="fields-grid-rows">
                            {#each cct.fields as field}
                              <div class="field-row">
                                <span class="field-name-text">{field.name}</span>
                                <span class="field-type-badge {getFieldTypeBadge(field.type)}">
                                  {getFieldTypeLabel(field.type)}
                                </span>
                              </div>
                            {/each}
                          </div>
                        </div>
                      {:else}
                        <p class="no-fields-text">Žiadne custom meta polia.</p>
                      {/if}
                    </div>
                  {/each}
                {/if}

                <!-- Taxonomies -->
                {#if selectedBlueprint.data.taxonomies && selectedBlueprint.data.taxonomies.length > 0}
                  {#each selectedBlueprint.data.taxonomies as tax}
                    <div class="entity-card border-tax">
                      <div class="entity-header">
                        <div class="entity-type-group">
                          <span class="entity-type-badge badge-tax">Taxonomy</span>
                          <span class="entity-name">{tax.name}</span>
                        </div>
                      </div>
                      <div class="fields-section">
                        <span class="fields-header">Prepojené s typmi (post_types)</span>
                        <div class="tax-linked-types">
                          {#each tax.post_types as pt}
                            <span class="tax-pt-tag">{pt}</span>
                          {/each}
                        </div>
                      </div>
                    </div>
                  {/each}
                {/if}

              </div>
            </div>

            <!-- Relations Map -->
            {#if selectedBlueprint.data.relations && selectedBlueprint.data.relations.length > 0}
              <div class="relations-block">
                <h4 class="block-title">Relations Layer (Vzťahy medzi entitami)</h4>
                <div class="relations-list-box">
                  {#each selectedBlueprint.data.relations as rel}
                    <div class="relation-row-visual">
                      <div class="relation-entity-node entity-cpt">{rel.from}</div>
                      <div class="relation-connector-line">
                        <span class="relation-connector-label">🔗 relation</span>
                      </div>
                      <div class="relation-entity-node entity-tax">{rel.to}</div>
                    </div>
                  {/each}
                </div>
              </div>
            {/if}

          <!-- Tab Content: Raw JSON -->
          {:else}
            <div class="raw-lines-container">
              <pre class="json-pre"><code>{JSON.stringify(selectedBlueprint.data, null, 2)}</code></pre>
            </div>
          {/if}

        </div>
      {:else}
        <div class="detail-empty">
          <p>Vyberte blueprint zo zoznamu</p>
        </div>
      {/if}
    </section>

  </div>
</main>
