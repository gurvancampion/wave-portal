<template>
  <div class="container">
    <div>
      <div class="text-5xl my-2">👋 Hey there!</div>
      <div class="my-4">
        <p>
          I'm Gurvan, currently following the Solidity + Ethereum Smart
          Contracts course at
          <a class="link-primary" href="https://buildspace.so/" target="_blank">
            buildspace.so
          </a>
        </p>
        <p>
          Connect your Ethereum wallet via Metamask (Use Rinkeby Network) and
          wave at me!
        </p>
        <p>PS: Get a chance to win some ETH 😉</p>
      </div>

      <button
        v-if="!currAccount"
        class="btn btn-primary btn-wide btn-large"
        @click="connectWallet"
      >
        Connect Wallet
      </button>

      <button
        v-if="currAccount"
        class="btn btn-primary btn-wide btn-large"
        @click="displayDialog = true"
      >
        Wave at me
      </button>

      <div class="mt-3">Total waves: {{ waves.length }}</div>

      <div v-if="waves.length > 0" class="overflow-x-auto mt-3">
        <table class="table w-full">
          <thead>
            <tr>
              <th>Address</th>
              <th>Message</th>
              <th>Won prize</th>
              <th>Datetime</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="(w, index) in waves" :key="index">
              <th class="!bg-transparent">{{ w.address }}</th>
              <th class="!bg-transparent">{{ w.message }}</th>
              <th class="!bg-transparent">
                {{ w.wonPrize ? '🎉' : '❌' }}
              </th>
              <th class="!bg-transparent">{{ w.timestamp }}</th>
            </tr>
          </tbody>
        </table>
      </div>

      <div
        v-if="displayDialog"
        :class="['modal', { 'modal-open	': displayDialog }]"
      >
        <div class="modal-box">
          <div class="form-control">
            <label class="label">
              <span class="label-text">Your message:</span>
            </label>
            <textarea
              v-model="message"
              class="textarea textarea-bordered h-24"
              placeholder="Write your message!"
            />
          </div>

          <div class="modal-action">
            <button
              :class="[
                'btn btn-primary',
                {
                  loading: isWaving,
                },
              ]"
              @click="wave"
            >
              Send
            </button>
            <button
              :class="[
                'btn',
                {
                  'btn-disabled': isWaving,
                },
              ]"
              @click="displayDialog = false"
            >
              Close
            </button>
          </div>
        </div>
      </div>
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
const isWaving = ref(false)
const displayDialog = ref(false)
const message = ref('')
const waves = ref([])
let wavePortalContract = reactive({})

const setWallet = async () => {
  const { ethereum } = window
  // Check if we have access to an authorized account
  const accounts = await ethereum.request({ method: 'eth_accounts' })

  if (accounts && accounts.length) {
    currAccount.value = accounts[0]
    await getAllWaves()
  }
}

const connectWallet = async () => {
  const { ethereum } = window
  if (!ethereum) alert('Get Metamask extension!')

  const accounts = await ethereum.request({ method: 'eth_requestAccounts' })

  if (accounts && accounts.length) {
    currAccount.value = accounts[0]
    await getAllWaves()
  }
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

  wavePortalContract.on('NewWave', (from, message, wonPrize, timestamp) => {
    waves.value.unshift({
      address: from,
      message,
      wonPrize,
      timestamp: new Date(timestamp * 1000).toLocaleString(),
    })
    if (wonPrize) confetti()
  })
}

const resetForm = () => {
  // Reset form
  displayDialog.value = false
  message.value = ''
}

const wave = async () => {
  try {
    const waveTxn = await wavePortalContract.wave(message.value, {
      gasLimit: 300000,
    })
    isWaving.value = true
    await waveTxn.wait()
    isWaving.value = false
    resetForm()
  } catch {
    isWaving.value = false
  }
}

const getAllWaves = async () => {
  waves.value = (await wavePortalContract.getAllWaves())
    .map((wave) => {
      return {
        address: wave.waver,
        message: wave.message,
        wonPrize: wave.wonPrize,
        timestamp: new Date(wave.timestamp * 1000).toLocaleString(),
      }
    })
    .reverse()
}

onMounted(async () => {
  try {
    await setWallet()
    await setWavePortalContract()
  } catch {}
})
</script>

<style>
.container {
  @apply min-h-screen flex justify-center items-center text-center mx-auto;
}
</style>
