<template>
  <div class="ai-model-select-cpt">
    <!-- Current Jian Coins balance -->
    <div class="content-box">
      <h1 class="title">
        Your current Jian Coins
        <div class="get-bi-method" @click="openGetDialog">Get Coins</div>
      </h1>
      <div class="content">
        <p class="jb-num">
          {{ formattedIntegral }}
          <img width="22" src="@/assets/images/jianB.png" alt="Jian Coin" />
        </p>
      </div>
    </div>

    <!-- Model selector -->
    <div class="model-selector">
      <h1 class="title"> Select AI Model </h1>
      <el-radio-group v-model="selectedModel" @change="handleModelChange">
        <el-tooltip effect="dark" content="For members only" placement="top">
          <el-radio label="" size="large" border :disabled="!isMember">
            Free Model
            <span class="free-tag">Free</span>
            <!-- Crown icon -->
            <img
              class="vip-icon"
              src="@/assets/images/membership.svg"
              alt="Member"
              title="Member"
              width="20"
            />
          </el-radio>
        </el-tooltip>
        <template v-if="modelList.length > 0">
          <el-tooltip
            v-for="(item, index) in modelList"
            :key="index"
            effect="dark"
            :content="`Costs ${Math.abs(payValue)} Jian Coins per use`"
            placement="top"
          >
            <el-radio :label="item" size="large" border>
              {{ item }}
              <span class="tips">
                {{ Math.abs(payValue) }}
                <img width="22" src="@/assets/images/jianB.png" alt="Jian Coin" title="Jian Coin" />
              </span>
            </el-radio>
          </el-tooltip>
        </template>
      </el-radio-group>
    </div>
  </div>

  <!-- Get Jian Coins dialog -->
  <pay-integral-dialog
    :title="title"
    :dialog-get-integral-visible="dialogGetIntegralVisible"
    :pay-number="-Math.abs(payValue) || 0"
    placeholder="Get Jian Coins"
    @cancle="handleCancleDialog"
    @confirm="handleConfirmDialog"
  />
</template>

<script lang="ts" setup>
  import { ref, computed } from 'vue';
  import { ElMessage } from 'element-plus';
  import {
    getGenerateResumeIntegralAsync,
    getGenerateResumeModelListAsync,
    getOptimizeResumeIntegralAsync,
    getOptimizeResumeModelListAsync
  } from '@/http/api/ai';
  import { formatNumberWithCommas } from '@/utils/common';
  import appStore from '@/store';
  import { storeToRefs } from 'pinia';

  const emit = defineEmits(['handleModelChange']);

  interface TDialog {
    dialogAiOptimizeVisible: boolean;
    aiType?: string;
  }

  const props = withDefaults(defineProps<TDialog>(), {
    dialogAiOptimizeVisible: false,
    aiType: 'optimizeResume'
  });

  // Check if user is a member
  const { membershipInfo } = storeToRefs(appStore.useMembershipStore);
  const isMember = computed(() => {
    if (membershipInfo.value.hasMembership && membershipInfo.value.daysRemaining > 0) {
      return true;
    } else {
      return false;
    }
  });

  // Reactive data
  const selectedModel = ref<string>('');
  const modelList = ref<string[]>([]);
  const payValue = ref<number>(0);
  const dialogGetIntegralVisible = ref<boolean>(false);
  const title = ref<string>('');

  // Model selection change handler
  const handleModelChange = (value: string) => {
    console.log('value:', value);
    selectedModel.value = value;
    emit('handleModelChange', {
      value: value,
      payValue: payValue.value
    });
  };

  // Computed properties
  const formattedIntegral = computed(() => {
    return formatNumberWithCommas(appStore.useUserInfoStore.userIntegralInfo.integralTotal);
  });

  onMounted(async () => {
    if (props.aiType === 'generateResume') {
      // Get coin balance and model list
      await Promise.all([getGenerateResumeCoin(), getGenerateResumeModelList()]);
    } else {
      // Get coin balance and model list
      await Promise.all([getOptimizeResumeCoin(), getOptimizeResumeModelList()]);
    }
  });

  // Get coins required for resume optimization
  const getOptimizeResumeCoin = async () => {
    const response = await getOptimizeResumeIntegralAsync();
    if (response.data.status === 200) {
      payValue.value = response.data.data;
    } else {
      ElMessage.error(response.data.message);
    }
  };

  // Get coins required for resume generation
  const getGenerateResumeCoin = async () => {
    const response = await getGenerateResumeIntegralAsync();
    if (response.data.status === 200) {
      payValue.value = response.data.data;
      handleModelChange(selectedModel.value);
    } else {
      ElMessage.error(response.data.message);
    }
  };

  // Get model list for resume optimization
  const getOptimizeResumeModelList = async () => {
    try {
      const response = await getOptimizeResumeModelListAsync();
      if (response.data.status === 200) {
        modelList.value = response.data.data;
        if (modelList.value.length > 0) {
          selectedModel.value = modelList.value[0];
          handleModelChange(selectedModel.value);
        }
      } else {
        ElMessage.error(response.data.message);
      }
    } catch (error) {
      throw new Error('Failed to get model list');
    }
  };

  // Get model list for resume generation
  const getGenerateResumeModelList = async () => {
    try {
      const response = await getGenerateResumeModelListAsync();
      if (response.data.status === 200) {
        modelList.value = response.data.data;
        if (modelList.value.length > 0) {
          selectedModel.value = modelList.value[0];
          handleModelChange(selectedModel.value);
        }
      } else {
        ElMessage.error(response.data.message);
      }
    } catch (error) {
      throw new Error('Failed to get model list');
    }
  };

  // Open get coins dialog
  const openGetDialog = () => {
    title.value = 'How to get Jian Coins';
    dialogGetIntegralVisible.value = true;
  };

  // Cancel dialog
  const handleCancleDialog = () => {
    dialogGetIntegralVisible.value = false;
  };
  
  // Confirm dialog
  const handleConfirmDialog = () => {
    dialogGetIntegralVisible.value = false;
  };
</script>

<style lang="scss" scoped>
  /* CSS remains exactly the same as it contains no Chinese text */
  /* ... existing CSS code ... */
</style>
