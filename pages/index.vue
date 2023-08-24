<template>
    <v-form v-model="valid">
        <!-- ローディング表示(全画面) -->
        <v-overlay v-model="loading" opacity="0.8" :persistent="true" class="align-center justify-center">
            <v-progress-circular indeterminate="loading" size="64" />
        </v-overlay>
        <v-container>
            <p>下記APIを利用して郵便番号を検索します</p>
            <p>
                <a href="https://zipcloud.ibsnet.co.jp/doc/api" target="_blank" rel="noopener noreferrer">
                    郵便番号検索API - zipcloud
                </a>
            </p>
            <!-- エラーメッセージ -->
            <v-alert type="error" v-model="errorText" v-if="errorText !== null" dense dismissible>{{ errorText }}
            </v-alert>
            <v-alert type="warning" v-model="warnText" v-if="warnText !== null" dense dismissible>{{ warnText }}
            </v-alert>
        </v-container>
        <v-container>
            <!-- 検索ダイアログ -->
            <v-text-field type="tel" v-model="zipcode" :rules="zipcodeRules" maxlength="8" :counter="7" label="郵便番号"
                @blur="removeHyphen" required></v-text-field>
            <!-- 検索ボタン -->
            <v-btn color="primary" :disabled="!valid || loading" @click="search">検索</v-btn>
        </v-container>
        <v-container>
            <!-- 検索結果 -->
            <v-data-table v-if="results !== null" :headers="headers" :items="results" :items-per-page="5" />
        </v-container>
    </v-form>
</template>

<script setup>
import { ref } from 'vue'
import { VDataTable } from 'vuetify/labs/VDataTable'

useHead({ title: '郵便番号検索' })

const valid = ref(false)        // 検証ステータス
const errorText = ref(null)     // エラーメッセージ
const warnText = ref(null)      // 警告メッセージ
const zipcode = ref('')         // 郵便番号（検索キー）
const loading = ref(false)      // 検索中
const results = ref(null)       // 検索結果
// 郵便番号検証ルール
const zipcodeRules = [
    v => !!v || '郵便番号は必須です。',
    v => v.length === 7 || '郵便番号は7桁で入力してください。',
    v =>
        /[0-9-]{7,8}/.test(v) ||
        '郵便番号はハイフンなしの数字で入力してください。',
]
// 検索結果一覧ヘッダー
const headers = [
    { title: '郵便番号', key: 'zipcode' },
    { title: '都道府県コード', key: 'prefcode' },
    { title: '都道府県名', key: 'address1' },
    { title: '市区町村名', key: 'address2' },
    { title: '町域名', key: 'address3' },
]

// 郵便番号成型(半角ハイフン除去)
const removeHyphen = () => {
    zipcode.value = zipcode.value.replace('-', '')
};

// 検索ボタン押下
const search = async () => {
    errorText.value = null
    warnText.value = null
    results.value = null

    // 郵便番号検索API呼び出し開始
    loading.value = true
    const url = 'https://zipcloud.ibsnet.co.jp/api/search?zipcode=' + zipcode.value
    try {
        const response = await fetch(url)

        // ネットワーク接続エラー
        if (!response.ok) {
            const errorMessage = await response.text()
            throw new Error(errorMessage || 'Network response was not ok')
        }

        const responseData = await response.json()
        // 検索結果なし
        if (responseData.results === null) {
            warnText.value = '指定された郵便番号は存在しません'
            return
        }

        // 検索成功
        results.value = responseData.results

    } catch (error) {
        errorText.value = '検索APIの呼び出しに失敗しました。'
        console.error(error)
    } finally {
        loading.value = false
    }
}
</script>
