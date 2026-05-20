// Aplikasi Prakiraan Cuaca Jakarta
<template>
  <div class="container">
    <h1> Prakiraan Cuaca Jakarta</h1>
    <p class="subtitle">Data suhu per-jam dari Open-Meteo API</p>

    <div v-if="loading" class="loading">
      Mengambil data cuaca...
    </div>

    <div v-if="error" class="error">
      {{ error }}
    </div>

    <div v-if="!loading && !error">
      <p class="info"> Menampilkan 10 data pertama</p>
      <table>
        <thead>
          <tr>
            <th>No</th>
            <th>Suhu (°C)</th>
            <th>Waktu</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in cuacaData" :key="index">
            <td>{{ index + 1 }}</td>
            <td>{{ formatWaktu(item.time) }} </td>
            <td class="suhu" :class="getSuhuClass(item.temperature)">{{ item.temperature }} °C</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { defineComponent, ref, onMounted } from 'vue'

interface CuacaItem {
  time: string;
  temperature: number;
}


export default defineComponent({
  name: 'App',
  setup() {
    const cuacaData = ref<CuacaItem[]>([]);
    const loading = ref(true);
    const error = ref('')
    // Fungsi untuk mengambil data cuaca dari API
    const fetchCuaca = async () => {
      try {
        const url = 'https://api.open-meteo.com/v1/forecast?latitude=-6.2&longitude=106.8&hourly=temperature_2m'
        const response = await fetch(url);
        const data = await response.json();

        // Mengambil data waktu dan suhu dari response
        const times: string[] = data.hourly.time;
        const temperatures: number[] = data.hourly.temperature_2m;

        //Mengambil berapa data yang akan ditampilkan
        for (let i = 0; i < 100; i++) {
          cuacaData.value.push({
            time: times[i] || '',
            temperature: temperatures[i] || 0
          })
        }
      } catch (err) {
        error.value = 'Gagal mengambil data cuaca. Periksa koneksi internet Anda.';
      } finally {
        loading.value = false;
      }
    }
    //
    const formatWaktu = (time: string): string => {
      const data = new Date(time);
      const hari = data.toLocaleDateString('id-ID', {
        weekday: 'short',
        day: '2-digit',
        month: 'short',
      })
      const jam = data.toLocaleTimeString('id-ID', {
        hour: '2-digit',
        minute: '2-digit',
      })
      return `${hari} ${jam}`;
    }
    // Fungsi untuk menentukan kelas warna berdasarkan suhu
    const getSuhuClass = (temp: number): string => {
      if (temp >= 35) return 'sangat-panas'
      if (temp >= 30) return 'panas'
      if (temp >= 25) return 'hangat'
      return 'sejuk'
    }

    onMounted(() => {
      fetchCuaca();
    })

    return {
      cuacaData,
      loading,
      error,
      formatWaktu,
      getSuhuClass
    }
  }
})
</script>

<style>
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body{
  font-family: Arial, Helvetica, sans-serif;
  background-color: #f0f4f8
}
.container {
  max-width: 600px;
  margin: 40px auto;
  padding: 20px;
  background: white;
  border-radius: 10px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.1);
}

h1 {
  font-size: 22px;
  color : #1a73e8;
  margin-bottom: 5px;
}
.subtitle {
  color: #666;
  margin-bottom: 20px;
  font-size: 14px;
}
.info {
  font-size: 13px;
  color: #888;
  margin-bottom: 10px;
}

.loading {
  text-align: center;
  padding: 30px;
  color: #1a73e8;
}

.error {
  text-align: center;
  padding: 20px;
  color: #e53935;
}

table {
  width: 100%;
  border-collapse: collapse;
}

thead {
  background-color: #1a73e8;
  color: white;
}

th {
  padding: 10px 12px;
  text-align: left;
  font-size: 14px;
}

td {
  padding: 10px 12px;
  font-size: 14px;
  border-bottom: 1px solid #eee;
  color: #212121;
}

tr:hover {
  background-color: #f5f9ff;
}

.suhu {
  font-weight: bold;
} 

/* Kelas warna berdasarkan suhu */
.sangat-panas {
  color:  #b71c1c;
}
.panas {
  color: #bf360c;
}
.hangat {
  color: #e65100;
}
.sejuk {
  color: #0d47a1;
}
</style>