<script setup lang="ts">
const router = useRouter()
const go = (e: any) => {
  router.push(`/${e.target.innerText}`)
}
</script>

<template>
  <div>
    <h1 text-3xl>Generative Art</h1>
    <div mt-2 grid grid-cols-5>
      <button btn p-1 text-xl @click="go">plum</button>
    </div>
  </div>
</template>
