
<script setup>
import { ref, computed, watch } from 'vue'

// Mining party locations and default scoring rules
const presets = {
  bahari: {
    name: 'Bahari Bay',
    description: 'Iron, gold, and stone treasures',
    plushName: 'Obsidian Kitsuu Plush',
    plushPoints: 10,
    gems: [
      ['Citrine', 1],
      ['Garnet', 2],
      ['Molten Ore', 3],
      ['Aquamarine', 4],
      ['Jasper', 5],
      ['Sapphire', 6]
    ]
  },
  elderwood: {
    name: 'Elderwood',
    description: 'Whitestone, palium, and platinum treasures',
    plushName: 'Rainbow Kitsuu Plush',
    plushPoints: 20,
    gems: [
      ['Citrine', 1],
      ['Jasper', 3],
      ['Amethyst', 5],
      ['Ruby', 10],
      ['Molten Ore', 12]
    ]
  },
  highlands: {
    name: 'Royal Highlands',
    description: 'Whitestone, iron, gold, and jadium treasures',
    plushName: 'Jadium Kitsuu Plush',
    plushPoints: 15,
    gems: [
      ['Citrine', 1],
      ['Garnet', 2],
      ['Molten Ore', 3],
      ['Aquamarine', 4],
      ['Jasper', 5],
      ['Sapphire', 6],
      ['Sunstone', 7]
    ]
  }
}

// Keep saved data between visits
const storageKey = 'palia-mining-calculator-v2'

function makeDefaultData() {
  const result = {}

  for (const [key, preset] of Object.entries(presets)) {
    result[key] = {
      gems: preset.gems.map(([name, points], index) => ({
        id: index + 1,
        name,
        points,
        normal: 0,
        starred: 0
      })),
      plushName: preset.plushName,
      plushPoints: preset.plushPoints,
      plushCount: 0,
      scores: []
    }
  }

  return result
}

function loadData() {
  const defaults = makeDefaultData()

  try {
    const saved = JSON.parse(localStorage.getItem(storageKey))

    if (saved && typeof saved === 'object') {
      for (const key of Object.keys(defaults)) {
        if (saved[key] && Array.isArray(saved[key].gems)) {
          defaults[key] = {
            ...defaults[key],
            ...saved[key],
            scores: Array.isArray(saved[key].scores)
              ? saved[key].scores
              : []
          }
        }
      }
    }
  } catch {
    // Use defaults if there is no valid saved data
  }

  return defaults
}

const location = ref('bahari')
const data = ref(loadData())

const playerName = ref('')
const teamName = ref('')

const current = computed(() => data.value[location.value])
const currentPreset = computed(() => presets[location.value])

watch(data, value => {
  localStorage.setItem(storageKey, JSON.stringify(value))
}, { deep: true })

function safeNumber(value) {
  return Math.max(0, Number(value) || 0)
}

// Calculate normal and star-quality gems
const total = computed(() => {
  const gemTotal = current.value.gems.reduce((sum, gem) => {
    const points = safeNumber(gem.points)
    const normal = safeNumber(gem.normal)
    const starred = safeNumber(gem.starred)

    return sum + normal * points + starred * points * 2
  }, 0)

  const bonus = safeNumber(current.value.plushCount) *
    safeNumber(current.value.plushPoints)

  return gemTotal + bonus
})

// Add player scores together by team
const teamTotals = computed(() => {
  const teams = {}

  current.value.scores.forEach(player => {
    if (!teams[player.team]) {
      teams[player.team] = 0
    }

    teams[player.team] += player.total
  })

  return Object.entries(teams)
    .map(([name, points]) => ({ name, points }))
    .sort((a, b) => b.points - a.points)
})

// Sort individual players by highest score
const sortedPlayers = computed(() => {
  return [...current.value.scores]
    .sort((a, b) => b.total - a.total)
})

