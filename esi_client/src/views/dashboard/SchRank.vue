<template>
    <div>
      <n-tabs type="line" animated>
        <n-tab-pane name="MapDisplay" tab="MapDisplay">
          <MapContainer></MapContainer>
        </n-tab-pane>
        <!-- <n-tab-pane name="maintain" tab="ListDisplay" v-if="$isAdmin">
          <n-message-provider>
            <BioTab></BioTab>
          </n-message-provider>
          <n-message-provider>
            <test></test>
          </n-message-provider>
        </n-tab-pane> -->
  
      </n-tabs>
    </div>
  </template>
  
  <script setup>
  import {ref,reactive,inject} from 'vue'
  import {AddCircleOutline, RefreshCircleOutline, SearchOutline, TrashOutline, CreateOutline, DownloadOutline} from "@vicons/ionicons5"
  
  import * as echarts from 'echarts';
  
  import {useRouter, useRoute} from 'vue-router'
  const router = useRouter()
  const route = useRoute()
  
  const axios = inject("axios")
  const message = inject("message")
  const serverUrl = inject("serverUrl")
  const dialog = inject("dialog")
  const fileDownload = inject("fileDownload")
  
  import { onMounted, onUnmounted } from "vue";
  import MapContainer from '../../components/mapContainer.vue';
//   import BioTab from '../../components/BioTab.vue';
//   import test from '../../components/test.vue';
  
  
  
  </script>
  
  
  <style scoped>
  
  </style>
  
  