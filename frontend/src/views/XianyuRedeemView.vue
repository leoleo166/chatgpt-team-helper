<template>
  <RedeemShell>
    <div class="text-center space-y-6">
      <div class="inline-flex items-center gap-2.5 rounded-full bg-white/60 dark:bg-white/10 backdrop-blur-xl border border-white/40 dark:border-white/10 px-4 py-1.5 shadow-sm transition-transform hover:scale-105 duration-300 cursor-default">
        <span class="relative flex h-2.5 w-2.5">
          <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-blue-400 opacity-75"></span>
          <span class="relative inline-flex rounded-full h-2.5 w-2.5 bg-[#0A84FF]"></span>
        </span>
        <span class="text-[13px] font-medium text-gray-600 dark:text-gray-300 tracking-wide">闲鱼订单专用 · 自动匹配兑换码</span>
      </div>

      <div class="space-y-3">
        <h1 class="text-[40px] leading-tight font-extrabold tracking-tight text-transparent bg-clip-text bg-gradient-to-r from-blue-500 via-cyan-500 to-teal-400 dark:from-blue-400 dark:via-cyan-400 dark:to-teal-300 drop-shadow-sm animate-gradient-x">
          闲鱼订单兑换
        </h1>
      </div>
    </div>

    <div class="relative group perspective-1000">
      <div class="absolute -inset-1 bg-gradient-to-r from-blue-500 via-cyan-500 to-teal-400 rounded-[2rem] blur opacity-25 group-hover:opacity-50 transition duration-1000 group-hover:duration-200 animate-tilt"></div>
      <AppleCard
        variant="glass"
        class="relative overflow-hidden shadow-2xl shadow-black/10 border border-white/40 dark:border-white/10 ring-1 ring-black/5 backdrop-blur-3xl transition-all duration-500 hover:shadow-3xl hover:scale-[1.01] animate-float"
      >
        <div class="p-8 sm:p-10 space-y-8">
          <form @submit.prevent="handleRedeem" class="space-y-8">
            <div
              class="space-y-2 group animate-in fade-in slide-in-from-bottom-4 duration-700 delay-100 fill-mode-backwards"
              :class="{ 'animate-shake': errorMessage && !formData.email }"
            >
              <AppleInput
                v-model.trim="formData.email"
                label="您的账号邮箱地址"
                placeholder="name@example.com"
                type="email"
                variant="filled"
                :disabled="isLoading"
                helperText="请输入用于接收 ChatGPT 邀请的账号邮箱地址"
                :error="formData.email && !isValidEmail ? '请输入有效的账号邮箱地址' : ''"
                class="transition-all duration-300 group-hover:translate-x-1"
              />
            </div>

            <div
              class="space-y-2 group animate-in fade-in slide-in-from-bottom-4 duration-700 delay-200 fill-mode-backwards"
              :class="{ 'animate-shake': errorMessage && !formData.orderId }"
            >
              <AppleInput
                v-model="formData.orderId"
                label="您的兑换码"
                placeholder="请输入您的兑换码激活"
                type="text"
                variant="filled"
                :disabled="isLoading"
                helperText="您的兑换码即闲鱼自动回复时发送的兑换码"
                :error="formData.orderId && !isValidOrderId ? '兑换码格式不正确，请输入纯数字兑换码' : ''"
                @input="handleOrderInput"
                class="transition-all duration-300 group-hover:translate-x-1"
              />
            </div>

            <div class="pt-2 animate-in fade-in slide-in-from-bottom-4 duration-700 delay-300 fill-mode-backwards">
              <AppleButton
                type="submit"
                variant="primary"
                size="lg"
                class="w-full h-[50px] text-[17px] font-medium shadow-lg shadow-blue-500/20 hover:shadow-blue-500/30 transition-all duration-300 hover:scale-[1.02] active:scale-[0.98]"
                :loading="isLoading"
                :disabled="isLoading"
              >
                {{ submitButtonLabel }}
              </AppleButton>
            </div>
          </form>

          <div
            v-if="successInfo"
            class="absolute inset-0 z-20 flex items-center justify-center p-6 bg-white/60 dark:bg-black/60 backdrop-blur-md rounded-[2rem] animate-in fade-in duration-300"
          >
            <div class="w-full rounded-2xl bg-[#34C759]/10 border border-[#34C759]/20 p-5 flex gap-4 shadow-lg backdrop-blur-xl">
              <div class="flex-shrink-0 mt-0.5">
                <div class="h-6 w-6 rounded-full bg-[#34C759] flex items-center justify-center shadow-sm">
                  <CheckCircle2 class="h-4 w-4 text-white" />
                </div>
              </div>
              <div class="flex-1 space-y-3">
                <h3 class="text-[15px] font-semibold text-[#1d1d1f] dark:text-white">兑换成功！</h3>
                <div class="text-[14px] text-[#1d1d1f]/80 dark:text-white/80 space-y-3">
                  <p>系统已完成兑换并将您加入 ChatGPT Team 账号。</p>
                  <div class="space-y-1">
                    <p class="text-[13px]">
                      <span class="text-[#86868b]">您的账号邮箱地址：</span>
                      <span class="font-medium">{{ successInfo.accountEmail }}</span>
                    </p>
                    <p class="text-[13px]">
                      <span class="text-[#86868b]">当前人数：</span>
                      <span class="font-medium">{{ successInfo.userCount }}</span>
                    </p>
                    <p class="text-[13px] flex items-center gap-2">
                      <span class="text-[#86868b]">邀请状态：</span>
                      <span
                        class="inline-flex items-center px-2 py-0.5 rounded-full text-[12px] font-medium"
                        :class="(successInfo.inviteStatus || '').includes('已发送') ? 'bg-[#34C759]/10 text-[#34C759]' : 'bg-[#FF9F0A]/10 text-[#FF9F0A]'"
                      >
                        {{ successInfo.inviteStatus || '未知' }}
                      </span>
                    </p>
                  </div>
                  <p class="text-[13px] leading-normal text-[#86868b]">
                    <template v-if="successInfo.inviteStatus && successInfo.inviteStatus.includes('已发送')">
                      请留意发送到您的账号邮箱地址的邀请邮件，接受后即可登录使用。
                    </template>
                    <template v-else>
                      如未收到自动邀请，请联系管理员并提供您的账号邮箱地址。
                    </template>
                  </p>
                  <div class="pt-1">
                    <button
                      type="button"
                      class="text-xs text-[#0A84FF] hover:text-[#0066cc] font-medium transition"
                      @click="successInfo = null"
                    >
                      继续兑换
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div v-if="errorMessage" class="animate-in fade-in slide-in-from-bottom-4 duration-500 ease-out-expo">
            <div class="rounded-2xl bg-[#FF3B30]/10 border border-[#FF3B30]/20 p-5 flex gap-4">
              <div class="flex-shrink-0 mt-0.5">
                <div class="h-6 w-6 rounded-full bg-[#FF3B30] flex items-center justify-center shadow-sm">
                  <AlertCircle class="h-4 w-4 text-white" />
                </div>
              </div>
              <div class="flex-1">
                <h3 class="text-[15px] font-semibold text-[#1d1d1f] dark:text-white">兑换失败</h3>
                <p class="mt-1 text-[14px] text-[#1d1d1f]/80 dark:text-white/80">{{ errorMessage }}</p>
              </div>
            </div>
          </div>

          <div class="pt-6 border-t border-gray-200/60 dark:border-white/10 space-y-3">
            <h4 class="text-[13px] font-semibold text-[#86868b] uppercase tracking-wider">使用说明</h4>
            <ul class="space-y-3 text-[14px] text-[#1d1d1f]/70 dark:text-white/70">
              <li class="flex items-start gap-3">
                <span class="h-1.5 w-1.5 rounded-full bg-[#0A84FF] mt-2 flex-shrink-0"></span>
                <RouterLink
                  to="/redeem/account-recovery"
                  class="text-[#0A84FF] hover:text-[#0066cc] font-medium transition"
                >
                  掉号补录请点此处
                </RouterLink>
              </li>
              <li class="flex items-start gap-3">
                <span class="h-1.5 w-1.5 rounded-full bg-[#0A84FF] mt-2 flex-shrink-0"></span>
                <span>兑换失败/未收到邀请邮件，请直接发送您的账号邮箱地址给客服处理（请不要发截图哦～）</span>
              </li>
              <li class="flex items-start gap-3">
                <span class="h-1.5 w-1.5 rounded-full bg-[#0A84FF] mt-2 flex-shrink-0"></span>
                <span>兑换通常需要 30-60 秒，请耐心等待并保持页面打开，系统完成后会自动提示</span>
              </li>
            </ul>
          </div>
        </div>
      </AppleCard>
    </div>
  </RedeemShell>
