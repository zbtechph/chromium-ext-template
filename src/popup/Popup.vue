<template>
    <div>
        <div class="py-2">
            <h3 class="text-2xl">ZFlix v0.1</h3>
            <p>by: zbyte@phcorner</p>
        </div>
        <div>
            <div class="flex mb-3">
                <input type="text" id="tokenTxt" class="p-1 w-3/4 text-gray-700 focus:outline-none" placeholder="Enter Token"/>
                <button class="p-1 bg-gray-500 rounded border" id="importBtn" type="button">IMPORT</button>
            </div>
            <div class="flex justify-between">
                <button class="p-1 bg-red-500 text-red-100 rounded border text-sm" id="clearBtn" type="button">CLEAR</button>
                <button class="p-1 text-blue-200 rounded text-sm" id="exportBtn">Export</button>
            </div>         
        </div>
    </div>
</template>

<script setup>
import { onMounted } from 'vue'

onMounted(()=>{
    console.log("test")
})

</script>