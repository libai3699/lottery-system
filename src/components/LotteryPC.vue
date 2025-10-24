<template>
  <div class="lottery-container">
    <!-- 中间抽奖面板 -->
    <div class="lottery-panel">
      <img src="../assets/lottery-frame.png" alt="抽奖框" class="lottery-frame-bg" />

      <!-- 奖项网格 3x4 -->
      <div class="prizes-grid">
        <div class="prize-item"
          :class="{ 'active': currentIndex === index, 'rotating': isRotating, 'thanks-prize': prize.isThanks }"
          v-for="(prize, index) in prizes" :key="index">
          <img v-if="currentIndex === index" src="../assets/prize-bg.png" alt="中奖背景" class="prize-active-bg" />
          <div class="prize-content" :class="{ 'thanks-content': prize.isThanks }">
            <div class="prize-name">{{ prize.name }}</div>
            <div class="prize-level">{{ prize.level }}</div>
            <div class="prize-value" :style="{ color: prize.color }">{{ prize.value }}</div>
          </div>
        </div>
      </div>

      <!-- 抽奖按钮 -->
      <div class="lottery-btn" @click="handleStartLottery" :class="{ 'disabled': isRotating }">
        <img src="../assets/button.png" alt="点击抽奖" />
      </div>
    </div>

    <!-- 抽奖码弹窗 -->
    <div class="modal-overlay" v-if="showCodeModal" @click="closeCodeModal">
      <div class="modal-content code-modal" @click.stop>
        <img src="../assets/code-modal.png" alt="抽獎碼" class="modal-bg" />
        <input type="text" v-model="lotteryCode" placeholder="請輸入抽獎碼" class="code-input" maxlength="20" />
        <div class="modal-buttons">
          <img src="../assets/cancel-btn.png" alt="取消" class="btn-cancel" @click="closeCodeModal" />
          <img src="../assets/confirm-btn.png" alt="确认" class="btn-confirm" @click="handleConfirmCode" />
        </div>
      </div>
    </div>

    <!-- 中奖弹窗 -->
    <div class="modal-overlay" v-if="showWinModal" @click="closeWinModal">
      <div class="modal-content win-modal" @click.stop>
        <img src="../assets/win-modal.png" alt="中奖弹窗" class="modal-bg" />
        <div class="win-info">
          <div class="win-prize-name">{{ winPrize.fullName }}</div>
          <div class="win-prize-desc">{{ winPrize.desc }}</div>
        </div>
        <img src="../assets/win-confirm.png" alt="确认中奖" class="btn-win-confirm" @click="closeWinModal" />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'LotteryPC',
  props: {
    prizes: {
      type: Array,
      required: true,
      default: () => []
    }
  },
  data() {
    return {
      showCodeModal: false,
      showWinModal: false,
      lotteryCode: '',
      isRotating: false,
      currentIndex: 5,
      winPrize: {}
    }
  },
  methods: {
    handleStartLottery() {
      if (this.isRotating) return
      this.showCodeModal = true
    },
    closeCodeModal() {
      this.showCodeModal = false
      this.lotteryCode = ''
    },
    handleConfirmCode() {
      if (!this.lotteryCode.trim()) {
        this.$emit('show-toast', '請輸入抽獎碼', 'error')
        return
      }
      this.$emit('lottery', this.lotteryCode)
    },
    startRotate(finalIndex) {
      this.closeCodeModal()
      this.isRotating = true

      // 动画参数
      const phase1Duration = 1000  // 匀速1秒
      const phase2Duration = 2000  // 匀加速2秒
      const phase3Duration = 1000  // 匀速1秒
      const totalDuration = phase1Duration + phase2Duration + phase3Duration

      // 至少转3圈再到目标位置
      const minRotations = 3
      const totalSteps = minRotations * 12 + finalIndex

      const startTime = Date.now()

      const animate = () => {
        const elapsed = Date.now() - startTime

        if (elapsed >= totalDuration) {
          // 动画结束，精确停在目标位置
          this.currentIndex = finalIndex
          setTimeout(() => {
            this.isRotating = false
            this.showWinResult(finalIndex)
          }, 800)
          return
        }

        let progress = 0

        if (elapsed < phase1Duration) {
          // 阶段1：匀速
          const t = elapsed / phase1Duration
          progress = t * 0.2  // 占总进度的20%
        } else if (elapsed < phase1Duration + phase2Duration) {
          // 阶段2：匀加速
          const t = (elapsed - phase1Duration) / phase2Duration
          progress = 0.2 + (t * t) * 0.6  // 占总进度的60%，使用二次函数加速
        } else {
          // 阶段3：匀速
          const t = (elapsed - phase1Duration - phase2Duration) / phase3Duration
          progress = 0.8 + t * 0.2  // 占总进度的20%
        }

        // 计算当前应该在哪个位置
        const currentStep = Math.floor(progress * totalSteps)
        this.currentIndex = currentStep % 12

        requestAnimationFrame(animate)
      }

      animate()
    },
    showWinResult(index) {
      this.winPrize = this.prizes[index]
      this.showWinModal = true
    },
    closeWinModal() {
      this.showWinModal = false
      this.currentIndex = 0
    }
  }
}
</script>

