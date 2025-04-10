<script setup lang="ts">
import { ref, onMounted, watch } from 'vue';
import type { Card } from '../types';

const props = defineProps<{
  show: boolean;
  card: Card;
}>();

const emit = defineEmits<{
  (e: 'close'): void;
  (e: 'update', card: Card): void;
}>();

const title = ref('');
const icon = ref('');
const isGroup = ref(false);
const backgroundColor = ref('');
const textColor = ref('');
const iconColor = ref('');
const titleInput = ref<HTMLInputElement | null>(null);
const showImageSearchModal = ref(false);
const showExpressiaImageSearchModal = ref(false);
const imageSearchResults = ref<{ link: string; thumbnailLink?: string }[]>([]);
const imageSearchQuery = ref('');
const isSearching = ref(false);

onMounted(() => {
  title.value = props.card.title;
  icon.value = props.card.icon;
  isGroup.value = !!props.card.subcards;
  backgroundColor.value = props.card.backgroundColor || props.card.category.color;
  textColor.value = props.card.textColor || '';
  iconColor.value = props.card.iconColor || '';
});

// Focus title input when dialog opens
watch(() => props.show, (newValue) => {
  if (newValue) {
    setTimeout(() => {
      titleInput.value?.focus();
    }, 100);
  }
});

// Handle body scroll lock
watch(() => props.show, (newValue) => {
  if (newValue) {
    document.body.style.overflow = 'hidden';
  } else {
    document.body.style.overflow = '';
  }
});

const handleSubmit = () => {
  if (!title.value || !icon.value) {
    return;
  }

  emit('update', {
    ...props.card,
    title: title.value,
    icon: icon.value,
    subcards: isGroup.value ? (props.card.subcards || []) : undefined,
    backgroundColor: backgroundColor.value !== props.card.category.color ? backgroundColor.value : undefined,
    textColor: textColor.value || undefined,
    iconColor: iconColor.value || undefined,
  });
};

const handleKeyDown = (event: KeyboardEvent) => {
  if (event.key === 'Escape') {
    emit('close');
  }
};

const searchImages = async (page = 1) => {
  if (!imageSearchQuery.value) return;
  
  isSearching.value = true;
  try {
    const apiKey = "AIzaSyDdlGItp_cjFBfqoB8mtepToa_6D3p-H6Q";
    const cx = "22223a4a6be2a43bd";
    const start = (page - 1) * 20 + 1;
    const response = await fetch(`https://www.googleapis.com/customsearch/v1?key=${apiKey}&cx=${cx}&q=${imageSearchQuery.value}&searchType=image&start=${start}`);
    const data = await response.json();
    imageSearchResults.value = data.items.map((item: any) => ({
      link: item.link,
      thumbnailLink: item.link
    }));
  } catch (error) {
    console.error('Error searching images:', error);
  } finally {
    isSearching.value = false;
  }
};

const searchImagesExpressia = async () => {
  if (!imageSearchQuery.value) return;
  
  isSearching.value = true;
  try {
    const response = await fetch('https://us-central1-expressia-a2020.cloudfunctions.net/imageSearchV5', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        data: {
          search_text: imageSearchQuery.value,
          countryCode: 'br'
        }
      })
    });
    const data = await response.json();
    imageSearchResults.value = data.result.items.map((item: any) => ({
      link: item.link,
      thumbnailLink: item.image.thumbnailLink
    }));
  } catch (error) {
    console.error('Error searching images:', error);
  } finally {
    isSearching.value = false;
  }
};

const selectImage = (imageUrl: string) => {
  icon.value = imageUrl;
  showImageSearchModal.value = false;
  showExpressiaImageSearchModal.value = false;
  imageSearchResults.value = [];
  imageSearchQuery.value = '';
};
</script>

