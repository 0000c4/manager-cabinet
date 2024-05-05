<template>
  <FixedLeftColumn>
    <template v-slot:fixed>
      <div class="block SearchAndDate">
        <InputDate :range="true" :modelValue="date"
          @update:modelValue="(event) => { date = event; addParamsToLocation() }" />
        <Search :searchQuery="search" selectDisplayValue="icon" :searchTypes="searchTypes" :searchType="searchType"
          @update:searchQuery="(event) => { search = event; addParamsToLocation() }"
          @update:searchType="(event) => { searchType = event; addParamsToLocation() }" />
      </div>
      <div class="block yearContainer">
        <Select :items="[{ value: '2023' }, { value: '2022' }, { value: '2021' }]" :modelValue="year"
          @update:modelValue="(event) => { year = event; addParamsToLocation() }" />
        <div class="yearContainerButtons">
          <Toggle width="full">Показать</Toggle>
          <Toggle width="full" color="gray">Сбросить</Toggle>
        </div>
      </div>

    </template>
    <template v-slot:default>
      <div v-if="orders.length > 0" class="ItemCardContainer">
        
        <StatusCard v-for="order in orders" class="ItemCard" status="success" :key="order.id">
          №: {{ order.id }}
        </StatusCard>
        
      </div>
      <Empty v-else :withoutImage="true">Заказов еще нет</Empty>
    </template>

  </FixedLeftColumn>
</template>

<script setup lang="ts">

import { ref } from 'vue'

import { FixedLeftColumn } from "~/shared/ui/templates/index";
import { Search } from "~/shared/ui/search/index";
import { InputDate } from "~/shared/ui/inputs/input-date";
import { Select } from "~/shared/ui/select";
import { StatusCard } from '~/shared/ui/status-card';
import { Toggle } from "~/shared/ui/toggle";
import { Empty } from '~/shared/ui/empty';
const router = useRouter()
const searchTypes = {
  'order_number': { title: 'Номер заказа' },
  'psid': { title: 'Номер фотосессии' },
  'client_id': { title: 'Клиент ID' },
  'phone': { title: 'Телефон' },
  'email': { title: 'Email' },
  'payer': { title: 'Плательщик, ребенок' },
}

const orders = ref([]);

const date = ref([new Date(), new Date()])
const year = ref('2023')
const search = ref('')
const searchType = ref('order_number')


onMounted(async () => {
  const res = await fetch('/lk/method/orders.getTest')
  const { response } = await res.json();
  orders.value = response.data.orders
})

function addParamsToLocation() {
  router.push({
    query: {
      search_type: searchType.value,
      search_value: search.value === '' ? 'null' : search.value,
      date_start: formatDateToQuery(date.value[0]),
      date_finish: formatDateToQuery(date.value[1]),
      year: year.value
    },
  })

}

function formatDateToQuery(date: Date) {
  const mounth = String(date.getMonth() + 1).length === 1 ? '0' + String(date.getMonth() + 1) : String(date.getMonth() + 1)
  const day = String(date.getDate()).length === 1 ? '0' + String(date.getDate()) : String(date.getDate())

  return String(date.getFullYear()) + mounth + day
}
</script>

<style lang="scss">
@import "~/shared/assets/styles/components/layout";
@import "~/shared/assets/styles/components/content/content";

.SearchAndDate {
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 15px
}

.yearContainer {
  margin-top: 15px;
  padding: 16px
}

.yearContainerButtons {
  margin-top: 5px;
  display: flex;
  gap: 5px
}

.ItemCardContainer {
  height: 80vh;
  overflow: auto;

  .ItemCard {
    padding: 16px;
    margin-bottom: 15px;
  }
}
</style>
