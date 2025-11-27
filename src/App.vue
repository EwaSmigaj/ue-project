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

// NOWY TYP: Planer posiłków
interface MealPlanEntry {
  day: string; // np. Poniedziałek
  mealType: 'Śniadanie' | 'Obiad' | 'Kolacja';
  recipeId: number | null; // ID przepisu lub null, jeśli nic nie wybrano
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
    ingredients: ['makaron', 'mięso', 'sos pomidorowy', 'ser', 'cebula'],
    isFavorite: false
  },
  {
    id: 3,
    title: 'Koktajl na trawienie',
    author: 'Zdrowie 100%',
    img: '🥤',
    tags: ['ZDROWE'],
    ingredients: ['jogurt', 'banan', 'szpinak', 'nasiona chia'],
    isFavorite: true
  },
  {
    id: 4,
    title: 'Zupa Pomidorowa',
    author: 'Babcia Zosia',
    img: '🍅',
    tags: ['SZYBKIE'],
    ingredients: ['pomidory', 'wywar', 'śmietana', 'ryż', 'cebula'],
    isFavorite: false
  }
]);

// --- 2. Stan Aplikacji ---
type View = 'HOME' | 'LIST' | 'FAVORITES' | 'DETAIL' | 'PLANNER' | 'SHOPPING';
const currentView = ref<View>('HOME');
const selectedRecipe = ref<Recipe | null>(null);

// NOWY STAN: Planer posiłków (domyślny szablon na 3 dni)
const days = ['Poniedziałek', 'Wtorek', 'Środa', 'Czwartek', 'Piątek', 'Sobota', 'Niedziela'];
const mealTypes: MealPlanEntry['mealType'][] = ['Śniadanie', 'Obiad', 'Kolacja'];

const mealPlan = ref<MealPlanEntry[]>(
  days.flatMap(day => 
    mealTypes.map(mealType => ({ 
      day, 
      mealType, 
      recipeId: null 
    }))
  )
);

// Zmienna do przechowywania tymczasowego wyboru przepisu w Plannerze
const plannerSelectedRecipeId = ref<number | null>(null); 
const plannerTargetSlot = ref<{ day: string, mealType: MealPlanEntry['mealType'] } | null>(null);


// --- 3. Nawigacja i Akcje ---
const goHome = () => {
  currentView.value = 'HOME';
  selectedRecipe.value = null;
};

const goBack = () => {
  if (currentView.value === 'DETAIL') {
    currentView.value = 'LIST';
  } else if (currentView.value === 'PLANNER' && plannerTargetSlot.value) {
    plannerTargetSlot.value = null; // Wróć z widoku wyboru przepisu do głównego planera
  } else {
    goHome();
  }
};

const showList = () => currentView.value = 'LIST';
const showFavorites = () => currentView.value = 'FAVORITES';
const showPlanner = () => currentView.value = 'PLANNER';
const showShopping = () => currentView.value = 'SHOPPING';

const openRecipe = (recipe: Recipe) => {
  if (currentView.value === 'PLANNER' && plannerTargetSlot.value) {
    // Jeśli jesteśmy w trybie dodawania do planera
    addRecipeToPlan(plannerTargetSlot.value.day, plannerTargetSlot.value.mealType, recipe.id);
    plannerTargetSlot.value = null; // Wyjdź z trybu wyboru
    currentView.value = 'PLANNER';
  } else {
    // Normalne otwarcie szczegółów przepisu
    selectedRecipe.value = recipe;
    currentView.value = 'DETAIL';
  }
};

const toggleFavorite = (recipe: Recipe) => {
  recipe.isFavorite = !recipe.isFavorite;
};

// --- 4. Logika Planera Posiłków ---
const getRecipeTitleById = (id: number | null): string => {
  if (id === null) return '';
  const recipe = recipes.value.find(r => r.id === id);
  return recipe ? recipe.title : 'Nieznany Przepis';
};

const addRecipeToPlan = (day: string, mealType: MealPlanEntry['mealType'], recipeId: number) => {
  const entry = mealPlan.value.find(e => e.day === day && e.mealType === mealType);
  if (entry) {
    entry.recipeId = recipeId;
  }
};

const removeRecipeFromPlan = (day: string, mealType: MealPlanEntry['mealType']) => {
  const entry = mealPlan.value.find(e => e.day === day && e.mealType === mealType);
  if (entry) {
    entry.recipeId = null;
  }
};