<template>
  <div 
    v-if="show" 
    class="dialog-overlay" 
    @click.self="emit('close')"
    @keydown="handleKeyDown"
    role="dialog"
    aria-modal="true"
    aria-labelledby="dialog-title"
  >
    <div class="dialog-content">
      <div class="dialog-header">
        <h2 id="dialog-title">Editar Item</h2>
        <button 
          class="close-button" 
          @click="emit('close')" 
          aria-label="Fechar"
        >
          ×
        </button>
      </div>
      
      <div class="dialog-body">
        <form @submit.prevent="handleSubmit">
          <div class="form-group">
            <label for="title">Título:</label>
            <input
              ref="titleInput"
              id="title"
              v-model="title"
              type="text"
              required
              placeholder="Digite o título"
              aria-required="true"
            />
          </div>

          <div class="form-group">
            <label for="icon">URL do Ícone:</label>
            <div class="icon-input-wrapper">
              <input
                id="icon"
                v-model="icon"
                type="url"
                required
                placeholder="https://api.iconify.design/..."
                aria-required="true"
              />
              <div class="icon-search-buttons">
                <button type="button" @click="showImageSearchModal = true" class="search-button">
                  Buscar no Google
                </button>
                <button type="button" @click="showExpressiaImageSearchModal = true" class="search-button">
                  Buscar no Expressia
                </button>
              </div>
            </div>
          </div>

          <div class="form-group">
            <label class="checkbox-label">
              <input
                type="checkbox"
                v-model="isGroup"
                :disabled="true"
                aria-label="É um grupo?"
              />
              É um grupo?
            </label>
          </div>

          <div class="color-customization">
            <h3>Personalização de Cores</h3>
            
            <div class="form-group">
              <label for="backgroundColor">Cor de Fundo:</label>
              <div class="color-input-wrapper">
                <input
                  id="backgroundColor"
                  v-model="backgroundColor"
                  type="color"
                  class="color-picker"
                  aria-label="Selecione a cor de fundo"
                />
                <input
                  type="text"
                  v-model="backgroundColor"
                  :placeholder="props.card.category.color"
                  class="color-text"
                  aria-label="Digite o código da cor de fundo"
                />
              </div>
            </div>

            <div class="form-group">
              <label for="textColor">Cor do Texto:</label>
              <div class="color-input-wrapper">
                <input
                  id="textColor"
                  v-model="textColor"
                  type="color"
                  class="color-picker"
                  aria-label="Selecione a cor do texto"
                />
                <input
                  type="text"
                  v-model="textColor"
                  placeholder="#000000"
                  class="color-text"
                  aria-label="Digite o código da cor do texto"
                />
              </div>
            </div>

            <div class="form-group">
              <label for="iconColor">Cor do Ícone:</label>
              <div class="color-input-wrapper">
                <input
                  id="iconColor"
                  v-model="iconColor"
                  type="color"
                  class="color-picker"
                  aria-label="Selecione a cor do ícone"
                />
                <input
                  type="text"
                  v-model="iconColor"
                  placeholder="#000000"
                  class="color-text"
                  aria-label="Digite o código da cor do ícone"
                />
              </div>
            </div>
          </div>

          <!-- Google Image Search Modal -->
          <div v-if="showImageSearchModal" class="image-search-modal">
            <div class="modal-content">
              <div class="modal-header">
                <h3>Buscar Imagens no Google</h3>
                <button @click="showImageSearchModal = false" class="close-button">×</button>
              </div>
              <div class="search-input-wrapper">
                <input
                  v-model="imageSearchQuery"
                  type="text"
                  placeholder="Digite o que deseja buscar..."
                  @keyup.enter="searchImages(1)"
                />
                <button @click="searchImages(1)" :disabled="isSearching">
                  {{ isSearching ? 'Buscando...' : 'Buscar' }}
                </button>
              </div>
              <div class="image-grid">
                <div
                  v-for="(image, index) in imageSearchResults"
                  :key="index"
                  class="image-item"
                  @click="selectImage(image.link)"
                >
                  <img :src="image.thumbnailLink || image.link" :alt="'Imagem ' + (index + 1)" />
                </div>
              </div>
              <div class="pagination" v-if="imageSearchResults.length > 0">
                <button @click="searchImages(1)">1</button>
                <button @click="searchImages(2)">2</button>
                <button @click="searchImages(3)">3</button>
                <button @click="searchImages(4)">4</button>
                <button @click="searchImages(5)">5</button>
              </div>
            </div>
          </div>

          <!-- Expressia Image Search Modal -->
          <div v-if="showExpressiaImageSearchModal" class="image-search-modal">
            <div class="modal-content">
              <div class="modal-header">
                <h3>Buscar Imagens no Expressia</h3>
                <button @click="showExpressiaImageSearchModal = false" class="close-button">×</button>
              </div>
              <div class="search-input-wrapper">
                <input
                  v-model="imageSearchQuery"
                  type="text"
                  placeholder="Digite o que deseja buscar..."
                  @keyup.enter="searchImagesExpressia"
                />
                <button @click="searchImagesExpressia" :disabled="isSearching">
                  {{ isSearching ? 'Buscando...' : 'Buscar' }}
                </button>
              </div>
              <div class="image-grid">
                <div
                  v-for="(image, index) in imageSearchResults"
                  :key="index"
                  class="image-item"
                  @click="selectImage(image.link)"
                >
                  <img :src="image.thumbnailLink || image.link" :alt="'Imagem ' + (index + 1)" />
                </div>
              </div>
            </div>
          </div>

          <div class="dialog-buttons">
            <button 
              type="button" 
              @click="emit('close')"
              class="cancel-button"
            >
              Cancelar
            </button>
            <button 
              type="submit" 
              class="submit-button"
              :disabled="!title || !icon"
            >
              Salvar
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dialog-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  /* Safari fix for full height */
  min-height: -webkit-fill-available;
  height: 100vh;
}

