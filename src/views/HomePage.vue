<script setup lang="ts">
import { defineComponent } from 'vue'
import { Capacitor } from '@capacitor/core'
import { ref } from 'vue'
import { CapacitorBarcodeScanner } from '@capacitor/barcode-scanner'
import { alertController } from '@ionic/vue'

const scanResult = ref('')
const scanHistory = ref<any>([])

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
  </div>
</template>