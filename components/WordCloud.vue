<template>
  <div>
    <div id="chart2" ref="chartContainer" class="w-full h-96 mb-4"></div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
const echarts = require('echarts');
require('echarts-wordcloud');

type KeywordWithWeight = [string, number];

const chartContainer = ref(null);

const generateWordcloud = async () => {
  //从接受的请求当中读取todayChat
  // const { todayChat } = await chrome.storage.local.get('todayChat');
  const todayChat: string = '今天天气真好，我想去公园玩。';
  if (!todayChat) {
    console.error('Error: Failed to retrieve todayChat from chrome.storage.local');
    return;
  }
  try {
    const response = await fetch('https://nwkazoq0sc.execute-api.ap-southeast-2.amazonaws.com/production/', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ text: todayChat })
    });

    if (!response.ok) {
      throw new Error('Network response was not ok ' + response.statusText);
    }

    const words = await response.json();

    // 检查 chartContainer ref 是否已经设置
    console.log('words', words)
    if (chartContainer.value) {
      // 初始化 ECharts 实例
      const chart = echarts.init(chartContainer.value);
      console.log(chart);  // 查看ECharts实例
      // 配置图表选项
      const option = {
         toolbox: {
          feature: {
            saveAsImage: {
              show: true,  // 显示下载按钮
              title: '下载图表',  // 下载按钮的提示文字
              pixelRatio: 2,  // 下载的图像分辨率
            }
          }
        },
        series: [
          {
            type: 'wordCloud',

            data: words.keywords.map((item: KeywordWithWeight) => {
              return {
                name: item[0],  // 词
                value: item[1],  // 权重
              };
            }),
          },
        ],
      };

      // 设置图表选项
      chart.setOption(option);
    }
  } catch (error) {
    console.error('Error: Failed to fetch word cloud data from the API', error);
  }
};

// 在组件挂载完成后调用 generateWordcloud 函数
onMounted(generateWordcloud);

</script>
