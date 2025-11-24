<script setup lang="ts">
import { ref, computed } from 'vue';

// --- 1. Typy i Dane ---
interface Recipe {
  id: number;
  title: string;
  author: string;
  img: string;
  tags: string[];
  ingredients: string[];
  isFavorite: boolean;
}

const recipes = ref<Recipe[]>([
  {
    id: 1,
    title: 'Kurczak w Ziołach',
    author: 'Grażyna Gotuje',
    img: '🍗',
    tags: ['ŁATWE'],
    ingredients: ['kurczak', 'olej', 'sól', 'pieprz', 'czosnek'],
    isFavorite: false
  },
  {
    id: 2,
    title: 'Lasagne',
    author: 'Kuchnia Włoska',
    img: '🍝',
    tags: [],
    ingredients: ['makaron', 'mięso', 'sos pomidorowy', 'ser'],
    isFavorite: false
  },
  {
    id: 3,
    title: 'Koktajl na trawienie',
    author: 'Zdrowie 100%',
    img: '🥤',
    tags: ['ZDROWE'],
    ingredients: ['jogurt', 'banan', 'szpinak'],
    isFavorite: true
  },
  {
    id: 4,
    title: 'Zupa Pomidorowa',
    author: 'Babcia Zosia',
    img: '🍅',
    tags: ['SZYBKIE'],
    ingredients: ['pomidory', 'wywar', 'śmietana', 'ryż'],
    isFavorite: false
  }
]);

// --- 2. Stan Aplikacji ---
type View = 'HOME' | 'LIST' | 'FAVORITES' | 'DETAIL' | 'PLANNER' | 'SHOPPING';
const currentView = ref<View>('HOME');
const selectedRecipe = ref<Recipe | null>(null);

// --- 3. Nawigacja ---
const goHome = () => {
  currentView.value = 'HOME';
  selectedRecipe.value = null;
};

const goBack = () => {
  if (currentView.value === 'DETAIL') {
    currentView.value = 'LIST';
  } else {
    goHome();
  }
};

const showList = () => currentView.value = 'LIST';
const showFavorites = () => currentView.value = 'FAVORITES';
const showPlanner = () => currentView.value = 'PLANNER';
const showShopping = () => currentView.value = 'SHOPPING';

const openRecipe = (recipe: Recipe) => {
  selectedRecipe.value = recipe;
  currentView.value = 'DETAIL';
};

const toggleFavorite = (recipe: Recipe) => {
  recipe.isFavorite = !recipe.isFavorite;
};

const displayedRecipes = computed(() => {
  if (currentView.value === 'FAVORITES') {
    return recipes.value.filter(r => r.isFavorite);
  }
  return recipes.value;
});
</script>

<template>
  <div class="desktop-wrapper">
    <div class="app-container">
      
      <header class="top-bar">
        <div class="left-nav">
          <button v-if="currentView !== 'HOME'" @click="goBack" class="nav-btn back-btn">
            ↩ Wróć
          </button>
          <span v-else class="logo">🍲 BABUSHEX</span>
        </div>
        
        <div class="top-icons">
          <button class="icon-btn">👤 Profil</button>
          <button class="icon-btn">❓ Pomoc</button>
        </div>
      </header>

      <div class="content-area">
        <main v-if="currentView === 'HOME'" class="main-menu">
          <h1>Co robimy dzisiaj?</h1>
          
          <div class="menu-grid">
            <button class="menu-card yellow" @click="showList">
              <span class="icon">🍲</span>
              <span class="label">Znajdź Przepisy</span>
            </button>
            
            <button class="menu-card green" @click="showFavorites">
              <span class="icon">♡</span>
              <span class="label">Ulubione</span>
            </button>
            
            <button class="menu-card red" @click="showPlanner">
              <span class="icon">📅</span>
              <span class="label">Planer Posiłków</span>
            </button>
            
            <button class="menu-card blue" @click="showShopping">
              <span class="icon">🛒</span>
              <span class="label">Lista Zakupów</span>
            </button>
          </div>

          <div class="settings-area">
            <button class="settings-btn">⚙ Ustawienia</button>
          </div>
        </main>

        <main v-else-if="currentView === 'LIST' || currentView === 'FAVORITES'" class="list-view">
          <h2 class="section-title">{{ currentView === 'FAVORITES' ? 'Twoje Ulubione' : 'Książka Kucharska' }}</h2>
          
          <div class="recipes-grid">
            <div 
              v-for="recipe in displayedRecipes" 
              :key="recipe.id" 
              class="recipe-card"
              @click="openRecipe(recipe)"
            >
              <div class="recipe-img">{{ recipe.img }}</div>
              <span v-if="recipe.tags.length" class="tag">{{ recipe.tags[0] }}</span>
              <h3>{{ recipe.title }}</h3>
              <div class="card-footer">
                <span>{{ recipe.author }}</span>
                <span v-if="recipe.isFavorite" class="heart">❤</span>
              </div>
            </div>
          </div>
        </main>

        <main v-else-if="currentView === 'DETAIL' && selectedRecipe" class="detail-view">
          <div class="detail-layout">
            <div class="detail-left">
              <div class="big-img">{{ selectedRecipe.img }}</div>
              <button class="action-btn yellow" @click="toggleFavorite(selectedRecipe)">
                {{ selectedRecipe.isFavorite ? '💔 Usuń z ulubionych' : '❤ Dodaj do ulubionych' }}
              </button>
            </div>

            <div class="detail-right">
              <h2>{{ selectedRecipe.title }}</h2>
              <p class="author">Autor: {{ selectedRecipe.author }}</p>
              
              <div class="ingredients-box">
                <h3>POTRZEBUJESZ:</h3>
                <ul>
                  <li v-for="ing in selectedRecipe.ingredients" :key="ing">
                    <label>
                      <input type="checkbox" class="big-checkbox">
                      {{ ing }}
                    </label>
                  </li>
                </ul>
              </div>
            </div>
          </div>
        </main>

        <main v-else class="placeholder-view">
          <h2>Funkcja w budowie...</h2>
        </main>
      </div>

    </div>
  </div>
