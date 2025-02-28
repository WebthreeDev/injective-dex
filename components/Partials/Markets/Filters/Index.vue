<script lang="ts" setup>
import { PropType } from 'vue'
import { MarketType } from '@injectivelabs/sdk-ui-ts'
import { MarketCategoryType, MarketQuoteType } from '@/types'

const route = useRoute()
const router = useRouter()

const props = defineProps({
  showLowVolumeMarkets: Boolean,

  search: {
    type: String,
    required: true
  },

  activeType: {
    type: String,
    required: true
  },

  activeQuote: {
    type: String as PropType<MarketQuoteType>,
    required: true
  },

  activeCategory: {
    type: String as PropType<MarketCategoryType>,
    required: true
  }
})

const emit = defineEmits<{
  (e: 'update:activeCategory', state: MarketCategoryType): void
  (e: 'update:search', state: string): void
  (e: 'update:activeQuote', state: MarketQuoteType): void
  (e: 'update:activeType', state: string): void
  (e: 'update:showLowVolumeMarkets', state: boolean): void
}>()

const FilterList = {
  [MarketType.Favorite]: MarketType.Favorite,
  All: '',
  [MarketType.Spot]: MarketType.Spot,
  [MarketType.Derivative]: MarketType.Derivative
}
const marketCategoryTypes = Object.entries(MarketCategoryType).map(
  ([key, value]) => ({
    key: `market-category-type-${value}`,
    label: key,
    type: MarketCategoryType[key as keyof typeof MarketCategoryType]
  })
)
const quoteOptions = Object.entries(MarketQuoteType).map(([key, value]) => ({
  display: key,
  value
}))

const activeQuoteValue = computed({
  get: (): MarketQuoteType => props.activeQuote,
  set: (value: MarketQuoteType) => {
    emit('update:activeQuote', value)
  }
})

const showLowVolumeMarketsValue = computed({
  get: (): boolean => props.showLowVolumeMarkets,
  set: (type: boolean) => {
    emit('update:showLowVolumeMarkets', type)
  }
})

const activeFilterType = computed({
  get: (): string => props.activeType,
  set: (type: string) => {
    emit('update:activeType', type)

    if (type === props.activeType) {
      return
    }

    if (!type || type === '') {
      clearRouteQueryParam('type')
    } else {
      fillRouteQueryParams({ type: type.toLowerCase() })
    }
  }
})

function handleCategoryChange(category: string) {
  emit('update:activeCategory', category as MarketCategoryType)

  if (category === props.activeCategory) {
    return
  }

  if (!category || category === MarketCategoryType.All) {
    clearRouteQueryParam('category')
  } else {
    fillRouteQueryParams({ category: category.toLowerCase() })
  }
}

function handleSearchedEvent(search: string) {
  emit('update:search', search)
}

function handleQuoteChange(quote: string) {
  if (quote === props.activeQuote) {
    return
  }

  if (!quote || quote === MarketQuoteType.All) {
    clearRouteQueryParam('quote')
  } else {
    fillRouteQueryParams({ quote: quote.toLowerCase() })
  }
}

function clearRouteQueryParam(key: string) {
  const { query } = route
  const queryClone = { ...query }

  delete queryClone[key]

  router.replace({ query: queryClone })
}

function fillRouteQueryParams(params: Record<string, string>) {
  router.replace({
    path: route.path,
    query: {
      ...route.query,
      ...params
    }
  })
}
</script>

<template>
  <div>
    <CommonTabMenu>
      <AppSelectButton
        v-for="filterType in Object.values(FilterList)"
        :key="`market-tabs-${filterType}`"
        v-model="activeFilterType"
        :value="filterType"
      >
        <template #default="{ active }">
          <CommonTabMenuItem :active="active">
            <span
              v-if="filterType === FilterList[MarketType.Favorite]"
              class="flex items-center"
            >
              <BaseIcon name="star-border" class="mr-1" />
              <span>{{ $t('trade.favorites') }}</span>
            </span>

            <span v-else-if="filterType === FilterList.All">
              {{ $t('trade.allMarkets') }}
            </span>

            <span v-else-if="filterType === FilterList[MarketType.Spot]">
              {{ $t('trade.spots') }}
            </span>

            <span v-else-if="filterType === FilterList[MarketType.Derivative]">
              {{ $t('trade.futures') }}
            </span>
          </CommonTabMenuItem>
        </template>
      </AppSelectButton>

      <template #actions>
        <AppSearch
          name="search"
          class="sm:w-auto md:w-3xs"
          input-classes="placeholder-white"
          dense
          transparent-bg
          data-cy="markets-search-input"
          :placeholder="$t('trade.search_markets')"
          show-prefix
          :model-value="search"
          @update:modelValue="handleSearchedEvent"
        />
      </template>
    </CommonTabMenu>

    <div
      class="flex flex-col sm:flex-row items-start sm:items-center gap-4 sm:gap-0 justify-between mt-6 mb-2 px-3 sm:px-0"
    >
      <div class="flex items-center gap-2">
        <PartialsMarketsFiltersCategorySelector
          v-for="marketCategoryType in marketCategoryTypes"
          :key="marketCategoryType.key"
          :type="marketCategoryType.type"
          :active="activeCategory === marketCategoryType.type"
          :data-cy="`market-category-${marketCategoryType.key}-button`"
          @click="handleCategoryChange"
        >
          {{ marketCategoryType.label }}
        </PartialsMarketsFiltersCategorySelector>
      </div>

      <div class="flex items-center">
        <AppSelect
          v-model="activeQuoteValue"
          :options="quoteOptions"
          class="self-end"
          @update:modelValue="handleQuoteChange"
        >
          <template #prefix>
            <span class="text-xs text-gray-300 uppercase">
              {{ $t('markets.quote') }}
            </span>
          </template>

          <template #default="{ selected }">
            <span v-if="selected" class="text-xs text-blue-500 uppercase">
              {{ selected.display }}
            </span>
          </template>

          <template #option="{ option }">
            <span class="text-xs uppercase text-white">
              {{ option.display }}
            </span>
          </template>
        </AppSelect>

        <AppCheckbox v-model="showLowVolumeMarketsValue" class="ml-4" sm>
          {{ $t('markets.showLowVol') }}
        </AppCheckbox>
      </div>
    </div>
  </div>
</template>
