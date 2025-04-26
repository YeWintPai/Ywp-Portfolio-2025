<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { gsap } from 'gsap';

const cursor = ref<HTMLElement | null>(null)

// Expose the cursor element to the parent
defineExpose({ cursor });

onMounted(() => {
    // Custom cursor follow
    document.addEventListener('mousemove', (e) => {
        gsap.to(cursor.value, {
            x: e.clientX - 12,
            y: e.clientY - 12,
            duration: 0.2,
            opacity:1,
            ease: 'power2.out'
        })
    })
})
</script>

<style scoped>
    #cursor {
    transition: transform 0.1s ease;
    }
</style>

<template>
    <!-- Custom Cursor -->
    <img
     :src="`storage/images/black-star-4-corner.png`"
     ref="cursor" 
     class="fixed opacity-0 w-10 h-10 rounded-full pointer-events-none z-40"
    />
</template>