// Clear quantities without deleting saved scores
function resetCounts() {
  current.value.gems.forEach(gem => {
    gem.normal = 0
    gem.starred = 0
  })

  current.value.plushCount = 0
}

// Save player score to current location
function saveScore() {
  const name = playerName.value.trim()

  if (!name) {
    alert('Please enter a player name!')
    return
  }

  current.value.scores.push({
    id: `${Date.now()}-${Math.random()}`,
    name,
    team: teamName.value.trim() || 'Individual',
    total: total.value
  })

  playerName.value = ''
  resetCounts()
}

// Remove an individual player score
function removeScore(id) {
  current.value.scores = current.value.scores
    .filter(score => score.id !== id)
}

// Clear the current location's leaderboard
function clearScores() {
  if (confirm(`Clear all ${currentPreset.value.name} scores?`)) {
    current.value.scores = []
  }
}

// Restore default gem scoring for a location
function resetLocationRules() {
  if (!confirm('Reset this location to its default rules?')) {
    return
  }

  const preset = currentPreset.value

  current.value.gems = preset.gems.map(([name, points], index) => ({
    id: index + 1,
    name,
    points,
    normal: 0,
    starred: 0
  }))

  current.value.plushName = preset.plushName
  current.value.plushPoints = preset.plushPoints
  current.value.plushCount = 0
}

// Add a custom gem or mining item
function addGem() {
  const name = prompt('Enter the gem or item name:')

  if (!name?.trim()) return

  current.value.gems.push({
    id: `${Date.now()}-${Math.random()}`,
    name: name.trim(),
    points: 0,
    normal: 0,
    starred: 0
  })
}

// Remove a gem from the current location
function removeGem(id) {
  current.value.gems = current.value.gems
    .filter(gem => gem.id !== id)
}

// Download the saved scores
function exportScores() {
  if (!current.value.scores.length) {
    alert('There are no scores to export yet.')
    return
  }

  const rows = [
    ['Player', 'Team', 'Points'],
    ...current.value.scores.map(player => [
      player.name,
      player.team,
      player.total
    ])
  ]

  const csv = rows.map(row =>
    row.map(value =>
      `"${String(value).replace(/"/g, '""')}"`
    ).join(',')
  ).join('\r\n')

  const blob = new Blob([csv], {
    type: 'text/csv;charset=utf-8'
  })

  const url = URL.createObjectURL(blob)
  const link = document.createElement('a')

  link.href = url
  link.download = `palia-${location.value}-scores.csv`
  link.click()

  URL.revokeObjectURL(url)
}
</script>

