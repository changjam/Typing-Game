<script setup>
import KeyBoard from './components/KeyBoard.vue';
</script>

<template>
  <KeyBoard/>
</template>

<style lang="scss">
    @import "./assets/style.scss";

    *{
        box-sizing: border-box;
        padding: 0px;
        margin: 0px;
    }

    body{
        font-family: sans-serif;
        background-color: $bgColor;
        color: $textPrimary;
    }

</style>
