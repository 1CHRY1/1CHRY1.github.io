<template>
  <VPTeamPage>
    <VPTeamPageTitle>
      <template #title>🎯 Projects ✨</template>
      <template #lead>My Open Source Projects</template>
    </VPTeamPageTitle>
    <div class="px-6 md:px-12 lg:px-16">
      <div class="ProjectsContent" v-if="projectsInfo.length > 0">
        <ProjectCard
          v-for="(item, index) in projectsInfo"
          :key="index"
          v-bind="item"
        />
      </div>
      <div class="ProjectsContent" v-else>
        <div class="text-1xl text-center">🚧 施工中...</div>
      </div>
    </div>
  </VPTeamPage>
</template>

<script setup lang="ts">
import { VPTeamPage, VPTeamPageTitle } from 'vitepress/theme';
import ProjectCard from './ProjectCard.vue';
import { projectsInfo } from '../userConfig/projectsInfo';
</script>

<style scoped>
.ProjectsContent {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(288px, 1fr));
  max-width: 1152px;
  gap: 24px;
  margin: 0 auto;
}
</style>
