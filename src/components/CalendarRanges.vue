<template>
    <div class="ranges">
        <div class="text-center py-2">*Forked package*</div>
        <ul v-if="ranges" class="hidden-xs">
            <li
                v-for="(value, key) in ranges"
                @click="$emit('clickRange', value)"
                :data-range-key="key"
                :key="key"
            >{{key}}</li>
        </ul>
    </div>
</template>

<script>
    export default {
        props: ['ranges']
    }
</script>