<template>
  <main class="app">

    <!-- PAGE HEADER -->
    <header class="header">
      <div class="eyebrow">
        ✦ PALIA MINING PARTIES ✦
      </div>

      <h1>💎 Mining Scorekeeper</h1>

      <p>
        Count your gems. Score your teams. Crown your winners.
      </p>
    </header>

    <!-- MINING LOCATION -->
    <section class="card">
      <h2>🗺️ Choose Your Mining Location</h2>

      <div class="location-buttons">
        <button
          v-for="(preset, key) in presets"
          :key="key"
          :class="['location-button', { active: location === key }]"
          @click="location = key"
        >
          {{ preset.name }}
        </button>
      </div>

      <div class="location-info">
        <strong>{{ currentPreset.name }}</strong>

        <p>{{ currentPreset.description }}</p>

        <span>
          {{ current.gems.length }} scoring items available
        </span>
      </div>
    </section>

    <!-- PLAYER INFORMATION -->
    <section class="card">
      <h2>👤 Player Information</h2>

      <div class="two-columns">
        <label>
          Player Name
          <input
            v-model="playerName"
            placeholder="Enter player name"
          />
        </label>

        <label>
          Team Name
          <input
            v-model="teamName"
            placeholder="Example: Team 1"
          />
        </label>
      </div>
    </section>

    <!-- GEM COLLECTION -->
    <section class="card">
      <div class="section-title">
        <div>
          <h2>💎 Gem Collection</h2>
          <p>Star-quality items earn double points!</p>
        </div>

        <button
          class="small-button"
          @click="resetLocationRules"
        >
          Reset Rules
        </button>
      </div>

      <div class="gem-header">
        <span>Gem / Points</span>
        <span>Normal</span>
        <span>⭐ Star</span>
        <span></span>
      </div>

      <div
        v-for="gem in current.gems"
        :key="gem.id"
        class="gem-row"
      >
        <div class="gem-details">
          <input
            v-model="gem.name"
            class="gem-name"
            aria-label="Gem name"
          />

          <div class="points-edit">
            <input
              v-model.number="gem.points"
              type="number"
              min="0"
              aria-label="Points per gem"
            />

            <span>points each</span>
          </div>
        </div>

        <input
          v-model.number="gem.normal"
          type="number"
          min="0"
          aria-label="Normal quantity"
        />

        <input
          v-model.number="gem.starred"
          type="number"
          min="0"
          aria-label="Star quality quantity"
        />

        <button
          class="remove-button"
          title="Remove item"
          @click="removeGem(gem.id)"
        >
          ✕
        </button>
      </div>

      <button
        class="add-button"
        @click="addGem"
      >
        + Add Another Gem or Item
      </button>
    </section>

    <!-- KITSUU BONUS -->
    <section class="card">
      <h2>🦊 Kitsuu Plush Bonus</h2>

      <label class="plush-label">
        Bonus Item Name
        <input v-model="current.plushName" />
      </label>

      <div class="two-columns">
        <label>
          Quantity Collected
          <input
            v-model.number="current.plushCount"
            type="number"
            min="0"
          />
        </label>

        <label>
          Points Per Plush
          <input
            v-model.number="current.plushPoints"
            type="number"
            min="0"
          />
        </label>
      </div>
    </section>

    <!-- CURRENT PLAYER SCORE -->
    <section class="score-panel">

      <div class="score-heading">
        CURRENT PLAYER SCORE
      </div>

      <div class="score-display">
        {{ total }}
      </div>

      <div class="score-caption">
        TOTAL POINTS
      </div>

      <div class="score-actions">
        <button
          class="save-button"
          @click="saveScore"
        >
          💾 Save Player Score
        </button>

        <button
          class="reset-button"
          @click="resetCounts"
        >
          Reset Counts
        </button>
      </div>

    </section>

    <!-- TEAM LEADERBOARD -->
    <section class="card">
      <h2>🏆 Team Leaderboard</h2>

      <p class="subtitle">
        {{ currentPreset.name }}
      </p>

      <p
        v-if="!teamTotals.length"
        class="empty"
      >
        No teams scored yet. Let the mining begin!
      </p>

      <div
        v-for="(team, index) in teamTotals"
        :key="team.name"
        class="leader-row"
      >
        <span>
          <span v-if="index === 0">🥇</span>
          <span v-else-if="index === 1">🥈</span>
          <span v-else-if="index === 2">🥉</span>
          <span v-else>{{ index + 1 }}.</span>

          {{ team.name }}
        </span>

        <strong>{{ team.points }} pts</strong>
      </div>
    </section>

    <!-- INDIVIDUAL LEADERBOARD -->
    <section class="card">
      <div class="section-title">
        <h2>⭐ Individual Player Scores</h2>

        <button
          v-if="current.scores.length"
          class="small-button"
          @click="clearScores"
        >
          Clear Scores
        </button>
      </div>

      <p
        v-if="!sortedPlayers.length"
        class="empty"
      >
        Saved players will appear here.
      </p>

      <div
        v-for="(player, index) in sortedPlayers"
        :key="player.id"
        class="leader-row"
      >
        <div>
          <strong>
            {{ index + 1 }}. {{ player.name }}
          </strong>

          <small>{{ player.team }}</small>
        </div>

        <div class="score-right">
          <strong>{{ player.total }} pts</strong>

          <button
            class="remove-button"
            @click="removeScore(player.id)"
          >
            ✕
          </button>
        </div>
      </div>

      <button
        v-if="current.scores.length"
        class="export-button"
        @click="exportScores"
      >
        📥 Export Scores to CSV
      </button>
    </section>

    <!-- FOOTER -->
    <footer>
      ✦ Made for Palia Mining Parties ✦
      <p>Happy Mining!</p>
    </footer>

  </main>
