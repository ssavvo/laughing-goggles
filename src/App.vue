<script setup lang="ts">
import { VNode, h, ref } from 'vue';
import StackWithDividers from './components/StackWithDividers.vue'
import MyBadge from './components/MyBadge.vue'
import EndPeriod from './components/EndPeriod.vue';
import clients from './data.json'

import { FlexRender, createColumnHelper, getCoreRowModel, useVueTable, SortingState, getSortedRowModel, getFilteredRowModel } from '@tanstack/vue-table'

type Status = {
  status: 'not started' | 'active' | 'finished';
  dayCount: number;
}

export type Client = {
  "o_id": number;
  "client_name": string;
  "diets": string[];
  "tariff": string[];
  "address": string;
  "phone": string;
  "statuses": Status[],
  "discount": number;
  "order_sum": number;
  "order_payed": string;
  "pay_status": string;
  "courier_comment": string;
  "inner_comment": string;
}
const columnHelper = createColumnHelper<Client>();

const today = new Date().setHours(0, 0, 0, 0);

const tableData = clients.map((client) => {
  const newDates = client.dates.map(({ start_date, end_date }) => {
    const startDate = new Date(start_date).setHours(0, 0, 0, 0);
    const endDate = new Date(end_date).setHours(0, 0, 0, 0);
    const status = today < startDate ? 'not started' : today <= endDate ? 'active' : 'finished';
    const dayCount = Math.trunc(Math.abs((status === 'not started' ? startDate - today : today - endDate) / (1000 * 60 * 60 * 24)))
    return {
      status,
      dayCount
    }
  })
  return { ...client, 'statuses': newDates };
})

const statusSort = (a: Status, b: Status) => {
  if (a.status === b.status) {
    return a.dayCount - b.dayCount;
  }
  switch (a.status) {
    case 'not started':
      return -1;
    case 'active':
      return b.status === 'not started' ? 1 : -1;
    case 'finished':
      return 1;
  }
}

const columns = [
  columnHelper.accessor('o_id', {
    enableColumnFilter: false
  }),
  columnHelper.accessor('client_name', {
    enableColumnFilter: false
  }),
  columnHelper.accessor('diets', {
    enableColumnFilter: false,
    cell: ({ getValue }) => {
      //@ts-ignore
      return h(StackWithDividers, {
        list: getValue().map(v => ({ text: v })),
      }, ({ text }: { text: string }) => h(MyBadge, { variant: 'diet' }, () => text)
      )
    }
  }),
  columnHelper.accessor('tariff', {
    enableColumnFilter: false,
    cell: ({ getValue }) => {
      //@ts-ignore
      return h(StackWithDividers, {
        list: getValue().map(v => ({ text: v }))
      }, ({ text }: { text: string }) => h(MyBadge, { variant: 'tariff' }, () => text)
      )
    }
  }),
  columnHelper.accessor('pay_status', {
    enableColumnFilter: false
  }),
  columnHelper.accessor('address', {
    enableColumnFilter: false
  }),
  columnHelper.accessor('statuses', {
    cell: ({ getValue }) => {
      //@ts-ignore
      return h(StackWithDividers, { list: getValue() }, (slotProps: any): VNode => {
        return h(EndPeriod, slotProps)
      });
    },
    sortingFn: (rowA, rowB, colId) => {
      const a: Status[] = rowA.getValue(colId);
      const b: Status[] = rowB.getValue(colId);
      const ascA = a.sort(statusSort)[0];
      const ascB = b.sort(statusSort)[0];
      return statusSort(ascA, ascB)
    }
  })
]

const sorting = ref<SortingState>([])

const table = useVueTable({
  data: tableData as Client[],
  columns,
  state: {
    get sorting() {
      return sorting.value
    },
  },
  onSortingChange: updaterOrValue => {
    sorting.value =
      typeof updaterOrValue === 'function'
        ? updaterOrValue(sorting.value)
        : updaterOrValue
  },
  getCoreRowModel: getCoreRowModel(),
  getSortedRowModel: getSortedRowModel(),
  getFilteredRowModel: getFilteredRowModel(),
  debugTable: true,
})
</script>

<template>
  <div class="overflow-x-auto">
    <table class="table">
      <thead>
        <tr v-for="headerGroup in table.getHeaderGroups()" :key="headerGroup.id">
          <th v-for="header in headerGroup.headers" :key="header.id" :colSpan="header.colSpan"
            :class="header.column.getCanSort() ? 'cursor-pointer select-none' : ''"
            @click="header.column.getToggleSortingHandler()?.($event)">
            <template v-if="!header.isPlaceholder">
              <FlexRender :render="header.column.columnDef.header" :props="header.getContext()" />

              {{
                { asc: ' 🔼', desc: ' 🔽' }[
                  header.column.getIsSorted() as string
                ]
              }}
            </template>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="row in table.getRowModel().rows" :key="row.id">
          <td v-for="cell in row.getVisibleCells()" :key="cell.id">
            <FlexRender :render="cell.column.columnDef.cell" :props="cell.getContext()" />
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template> 

<style scoped></style>
