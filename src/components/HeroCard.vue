<script setup>
import { ref, computed, onMounted, onUnmounted, watchEffect } from 'vue';
import { getGenresMovie } from '../services/api-service';
import { useRouter } from 'vue-router';

const props = defineProps({
  movies: {
    type: Array,
    required: true,
    default: () => [],
  }
});

const router = useRouter();

const currentIndex = ref(0);
const progress = ref(0);
const isHovered = ref(false);
const genreMovieList = ref([]);

const chosenFilm = computed(() => props.movies[currentIndex.value]);

watchEffect(async () => {
  const dataGenres = await getGenresMovie();
  genreMovieList.value = dataGenres.data.genres;
});

const genresMovie = computed(() => {
  return chosenFilm.value?.genre_ids?.map(genreId => {
    const genre = genreMovieList.value?.find(genre => genre.id === genreId);
    return genre ? genre.name : '';
  }).join(', ');
});

const nextFilm = () => {
  currentIndex.value = (currentIndex.value + 1) % props.movies.length;
  progress.value = 0;
};

let intervalId = null;
let progressIntervalId = null;

const startAutoSlide = () => {
  intervalId = setInterval(nextFilm, 5000);
  progressIntervalId = setInterval(() => {
    if (progress.value < 100) {
      progress.value += 2;
    }
  }, 100);
};

const stopAutoSlide = () => {
  clearInterval(intervalId);
  clearInterval(progressIntervalId);
};

onMounted(() => {
  startAutoSlide();
});

onUnmounted(() => {
  stopAutoSlide();
});

const handleMouseEnter = () => {
  stopAutoSlide();
  isHovered.value = true;
};

const handleMouseLeave = () => {
  startAutoSlide();
  isHovered.value = false;
  progress.value = 0;
};

const goToDetailMovie = (id) => {
  router.push({ name: 'Detail', params: { id } });
}

defineExpose({
  currentIndex,
  progress,
})
</script>

<template>
  <button @click="goToDetailMovie(chosenFilm?.id)"
    class="hero-card w-full max-h-fit my-0 mx-auto flex flex-col cursor-pointer" @mouseenter="handleMouseEnter"
    @mouseleave="handleMouseLeave">
    <transition name="slide-fade" mode="out-in">
      <div class="w-full flex flex-col min-[1105px]:flex-row max-[1105px]:items-center min-[1105px]:justify-center xl:h-[50dvh] gap-8 xl:px-72 relative" :key="chosenFilm?.id">
        <div v-if="chosenFilm?.backdrop_path"
          class="image-backdrop hidden xl:block bg-[var(--black)] w-3/4 h-full absolute right-20 scale-100 grayscale-100" :style="{
            backgroundImage: `linear-gradient(to right, transparent, rgba(0, 0, 0, 0.2), transparent), url(https://image.tmdb.org/t/p/w500${chosenFilm?.backdrop_path})`,
            backgroundSize: 'contain',
            backgroundPosition: 'center',
            backgroundRepeat: 'no-repeat',
            opacity: 0.9,
          }">
          <div
            class="absolute top-0 left-0 w-full h-full bg-gradient-to-r from-[var(--black)] via-[var(--black)]/50 to-[var(--black)] from-8% via-50% to-76%">
          </div>
        </div>
        <div v-if="chosenFilm" class="relative flex justify-end items-center min-[1105px]:basis-1/4">
          <img :src="`https://image.tmdb.org/t/p/w500${chosenFilm?.poster_path}`"
            class="image-poster w-[300px] h-full object-cover" />
        </div>
        <div class="hidden min-[1280px]:flex min-[1105px]:basis-3/4 max-w-3xl flex-col md:justify-end z-10 py-4" style="gap: 1rem;">
          <div v-if="chosenFilm" class="flex flex-col items-center min-[1105px]:items-start" style="gap: 1rem;">
            <h2 v-if="chosenFilm?.original_title !== chosenFilm?.title"
              class="movie-title flex flex-col gap-2 font-bold text-center min-[1105px]:text-left text-5xl text-[var(--green)]">
              <span>{{ chosenFilm?.title }} ({{ chosenFilm?.release_date.split("-")[0] }})</span>
              <span class="text-2xl font-semibold">Original Title: {{ chosenFilm?.original_title }}</span>
            </h2>
            <h2 v-else class="w-full movie-title font-bold text-left text-5xl text-[var(--green)]">
              {{ chosenFilm?.title }} <span>({{ chosenFilm?.release_date.split("-")[0] }})</span>
            </h2>
            <p class="movie-score text-2xl pt-2">{{ genresMovie }}</p>
          </div>
          <!-- <p class="hidden min-[1105px]:block movie-overview text-justify hyphens-auto text-pretty line-clamp-4 text-xl">
            {{ chosenFilm?.overview }}
          </p> -->
        </div>
      </div>
    </transition>
    <div
      class="progress-bar h-1 bg-gradient-to-r from-[var(--black)] from-50% via-[var(--dark-green)] via-90% to-[var(--green)] to-100% rounded-r-2xl"
      :style="{ width: progress + '%', opacity: isHovered || progress <= 5 || progress >= 95 ? 0 : 1 }"></div>
  </button>
</template>

<style scoped>
.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.5s ease-in-out;
}

.slide-fade-enter-from,
.slide-fade-leave-to {
  opacity: 0;
}

.slide-fade-enter-to,
.slide-fade-leave-from {
  opacity: 1;
}

.progress-bar {
  transition: width 0.1s linear, opacity 300ms ease-out;
}
</style>