</template>

<style>
/* GENERAL PAGE STYLING */

* {
  box-sizing: border-box;
}

:root {
  font-family: Arial, Helvetica, sans-serif;
  color: #f0eafa;
  background: #10101d;
  font-synthesis: none;
}

html {
  min-height: 100%;
}

body {
  display: block;
  margin: 0;
  min-width: 320px;
  min-height: 100vh;
  background: #10101d;
  color: #f0eafa;
}

#app {
  width: 100%;
  max-width: none;
  margin: 0;
  padding: 0;
  text-align: left;
}

.app {
  width: 100%;
  max-width: 900px;
  margin: 0 auto;
  padding: 35px 18px;
}

/* HEADER */

.header {
  text-align: center;
  margin-bottom: 35px;
}

.eyebrow {
  color: #c99df2;
  font-size: 12px;
  letter-spacing: 3px;
  font-weight: bold;
}

h1 {
  font-size: clamp(28px, 5vw, 42px);
  color: #e6caff;
  margin: 18px 0;
}

h2 {
  color: #ead4ff;
  font-size: 20px;
  margin: 0 0 18px;
}

p {
  color: #bcb2cf;
  line-height: 1.6;
}

.header p {
  font-size: 15px;
}

/* MAIN CARDS */

.card {
  background: #1c1a2f;
  border: 1px solid #41375e;
  border-radius: 16px;
  padding: 25px;
  margin-bottom: 22px;
}

/* BUTTONS */

button {
  font: inherit;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-weight: bold;
  padding: 12px 16px;
  background: #654996;
  color: white;
  transition: background 0.2s;
}

button:hover {
  filter: brightness(1.15);
}

/* LOCATION SELECTOR */

.location-buttons {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 12px;
}

.location-button {
  background: #302844;
  border: 1px solid #55466f;
  color: #d5c8e8;
}

.location-button.active {
  background: #9364d1;
  color: #fff;
  border-color: #c19af1;
}

.location-info {
  margin-top: 20px;
  padding: 18px;
  background: #26203c;
  border-radius: 10px;
  border-left: 4px solid #a478e1;
}

.location-info strong {
  color: #e2c3ff;
}

.location-info p {
  margin: 8px 0;
}

.location-info span {
  color: #d0b4e7;
  font-size: 13px;
}

/* INPUT FIELDS */

.two-columns {
  display: grid;
  grid-template-columns: repeat(2, 1fr);
  gap: 18px;
}

label {
  display: flex;
  flex-direction: column;
  gap: 9px;
  color: #cfc2e3;
  font-size: 14px;
}

input {
  width: 100%;
  min-width: 0;
  padding: 11px;
  border-radius: 8px;
  border: 1px solid #55486d;
  background: #121222;
  color: #fff;
  font: inherit;
}

input:focus {
  outline: 2px solid #af83e8;
}

/* GEM COLLECTION */

.section-title {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 15px;
  margin-bottom: 15px;
}

.section-title h2 {
  margin-bottom: 6px;
}

.section-title p {
  margin: 0;
  color: #dabf78;
  font-size: 13px;
}

.small-button {
  background: #39304f;
  border: 1px solid #6b5986;
  font-size: 12px;
  padding: 9px 12px;
  white-space: nowrap;
}