.dialog-content {
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.2);
  width: 90%;
  max-width: 400px;
  max-height: 90vh;
  /* Safari fixes */
  max-height: -webkit-fill-available;
  display: flex;
  flex-direction: column;
  margin: 1rem;
  overflow: hidden; /* Prevent content overflow */
}

.dialog-header {
  padding: 1rem;
  border-bottom: 1px solid #eee;
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: white;
  border-radius: 12px 12px 0 0;
  position: relative;
  z-index: 1;
}

.dialog-header h2 {
  margin: 0;
  font-size: 1.25rem;
}

.close-button {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  padding: 0.5rem;
  line-height: 1;
  color: #666;
  -webkit-tap-highlight-color: transparent;
  transition: color 0.2s ease;
}

.close-button:hover {
  color: #000;
}

.close-button:focus {
  outline: 2px solid #2196F3;
  outline-offset: 2px;
}

.dialog-body {
  padding: 1rem;
  overflow-y: auto;
  -webkit-overflow-scrolling: touch;
  flex: 1;
  /* Safari fixes */
  position: relative;
  height: 100%;
}

.form-group {
  margin-bottom: 1rem;
}

.form-group label {
  display: block;
  margin-bottom: 0.5rem;
  font-weight: 500;
}

.form-group input[type="text"],
.form-group input[type="url"] {
  width: 100%;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 16px; /* Safari minimum font size for inputs */
  box-sizing: border-box;
  transition: border-color 0.2s ease;
}

.form-group input[type="text"]:focus,
.form-group input[type="url"]:focus {
  border-color: #2196F3;
  outline: none;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  cursor: pointer;
  -webkit-tap-highlight-color: transparent;
}

.dialog-buttons {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  margin-top: 1.5rem;
  padding: 1rem;
  border-top: 1px solid #eee;
  background: white;
  position: relative;
  z-index: 1;
}

.dialog-buttons button {
  padding: 0.75rem 1.25rem;
  border-radius: 6px;
  border: none;
  cursor: pointer;
  font-size: 1rem;
  min-width: 80px;
  -webkit-tap-highlight-color: transparent;
  -webkit-appearance: none;
  appearance: none;
  transition: opacity 0.2s ease, background-color 0.2s ease;
}

.dialog-buttons button:focus {
  outline: 2px solid #2196F3;
  outline-offset: 2px;
}

.dialog-buttons button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.cancel-button {
  background: #f5f5f5;
  color: #333;
}

.cancel-button:hover {
  background: #e0e0e0;
}

.submit-button {
  background: #2196F3;
  color: white;
}

.submit-button:hover {
  background: #1976D2;
}

