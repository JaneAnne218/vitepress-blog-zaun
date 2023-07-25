<template>
    <div class="outerBox">
      <div class="innerBox">
        <div class="side-rowbox">
          <div class="circle"></div>
          <div class="circle"></div>
        </div>
        <div class="center-rowbox">
          <div class="circle"></div>
        </div>
        <div class="side-rowbox">
          <div class="circle"></div>
          <div class="circle"></div>
        </div>
      </div>
    </div>
  </template>
  
  <script setup lang="ts">
 
  </script>
  
  <style scoped>
  .outerBox {
    width: 100px;
    height: 100px;
    border: 1px solid #000;
    padding: 12px;
    box-sizing: border-box;
  }
  
  .innerBox {
    height: 76px;
    width: 76px;
    display: flex;
    flex-direction: column;
  }
  
  .side-rowbox {
    flex: 1;
    display: flex;
    justify-content: space-between;
  }
  
  .center-rowbox {
    flex: 1;
    display: flex;
    justify-content: center;
  }
  
  .circle {
    height: 20px;
    width: 20px;
    background-color: black;
    border-radius: 50%;
  }
  </style>
  