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
  .ai-model-select-cpt {
    display: flex;
    flex-direction: column;
    position: relative; /* Positioning for model selector */
    .content-box {
      font-size: 12px;
      color: #777777;
      display: flex;
      flex-direction: column;
      align-items: flex-start;
      border-bottom: none;
      margin-bottom: 15px;
      p {
        height: 40px;
        font-size: 14px;
        display: flex;
        align-items: center;
        color: #fb8444;
        img {
          margin-left: 5px;
        }
      }
      .jb-num {
        font-size: 18px;
        font-weight: 600;
        background: -webkit-linear-gradient(top, #ff0000, #00ff00); /* Linear gradient */
        /* For cross-browser support */
        background-clip: text; /* Clip background to text */
        -webkit-text-fill-color: transparent; /* Transparent text fill */
        letter-spacing: 1px;
      }
      .content {
        font-size: 14px;
        color: #333333;
        margin-top: 10px;
        margin-left: 10px;
      }
    }
    .title {
      font-size: 16px;
      color: #009a74;
      position: relative;
      height: 20px;
      display: flex;
      align-items: center;
      margin-left: 10px;
      letter-spacing: 1px;
      &::before {
        content: '';
        position: absolute;
        height: 10px;
        width: 3px;
        background-color: #009a74;
        left: -10px;
      }
      .get-bi-method {
        display: flex;
        align-items: center;
        justify-content: center;
        padding: 6px 10px;
        height: 25px;
        background-color: #70f5c4;
        border-radius: 15px;
        font-size: 13px;
        transition: all 0.3s;
        margin: 0 auto;
        margin-left: 15px;
        cursor: pointer;
        letter-spacing: 1px;
        user-select: none;
        &:hover {
          opacity: 0.8;
        }
      }
    }
    .model-selector {
      width: 100%;
      margin-bottom: 20px;
      .el-select {
        width: 100%;
      }
      .model-tips {
        font-size: 12px;
        color: #999;
        margin-top: 8px;
        margin-left: 10px;
      }
    }
    .history-optimize-list {
      display: flex;
      align-items: center;
      justify-content: center;
      cursor: pointer;
      transition: all 0.3s;
      padding: 5px 0;
      width: 110px;
      background-color: #fbfbfb;
      font-size: 12px;
      color: #8d8a8a;
      user-select: none;
      &:hover {
        background-color: #eee;
        border-radius: 3px;
        color: green;
      }
      span {
        margin-left: 6px;
      }
    }
    .el-radio-group {
      margin: 20px 0 0 10px;
    }
    .el-radio {
      display: flex;
      align-items: center;
      margin-bottom: 10px;
      border-radius: 8px;
      padding: 10px;
      transition: all 0.3s ease;
      border: 1px solid #dcdfe6;
      position: relative;

      &:hover {
        background-color: #f5f7fa;
        border-color: #4e97fb;
      }

      &.is-checked {
        background-color: #e8f4ff;
        border-color: #4e97fb;

        .el-radio__label {
          color: #4e97fb;
        }
      }

      :deep(.el-radio__label) {
        display: flex;
        align-items: center;
        justify-content: space-between;
        width: 100%;
        font-size: 14px;
        color: #606266;

        .free-tag {
          margin-left: 10px;
          padding: 4px 8px;
          background-color: #e8f5e9;
          color: #4caf50;
          border-radius: 12px;
          font-size: 12px;
          font-weight: bold;
        }

        .tips {
          display: flex;
          align-items: center;
          justify-content: center;
          margin-left: 10px;
          font-size: 14px;
          color: #4e97fb;
          font-weight: bold;

          img {
            margin-left: 5px;
          }
        }
      }
      .vip-icon {
        position: absolute;
        top: -12px;
        right: -6px;
      }
    }
  }
</style>
