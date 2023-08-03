<script lang="ts">
import { ref, reactive, computed } from 'vue'

export default {
  name: 'Table',
  props: {
    data: {
      type: Array,
      required: true
    },
    columns: {
      type: Array,
      required: true
    },
    pagination: Boolean,
    searchable: Boolean,
    defaultSorting: Boolean,
    itemsPerPage: {
      type: Number,
      required: true
    }
  },
  setup(props) {
    const columnsData = ref([...props.columns])
    const originalData = ref([...props.data])

    const state = reactive({
      data: originalData.value?.slice(), // Create a copy of the original data as a reactive array
      field: '',
      dir: 'asc',
      currentPage: 1
    })

    const totalPages = computed(() => Math.ceil(state.data.length / props.itemsPerPage))

    /**
     * Instead of using a watchEffect to update the reactive state.data when sorting,
     * we can use a computed property for the sorted data.
     *  This will improve performance and simplify the code.
     */
    const sortedData = computed(() => {
      // If there is no field to sort, we can return state data
      if (!state.field) return state.data

      const column = columnsData.value.find((col) => col.key === state.field)
      if (!column || !column.sortable) return state.data
      // We only sort if there is a column and the column is sortable
      return column.sort(state.data, state.dir === 'asc')
    })

    const paginatedData = computed(() => {
      const start = (state.currentPage - 1) * props.itemsPerPage
      const end = start + props.itemsPerPage
      return sortedData.value.slice(start, end)
    })

    const prevPage = () => {
      if (state.currentPage > 1) {
        state.currentPage--
      }
    }

    const nextPage = () => {
      if (state.currentPage < totalPages.value) {
        state.currentPage++
      }
    }

    const sort = (column: any) => {
      if (!column.sortable) {
        return
      }

      state.field = column.key
      state.data = column.sort(state.data, state.dir === 'asc' ? true : false)
      state.dir = state.dir === 'asc' ? 'desc' : 'asc'
    }

    // Sort by default
    props.columns?.map((columnSort) => {
      if (columnSort.defaultSorting) sort(columnSort)
    })

    return { columnsData, sort, state, paginatedData, totalPages, prevPage, nextPage }
  }
}
</script>

<template>
  <div class="p-6 mx-auto max-w-4xl">
    <table
      class="w-full border-collapse border border-gray-300 shadow-sm rounded-lg overflow-hidden"
    >
      <thead class="bg-indigo-500 text-white">
        <tr>
          <th class="py-2 px-4 cursor-pointer" @click="sort(column)" v-for="column in columnsData">
            <div class="flex items-center">
              {{ column.name }}
              <vue-feather
                class="ml-auto"
                type="chevron-up"
                v-if="column.key === state.field && state.dir === 'asc'"
              ></vue-feather>
              <vue-feather
                class="ml-auto"
                v-if="column.key === state.field && state.dir === 'desc'"
                type="chevron-down"
              ></vue-feather>
            </div>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-if="state.data?.length === 0">
          <td colspan="{{columnsData.length}}">Sem resultados</td>
        </tr>
        <tr v-else v-for="(item, index) in paginatedData" :key="item.id || index">
          <td
            :key="index"
            class="py-3 px-4 border-b border-gray-300"
            v-for="(column, indexColumn) in columnsData"
            v-html="column.render(column, item)"
          ></td>
        </tr>
      </tbody>
    </table>

    <!-- Pagination -->
    <div class="flex justify-center mt-4">
      <nav class="pagination">
        <button
          @click="prevPage"
          :disabled="state.currentPage === 1"
          class="px-3 py-1 rounded-lg focus:outline-none focus:ring focus:border-indigo-300"
          :class="{ 'text-gray-500': state.currentPage === 1 }"
        >
          Previous
        </button>
        {{ state.currentPage + '/' + totalPages }}
        <button
          @click="nextPage"
          :disabled="state.currentPage === totalPages"
          class="px-3 py-1 rounded-lg focus:outline-none focus:ring focus:border-indigo-300 ml-2"
          :class="{ 'text-gray-500': state.currentPage === totalPages }"
        >
          Next
        </button>
      </nav>
    </div>
  </div>
</template>
