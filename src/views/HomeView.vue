<script setup>
import { onMounted, ref } from 'vue'
import { storeToRefs } from 'pinia'
import { useCitiesStore } from '@/stores/cities.js'
import Block from '@/components/Block/Block.vue'
import Loader from '@/components/Base/Loader.vue'
import Button from '@/components/Base/Button.vue'
import Modal from '@/components/Base/Modal.vue'

const store = useCitiesStore()
const { cities } = storeToRefs(store)
const { favoriteCitiesId } = storeToRefs(store)

const modalEl = ref()
const loading = ref(true)
const loadingBtn = ref(false)

const onShowModal = () => {
  modalEl.value.open()
}

const fetchData = async () => {
  if (cities.value.length) {
    loading.value = false
    return
  }
  try {
    await store.getIpUserFromIpAddress()
    await store.getWeatherOneCallCityObj()
  } catch (err) {
    console.error(err)
  } finally {
    loading.value = false
  }
}

const onAddCity = () => {
  loadingBtn.value = true
  try {
    const response = store.addCity()
    if (!response) {
      onShowModal()
    }
  } catch (err) {
    console.error(err)
  } finally {
    loadingBtn.value = false
  }
}

onMounted(() => {
  fetchData()
})
</script>

<template>
  <main class="home-page">
    <div class="container">
      <Loader v-if="loading" />
      <template v-else>
        <div v-if="cities.length" class="home-page__items">
          <Block
            v-for="city of cities"
            :key="city.id"
            :city="city"
            :is-fav="favoriteCitiesId.some((item) => item.idRes === city.idRes)"
          />
        </div>
        <div v-else class="home-page__empty">{{ $t('add_weather') }}</div>
        <div class="home-page__btn">
          <Button @click="onAddCity" :loading="loadingBtn">{{ $t('add_city') }}</Button>
        </div>
      </template>
      <Modal ref="modalEl">
        <div class="home-page__modal">{{ $t('maximum_cities_5') }}</div>
      </Modal>
    </div>
  </main>
</template>

<style lang="scss" scoped>
.home-page {
  position: relative;
  padding-bottom: 5rem;
  flex: 1 1 auto;

  &__items {
    margin-bottom: 4rem;

    .block {
      margin-bottom: 4rem;

      &:last-child {
        margin-bottom: 0;
      }
    }
  }

  &__btn {
    display: flex;
    justify-content: center;
  }

  &__empty {
    text-align: center;
    font-size: 2rem;
    font-weight: 500;
    margin-bottom: 3rem;
  }

  &__modal {
    font-size: 2rem;
    text-align: center;
  }
}
</style>