const startRecipeSelection = (day: string, mealType: MealPlanEntry['mealType']) => {
  plannerTargetSlot.value = { day, mealType };
  currentView.value = 'LIST'; // Możemy użyć istniejącego widoku listy do wyboru
};

// Funkcja pomocnicza do grupowania planu
const groupedMealPlan = computed(() => {
  return days.map(day => ({
    day,
    meals: mealTypes.map(mealType => mealPlan.value.find(e => e.day === day && e.mealType === mealType))
  }));
});


// --- 5. Logika Listy Zakupów ---
const shoppingList = computed(() => {
  const ingredientsMap = new Map<string, boolean>(); // Składnik -> stan (czy kupiony)
  
  // 1. Zbierz składniki z zaplanowanych przepisów
  const plannedRecipeIds = new Set(mealPlan.value.map(e => e.recipeId).filter(id => id !== null) as number[]);
  
  plannedRecipeIds.forEach(recipeId => {
    const recipe = recipes.value.find(r => r.id === recipeId);
    if (recipe) {
      recipe.ingredients.forEach(ingredient => {
        const normalizedIng = ingredient.toLowerCase().trim();
        if (!ingredientsMap.has(normalizedIng)) {
          ingredientsMap.set(normalizedIng, false); // Domyślnie niekupione
        }
      });
    }
  });

  // 2. Konwertuj mapę na tablicę do wyświetlenia
  return Array.from(ingredientsMap.keys()).sort();
});


