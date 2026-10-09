<template>
  <div class="spreadsheet-container">

    <!-- Jspreadsheet -->
    <div
      id="worksheet"
      ref="refspreadsheet"
    ></div>


    <!--
      Vue側からsheetDataを変更するための入力欄
    -->
    <div class="input-panel">

      <h3>Vue → Jspreadsheet テスト</h3>

      <label>
        A2:
        <input
          v-model="sheetData[1][0]"
          type="text"
        />
      </label>

      <label>
        E2:
        <input
          v-model="sheetData[1][4]"
          type="text"
        />
      </label>

      <label>
        F2:
        <input
          v-model="sheetData[1][5]"
          type="text"
        />
      </label>

    </div>


    <!--
      Vue側の現在値確認用
    -->
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
  watch,
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
 * --------------------------------------------------
 */
const jspreadsheetObj = shallowRef(null)


/**
 * --------------------------------------------------
 * 「Jspreadsheet → Vue」の更新中かどうか
 *
 * trueの場合、
 * sheetData変更をwatchしても
 * setData()を実行しない。
 * --------------------------------------------------
 */
const updatingFromJspreadsheet = ref(false)


/**
 * --------------------------------------------------
 * Jspreadsheet設定
 * --------------------------------------------------
 */
const jSpreadSheetOptions = computed(() => {

  return {

    data: sheetData.value,

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

    tableOverflow: true,

    tableHeight: 400,


    /**
     * ----------------------------------------------
     * Jspreadsheet → Vue
     * ----------------------------------------------
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
        '[Jspreadsheet → Vue]',
        {
          x,
          y,
          newValue,
          oldValue
        }
      )


      if (
        sheetData.value[y] &&
        x !== undefined &&
        y !== undefined
      ) {

        /**
         * JspreadsheetからVueへの更新中
         */
        updatingFromJspreadsheet.value = true


        /**
         * 変更されたセルだけ更新
         */
        sheetData.value[y][x] = newValue


        /**
         * Vueのreactive更新処理が
         * 完了するタイミングを考慮して解除
         */
        setTimeout(() => {
          updatingFromJspreadsheet.value = false
        }, 0)
      }
    }
  }
})


/**
 * --------------------------------------------------
 * Vue → Jspreadsheet
 *
 * sheetDataが外部から変更された場合に
 * Jspreadsheetへ反映
 * --------------------------------------------------
 */
watch(
  sheetData,
  (newData) => {

    console.log(
      '[Vue → Jspreadsheet]',
      newData
    )


    /**
     * Jspreadsheetからの変更なら、
     * ここではsetDataしない。
     */
    if (updatingFromJspreadsheet.value) {
      return
    }


    /**
     * Jspreadsheetがまだ生成されていない場合
     */
    if (!jspreadsheetObj.value) {
      return
    }


    /**
     * Vue側で変更されたデータを
     * Jspreadsheetへ反映
     */
    jspreadsheetObj.value.setData(
      newData
    )

  },

  {
    deep: true
  }
)


/**
 * --------------------------------------------------
 * mounted
 * --------------------------------------------------
 */
onMounted(() => {

  jspreadsheetObj.value = jspreadsheet(
    refspreadsheet.value,
    jSpreadSheetOptions.value
  )


  /**
   * グレー表示
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
 * beforeUnmount
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
 * Vue → Jspreadsheet
 * テスト入力エリア
 */
.input-panel {

  margin-top: 25px;

  padding: 15px;

  border: 1px solid #aaa;

  background: #f5f5f5;
}


.input-panel h3 {

  margin-top: 0;

  font-size: 16px;
}


.input-panel label {

  display: block;

  margin: 8px 0;
}


.input-panel input {

  width: 300px;

  margin-left: 10px;

  padding: 5px;
}


/**
 * デバッグ表示
 */
.debug-panel {

  margin-top: 25px;

  padding: 15px;

  border: 1px solid #ccc;

  background: #f8f8f8;
}


.debug-panel h3 {

  margin-top: 0;

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