<style scoped>
.lottery-container {
  position: fixed;
  inset: 0;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
  background-image: url('../assets/bg.png');
  background-size: 100% 100%;
  background-position: center center;
  background-repeat: no-repeat;
}

.lottery-panel {
  position: fixed;
  top: 56.5%;
  left: 50%;
  transform: translate(-50%, -50%);
  z-index: 10;
  width: 50vw;
}

.lottery-frame-bg {
  width: 100%;
  max-height: 85vh;
  height: auto;
  display: block;
  object-fit: contain;
}

.prizes-grid {
  position: absolute;
  top: 18%;
  left: 49.23%;
  transform: translateX(-50%);
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  justify-items: center;
  align-items: center;
  width: 69%;
  z-index: 1;
  padding: 2.5vh 3vw 0 3vw;
  box-sizing: border-box;
}

.prize-item {
  position: relative;
  width: 100%;
  max-width: 100%;
  aspect-ratio: 1;
  background: transparent;
  border: none;
  outline: none;
  border-radius: 0.5vw;
  padding: 0;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  text-align: center;
  transition: all 0.3s ease;
  cursor: pointer;
  overflow: hidden;
}

.prize-active-bg {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  width: 90%;
  height: 90%;
  object-fit: contain;
  z-index: 0;
}

.prize-content {
  position: relative;
  z-index: 1;
}

.prize-item.active {
  transform: scale(1.05);
}

.prize-item.rotating {
  transition: all 0.08s linear;
}

.prize-name {
  font-size: 0.9vw;
  font-weight: bold;
  color: #1e293b;
  margin-bottom: 0.3vh;
}

.prize-level {
  font-size: 0.8vw;
  margin-bottom: 0.5vh;
}

.prize-value {
  font-size: 1vw;
  font-weight: bold;
  white-space: pre-line;
  line-height: 1.3;
}

/* 銘謝惠顧特殊样式 */
.prize-item.thanks-prize .prize-name {
  font-size: 1.1vw;
  color: #1e293b;
  font-weight: bold;
}

.prize-item.thanks-prize .prize-level {
  font-size: 1.1vw;
  color: #1e293b;
  font-weight: bold;
}

.prize-item.thanks-prize .prize-value {
  display: none;
}

.lottery-btn {
  position: absolute;
  bottom: 14.3%;
  left: 50%;
  transform: translateX(-50%);
  cursor: pointer;
  transition: transform 0.2s;
  z-index: 1;
  width: 33%;
}

.lottery-btn:hover {
  transform: translateX(-50%) scale(1.05);
}

.lottery-btn:active {
  transform: translateX(-50%) scale(0.95);
}

.lottery-btn.disabled {
  pointer-events: none;
  opacity: 0.6;
}

.lottery-btn img {
  width: 100%;
  height: auto;
  display: block;
}

/* 弹窗样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }

  to {
    opacity: 1;
  }
}

.modal-content {
  position: relative;
  animation: scaleIn 0.3s ease;
}

@keyframes scaleIn {
  from {
    transform: scale(0.8);
    opacity: 0;
  }

  to {
    transform: scale(1);
    opacity: 1;
  }
}

.modal-bg {
  width: 100%;
  height: auto;
  display: block;
}

/* 抽奖码弹窗 */
.code-modal {
  width: 55vw;
}

.code-input {
  position: absolute;
  top: 50%;
  left: 53%;
  transform: translateX(-50%);
  width: 60%;
  padding: 2vh 2vw;
  font-size: 1.3vw;
  border: 0.2vw solid #cbd5e1;
  border-radius: 0.8vw;
  text-align: center;
  outline: none;
  transition: border-color 0.3s;
}

.code-input:focus {
  border-color: #3b82f6;
}

.modal-buttons {
  position: absolute;
  bottom: 10%;
  left: 53%;
  transform: translateX(-50%);
  display: flex;
  gap: 5vw;
}

.btn-cancel,
.btn-confirm {
  width: 15vw;
  max-width: 120px;
  min-width: 80px;
  height: auto;
  cursor: pointer;
  transition: transform 0.2s;
}

.btn-cancel:hover,
.btn-confirm:hover {
  transform: scale(1.1);
}

.btn-cancel:active,
.btn-confirm:active {
  transform: scale(0.95);
}

/* 中奖弹窗 */
.win-modal {
  width: 40vw;
  max-width: 650px;
  min-width: 450px;
}

.win-info {
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translateX(-50%);
  text-align: center;
  width: 80%;
}

.win-prize-name {
  font-size: 4.5vw;
  font-weight: bold;
  color: #dc2626;
  margin-bottom: 1vh;
  text-shadow: 0.2vw 0.2vw 0.4vw rgba(0, 0, 0, 0.2);
}

.win-prize-desc {
  font-size: 3vw;
  color: #1e293b;
  font-weight: 500;
}

.btn-win-confirm {
  position: absolute;
  bottom: 15%;
  left: 52%;
  transform: translateX(-50%);
  width: 20vw;
  height: auto;
  cursor: pointer;
  transition: transform 0.2s;
}

.btn-win-confirm:hover {
  transform: translateX(-50%) scale(1.1);
}

.btn-win-confirm:active {
  transform: translateX(-50%) scale(0.95);
}

/* 超大屏幕适配 */
@media (min-width: 1920px) {
  .lottery-panel {
    width: 850px;
  }
}
</style>