// --- 6. Computed dla wyświetlania ---
const displayedRecipes = computed(() => {
  // Jeśli jesteśmy w trybie wyboru przepisu dla planera, wyświetl wszystkie lub ulubione, jeśli jest włączony widok 'FAVORITES'
  if (currentView.value === 'LIST' && plannerTargetSlot.value) {
    return recipes.value;
  }

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
          <button 
            v-if="currentView !== 'HOME'" 
            @click="goBack" 
            class="nav-btn back-btn"
          >
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
          <h2 class="section-title">
            {{ plannerTargetSlot ? 'Wybierz Przepis' : 
               (currentView === 'FAVORITES' ? 'Twoje Ulubione' : 'Książka Kucharska') }}
          </h2>
          <p v-if="plannerTargetSlot" class="planner-info">
            Wybierasz przepis na **{{ plannerTargetSlot.mealType }}** w **{{ plannerTargetSlot.day }}**. Kliknij przepis, aby go dodać.
          </p>
          
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
              
              <button class="action-btn green" style="margin-top: 15px;" @click="showPlanner">
                📅 Dodaj do Planera
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

        <main v-else-if="currentView === 'PLANNER'" class="planner-view">
          <h2 class="section-title">📅 Planer Posiłków</h2>
          <div class="planner-grid">
            <div class="planner-header"></div>
            <div v-for="meal in mealTypes" :key="meal" class="planner-header meal-type">{{ meal }}</div>
            
            <template v-for="group in groupedMealPlan" :key="group.day">
              <div class="day-label">{{ group.day }}</div>
              <div v-for="mealEntry in group.meals" :key="mealEntry!.mealType" class="meal-slot">
                
                <button 
                  v-if="mealEntry!.recipeId" 
                  class="recipe-btn filled" 
                  @click="openRecipe(recipes.find(r => r.id === mealEntry!.recipeId)!)"
                >
                  {{ getRecipeTitleById(mealEntry!.recipeId) }}
                  <span class="remove-btn" @click.stop="removeRecipeFromPlan(mealEntry!.day, mealEntry!.mealType)">✖</span>
                </button>
                
                <button v-else class="recipe-btn empty" @click="startRecipeSelection(mealEntry!.day, mealEntry!.mealType)">
                  + Dodaj Przepis
                </button>
              </div>
            </template>
          </div>
          <p class="planner-tip">Kliknij na przepis, aby zobaczyć szczegóły, lub na slot, aby wybrać nowy.</p>
        </main>

        <main v-else-if="currentView === 'SHOPPING'" class="shopping-view">
          <h2 class="section-title">🛒 Lista Zakupów</h2>
          <p class="shopping-info">Lista generowana na podstawie wszystkich przepisów w **Planerze Posiłków**.</p>
          
          <div v-if="shoppingList.length === 0" class="empty-shopping">
            <span class="icon">📝</span>
            <p>Twój planer posiłków jest pusty! Dodaj przepisy, by wygenerować listę.</p>
            <button class="action-btn blue" @click="showPlanner">Przejdź do Planera</button>
          </div>

          <ul v-else class="shopping-list">
            <li v-for="(ingredient, index) in shoppingList" :key="index">
              <label class="shopping-item">
                <input type="checkbox" class="big-checkbox">
                {{ ingredient }}
              </label>
            </li>
          </ul>

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
.planner-info { font-size: 1.2rem; margin-bottom: 20px; padding: 10px; border: 2px dashed #FF5964; background: #FFE74C50; border-radius: 10px; } /* Dodano styl dla info w trybie wyboru */

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

/* --- NOWE STYLE: PLANER POSIŁKÓW --- */
.planner-grid {
  display: grid;
  grid-template-columns: 150px repeat(3, 1fr); /* Dzień + Śniadanie/Obiad/Kolacja */
  border: 3px solid #000;
  border-radius: 15px;
  overflow: hidden;
  box-shadow: 8px 8px 0 #000;
}
.planner-header {
  padding: 15px 10px;
  font-weight: bold;
  font-size: 1.2rem;
  border-bottom: 3px solid #000;
  border-right: 3px solid #000;
  text-align: center;
  background-color: #FFE74C; /* Yellow */
}
.planner-header:last-child { border-right: none; }
.day-label {
  padding: 15px 10px;
  font-weight: bold;
  font-size: 1.2rem;
  background-color: #f0f0f0;
  border-right: 3px solid #000;
  border-bottom: 1px solid #ddd;
  display: flex;
  align-items: center;
  justify-content: flex-start;
}
/* Usuń dolną linię dla ostatniego dnia */
.planner-grid > .day-label:nth-last-child(-n+4) { border-bottom: none; }

.meal-slot {
  padding: 10px;
  border-right: 1px solid #ddd;
  border-bottom: 1px solid #ddd;
  min-height: 80px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.meal-slot:nth-child(4n) { border-right: none; }

/* Usuń dolną linię dla ostatniego wiersza */
.planner-grid > .meal-slot:nth-last-child(-n+3) { border-bottom: none; }

.recipe-btn {
  width: 100%;
  height: 100%;
  min-height: 60px;
  padding: 10px;
  border-radius: 8px;
  border: 2px dashed #000;
  font-size: 1rem;
  font-weight: bold;
  cursor: pointer;
  transition: background-color 0.1s;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
  position: relative;
}
.recipe-btn.empty {
  background-color: #f9f9f9;
  color: #555;
  border-style: dashed;
}
.recipe-btn.empty:hover {
  background-color: #FFE74C; /* Lekki Yellow */
  border-style: solid;
}
.recipe-btn.filled {
  background-color: #6BF178; /* Green */
  color: #000;
  border-style: solid;
  box-shadow: 2px 2px 0 #000;
}
.remove-btn {
  position: absolute;
  top: -8px;
  right: -8px;
  background: #FF5964; /* Red */
  border: 2px solid #000;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  line-height: 16px;
  font-size: 0.8rem;
  font-weight: bold;
  text-align: center;
  z-index: 10;
  box-shadow: 1px 1px 0 #000;
}
.planner-tip {
  margin-top: 20px;
  font-size: 1rem;
  text-align: center;
  color: #666;
}

/* NOWE STYLE: LISTA ZAKUPÓW */
.shopping-info {
  font-size: 1.2rem;
  margin-bottom: 30px;
  color: #666;
  text-align: center;
}
.shopping-list {
  list-style: none;
  padding: 0;
  display: grid;
  grid-template-columns: repeat(3, 1fr); /* 3 kolumny dla przejrzystości */
  gap: 15px 30px;
  font-size: 1.5rem;
  border: 3px solid #000;
  border-radius: 15px;
  padding: 30px;
  background: #f0f0f0;
  box-shadow: 5px 5px 0 #000;
}
.shopping-item {
  display: flex;
  align-items: center;
  cursor: pointer;
}
.empty-shopping {
  text-align: center;
  padding: 50px;
  border: 3px dashed #FF5964;
  border-radius: 15px;
  background: #FFE74C50;
}
.empty-shopping .icon {
  font-size: 5rem;
  display: block;
  margin-bottom: 20px;
}
.empty-shopping p {
  font-size: 1.5rem;
  margin-bottom: 30px;
}
</style>