.color-customization {
  margin-top: 1.5rem;
  padding-top: 1.5rem;
  border-top: 1px solid #ddd;
}

.color-customization h3 {
  margin: 0 0 1rem;
  font-size: 1.1rem;
}

.color-input-wrapper {
  display: flex;
  gap: 0.5rem;
  align-items: center;
}

.color-picker {
  width: 44px;
  height: 44px;
  padding: 0;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  -webkit-appearance: none;
  appearance: none;
}

.color-text {
  flex: 1;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 16px;
  -webkit-appearance: none;
  appearance: none;
  transition: border-color 0.2s ease;
}

.color-text:focus {
  border-color: #2196F3;
  outline: none;
}

/* Mobile Safari specific fixes */
@supports (-webkit-touch-callout: none) {
  .dialog-overlay {
    padding: env(safe-area-inset-top) env(safe-area-inset-right) env(safe-area-inset-bottom) env(safe-area-inset-left);
  }
  
  .dialog-content {
    max-height: calc(100vh - env(safe-area-inset-top) - env(safe-area-inset-bottom) - 2rem);
  }
}

/* Mobile-specific styles */
@media (max-width: 768px) {
  .dialog-overlay {
    align-items: flex-end;
    padding: 0;
  }

  .dialog-content {
    width: 100%;
    max-width: 100%;
    margin: 0;
    border-radius: 20px 20px 0 0;
    max-height: 85vh;
  }

  .dialog-header {
    padding: 1.25rem;
  }

  .dialog-body {
    padding: 1.25rem;
  }

  .form-group input[type="text"],
  .form-group input[type="url"],
  .color-text {
    font-size: 16px;
    padding: 0.875rem;
  }

  .dialog-buttons {
    padding: 1.25rem;
    margin-top: 0;
  }

  .dialog-buttons button {
    flex: 1;
    padding: 1rem;
    font-size: 16px;
  }
}

@media (max-width: 480px) {
  .dialog-header {
    padding: 1rem;
  }

  .dialog-body {
    padding: 1rem;
  }

  .form-group input[type="text"],
  .form-group input[type="url"],
  .color-text {
    padding: 0.75rem;
  }

  .dialog-buttons {
    padding: 1rem;
  }

  .dialog-buttons button {
    padding: 0.875rem;
  }
}

.icon-input-wrapper {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.icon-search-buttons {
  display: flex;
  gap: 0.5rem;
}

.search-button {
  flex: 1;
  background: #f5f5f5;
  color: #333;
  padding: 0.5rem;
  border-radius: 4px;
  border: 1px solid #ddd;
  font-size: 0.9rem;
}

.search-button:hover {
  background: #e0e0e0;
}

.image-search-modal {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}

.image-search-modal .modal-content {
  background: white;
  border-radius: 12px;
  width: 90%;
  max-width: 800px;
  max-height: 90vh;
  display: flex;
  flex-direction: column;
  overflow: hidden;
}

.modal-header {
  padding: 1rem;
  border-bottom: 1px solid #eee;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.modal-header h3 {
  margin: 0;
  font-size: 1.1rem;
}

.search-input-wrapper {
  padding: 1rem;
  display: flex;
  gap: 0.5rem;
}

.search-input-wrapper input {
  flex: 1;
  padding: 0.75rem;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 16px;
}

.search-input-wrapper button {
  padding: 0.75rem 1.25rem;
  background: #2196F3;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}

.search-input-wrapper button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.image-grid {
  padding: 1rem;
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 1rem;
  overflow-y: auto;
  max-height: 60vh;
}

.image-item {
  aspect-ratio: 1;
  border-radius: 8px;
  overflow: hidden;
  cursor: pointer;
  transition: transform 0.2s ease;
}

.image-item:hover {
  transform: scale(1.05);
}

.image-item img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.pagination {
  padding: 1rem;
  display: flex;
  justify-content: center;
  gap: 0.5rem;
  border-top: 1px solid #eee;
}

.pagination button {
  padding: 0.5rem 1rem;
  background: #f5f5f5;
  color: #333;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.pagination button:hover {
  background: #e0e0e0;
}
</style>