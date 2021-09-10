<template>
  <div class="container">
    <div>
      <Logo />
      <div class="text-5xl text-gray-800 dark:text-white text my-2">
        👋 Hey there!
      </div>
      <div class="dark:text-white my-4">
        I'm Gurvan currently following the Solidity + Ethereum Smart Contracts
        course at buildspace.so. Connect your Ethereum wallet and wave at me!
      </div>
      <button
        v-if="!currAccount"
        class="btn btn-wide btn-large"
        @click="connectWallet"
      >
        Connect Wallet
      </button>
      <button
        v-if="currAccount"
        :class="[
          'btn btn-wide btn-large',
          {
            loading: isWaving,
          },
        ]"
        @click="wave"
      >
        {{ isWaving ? '' : 'Wave at Me' }}
      </button>
      <div class="dark:text-white mt-3">Total waves: {{ wavesCount }}</div>
    </div>
  </div>
</template>

<script setup>
import { ethers } from 'ethers'
import confetti from 'canvas-confetti'
import { onMounted, ref, reactive, useContext } from '@nuxtjs/composition-api'

import WavePortal from '@/contracts/WavePortal.json'

const { $config } = useContext()

const currAccount = ref('')
const wavesCount = ref(0)
const isWaving = ref(false)

let wavePortalContract = reactive({})

const isWalletConnected = async () => {
  const { ethereum } = window
  // Check if we have access to an authorized account
  const accounts = await ethereum.request({ method: 'eth_accounts' })

  if (accounts && accounts.length) currAccount.value = accounts[0]
}

const connectWallet = async () => {
  const { ethereum } = window
  if (!ethereum) alert('Get Metamask extension!')

  const accounts = await ethereum.request({ method: 'eth_requestAccounts' })

  if (accounts && accounts.length) currAccount.value = accounts[0]
}

// eslint-disable-next-line no-unused-vars
const setWavePortalContract = () => {
  const provider = new ethers.providers.Web3Provider(window.ethereum)
  const signer = provider.getSigner()
  const contractABI = WavePortal.abi
  wavePortalContract = new ethers.Contract(
    $config.wavePortalContract,
    contractABI,
    signer
  )
}

const wave = async () => {
  const waveTxn = await wavePortalContract.wave()
  isWaving.value = true
  await waveTxn.wait()
  isWaving.value = false
  await getTotalWaves()
  confetti()
}

const getTotalWaves = async () => {
  const count = await wavePortalContract.getTotalWaves()
  wavesCount.value = Number(count)
}

onMounted(async () => {
  try {
    await isWalletConnected()
    await setWavePortalContract()
    await getTotalWaves()
  } catch {}
})
</script>

<style>
.container {
  @apply min-h-screen flex justify-center items-center text-center mx-auto;
}
</style>
