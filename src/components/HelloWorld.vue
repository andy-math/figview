<template>
  <n-config-provider :theme="theme">
    <n-layout id="HelloWorldRoot">
      <n-layout-header id="HelloWorldHeader" bordered>
        <n-space justify="space-around" align="center">
          <n-button strong size="large" @click="changeTheme">切换主题</n-button>
          <h1>Figview</h1>
          <n-space align="center" justify="center">
            <n-button strong circle size="large" @click="numOfCols -= numOfCols == 1 ? 0 : 1">
              <template #icon>
                <n-icon>
                  <add-icon />
                </n-icon>
              </template>
            </n-button>
            <n-tag>{{ numOfCols }}列</n-tag>
            <n-button strong circle size="large" @click="numOfCols += 1">
              <template #icon>
                <n-icon>
                  <remove-icon />
                </n-icon>
              </template>
            </n-button>
          </n-space>
        </n-space>
        <n-space align="center" justify="center"></n-space>
      </n-layout-header>
      <n-grid :x-gap="12" :y-gap="8" :cols="numOfCols">
        <n-grid-item v-for="fig in data">
          <n-carousel show-arrow>
            <div v-for="svg in fig.id_list">
              <!--
              <img class="figure" :src="'http://localhost:8080/' + String(svg.src)" />
              <n-card :title="'Figure ' + String(fig.figure)">
                {{ svg.time }}
                <n-button>上传</n-button>
              </n-card>
              -->
              <n-card :title="'Figure ' + String(fig.figure)">
                <template #header-extra>
                  <n-upload
                    :action="'http://localhost:8080/addfig?fig=' + String(fig.figure)"
                    :show-file-list="false"
                  >
                    <n-button>上传文件</n-button>
                  </n-upload>
                </template>
                <img class="figure" :src="'http://localhost:8080/' + String(svg.src)" />
                <template #footer>{{ svg.time }}</template>
              </n-card>
            </div>
          </n-carousel>
        </n-grid-item>
      </n-grid>
    </n-layout>
  </n-config-provider>
</template>

<script setup lang="ts">
import { AddCircle as AddIcon, RemoveCircle as RemoveIcon } from '@vicons/ionicons5';
import axios, { AxiosResponse } from 'axios';
import { darkTheme, lightTheme, NButton, NCard, NCarousel, NConfigProvider, NGrid, NGridItem, NIcon, NLayout, NLayoutHeader, NSpace, NTag, NUpload } from 'naive-ui';
import { BuiltInGlobalTheme } from 'naive-ui/lib/themes/interface';
import { Ref, ref } from 'vue';

interface Blob {
  id: string;
  src: string;
  time: string;
}

interface Figure {
  figure: number;
  name: string;
  id_list: Blob[];
}

const onError = (error: AxiosResponse<Figure[]>) => {
  console.log(error);
  alert("断流！");
}

const checkStatus = (response: AxiosResponse<Figure[]>, f: (arg0: AxiosResponse<Figure[]>) => void) => {
  if (response.status === 200) {
    f(response);
  } else {
    onError(response);
  }
}

const onRefresh = function (response: AxiosResponse<Figure[]>) {
  checkStatus(response, (response) => {
    //console.log(response);
    data.value = response.data;
  })
}

setInterval(() => {
  axios.get('http://localhost:8080/listfig').then(onRefresh).catch(onError);
}, 100);

const changeTheme = () => {
  if (theme.value.name == darkTheme.name) {
    theme.value = lightTheme;
  } else {
    theme.value = darkTheme;
  }
}


const theme: Ref<BuiltInGlobalTheme> = ref(darkTheme)
const data: Ref<Figure[]> = ref([])
const numOfCols = ref(3)
</script>

<style scoped>
.n-carousel {
  width: auto;
  height: auto;
  max-width: 100%;
  vertical-align: bottom;
}
img.figure {
  width: auto;
  height: auto;
  max-width: 100%;
  vertical-align: bottom;
}
</style>