</template>

<script setup lang="ts">
import { computed, ref } from 'vue'
import AppleButton from '@/components/ui/apple/Button.vue'
import AppleCard from '@/components/ui/apple/Card.vue'
import AppleInput from '@/components/ui/apple/Input.vue'
import RedeemShell from '@/components/RedeemShell.vue'
import { redemptionCodeService } from '@/services/api'
import { EMAIL_REGEX } from '@/lib/validation'
import type { RedeemSuccessInfo } from '@/composables/useRedeemForm'
import { AlertCircle, CheckCircle2 } from 'lucide-vue-next'

const ORDER_REGEX = /^\d{6,30}$/

const formData = ref({
  email: '',
  orderId: ''
})
const isLoading = ref(false)
const loadingStage = ref<'idle' | 'check' | 'sync' | 'redeem'>('idle')
const errorMessage = ref('')
const successInfo = ref<RedeemSuccessInfo | null>(null)

const isValidEmail = computed(() => {
  if (!formData.value.email) return true
  return EMAIL_REGEX.test(formData.value.email.trim())
})

const isValidOrderId = computed(() => {
  if (!formData.value.orderId) return true
  return ORDER_REGEX.test(formData.value.orderId.trim())
})

const submitButtonLabel = computed(() => {
  if (!isLoading.value) {
    return '立即兑换'
  }
  if (loadingStage.value === 'check') {
    return '正在核验兑换码...'
  }
  if (loadingStage.value === 'sync') {
    return '正在同步兑换信息...'
  }
  return '正在兑换...'
})

