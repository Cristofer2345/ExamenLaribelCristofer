<script setup lang="ts">
import { defineComponent, onMounted } from 'vue'
import { Capacitor } from '@capacitor/core'
import { ref } from 'vue'
import { CapacitorBarcodeScanner } from '@capacitor/barcode-scanner'
import { alertController } from '@ionic/vue'
import mapboxgl from 'mapbox-gl';
import 'mapbox-gl/dist/mapbox-gl.css';


const scanResult = ref('')
const scanHistory = ref<any>([])
 mapboxgl.accessToken ='pk.eyJ1IjoibGFyaWNyaXMiLCJhIjoiY205dWxraWk3MGF3czJxb29waHNkeWs2eSJ9.y64qlXLkMxVSFXV4ZSbsFg'
 const longitude = ref<any>([]);
  const latitud = ref<any>([]);
  const map = ref<any>(null);
const startScan = async () => {
 
  const status = await CapacitorBarcodeScanner.scanBarcode({ hint: 17 });
  if (status.ScanResult) {
    scanResult.value = status.ScanResult;
    scanHistory.value.push(status.ScanResult);

    const result = status.ScanResult;

   
    const emailRegex = /@/;
    if (emailRegex.test(result)) {
      alert(`No abrió Correo`);
      const email = `mailto:${result}?subject=Escaneo&body=Resultado del escaneo: ${result}`;
      window.location.href = email;
    } 

    else if (result.startsWith('http://') || result.startsWith('https://')) {
   
      window.location.href = result;
    } 

    else if (result.startsWith('{') && result.endsWith('}')) {
      try {
      const coordinates = JSON.parse(result);
      if (coordinates.lng && coordinates.lat) {
        longitude.value = coordinates.lng;
        latitud.value = coordinates.lat;
      
        new mapboxgl.Marker()
        .setLngLat([coordinates.lng, coordinates.lat])
        .addTo(map.value);
      } else {
        alert('Las coordenadas no son válidas.');
      }
      } catch (error) {
      alert('Error al procesar las coordenadas.');
      }
    }
    else {
      alert(`Resultado del escaneo: ${result}`);
    }


  } else {
    const alert = await alertController.create({
      header: 'Error',
      message: 'No se pudo escanear el código de barras.',
      buttons: ['OK']
    });
    await alert.present();
  }
};
onMounted(() => {
   map.value = new mapboxgl.Map({
	container: 'map', // container ID
	style: 'mapbox://styles/mapbox/streets-v12', // style URL
  center: [-85.44106138014348,10.634095596662831], // starting position [lng, lat]
	zoom: 9, // starting zoom
});
 
});
</script>
<template>
  <div style="text-align: center; font-family: Arial, sans-serif; margin-top: 20px;">
    <button 
      @click="startScan" 
      style="padding: 10px 20px; background-color: #007BFF; color: white; border: none; border-radius: 5px; cursor: pointer;">
      Iniciar Escaneo
    </button>
    <p style="margin-top: 20px; font-size: 18px; color: #333;">
      <strong>Resultado del escaneo:</strong> {{ scanResult }}
    </p>
    
    <div style="margin-top: 30px;">
      <p style="font-size: 16px; font-weight: bold; color: #555;">Historial de Escaneos</p>
      <ul style="list-style-type: none; padding: 0;">
        <li 
          v-for="(result, index) in scanHistory" 
          :key="index" 
          style="background-color: #f9f9f9; margin: 5px 0; padding: 10px; border-radius: 5px; box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);">
          {{ result }}
        </li>
      </ul>
    </div>

    <!-- Map container -->
    <div id="map" style="width: 100%; height: 400px; margin-top: 20px;"></div>
  </div>
</template>