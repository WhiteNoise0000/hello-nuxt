<template>
    <v-form v-model="valid">
        <!-- ローディング表示(全画面) -->
        <v-overlay :opacity="0.8" :value="loading">
            <v-progress-circular indeterminate size="64"></v-progress-circular>
        </v-overlay>
        <p>下記APIを利用して郵便番号を検索します</p>
        <p>
            <a href="https://zipcloud.ibsnet.co.jp/doc/api" target="_blank" rel="noopener noreferrer">
                郵便番号検索API - zipcloud
            </a>
        </p>
        <v-divider></v-divider>
        <p>
            <!-- エラーメッセージ -->
            <v-alert type="error" v-model:="errorText" v-if="errorText != null" dense dismissible>{{ errorText }}
            </v-alert>
            <v-alert type="warning" v-model:="warnText" v-if="warnText != null" dense dismissible>{{ warnText }}
            </v-alert>
        </p>
        <p>
            <!-- 検索ダイアログ -->
            <v-text-field type="tel" v-model="zipcode" :rules="zipcodeRules" maxlength=8 :counter=7 label="郵便番号"
                @blur="removeHypen" required>
            </v-text-field>
        </p>
        <p>
            <!-- 検索ボタン -->
            <v-btn color="primary" :disabled="!valid || loading" @click="search">検索</v-btn>
        </p>
        <p>
            <!-- 検索結果 -->
            <v-data-table v-if="results != null" :headers="headers" :items="results" :items-per-page="5">
            </v-data-table>
        </p>
    </v-form>
</template>

<script>
import axios from 'axios';
export default {
    name: 'index',
    data: () => ({
        // 検証ステータス
        valid: false,
        // エラーメッセージ
        errorText: null,
        // 警告メッセージ
        warnText: null,
        // 郵便番号（検索キー）
        zipcode: '',
        // 郵便番号検証ルール
        zipcodeRules: [
            v => !!v || '郵便番号は必須です。',
            v => v.length == 7 || '郵便番号は7桁で入力してください。',
            v => /[0-9-]{7,8}/.test(v) || '郵便番号はハイフンなしの数字で入力してください。',
        ],
        // 検索中
        loading: false,
        // 検索結果一覧ヘッダー
        headers: [
            { text: '郵便番号', value: 'zipcode', },
            { text: '都道府県コード', value: 'prefcode' },
            { text: '都道府県名', value: 'address1' },
            { text: '市区町村名', value: 'address2' },
            { text: '町域名', value: 'address3' },
        ],
        // 検索結果
        results: null,
    }),
    methods: {
        // 郵便番号成型(半角ハイフン除去)
        removeHypen() {
            this.zipcode = this.zipcode.replace('-', '')
        },
        // 検索
        search() {
            this.errorText = null
            this.warnText = null
            this.results = null

            // 郵便番号検索API呼び出し開始
            this.loading = true
            const url = 'https://zipcloud.ibsnet.co.jp/api/search?zipcode=' + this.zipcode
            axios.get(url).then(res => {
                console.log(res.data)
                // 不正な検索
                if (res.status == 400) {
                    this.warnText = res.data.statusText
                    return
                }
                // 検索結果なし
                if (res.data.results == null) {
                    this.warnText = "指定された郵便番号は存在しません"
                    return
                }
                // 検索成功
                this.results = res.data.results
            }).catch(err => {
                // API呼び出し失敗
                this.errorText = "検索APIの呼び出しに失敗しました。"
            }).finally(() => {
                // 検索終了
                this.loading = false
            })
        },
    }
}
</script>
