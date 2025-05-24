<template>
  <div class="image-gallery" @wheel.passive="handleScroll">
    <section v-for="(img, index) in images" :key="index" :ref="el => setImageRef(el, index)" class="image-section"
      :class="{ visible: visibleImages[index] }">
      <div v-if="index === images.length - 1" class="final-message">Bonne fête ❤️</div>
      <div v-else>
        <div class="background-image" :style="{ backgroundImage: `url(${img})` }"></div>
        <img :src="img" :alt="'Artwork ' + (index + 1)" />
        <div class="description" v-if="visibleImages[index]">
          {{ descriptions[img] }}
        </div>
      </div>
    </section>
  </div>
</template>

<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

// Liste des images
const images = [
  "/1.jpeg", "/2.jpeg", "/3.jpeg", "/4.jpeg",
  "/5.jpeg", "/6.jpeg", "/8.jpeg",
  "/9.jpeg", "/10.jpeg", "/11.jpeg", "/12.jpeg",
  "/13.jpeg", "/14.jpeg", "/15.jpeg", "/16.jpeg", "FINAL"
]

// ... autres imports et refs

const descriptions = {
  "/1.jpeg": "Vue de Marseille depuis la basilique Notre-Dame de la Garde ",
  "/2.jpeg": "Musée de le grotte Cosquer de Marseille",
  "/3.jpeg": "Peintures dans marseille",
  "/4.jpeg": "Domaine du Rayol",
  "/5.jpeg": "Domaine du Rayol",
  "/6.jpeg": "Domaine du Rayol",
  "/8.jpeg": "Plage Notre-Dame de Porquerolles",
  "/9.jpeg": "Vue du Lavandou depuis en haut ;)",
  "/10.jpeg": "La méditerranée depuis notre location",
  "/11.jpeg": "Vue sur la mer et le port",
  "/12.jpeg": "La mer depuis Porquerolles",
  "/13.jpeg": "Calanques à Porquerolles",
  "/14.jpeg": "Plage Notre-Dame de Porquerolles",
  "/15.jpeg": "Une jolie photo, toujours à Porquerolles",
  "/16.jpeg": "De retour au domaine du Rayol"
}


// Refs des sections
const imageRefs = ref([])
const visibleImages = ref(Array(images.length).fill(false))
const currentIndex = ref(0)
let scrolling = false

// Associer chaque élément DOM à son index
const setImageRef = (el, index) => {
  if (el) imageRefs.value[index] = el
}

// Scroll manuel par image
const scrollToSection = (index) => {
  if (index >= 0 && index < imageRefs.value.length) {
    imageRefs.value[index].scrollIntoView({ behavior: 'smooth' })
    currentIndex.value = index
  }
}

// Gestion du scroll molette
const handleScroll = (e) => {
  if (scrolling) return
  scrolling = true

  if (e.deltaY > 0 && currentIndex.value < images.length - 1) {
    scrollToSection(currentIndex.value + 1)
  } else if (e.deltaY < 0 && currentIndex.value > 0) {
    scrollToSection(currentIndex.value - 1)
  }

  setTimeout(() => {
    scrolling = false
  }, 1000)
}

// Observer pour déclencher les animations
onMounted(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        const index = imageRefs.value.indexOf(entry.target)
        if (entry.isIntersecting && index !== -1) {
          visibleImages.value[index] = true
        }
      })
    },
    { threshold: 0.5 }
  )

  imageRefs.value.forEach((el) => {
    if (el) observer.observe(el)
  })

  scrollToSection(0)
})

// Nettoyage
onBeforeUnmount(() => {
  window.removeEventListener('wheel', handleScroll)
})
</script>

<style scoped>
.image-gallery {
  height: 100vh;
  width: 100vw;
  overflow-y: hidden;
  /* Désactive le scroll natif */
  scroll-snap-type: y mandatory;
  position: relative;
}

.image-section {
  scroll-snap-align: start;
  position: relative;
  height: 100vh;
  width: 100vw;
  display: flex;
  align-items: center;
  justify-content: center;
  overflow: hidden;
  opacity: 0;
  filter: blur(12px);
  transform: scale(0.95);
  transition: all 1.2s ease;
}

.image-section.visible {
  opacity: 1;
  filter: blur(0);
  transform: scale(1);
}

.image-section img {
  position: relative;
  z-index: 2;
  width: 100%;
  height: 100%;
  object-fit: contain;
  object-position: center;
}

.background-image {
  position: absolute;
  inset: 0;
  background-size: cover;
  background-position: center;
  filter: blur(20px) brightness(0.6);
  transform: scale(1.1);
  z-index: 1;
  transition: opacity 1.2s ease, transform 1.2s ease;
  opacity: 0;
}

.image-section.visible .background-image {
  opacity: 1;
  transform: scale(1);
}

.description {
  position: absolute;
  bottom: 2rem;
  left: 50%;
  transform: translateX(-50%) translateY(20px);
  background-color: rgba(0, 0, 0, 0.6);
  color: white;
  padding: 0.75rem 1.25rem;
  border-radius: 1rem;
  font-size: 1.1rem;
  font-weight: 500;
  text-align: center;
  max-width: 80%;
  opacity: 0;
  transition: opacity 1s ease, transform 1s ease;
  z-index: 3;
  backdrop-filter: blur(6px);
}

.image-section.visible .description {
  opacity: 1;
  transform: translateX(-50%) translateY(0);
}



.final-section {
  display: flex;
  align-items: center;
  justify-content: center;
  background: radial-gradient(circle, #ffdee9, #b5fffc);
  position: relative;
  height: 100vh;
  width: 100vw;
  opacity: 0;
  transform: scale(0.95);
  transition: all 1.2s ease;
  scroll-snap-align: start;
}

.image-section.visible+.final-section {
  opacity: 1;
  transform: scale(1);
}

.final-message {
  font-size: 3rem;
  font-weight: bold;
  color: #333;
  text-shadow: 2px 2px 10px white;
  animation: pop-in 1.2s ease forwards;
  padding: 1rem 2rem;
  border-radius: 1rem;
  background: rgba(255, 255, 255, 0.8);
  backdrop-filter: blur(6px);
}

@keyframes pop-in {
  from {
    transform: scale(0.8);
    opacity: 0;
  }

  to {
    transform: scale(1);
    opacity: 1;
  }
}
</style>