</template>

<style scoped>
/* --- PALETA BARW Z PLIKU PNG --- */
/*
  Yellow: #FFE74C
  Red:    #FF5964
  White:  #FFFFFF
  Green:  #6BF178
  Blue:   #35A7FF
*/

.desktop-wrapper {
  background-color: #35A7FF; /* Blue jako tło pulpitu */
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 40px;
  box-sizing: border-box;
  font-family: 'Comic Sans MS', 'Arial', sans-serif;
}

.app-container {
  background-color: #FFFFFF; /* White jako "kartka" */
  width: 100%;
  max-width: 1200px; /* Szerokość desktopowa */
  min-height: 80vh;
  border-radius: 20px;
  box-shadow: 10px 10px 0px rgba(0,0,0,0.2);
  border: 4px solid #000;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* --- BUTTON COLORS --- */
.yellow { background-color: #FFE74C; }
.red { background-color: #FF5964; }
.green { background-color: #6BF178; }
.blue { background-color: #35A7FF; }

/* --- NAGŁÓWEK --- */
.top-bar {
  padding: 20px 40px;
  border-bottom: 4px solid #000;
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #fff;
}
.logo { font-size: 2rem; font-weight: bold; }
.nav-btn, .icon-btn {
  border: 3px solid #000;
  background: #fff;
  padding: 10px 20px;
  font-size: 1.2rem;
  border-radius: 12px;
  cursor: pointer;
  font-weight: bold;
  box-shadow: 4px 4px 0 #000;
  transition: transform 0.1s;
}
.nav-btn:active, .icon-btn:active { transform: translate(2px, 2px); box-shadow: 2px 2px 0 #000; }
.top-icons { display: flex; gap: 15px; }

/* --- TREŚĆ --- */
.content-area { padding: 40px; flex-grow: 1; }

/* --- MENU GŁÓWNE --- */
.main-menu h1 { text-align: center; font-size: 3rem; margin-bottom: 50px; }
.menu-grid {
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* 4 kolumny dla Desktopu */
  gap: 30px;
  margin-bottom: 50px;
}
.menu-card {
  height: 250px;
  border: 4px solid #000;
  border-radius: 20px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  cursor: pointer;
  box-shadow: 8px 8px 0 #000;
  transition: transform 0.2s;
}
.menu-card:hover { transform: translateY(-5px); }
.menu-card .icon { font-size: 4rem; margin-bottom: 15px; }
.menu-card .label { font-size: 1.8rem; font-weight: bold; text-align: center; color: #000; }

.settings-area { text-align: center; }
.settings-btn { background: none; border: none; font-size: 1.5rem; text-decoration: underline; cursor: pointer; }

/* --- LISTA PRZEPISÓW --- */
.section-title { font-size: 2.5rem; margin-bottom: 30px; border-bottom: 2px dashed #000; display: inline-block; }
.recipes-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 30px;
}
.recipe-card {
  border: 3px solid #000;
  border-radius: 15px;
  padding: 20px;
  text-align: center;
  cursor: pointer;
  position: relative;
  background: #fff;
  box-shadow: 5px 5px 0 #000;
  transition: 0.2s;
}
.recipe-card:hover { background-color: #f9f9f9; transform: scale(1.02); }
.recipe-img { font-size: 5rem; margin-bottom: 10px; }
.tag {
  position: absolute; top: 10px; right: 10px;
  background: #FFE74C; border: 2px solid #000; padding: 2px 8px; font-size: 0.9rem; border-radius: 5px; font-weight: bold;
}
.card-footer { display: flex; justify-content: space-between; margin-top: 15px; font-size: 0.9rem; color: #555; }
.heart { color: #FF5964; font-size: 1.5rem; }

/* --- SZCZEGÓŁY (Layout 2-kolumnowy) --- */
.detail-layout { display: grid; grid-template-columns: 1fr 2fr; gap: 50px; }
.big-img { font-size: 10rem; text-align: center; border: 3px solid #000; border-radius: 20px; padding: 20px; margin-bottom: 20px; background: #fff; }
.action-btn { width: 100%; padding: 15px; font-size: 1.2rem; border: 3px solid #000; border-radius: 10px; font-weight: bold; cursor: pointer; box-shadow: 5px 5px 0 #000; }
.detail-right h2 { font-size: 3rem; margin: 0 0 10px 0; }
.author { font-size: 1.5rem; color: #666; margin-bottom: 30px; }
.ingredients-box { background: #f0f0f0; border: 3px solid #000; padding: 30px; border-radius: 15px; position: relative; }
.ingredients-box ul { list-style: none; padding: 0; font-size: 1.5rem; }
.ingredients-box li { margin-bottom: 15px; }
.big-checkbox { width: 30px; height: 30px; margin-right: 15px; accent-color: #6BF178; }
</style>