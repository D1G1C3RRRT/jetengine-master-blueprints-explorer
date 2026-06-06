<script>
  import { onMount } from 'svelte';
  import rawBlueprints from './data/blueprints.json';

  // State
  let searchQuery = '';
  let selectedBlueprintKey = null;
  let selectedTab = 'visual'; // 'visual' or 'raw'
  let activeFilter = 'all';

  // Map files to user-friendly metadata
  const blueprintMeta = {
    'blueprints.pages': {
      title: 'JetEngine Master Template System',
      description: 'Kompletný znovupoužiteľný meta systém pre 20 produktových demo webov.',
      category: 'Core',
      gradient: 'linear-gradient(135deg, #3b82f6, #6366f1)',
      icon: '🧠'
    },
    'universalny back.pages': {
      title: 'Super-Univerzálny Backend',
      description: 'Základná databázová architektúra (8 jadrových entít) pre akýkoľvek dynamic web.',
      category: 'Core',
      gradient: 'linear-gradient(135deg, #8b5cf6, #d946ef)',
      icon: '⚡'
    },
    '20webblueprints AI.pages': {
      title: '20 Webových Blueprintov',
      description: 'Sada 20 webových konceptov od trhovísk po zložité rezervačné systémy.',
      category: 'Core',
      gradient: 'linear-gradient(135deg, #ec4899, #f43f5e)',
      icon: '📐'
    },
    'bigmarketplace.pages': {
      title: 'Veľké Digitálne Trhovisko',
      description: 'Kompletná logická štruktúra pre B2C + B2B trhovisko, retail a veľkoobchod.',
      category: 'Marketplace',
      gradient: 'linear-gradient(135deg, #06b6d4, #3b82f6)',
      icon: '🛒'
    },
    'marketplace.pages': {
      title: 'Digitálne Trhovisko (Creator / Retail)',
      description: 'Predaj digitálnych produktov, služieb, stánkov a správa objednávok.',
      category: 'Marketplace',
      gradient: 'linear-gradient(135deg, #14b8a6, #10b981)',
      icon: '🛍️'
    },
    'realestate.pages': {
      title: 'Realitný Web & Kancelária',
      description: 'Architektúra pre maklérov, ponuky nehnuteľností, predaje, prenájmy a projekty.',
      category: 'Real Estate',
      gradient: 'linear-gradient(135deg, #f59e0b, #ea580c)',
      icon: '🏠'
    },
    'travels.pages': {
      title: 'Travel & Ubytovanie',
      description: 'Dovolenky, destinácie, zájazdy, ubytovanie a kapacity.',
      category: 'Directory',
      gradient: 'linear-gradient(135deg, #0ea5e9, #6366f1)',
      icon: '✈️'
    },
    'startupy.pages': {
      title: 'Startup & Crowdfunding',
      description: 'Minimalistická schéma pre startup platformy, investovanie a kampane.',
      category: 'Fundraising',
      gradient: 'linear-gradient(135deg, #10b981, #06b6d4)',
      icon: '🚀'
    },
    'raising.pages': {
      title: 'Digital Fundraising & Kampane',
      description: 'Architektúra MVP pre enterprise organizácie, charity-only aj startup fundraising.',
      category: 'Fundraising',
      gradient: 'linear-gradient(135deg, #a855f7, #ec4899)',
      icon: '💖'
    },
    'online magazine.pages': {
      title: 'Online Magazín & Journal',
      description: 'Univerzálny model pre spravodajstvo, portál, blogy, autorov a články.',
      category: 'Content',
      gradient: 'linear-gradient(135deg, #2563eb, #06b6d4)',
      icon: '📰'
    },
    'Booking-Shop-Structure.pages': {
      title: 'Booking Shop & Rezervácie',
      description: 'Všeobecný model pre rezervácie služieb, produktov a termínov.',
      category: 'Booking',
      gradient: 'linear-gradient(135deg, #f97316, #f43f5e)',
      icon: '📅'
    },
    'Buseness-Web-Structure.pages': {
      title: 'Business & Corporate Web',
      description: 'Jednoduchá firemná prezentácia: služby, referencie, blog, kontakt.',
      category: 'Content',
      gradient: 'linear-gradient(135deg, #6366f1, #a855f7)',
      icon: '🏢'
    },
    'ostatnetypyjobevent.pages': {
      title: 'Case Studies & Moduly',
      description: 'Doplnkové moduly ako B2B referencie, Lead manažment a Job Events.',
      category: 'Core',
      gradient: 'linear-gradient(135deg, #d946ef, #f43f5e)',
      icon: '🧱'
    },
    'typy-webov.pages': {
      title: 'Klasifikácia Typov Webov',
      description: 'Prehľad a štruktúra základných typov webov na webe.',
      category: 'Directory',
      gradient: 'linear-gradient(135deg, #059669, #10b981)',
      icon: '📊'
    },
    'dta.pages': {
      title: 'SaaS Podnikateľské Modely',
      description: 'Príklady architektúr ako Notion, Shopify, Canva.',
      category: 'Core',
      gradient: 'linear-gradient(135deg, #475569, #1e293b)',
      icon: '💡'
    }
  };

  const categories = ['all', 'Core', 'Marketplace', 'Real Estate', 'Directory', 'Fundraising', 'Booking', 'Content'];

  // Identify field type based on keywords
  function getFieldType(fieldName) {
    const fn = fieldName.toLowerCase();
    if (fn.includes('image') || fn.includes('galer') || fn.includes('logo') || fn.includes('screenshot') || fn.includes('img') || fn.includes('foto')) return 'Obrázok / Galéria';
    if (fn.includes('price') || fn.includes('cena') || fn.includes('budget') || fn.includes('amount') || fn.includes('raised') || fn.includes('target') || fn.includes('finance')) return 'Číslo / Menový formát';
    if (fn.includes('date') || fn.includes('datum') || fn.includes('duration') || fn.includes('trvanie')) return 'Dátum / Čas';
    if (fn.includes('id') || fn.includes('slug') || fn.includes('cct') || fn.includes('cpt')) return 'Systémový kľúč / ID';
    if (fn.includes('status') || fn.includes('stav') || fn.includes('type') || fn.includes('typ')) return 'Výber (Select / Enum)';
    if (fn.includes('email') || fn.includes('phone') || fn.includes('tel') || fn.includes('kontakt') || fn.includes('mess')) return 'Kontaktné info';
    return 'Textové pole';
  }

  // Improved strict parser for cleaner extraction and better UI model representation
  function parseBlueprint(lines) {
    let pages = [];
    let cpts = [];
    let currentCpt = null;
    let currentGroup = 'Všeobecné';
    let mode = 'general';
    let generalNotes = [];

    for (let line of lines) {
      const trimmed = line.trim();
      if (!trimmed) continue;

      const lower = trimmed.toLowerCase();
      if (lower === 'stránky' || lower === 'pages' || lower === 'stranky') {
        mode = 'pages';
        continue;
      }
      if (trimmed.startsWith('CPT:') || trimmed.startsWith('CCT:')) {
        mode = 'cpt';
        const parts = trimmed.split(':');
        currentCpt = {
          type: parts[0].trim(),
          name: parts[1] ? parts[1].trim() : '',
          slug: '',
          fields: [],
          relations: []
        };
        cpts.push(currentCpt);
        currentGroup = 'Základné polia';
        continue;
      }
      if (lower.startsWith('slug:')) {
        if (currentCpt) {
          currentCpt.slug = trimmed.substring(5).trim();
        }
        continue;
      }
      if (lower === 'meta' || lower === 'meta fields' || lower === 'fields') {
        mode = 'meta';
        continue;
      }
      if (lower === 'vzťahy' || lower === 'relations' || lower === 'relácie' || lower === 'väzby') {
        mode = 'relations';
        continue;
      }

      const isListItem = trimmed.startsWith('-') || trimmed.startsWith('*') || /^\d+\./.test(trimmed);
      const cleanLine = trimmed.replace(/^[-*\d\.]\s*/, '');

      if (mode === 'pages') {
        if (trimmed.includes('.') && (trimmed.endsWith('.png') || trimmed.endsWith('.jpg') || trimmed.endsWith('.pdf'))) {
          continue;
        }
        pages.push(cleanLine);
      } else if (mode === 'meta') {
        if (currentCpt) {
          if (trimmed.length < 25 && /^[A-ZÁÄČĎÉÍĹĽŇÓÔŔŠŤÚÝŽ]/.test(trimmed) && !trimmed.includes(':') && isNaN(trimmed[0])) {
            currentGroup = trimmed;
          } else {
            let fieldName = trimmed;
            let typeLabel = getFieldType(trimmed);
            
            if (trimmed.includes(':')) {
              const parts = trimmed.split(':');
              fieldName = parts[0].trim();
              typeLabel = parts[1].trim();
            }
            
            currentCpt.fields.push({
              group: currentGroup,
              name: fieldName,
              type: typeLabel
            });
          }
        } else {
          generalNotes.push({ text: cleanLine, isListItem });
        }
      } else if (mode === 'relations') {
        if (currentCpt) {
          currentCpt.relations.push(cleanLine);
        } else {
          generalNotes.push({ text: cleanLine, isListItem });
        }
      } else {
        generalNotes.push({ text: cleanLine, isListItem });
      }
    }

    generalNotes = generalNotes.filter(n => !n.text.includes('.png') && !n.text.includes('.jpg') && !n.text.includes('.pdf') && n.text.length > 2);

    return { pages, cpts, generalNotes };
  }

  // Reactive listings
  $: blueprints = Object.entries(rawBlueprints).map(([key, lines]) => {
    const meta = blueprintMeta[key] || {
      title: key.replace('.pages', ''),
      description: 'Základná štruktúra webovej aplikácie.',
      category: 'General',
      gradient: 'linear-gradient(135deg, #475569, #334155)',
      icon: '⚙️'
    };
    
    return {
      key,
      ...meta,
      parsed: parseBlueprint(lines),
      rawLines: lines
    };
  });

  $: filteredBlueprints = blueprints.filter(bp => {
    const matchesSearch = bp.title.toLowerCase().includes(searchQuery.toLowerCase()) ||
                          bp.description.toLowerCase().includes(searchQuery.toLowerCase()) ||
                          bp.rawLines.some(line => line.toLowerCase().includes(searchQuery.toLowerCase()));
    
    const matchesFilter = activeFilter === 'all' || bp.category === activeFilter;
    
    return matchesSearch && matchesFilter;
  });

  $: selectedBlueprint = blueprints.find(bp => bp.key === selectedBlueprintKey);

  onMount(() => {
    if (blueprints.length > 0) {
      selectedBlueprintKey = blueprints[0].key;
    }
  });

  function selectBlueprint(key) {
    selectedBlueprintKey = key;
    if (window.innerWidth < 1024) {
      setTimeout(() => {
        document.getElementById('blueprint-detail')?.scrollIntoView({ behavior: 'smooth' });
      }, 100);
    }
  }

  // Helper to group fields by their group names
  function getGroupedFields(fields) {
    const groups = {};
    fields.forEach(f => {
      if (!groups[f.group]) groups[f.group] = [];
      groups[f.group].push(f);
    });
    return Object.entries(groups);
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
          PWA v1.2.0
        </span>
      </div>
    </div>
  </header>

  <!-- Content Grid -->
  <div class="content-grid">
    
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
            on:click={() => selectBlueprint(bp.key)}
            class="blueprint-card {selectedBlueprintKey === bp.key ? 'active' : ''}"
          >
            <div class="card-inner">
              <div class="card-icon" style="background: {bp.gradient}">
                {bp.icon}
              </div>
              <div class="card-info">
                <div class="card-header-row">
                  <h3 class="card-title">{bp.title}</h3>
                  <span class="card-cat-badge">{bp.category}</span>
                </div>
                <p class="card-desc">{bp.description}</p>
                <div class="card-stats">
                  <span>📂 {bp.parsed.pages.length} Stránok</span>
                  <span>📦 {bp.parsed.cpts.length} Entít</span>
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
        <div class="detail-panel">
          
          <!-- Detail Header -->
          <div class="detail-header">
            <div class="detail-header-main">
              <div class="detail-icon" style="background: {selectedBlueprint.gradient}">
                {selectedBlueprint.icon}
              </div>
              <div>
                <h2 class="detail-title">{selectedBlueprint.title}</h2>
                <p class="detail-subtitle">{selectedBlueprint.key}</p>
              </div>
            </div>

            <!-- Tab selector -->
            <div class="tab-selector">
              <button 
                on:click={() => selectedTab = 'visual'}
                class="tab-btn {selectedTab === 'visual' ? 'active' : ''}"
              >
                Vizuálna schémá
              </button>
              <button 
                on:click={() => selectedTab = 'raw'}
                class="tab-btn {selectedTab === 'raw' ? 'active' : ''}"
              >
                Raw Riadky
              </button>
            </div>
          </div>

          <!-- Tab Content: Visual -->
          {#if selectedTab === 'visual'}
            <!-- Goals / Spec -->
            {#if selectedBlueprint.parsed.generalNotes.length > 0}
              <div class="info-block">
                <h4 class="block-title">Ciele a špecifikácia</h4>
                <div class="info-content">
                  {#each selectedBlueprint.parsed.generalNotes as note}
                    {#if note.isListItem}
                      <div class="bullet-item">
                        <span class="bullet-dot">•</span>
                        <p>{note.text}</p>
                      </div>
                    {:else}
                      <p class="paragraph-text">{note.text}</p>
                    {/if}
                  {/each}
                </div>
              </div>
            {/if}

            <!-- Pages Structure -->
            {#if selectedBlueprint.parsed.pages.length > 0}
              <div class="pages-block">
                <h4 class="block-title">Navrhovaná štruktúra stránok</h4>
                <div class="pages-tags">
                  {#each selectedBlueprint.parsed.pages as page}
                    <div class="page-tag">
                      <span class="page-tag-icon">📂</span> {page}
                    </div>
                  {/each}
                </div>
              </div>
            {/if}

            <!-- Entities & Fields -->
            {#if selectedBlueprint.parsed.cpts.length > 0}
              <div class="entities-block">
                <h4 class="block-title">Databázový Model & Entity</h4>
                
                <div class="entities-grid">
                  {#each selectedBlueprint.parsed.cpts as cpt}
                    <div class="entity-card">
                      <!-- Entity Header -->
                      <div class="entity-header">
                        <div class="entity-type-group">
                          <span class="entity-type-badge {cpt.type.toLowerCase() === 'cpt' ? 'badge-cpt' : 'badge-cct'}">{cpt.type}</span>
                          <span class="entity-name">{cpt.name}</span>
                        </div>
                        {#if cpt.slug}
                          <span class="entity-slug">slug: <code>{cpt.slug}</code></span>
                        {/if}
                      </div>

                      <!-- Grouped Custom Fields -->
                      {#if cpt.fields.length > 0}
                        <div class="fields-section">
                          {#each getGroupedFields(cpt.fields) as [groupName, groupFields]}
                            <div class="field-group-container">
                              <span class="field-group-title">{groupName}</span>
                              <div class="fields-grid-rows">
                                {#each groupFields as field}
                                  <div class="field-row">
                                    <span class="field-name-text">{field.name}</span>
                                    <span class="field-type-badge">{field.type}</span>
                                  </div>
                                {/each}
                              </div>
                            </div>
                          {/each}
                        </div>
                      {/if}

                      <!-- Relations -->
                      {#if cpt.relations.length > 0}
                        <div class="entity-relations">
                          <span class="fields-header">Väzby a prepojenia</span>
                          <div class="relations-list">
                            {#each cpt.relations as rel}
                              <div class="relation-item">
                                <span class="relation-icon">🔗</span> {rel}
                              </div>
                            {/each}
                          </div>
                        </div>
                      {/if}
                    </div>
                  {/each}
                </div>
              </div>
            {/if}

          <!-- Tab Content: Raw Lines -->
          {:else}
            <div class="raw-lines-container">
              {#each selectedBlueprint.rawLines as line}
                <div class="raw-line">{line}</div>
              {/each}
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
