<template>
  <div class="spreadsheet-container">

    <!-- Jspreadsheet -->
    <div
      id="worksheet"
      ref="refspreadsheet"
    ></div>

    <!-- Vue側のリアクティブデータ確認用 -->
    <div class="debug-panel">
      <h3>Vue reactive: sheetData</h3>

      <pre>{{ JSON.stringify(sheetData, null, 2) }}</pre>
    </div>

  </div>
</template>

<script setup>
import {
  ref,
  shallowRef,
  computed,
  onMounted,
  onBeforeUnmount
} from 'vue'

import jspreadsheet from 'jspreadsheet-ce'
import 'jspreadsheet-ce/dist/jspreadsheet.css'

import 'jsuites'
import 'jsuites/dist/jsuites.css'


/**
 * --------------------------------------------------
 * DOM参照
 * --------------------------------------------------
 */
const refspreadsheet = ref(null)


/**
 * --------------------------------------------------
 * Jspreadsheetのデータ
 *
 * Vueのリアクティブデータ
 * --------------------------------------------------
 */
const sheetData = ref([
  [true, 1, '2024-01-01', '', '', '', '', '', 'JA', 'foo'],
  [true, '', '', 'NHR', '肥料費', 50000, '', '=G2-F2', '', ''],
  [true, '', '', 'KBZ', '仮払い消費税', 5000, '', '=SUM(G1:G3)-SUM(F1:F3)', '', ''],
  [false, 2, '2024-01-02', '', '', '', '', '', '○○○○', 'foo'],
  [false, '', '', 'URA', '売上', '', 80000, '=SUM(G1:G5)-SUM(F1:F5)', '', ''],
  [false, '', '', 'KUZ', '仮受け消費税', '', 8000, '=SUM(G1:G6)-SUM(F1:F6)', '', '']
])


/**
 * --------------------------------------------------
 * Jspreadsheetインスタンス
 *
 * 外部ライブラリのオブジェクトなので
 * reactive()ではなくshallowRef()を使用
 * --------------------------------------------------
 */
const jspreadsheetObj = shallowRef(null)


/**
 * --------------------------------------------------
 * Jspreadsheet設定
 * --------------------------------------------------
 */
const jSpreadSheetOptions = computed(() => {

  return {

    /**
     * 初期データ
     */
    data: sheetData.value,

    /**
     * 列設定
     */
    columns: [
      {
        type: 'checkbox',
        title: ' ',
        width: 25
      },
      {
        type: 'numeric',
        title: 'No.',
        width: 50
      },
      {
        type: 'text',
        title: '取引日付',
        width: 100
      },
      {
        type: 'text',
        title: ' ',
        width: 60
      },
      {
        type: 'text',
        title: '勘定科目',
        width: 140
      },
      {
        type: 'numeric',
        title: '借方',
        width: 80,
        decimal: ','
      },
      {
        type: 'numeric',
        title: '貸方',
        width: 80,
        decimal: ','
      },
      {
        type: 'numeric',
        title: '残高',
        width: 80,
        decimal: ','
      },
      {
        type: 'text',
        title: '摘要 / 参照',
        width: 150
      },
      {
        type: 'text',
        title: '事業主',
        width: 100
      }
    ],

    /**
     * 表のスクロール
     */
    tableOverflow: true,

    /**
     * 表の高さ
     */
    tableHeight: 400,


    /**
     * ------------------------------------------------
     * セル編集確定時
     * ------------------------------------------------
     *
     * Jspreadsheetから
     *
     *   x       : 列番号
     *   y       : 行番号
     *   newValue: 新しい値
     *   oldValue: 変更前の値
     *
     * を受け取る。
     *
     * 全データを差し替えるのではなく、
     * 変更されたセルだけVue側へ反映する。
     */
    onchange: (
      instance,
      cell,
      x,
      y,
      newValue,
      oldValue
    ) => {

      console.log(
        '[Jspreadsheet onchange]',
        {
          x,
          y,
          newValue,
          oldValue
        }
      )

      /**
       * 変更されたセルだけ更新
       *
       * 例：
       * x = 4
       * y = 1
       *
       * → sheetData[1][4]
       */
      if (
        sheetData.value[y] &&
        x !== undefined &&
        y !== undefined
      ) {
        sheetData.value[y][x] = newValue
      }
    }
  }
})


/**
 * --------------------------------------------------
 * mounted
 * --------------------------------------------------
 */
onMounted(() => {

  /**
   * Jspreadsheet生成
   */
  jspreadsheetObj.value = jspreadsheet(
    refspreadsheet.value,
    jSpreadSheetOptions.value
  )


  /**
   * ------------------------------------------------
   * グレー表示するセル
   * ------------------------------------------------
   */
  const gray = '#DDD'
  const back = 'background-color'

  const grayCells = [
    'A1', 'B1', 'C1', 'D1', 'E1',
    'F1', 'G1', 'H1', 'I1', 'J1',

    'A4', 'B4', 'C4', 'D4', 'E4',
    'F4', 'G4', 'H4', 'I4', 'J4'
  ]

  grayCells.forEach(cell => {

    jspreadsheetObj.value.setStyle(
      cell,
      back,
      gray
    )

  })

})


/**
 * --------------------------------------------------
 * コンポーネント破棄時
 * --------------------------------------------------
 */
onBeforeUnmount(() => {

  if (
    jspreadsheetObj.value &&
    typeof jspreadsheetObj.value.destroy === 'function'
  ) {

    jspreadsheetObj.value.destroy()

    jspreadsheetObj.value = null
  }

})
</script>


<style scoped>

.spreadsheet-container {
  width: 100%;
}


/**
 * Vue側のデータ確認エリア
 */
.debug-panel {
  margin-top: 30px;
  padding: 15px;

  border: 1px solid #ccc;

  background: #f8f8f8;
}


.debug-panel h3 {
  margin-top: 0;
  margin-bottom: 10px;

  font-size: 16px;
}


.debug-panel pre {
  margin: 0;

  padding: 10px;

  overflow: auto;

  max-height: 400px;

  background: white;

  border: 1px solid #ddd;

  font-size: 12px;

  line-height: 1.4;
}

</style>
