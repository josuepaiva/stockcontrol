<script setup lang="ts">
import { ref } from 'vue'

const currentRoute = ref('home')

function navigateTo(route: string) {
  currentRoute.value = route
  emit('navigate', route)
}

const emit = defineEmits(['navigate'])
</script>

<template>
  <div class="sidebar">
    <div class="logo-section">
      <img src="@/assets/profile.jpg" alt="Profile" class="profile-image" />
    </div>

    <nav class="navigation">
      <ul>
        <li>
          <a 
            @click="navigateTo('home')" 
            :class="{ active: currentRoute === 'home' }"
          >
            <i class="fas fa-home"></i>
            Perfil
          </a>
        </li>
        <li class="config-item">
          <a 
            @click="navigateTo('products')" 
            :class="{ active: currentRoute === 'products' }"
          >
            <i class="fas fa-cog"></i>
            Produtos
          </a>
        </li>
      </ul>
    </nav>
  </div>
</template>

<style scoped>
.sidebar {
  width: 200px;
  background-color: #1a1a1a;
  color: white;
  height: 100vh;
  position: fixed;
  left: 0;
  top: 0;
  padding-top: 20px;
}

.logo-section {
  text-align: center;
  padding: 20px 0;
}

.profile-image {
  width: 120px;
  height: 120px;
  border-radius: 50%;
  border: 3px solid #333;
}

.navigation {
  margin-top: 30px;
}

.navigation ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.navigation li {
  margin: 5px 0;
}

.navigation a {
  display: flex;
  align-items: center;
  padding: 12px 20px;
  color: #999;
  text-decoration: none;
  cursor: pointer;
  transition: all 0.3s ease;
}

.navigation a:hover,
.navigation a.active {
  color: #fff;
  background-color: rgba(255, 255, 255, 0.1);
}

.navigation i {
  margin-right: 10px;
  width: 20px;
  text-align: center;
}

.config-item {
  margin-top: 10px;
}
</style> 