<template>
  <div>
    <button @click="showModal = true"><slot></slot></button>
    <modal v-if="showModal">I am a modal for {{ button }}</modal>
  </div>
</template>
