<template>
    <div class="lottery-container">
        <!-- 中间抽奖面板 -->
        <div class="lottery-panel">
            <img src="../assets/mobileImg/抽奖栏.png" alt="抽奖框" class="lottery-frame-bg" />

            <!-- 奖项网格 3x4 -->
            <div class="prizes-grid">
                <div class="prize-item"
                    :class="{ 'active': currentIndex === index, 'rotating': isRotating, 'thanks-prize': prize.isThanks }"
                    v-for="(prize, index) in prizes" :key="index">
                    <img v-if="currentIndex === index" src="../assets/mobileImg/中奖选项.png" alt="中奖背景"
                        class="prize-active-bg" />
                    <div class="prize-content" :class="{ 'thanks-content': prize.isThanks }">
                        <div class="prize-name">{{ prize.name }}</div>
                        <div class="prize-level">{{ prize.level }}</div>
                        <div class="prize-value" :style="{ color: prize.color }">{{ prize.value }}</div>
                    </div>
                </div>
            </div>

            <!-- 抽奖按钮 -->
            <div class="lottery-btn" @click="handleStartLottery" :class="{ 'disabled': isRotating }">
                <img src="../assets/mobileImg/抽奖按钮.png" alt="点击抽奖" />
            </div>
        </div>

        <!-- 抽奖码弹窗 -->
        <div class="modal-overlay" v-if="showCodeModal" @click="closeCodeModal">
            <div class="modal-content code-modal" @click.stop>
                <img src="../assets/mobileImg/抽奖码背景.png" alt="抽獎碼" class="modal-bg" />
                <input type="text" v-model="lotteryCode" placeholder="請輸入抽獎碼" class="code-input" maxlength="20" />
                <div class="modal-buttons">
                    <img src="../assets/mobileImg/取消.png" alt="取消" class="btn-cancel" @click="closeCodeModal" />
                    <img src="../assets/mobileImg/确认.png" alt="确认" class="btn-confirm" @click="handleConfirmCode" />
                </div>
            </div>
        </div>

        <!-- 中奖弹窗 -->
        <div class="modal-overlay" v-if="showWinModal" @click="closeWinModal">
            <div class="modal-content win-modal" @click.stop>
                <img src="../assets/mobileImg/中奖弹窗背景.png" alt="中奖弹窗" class="modal-bg" />
                <div class="win-info">
                    <div class="win-prize-name">{{ winPrize.fullName }}</div>
                    <div class="win-prize-desc">{{ winPrize.desc }}</div>
                </div>
                <img src="../assets/mobileImg/中奖按钮.png" alt="确认中奖" class="btn-win-confirm" @click="closeWinModal" />
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'LotteryMobile',
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
                    }, 500)
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
    position: relative;
    width: 100vw;
    min-height: 100vh;
    padding: 10vh 0 0vh 0;
    box-sizing: border-box;
    overflow: visible;
    background-image: url('../assets/mobileImg/BG.png');
    background-size: 100% 100%;
    background-position: center;
    background-repeat: no-repeat;
    display: flex;
    align-items: center;
    justify-content: center;
}

.lottery-panel {
    position: relative;
    margin: 0 auto;
    z-index: 10;
    width: 100vw;
    max-width: 450px;
    padding-bottom: 5vh;
}

.lottery-frame-bg {
    width: 100%;
    height: auto;
    display: block;
    object-fit: contain;
}

.prizes-grid {
    position: absolute;
    top: 18.2%;
    left: 50%;
    transform: translateX(-50%);
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    grid-template-rows: repeat(4, 1fr);
    width: 52%;
    z-index: 1;
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
    object-fit: contain;
    width: 90%;
    height: 90%;
    z-index: 0;
}

.prize-content {
    position: relative;
    z-index: 1;
    padding: 1vw;
}

.prize-item.active {
    transform: scale(1.08);
}

.prize-item.rotating {
    transition: all 0.08s linear;
}

.prize-name {
    font-size: 2.8vw;
    font-weight: bold;
    color: #1e293b;
}

.prize-level {
    font-size: 2.5vw;
    color: #1e293b;
    font-weight: bold;
    margin-bottom: 0.2vh;
}

.prize-value {
    font-size: 2.6vw;
    font-weight: bold;
    white-space: pre-line;
    line-height: 1.5;
}

/* 銘謝惠顧特殊样式 */
.prize-item.thanks-prize .prize-content {
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
    width: 100%;
}

.prize-item.thanks-prize .prize-name {
    font-size: 4vw;
    margin-bottom: 0.3vh;
    text-align: center;
    width: 100%;
}

.prize-item.thanks-prize .prize-level {
    font-size: 4vw;
    margin-bottom: 0;
    text-align: center;
    width: 100%;
}

.prize-item.thanks-prize .prize-value {
    display: none;
}

.lottery-btn {
    position: absolute;
    bottom: 11.5vh;
    left: 50%;
    transform: translateX(-50%);
    cursor: pointer;
    transition: transform 0.2s;
    z-index: 1;
    width: 43%;
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
    width: 90vw;
}

.code-input {
    position: absolute;
    top: 48%;
    left: 55%;
    transform: translateX(-50%);
    width: 65%;
    padding: 3vw;
    font-size: 4vw;
    border: 0.5vw solid #cbd5e1;
    border-radius: 2vw;
    text-align: center;
    outline: none;
    transition: border-color 0.3s;
}

.code-input:focus {
    border-color: #3b82f6;
}

.modal-buttons {
    position: absolute;
    bottom: 8%;
    left: 53%;
    transform: translateX(-50%);
    display: flex;
    gap: 8vw;
}

.btn-cancel,
.btn-confirm {
    width: 30vw;
    max-width: 120px;
    height: auto;
    cursor: pointer;
    transition: transform 0.2s;
}

.btn-cancel:active,
.btn-confirm:active {
    transform: scale(0.95);
}

/* 中奖弹窗 */
.win-modal {
    width: 80vw;
    max-width: 400px;
}

.win-info {
    position: absolute;
    top: 35%;
    left: 53%;
    transform: translateX(-50%);
    text-align: center;
    width: 80%;
}

.win-prize-name {
    font-size: 7vw;
    font-weight: bold;
    color: #dc2626;
    margin-bottom: 2vh;
    text-shadow: 0.3vw 0.3vw 0.6vw rgba(0, 0, 0, 0.2);
}

.win-prize-desc {
    font-size: 5vw;
    color: #1e293b;
    font-weight: 500;
}

.btn-win-confirm {
    position: absolute;
    bottom: 13.5%;
    left: 53%;
    transform: translateX(-50%);
    width: 40vw;
    max-width: 180px;
    height: auto;
    cursor: pointer;
    transition: transform 0.2s;
}

.btn-win-confirm:active {
    transform: translateX(-50%) scale(0.95);
}
</style>
