<template>
  <div>
    <!-- PC端 -->
    <LotteryPC v-if="!isMobile" ref="lotteryPC" :prizes="allPrizes" @lottery="handleLottery" @show-toast="showToast" />

    <!-- 移动端 -->
    <LotteryMobile v-if="isMobile" ref="lotteryMobile" :prizes="allPrizes" @lottery="handleLottery"
      @show-toast="showToast" />

    <!-- Toast 提示 -->
    <div v-if="toast.show" class="toast" :class="toast.type">
      {{ toast.message }}
    </div>
  </div>
</template>

<script>
import LotteryPC from './components/LotteryPC.vue'
import LotteryMobile from './components/LotteryMobile.vue'

export default {
  name: 'App',
  components: {
    LotteryPC,
    LotteryMobile
  },
  data() {
    return {
      allPrizes: [],
      isMobile: false,
      toast: {
        show: false,
        message: '',
        type: 'error'
      }
    }
  },
  mounted() {
    this.checkDevice()
    this.getWheelInfo()
    window.addEventListener('resize', this.handleResize)
  },
  beforeUnmount() {
    window.removeEventListener('resize', this.handleResize)
  },
  methods: {
    checkDevice() {
      // 判断是否为移动端：User Agent + 窗口宽度
      const isMobileUA = /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent)
      const isMobileWidth = window.innerWidth <= 768
      this.isMobile = isMobileUA || isMobileWidth
    },
    handleResize() {
      this.checkDevice()
    },
    async getWheelInfo() {
      try {
        const response = await fetch(`${window.location.origin}/renren-fast/app/weel/getWheelInfo`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          }
        })

        const result = await response.json()

        if (result.code === 0 && result.data) {
          // 将接口返回的奖品数据映射到页面显示格式
          this.allPrizes = result.data.map(prize => {
            const name = prize.name || ''
            const award = prize.award || ''

            // 处理 name：銘謝惠顧特殊处理，其他超过4个字符就换行
            let displayName = ''
            let displayLevel = ''
            if (name === '銘謝惠顧') {
              displayName = '銘謝'
              displayLevel = '惠顧'
            } else if (name.length > 4) {
              displayName = name.substring(0, 4)
              displayLevel = name.substring(4)
            } else {
              displayName = name
              displayLevel = ''
            }

            // 处理 award：如果是"銘謝惠顧"直接显示，否则处理括号
            let displayValue = ''
            if (award === '很抱歉你没有中奖') {
              displayValue = ''
            } else if (award.includes('USDT')) {
              // USDT 单独一行
              const match = award.match(/^(.+?)(USDT.*)$/)
              if (match) {
                displayValue = `${match[1]}\n${match[2]}`
              } else {
                displayValue = award
              }
            } else if (award.includes('（') && award.includes('）')) {
              // 提取括号前的内容和括号内的内容
              const match = award.match(/^(.+)（(.+)）$/)
              if (match) {
                displayValue = `${match[1]}\n${match[2]}`
              } else {
                displayValue = award
              }
            } else {
              displayValue = award
            }

            // 判断颜色和是否为銘謝惠顧
            const color = name === '銘謝惠顧' ? '#333' : '#ef4444'
            const isThanks = name === '銘謝惠顧'

            return {
              id: prize.id,
              name: displayName,
              level: displayLevel,
              value: displayValue,
              color: color,
              fullName: name,
              desc: award,
              isThanks: isThanks
            }
          })
        } else {
          console.error('获取转盘信息失败:', result.msg)
          // 使用默认数据
          this.setDefaultPrizes()
        }
      } catch (error) {
        console.error('获取转盘信息接口调用失败:', error)
        // 使用默认数据
        this.setDefaultPrizes()
      }
    },
    setDefaultPrizes() {
      this.allPrizes = [
        { id: 1, name: '试炼倉位', level: '一獎', value: '+5%倉位\n3天', color: '#ef4444', fullName: '试炼倉位一獎', desc: '+5%倉位（3天）', isThanks: false },
        { id: 2, name: '试炼倉位', level: '二獎', value: '+5%倉位\n7天', color: '#ef4444', fullName: '试炼倉位二獎', desc: '+5%倉位（7天）', isThanks: false },
        { id: 3, name: '试炼倉位', level: '三獎', value: '+10%倉位\n7天', color: '#ef4444', fullName: '试炼倉位三獎', desc: '+10%倉位（7天）', isThanks: false },
        { id: 4, name: '銘謝', level: '惠顧', value: '', color: '#333', fullName: '銘謝惠顧', desc: '感謝參與', isThanks: true },
        { id: 5, name: '送福現金', level: '一獎', value: '40\nUSDT', color: '#ef4444', fullName: '送福現金一獎', desc: '40 USDT（價值1088TWD）', isThanks: false },
        { id: 6, name: '送福現金', level: '二獎', value: '100\nUSDT', color: '#ef4444', fullName: '送福現金二獎', desc: '100 USDT（價值2888TWD）', isThanks: false },
        { id: 7, name: '送福現金', level: '三獎', value: '2300\nUSDT', color: '#ef4444', fullName: '送福現金三獎', desc: '2300 USDT（價值68888TWD）', isThanks: false },
        { id: 8, name: '送福現金', level: '特獎', value: '5200\nUSDT', color: '#ef4444', fullName: '送福現金特獎', desc: '5200 USDT（價值158888TWD）', isThanks: false },
        { id: 9, name: '黃金獎', level: '黃金', value: '玫瑰花', color: '#ef4444', fullName: '黃金獎', desc: '黃金玫瑰花', isThanks: false },
        { id: 10, name: '黃金佳獎', level: '曦光', value: '墜子', color: '#ef4444', fullName: '黃金佳獎', desc: '曦光墜子', isThanks: false },
        { id: 11, name: '黃金豪獎', level: '迎鶴', value: '金條', color: '#ef4444', fullName: '黃金豪獎', desc: '迎鶴金條', isThanks: false },
        { id: 12, name: '黃金禮獎', level: '財神爺', value: '金條', color: '#ef4444', fullName: '黃金禮獎', desc: '財神爺金條', isThanks: false }
      ]
    },

    showToast(message, type = 'error') {
      this.toast.show = true
      this.toast.message = message
      this.toast.type = type

      setTimeout(() => {
        this.toast.show = false
      }, 3000)
    },

    async handleLottery(code) {
      try {
        // 调用抽奖接口
        const response = await fetch(`${window.location.origin}/renren-fast/app/weel/wheelLottery`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json'
          },
          body: JSON.stringify({
            wheelNo: code
          })
        })

        const result = await response.json()

        if (result.code === 0 && result.data) {
          // 根据返回的奖品 ID 找到对应的索引
          const prizeId = result.data.id
          const finalIndex = this.allPrizes.findIndex(prize => prize.id === prizeId)

          if (finalIndex === -1) {
            this.showToast('獎品數據異常，請重試')
            return
          }

          // 调用子组件的旋转方法
          const lotteryComponent = this.isMobile ? this.$refs.lotteryMobile : this.$refs.lotteryPC
          lotteryComponent.startRotate(finalIndex)
        } else {
          this.showToast(result.msg || '抽獎失敗，請重試')
        }
      } catch (error) {
        console.error('抽獎接口調用失敗:', error)
        this.showToast('網絡錯誤，請重試')
      }
    }
  }
}
</script>

<style scoped>
.toast {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  padding: 12px 24px;
  border-radius: 8px;
  color: white;
  font-size: 14px;
  z-index: 9999;
  animation: slideDown 0.3s ease-out;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.toast.error {
  background-color: #211d25;
}

.toast.success {
  background-color: #10b981;
}

.toast.info {
  background-color: #3b82f6;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateX(-50%) translateY(-20px);
  }

  to {
    opacity: 1;
    transform: translateX(-50%) translateY(0);
  }
}
</style>
