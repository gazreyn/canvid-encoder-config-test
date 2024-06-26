<script setup lang="ts">
import { reactive, ref } from 'vue';

/**
 * Array of supported pixel formats
 */
const verticalPixelList = [2160, 1440, 1080, 720, 576];

/**
 * Store all the configs that have been tested
 */
const configsTested = reactive<VideoEncoderConfig[]>([]);

const initialConfig = ref<VideoEncoderConfig>({
  codec: 'avc1.640032',
  width: 3840,
  height: 2160,
  bitrate: 2000000,
  framerate: 30,
  hardwareAcceleration: 'prefer-hardware',
  latencyMode: 'quality',
  bitrateMode: 'quantizer',
});

/**
 * Store the result of the test. Used to display the result of the test
 */
const result = reactive({
  status: '',
  config: initialConfig.value,
});

/**
 * Run the test. Generates a list of alternative configurations and tests them
 */
async function run() {
  const configs = generateConfigs(initialConfig.value);

  // Clear the list of tested configs
  configsTested.length = 0;

  for (const config of configs) {
    const supported = await test(config);
    configsTested.push(config);

    if (supported) {
      result.status = 'Config Supported';
      result.config = config;
      return;
    }
  }

  if (result.status === '') {
    result.status = 'Config Not Supported';
  }
}

/**
 * Uses VideoEncoder to test if a config is supported
 * @param config
 * @returns boolean
 */
async function test(config: VideoEncoderConfig) {
  const { supported } = await VideoEncoder.isConfigSupported(config);
  return supported;
}

function generateConfigs(initialConfig: VideoEncoderConfig) {
  // TODO: Doesn't consider vertical aspect ratios...
  const configArray: VideoEncoderConfig[] = [];

  // Add the initial config to the list to be tested
  configArray.push(initialConfig);

  // Get aspect ratio of the initial config
  const aspectRatio = initialConfig.width / initialConfig.height;

  // Generate a list of alternative configurations
  for (const height of verticalPixelList) {
    if (height >= initialConfig.height) continue;

    const width = height * aspectRatio;

    const config: VideoEncoderConfig = {
      codec: initialConfig.codec,
      width: width,
      height: height,
      bitrate: initialConfig.bitrate,
      framerate: initialConfig.framerate,
      hardwareAcceleration: initialConfig.hardwareAcceleration,
      latencyMode: initialConfig.latencyMode,
      bitrateMode: initialConfig.bitrateMode,
    };

    configArray.push(config);
  }

  return configArray;
}
</script>

<template>
  <div>
    <div style="margin-bottom: 1rem">
      <div
        style="
          display: flex;
          flex-direction: column;
          gap: 1rem;
          margin-bottom: 1rem;
        "
      >
        <label
          >Width:
          <input type="number" v-model="initialConfig.width" />
        </label>
        <label
          >Height:<input type="number" v-model="initialConfig.height"
        /></label>
      </div>
      <button @click="run">Test</button>
    </div>
    <div v-if="result.status">
      <div
        style="
          text-align: left;
          background-color: #1c1c1c;
          padding: 1rem;
          border-radius: 0.5rem;
        "
      >
        <div>
          Status:
          <span
            style="font-weight: bold"
            :style="{
              color: result.status === 'Config Supported' ? 'green' : 'red',
            }"
            >{{ result.status }}</span
          >
        </div>
        <div>
          Working Config:
          <pre>{{ result.config }}</pre>
        </div>
      </div>
      <div
        style="
          margin-top: 1rem;
          text-align: left;
          background-color: #1c1c1c;
          padding: 1rem;
          border-radius: 0.5rem;
        "
      >
        <span>Tested Configs:</span>
        <pre>{{ configsTested }}</pre>
      </div>
    </div>
  </div>
</template>