.gem-header,
.gem-row {
  display: grid;
  grid-template-columns: minmax(0, 3fr) 1fr 1fr 30px;
  gap: 12px;
  align-items: center;
}

.gem-header {
  text-align: center;
  font-size: 12px;
  color: #b8a9cc;
  margin: 20px 0 8px;
}

.gem-header span:first-child {
  text-align: left;
}

.gem-row {
  padding: 12px 0;
  border-bottom: 1px solid #37304b;
}

.gem-row > input {
  text-align: center;
}

.gem-name {
  color: #e7d2ff;
  font-weight: bold;
}

.points-edit {
  margin-top: 7px;
  display: flex;
  align-items: center;
  gap: 8px;
  color: #b8a9cc;
  font-size: 12px;
}

.points-edit input {
  width: 65px;
  padding: 6px;
}

.remove-button {
  background: transparent;
  color: #e9a0b7;
  padding: 6px;
}

.add-button {
  margin-top: 20px;
  background: #43315e;
  border: 1px solid #795ba6;
  width: 100%;
}

/* KITSUU BONUS */

.plush-label {
  margin-bottom: 18px;
}

/* PLAYER TOTAL - SPACING FIX */

.score-panel {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 20px;

  width: 100%;
  padding: 40px 25px;
  margin-bottom: 22px;

  text-align: center;
  background: linear-gradient(135deg, #4b316b, #272044);
  border: 1px solid #a079cd;
  border-radius: 16px;
}

.score-heading {
  display: block;
  color: #e3cfff;
  font-size: 14px;
  font-weight: bold;
  letter-spacing: 3px;
  line-height: 1.5;
}

.score-display {
  display: block;
  font-size: clamp(65px, 12vw, 90px);
  font-weight: bold;
  color: #ffffff;
  line-height: 1.2;
  margin: 0;
  padding: 8px 0;
}

.score-caption {
  display: block;
  color: #dbbaff;
  font-size: 13px;
  letter-spacing: 3px;
  line-height: 1.5;
}

.score-actions {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 15px;
  flex-wrap: wrap;
  width: 100%;
  margin-top: 15px;
}

.save-button {
  background: #c19af1;
  color: #241731;
}

.reset-button {
  background: #33284c;
  border: 1px solid #8b6eae;
}

/* LEADERBOARDS */

.subtitle {
  font-size: 13px;
  margin-top: -10px;
}

.leader-row {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 12px;
  padding: 17px 5px;
  border-bottom: 1px solid #3d3451;
}

.leader-row strong {
  color: #ead08a;
}

.leader-row small {
  display: block;
  margin-top: 5px;
  color: #a99cba;
}

.score-right {
  display: flex;
  align-items: center;
  gap: 12px;
}

.empty {
  text-align: center;
  padding: 20px;
  font-size: 14px;
}

.export-button {
  margin-top: 20px;
  width: 100%;
  background: #43315e;
  border: 1px solid #795ba6;
}

/* FOOTER */

footer {
  text-align: center;
  color: #a99abd;
  padding: 25px;
  font-size: 13px;
}

footer p {
  font-size: 12px;
}

/* MOBILE LAYOUT */

@media (max-width: 600px) {
  .location-buttons {
    grid-template-columns: 1fr;
  }

  .two-columns {
    grid-template-columns: 1fr;
  }

  .card {
    padding: 17px;
  }

  .gem-header,
  .gem-row {
    gap: 6px;
    grid-template-columns: minmax(0, 2.5fr) 1fr 1fr 24px;
  }

  .gem-row input {
    padding: 8px 4px;
  }

  .section-title {
    flex-wrap: wrap;
  }

  .points-edit {
    flex-wrap: wrap;
  }

  .score-panel {
    padding: 35px 15px;
    gap: 18px;
  }

  .score-actions {
    flex-direction: column;
  }

  .score-actions button {
    width: 100%;
  }
}
</style>
