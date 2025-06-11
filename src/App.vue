<template lang="pug">
.dashboard
  .header
    h1 Feyezan Rapor Dashboard
    .file-upload
      input(type="file" @change="handleFileUpload" accept=".xlsx")
  .charts(v-if="hasData")
    .chart-container
      h3 Sepete Eklemelerin Dönüşüm Değeri
      LineChart(:chartData="basketConversionData" :options="chartOptions")
    .chart-container
      h3 Alışveriş Dönüşüm Değeri (Ciro)
      LineChart(:chartData="revenueData" :options="chartOptions")
    .chart-container
      h3 ROAS (Reklam Harcaması Getirisi)
      LineChart(:chartData="roasData" :options="chartOptions")
    .chart-container
      h3 Toplam Harcanan Tutar
      LineChart(:chartData="spentAmountData" :options="chartOptions")
</template>

<script setup>
import { ref, computed } from 'vue'
import { LineChart } from 'vue-chart-3'
import { Chart, registerables } from 'chart.js'
import * as XLSX from 'xlsx'

Chart.register(...registerables)

const hasData = ref(false)
const monthlyData = ref({
  basketConversion: Array(12).fill(0),
  revenue: Array(12).fill(0),
  roas: Array(12).fill(0),
  spentAmount: Array(12).fill(0)
})

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  scales: {
    y: {
      beginAtZero: true
    }
  }
}

const months = [
  'Ocak', 'Şubat', 'Mart', 'Nisan', 'Mayıs', 'Haziran',
  'Temmuz', 'Ağustos', 'Eylül', 'Ekim', 'Kasım', 'Aralık'
]

const createChartData = (label, data) => ({
  labels: months,
  datasets: [{
    label,
    data,
    borderColor: '#' + Math.floor(Math.random() * 16777215).toString(16),
    tension: 0.1
  }]
})

const basketConversionData = computed(() =>
  createChartData('Sepete Eklemelerin Dönüşüm Değeri', monthlyData.value.basketConversion)
)

const revenueData = computed(() =>
  createChartData('Alışveriş Dönüşüm Değeri', monthlyData.value.revenue)
)

const roasData = computed(() =>
  createChartData('ROAS', monthlyData.value.roas)
)

const spentAmountData = computed(() =>
  createChartData('Harcanan Tutar', monthlyData.value.spentAmount)
)

const handleFileUpload = (event) => {
  const file = event.target.files[0]
  const reader = new FileReader()

  reader.onload = (e) => {
    const data = new Uint8Array(e.target.result)
    const workbook = XLSX.read(data, { type: 'array' })
    const firstSheet = workbook.Sheets[workbook.SheetNames[0]]
    const jsonData = XLSX.utils.sheet_to_json(firstSheet)

    if (jsonData.length > 0) {
      // Rapor ayını belirle
      const reportDate = new Date(jsonData[0]['Rapor Başlangıcı'])
      const monthIndex = reportDate.getMonth()

      // Verileri topla
      const totalBasketConversion = jsonData.reduce((sum, row) =>
        sum + (row['Sepete Eklemelerin Dönüşüm Değeri'] || 0), 0)

      const totalRevenue = jsonData.reduce((sum, row) =>
        sum + (row['Alışveriş dönüşüm değeri'] || 0), 0)

      const totalRoas = jsonData.reduce((sum, row) =>
        sum + (row['Alışveriş Reklam Harcamasının Getirisi'] || 0), 0)

      const averageRoas = totalRoas / jsonData.length

      const totalSpentAmount = jsonData.reduce((sum, row) =>
        sum + (row['Harcanan Tutar (TRY)'] || 0), 0)

      // Verileri aylık dizilere ekle
      monthlyData.value.basketConversion[monthIndex] = totalBasketConversion
      monthlyData.value.revenue[monthIndex] = totalRevenue
      monthlyData.value.roas[monthIndex] = averageRoas
      monthlyData.value.spentAmount[monthIndex] = totalSpentAmount

      hasData.value = true
    }
  }

  reader.readAsArrayBuffer(file)
}
</script>

<style lang="scss">
.dashboard {
  max-width: 1400px;
  margin: 0 auto;
  padding: 20px;

  .header {
    text-align: center;
    margin-bottom: 30px;

    h1 {
      color: #2c3e50;
      margin-bottom: 20px;
    }

    .file-upload {
      input {
        padding: 10px;
        border: 2px dashed #2c3e50;
        border-radius: 5px;
        cursor: pointer;

        &:hover {
          border-color: #42b983;
        }
      }
    }
  }

  .charts {
    display: grid;
    gap: 20px;

    // Telefon ekranı için (< 768px)
    grid-template-columns: 1fr;

    // Tablet dikey ekran için (>= 768px)
    @media (min-width: 768px) {
      grid-template-columns: repeat(2, 1fr);
    }

    // Masaüstü ve tablet yatay ekran için (>= 1024px)
    @media (min-width: 1024px) {
      grid-template-columns: repeat(3, 1fr);
    }

    .chart-container {
      background: white;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);

      h3 {
        text-align: center;
        color: #2c3e50;
        margin-bottom: 15px;
      }

      canvas {
        height: 300px !important;
      }
    }
  }
}

body {
  background: #f5f5f5;
  margin: 0;
  font-family: Arial, sans-serif;
}
</style>
