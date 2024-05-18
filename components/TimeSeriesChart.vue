<script lang="ts" setup>
const props: any = defineProps({
    labels: null,
    dataset: null
})

import {
    Chart as ChartJS,
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
    Legend
} from 'chart.js'
import { Line } from 'vue-chartjs'

ChartJS.register(
    CategoryScale,
    LinearScale,
    PointElement,
    LineElement,
    Title,
    Tooltip,
)

const chartData = ref({
    labels: props.labels,
    datasets: [
        {
            label: 'Penjualan Harian',
            backgroundColor: '#f87979',
            data: props.dataset,
        },
    ],
})

const chartOptions = ref({
    responsive: true,
    maintainAspectRatio: false,
    scales: {
      x: {
        display: true,
        title: {
          display: true
        }
      },
      y: {
        display: true,
        title: {
          display: true,
          text: 'Penjualan'
        },
        suggestedMin: 0,
        suggestedMax: (props.dataset[-1] * 1.8)
      }
    }
})

</script>

<template>
    <div class="h-[40vh]">
        <Line :data="chartData" :options="chartOptions" />
    </div>
</template>