const normalizeXianyuUserMessage = (message: string) => {
  const normalized = message.trim()
  if (!normalized) {
    return '兑换失败，请稍后再试'
  }

  const specialMappings: Array<[RegExp, string]> = [
    [/订单详情解析失败，请确认订单号是否正确/g, '兑换码解析失败，请确认兑换码是否正确'],
    [/未找到对应订单，请确认订单号是否正确/g, '未找到对应兑换信息，请确认兑换码是否正确'],
    [/未找到对应订单，请稍后再试/g, '未找到对应兑换信息，请稍后再试'],
    [/查询订单失败，请稍后再试/g, '核验兑换码失败，请稍后再试'],
    [/请输入有效的闲鱼订单号/g, '请输入有效的兑换码'],
    [/请输入闲鱼订单号/g, '请输入您的兑换码'],
    [/请输入邮箱地址/g, '请输入您的账号邮箱地址'],
    [/请输入有效的邮箱地址/g, '请输入有效的账号邮箱地址']
  ]

  let userMessage = normalized
  for (const [pattern, replacement] of specialMappings) {
    userMessage = userMessage.replace(pattern, replacement)
  }

  return userMessage
    .replace(/数字订单号/g, '兑换码')
    .replace(/闲鱼订单号/g, '兑换码')
    .replace(/对应订单/g, '对应兑换信息')
    .replace(/订单号格式不正确/g, '兑换码格式不正确')
    .replace(/订单号是否正确/g, '兑换码是否正确')
    .replace(/订单号/g, '兑换码')
}

const handleOrderInput = (value: string | Event) => {
  let input = ''
  if (typeof value === 'string') {
    input = value
  } else {
    input = (value.target as HTMLInputElement).value
  }
  formData.value.orderId = input.replace(/[^\d]/g, '')
}

const handleRedeem = async () => {
  errorMessage.value = ''
  successInfo.value = null

  const normalizedEmail = formData.value.email.trim()
  const normalizedOrderId = formData.value.orderId.trim()

  if (!normalizedEmail) {
    errorMessage.value = '请输入您的账号邮箱地址'
    return
  }

  if (!isValidEmail.value) {
    errorMessage.value = '请输入有效的账号邮箱地址'
    return
  }

  if (!normalizedOrderId) {
    errorMessage.value = '请输入您的兑换码'
    return
  }

  if (!isValidOrderId.value) {
    errorMessage.value = '兑换码格式不正确，请检查后重试'
    return
  }

  isLoading.value = true
  loadingStage.value = 'check'
  try {
    const checkResponse = await redemptionCodeService.checkXianyuOrder({
      orderId: normalizedOrderId
    })

    let orderExists = Boolean(checkResponse.data?.order)

    if (!orderExists) {
      loadingStage.value = 'sync'
      await redemptionCodeService.syncXianyuOrder({
        orderId: normalizedOrderId
      })
      const verifyResponse = await redemptionCodeService.checkXianyuOrder({
        orderId: normalizedOrderId
      })
      orderExists = Boolean(verifyResponse.data?.order)
      if (!orderExists) {
        throw new Error('未找到对应兑换信息，请确认兑换码是否正确')
      }
    }

    loadingStage.value = 'redeem'
    const response = await redemptionCodeService.redeemXianyuOrder({
      email: normalizedEmail,
      orderId: normalizedOrderId
    })
    successInfo.value = {
      accountEmail: response.data.data.accountEmail,
      userCount: response.data.data.userCount,
      inviteStatus: response.data.data.inviteStatus
    }
    formData.value = {
      email: '',
      orderId: ''
    }
  } catch (error: any) {
    const status = error?.response?.status
    const errorCode = error?.response?.data?.errorCode
    const raw =
      error?.response?.data?.error ||
      error?.response?.data?.message ||
      error?.message ||
      ''
    const rawMessage = typeof raw === 'string' && raw.trim() ? raw : '兑换失败，请稍后再试'
    const message = normalizeXianyuUserMessage(rawMessage)

    const shouldContactSupport =
      errorCode === 'xianyu_codes_not_configured' ||
      errorCode === 'xianyu_no_today_codes' ||
      errorCode === 'xianyu_today_codes_exhausted' ||
      errorCode === 'xianyu_codes_unavailable' ||
      (status === 503 && rawMessage.includes('闲鱼') && rawMessage.includes('兑换码') && rawMessage.includes('今日'))

    errorMessage.value = shouldContactSupport ? '暂无可用兑换码，请联系管理员补货' : message
  } finally {
    isLoading.value = false
    loadingStage.value = 'idle'
  }
}
</script>

<style scoped>
@keyframes shake {
  0%,
  100% {
    transform: translateX(0);
  }
  25% {
    transform: translateX(-4px);
  }
  75% {
    transform: translateX(4px);
  }
}

.animate-shake {
  animation: shake 0.4s cubic-bezier(0.36, 0.07, 0.19, 0.97) both;
}
</style>
