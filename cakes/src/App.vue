vue
<template>
  <div class="container">
    <div class="cakes-list" v-infinite-scroll="fetchCakes" infinite-scroll-disabled="loading" infinite-scroll-distance="10">
      <div
        class="cake-item"
        v-for="cake in cakes"
        :key="cake.id"
      >
        <img
          v-if="cake.images.length > 0"
          :src="getImageSrc(cake.images[0].image)"
          alt="cake image"
          class="cake-image"
        />
        <h3>{{ cake.name }}</h3>
        <p>{{ cake.description }}</p>
      </div>
    </div>
    <div v-if="loading" class="loading">Загрузка...</div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      cakes: [],
      page: 1,
      loading: false,
      hasMore: true,
    };
  },
  mounted() {
    this.fetchCakes();
  },
  methods: {
    async fetchCakes() {
      if (this.loading || !this.hasMore) return;

      this.loading = true;

      try {
        const response = await fetch(`http://127.0.0.1:9000/api/cakes/?page=${this.page}`);
        const data = await response.json();

        if (data.results && data.results.length > 0) {
          this.cakes.push(...data.results);
          this.hasMore = !!data.next; // Проверяем, есть ли следующая страница
          this.page += 1; // Увеличиваем номер страницы
        } else {
          this.hasMore = false; // Если нет результатов, больше страниц нет
        }
      } catch (error) {
        console.error("Ошибка при загрузке тортов:", error);
      } finally {
        this.loading = false;
      }
    },
    getImageSrc(image) {
      return `data:image/png;base64,${image}`;
    },
    handleScroll() {
      const scrollHeight = document.documentElement.scrollHeight;
      const scrollTop = window.scrollY;
      const clientHeight = window.innerHeight;

      console.log('Scroll position:', scrollTop + clientHeight, 'Scroll height:', scrollHeight);

      if (scrollTop + clientHeight >= scrollHeight - 5) {
        console.log('Loading more cakes...');
        this.fetchCakes();
      }
    },
  },
};
</script>

<style scoped>
.container {
  padding: 20px;
  display: flex;
  justify-content: center; /* Центрируем содержимое по горизонтали */
}

.cakes-list {
  display: flex;
  flex-wrap: wrap;
  justify-content: center; /* Центрируем элементы в списке */
  max-width: 1200px; /* Максимальная ширина контейнера */
  width: 100%; /* Ширина на 100% */
}

.cake-item {
  width: 200px;
  margin: 10px;
  text-align: center;
}

.cake-image {
  width: 100%;
  height: 150px; /* Задайте фиксированную высоту */
  object-fit: cover; /* Обеспечивает обрезку изображения, чтобы оно соответствовало заданным пропорциям */
}

.loading {
  text-align: center;
  margin-top: 20px;
}
</style>