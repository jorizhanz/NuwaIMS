<template>
    <div class="background">
      <div class="welcome-container">
        <img src="../assets/nuwa-logo-full.png" alt="Logo" class="logo">
        <h1 class="welcome-message">Welcome to Nuwa's Inventory Management System!</h1>
      </div>
    </div>
  </template>
  
  <script>
  export default {
    name: 'Welcome',
  }
  </script>
  
  <style scoped>
  html, body {
    margin: 0;
    padding: 0;
    height: 100%;
    width: 100%;
  }
  
  .background {
    background-image: url('../assets/home-bg.jpg');
    background-size: cover;
    background-position: center;
    height: 100vh;
    width: 100vw;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  
  .welcome-container {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
  
  .logo {
    max-width: 500px;
    height: auto;
  }
  
  .welcome-message {
    font-size: 24px;
    margin-top: 20px;
    color: white;
  }
  </style>
  