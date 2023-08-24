<template>
    <v-app>
        <v-app-bar color="primary">
            <v-app-bar-nav-icon variant="text" @click.stop="drawer = !drawer" />
            <v-app-bar-title>{{ title }}</v-app-bar-title>
        </v-app-bar>

        <v-navigation-drawer v-model="drawer">
            <v-list>
                <v-list-item v-for="item in items" :title="item.title" :to="item.to"
                    :prepend-icon="item.icon || null" />
                <!-- GitHubへのリンク -->
                <v-list-item title="GitHubリポジトリ" href="https://github.com/WhiteNoise0000/hello-nuxt" target="_blank"
                    rel="noopener" prepend-icon="mdi-github" />
            </v-list>
        </v-navigation-drawer>


        <v-main>
            <v-container>
                <slot />
            </v-container>
        </v-main>
    </v-app>
</template>
  
<script setup>
import { ref } from 'vue'
const drawer = ref(null)
</script>
  
<script>
export default {
    data: () => ({
        drawer: null,
        items: [
            {
                icon: 'mdi-map-search',
                title: '郵便番号検索',
                to: '/'
            },
        ],
        title: '郵便番号検索'
    })
}
</script>
  