<template>
  <div>
    <div class="search-container">
      <input
        v-model="searchKeyword"
        placeholder="輸入關鍵字"
      />
    </div>
    <div class="table-container">
      <table class="custom-table">
        <thead>
          <tr>
            <th @click="sortByColumn(0)">
              代稱
              <div
                class="sort-icon"
                :class="{
                  ascending:
                    sortDirection.column === 0 && sortDirection.ascending,
                  descending:
                    sortDirection.column === 0 && !sortDirection.ascending,
                }"
              >
                <span class="arrow-up"></span>
                <span class="arrow-down"></span>
              </div>
            </th>
            <th @click="sortByColumn(1)">
              名稱
              <div
                class="sort-icon"
                :class="{
                  ascending:
                    sortDirection.column === 1 && sortDirection.ascending,
                  descending:
                    sortDirection.column === 1 && !sortDirection.ascending,
                }"
              >
                <span class="arrow-up"></span>
                <span class="arrow-down"></span>
              </div>
            </th>
            <th @click="sortByColumn(2)">
              最高
              <div
                class="sort-icon"
                :class="{
                  ascending:
                    sortDirection.column === 2 && sortDirection.ascending,
                  descending:
                    sortDirection.column === 2 && !sortDirection.ascending,
                }"
              >
                <span class="arrow-up"></span>
                <span class="arrow-down"></span>
              </div>
            </th>
            <th @click="sortByColumn(3)">
              最低
              <div
                class="sort-icon"
                :class="{
                  ascending:
                    sortDirection.column === 3 && sortDirection.ascending,
                  descending:
                    sortDirection.column === 3 && !sortDirection.ascending,
                }"
              >
                <span class="arrow-up"></span>
                <span class="arrow-down"></span>
              </div>
            </th>
            <th @click="sortByColumn(4)">
              振幅
              <div
                class="sort-icon"
                :class="{
                  ascending:
                    sortDirection.column === 4 && sortDirection.ascending,
                  descending:
                    sortDirection.column === 4 && !sortDirection.ascending,
                }"
              >
                <span class="arrow-up"></span>
                <span class="arrow-down"></span>
              </div>
            </th>
          </tr>
        </thead>
      </table>
    </div>
    <div class="table-content">
      <table class="custom-table">
        <tbody>
          <tr
            v-for="(item, index) in filteredItems"
            :key="index"
          >
            <td
              v-for="(cell, cellIndex) in item"
              :key="cellIndex"
              :class="{ highlighted: shouldHighlight(cell) }"
            >
              <span v-html="highlightKeyword(cell)"></span>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";
import * as XLSX from "xlsx";

const items = ref([]);
const sortDirection = ref({ column: null, ascending: true });
const searchKeyword = ref("");

const fetchData = async () => {
  try {
    const response = await axios.get("output.xlsx", {
      responseType: "arraybuffer",
    });
    const arrayBuffer = response.data;
    const data = parseExcelData(arrayBuffer);
    items.value = data;
  } catch (error) {
    console.error("讀取資料失敗", error);
  }
};

const parseExcelData = (arrayBuffer) => {
  const data = [];
  const workbook = XLSX.read(arrayBuffer, { type: "array" });
  const worksheet = workbook.Sheets[workbook.SheetNames[0]];
  const rows = XLSX.utils.sheet_to_json(worksheet, { header: 1 });

  for (let i = 1; i < rows.length; i++) {
    const row = rows[i];
    const thirdColumn = parseFloat(row[2]);
    const fourthColumn = parseFloat(row[3]);

    // 計算第五列的值
    let fifthColumn = (thirdColumn / fourthColumn - 1) * 100;

    // 檢查是否為NaN或Infinity，是的話就顯示0
    if (isNaN(fifthColumn) || !isFinite(fifthColumn)) {
      fifthColumn = 0;
    }

    row.push(fifthColumn.toFixed(1) + "%"); // 四捨五入取小數點第一位

    data.push(row);
  }

  return data;
};

const sortByColumn = (columnIndex) => {
  const column = columnIndex;
  const ascending =
    sortDirection.value.column === column
      ? !sortDirection.value.ascending
      : true;

  sortDirection.value = { column, ascending };
  items.value = bubbleSort(items.value, column, ascending);
};

const shouldHighlight = (cell) => {
  if (!searchKeyword.value) {
    return false;
  }
  const lowerSearchTerm = searchKeyword.value.toLowerCase();
  return cell.toString().toLowerCase().includes(lowerSearchTerm);
};

const highlightKeyword = (cell) => {
  if (!searchKeyword.value) {
    return cell; // 如果沒搜索內容不反色
  }
  const lowerSearchTerm = searchKeyword.value.toLowerCase();
  const cellText = cell.toString();
  const startIndex = cellText.toLowerCase().indexOf(lowerSearchTerm);
  if (startIndex !== -1) {
    const highlightedText =
      cellText.substring(0, startIndex) +
      "<span class='highlight'>" +
      cellText.substring(startIndex, startIndex + lowerSearchTerm.length) +
      "</span>" +
      cellText.substring(startIndex + lowerSearchTerm.length);
    return highlightedText;
  } else {
    return cell; // 都沒匹配顯示全資料
  }
};

const filteredItems = computed(() => {
  return items.value.filter((item) => {
    return item.some((cell) =>
      cell.toString().toLowerCase().includes(searchKeyword.value.toLowerCase())
    );
  });
});

onMounted(() => {
  fetchData();
});

const bubbleSort = (arr, column, ascending) => {
  return arr.slice().sort((a, b) => {
    const valueA = a[column];
    const valueB = b[column];

    if (typeof valueA === "number" && typeof valueB === "number") {
      return ascending ? valueA - valueB : valueB - valueA; // 數字排序
    } else if (typeof valueA === "string" && typeof valueB === "string") {
      return ascending
        ? valueA.localeCompare(valueB)
        : valueB.localeCompare(valueA); // 字串排序
    } else {
      return 0; // 其他類型保持原樣
    }
  });
};
</script>

<style scoped>
.search-container {
  display: flex;
  justify-content: center;
}

.search-container input {
  width: 100%;
  font-size: 30px;
}

.table-container {
  max-height: 100vh;
  overflow-y: auto;
  top: 0;
  background-color: white;
}

.table-content {
  max-height: 85vh;
  overflow-y: auto;
}

.custom-table {
  width: 100%;
  table-layout: fixed;
  border-collapse: collapse;
  margin-top: 20px;
  border: 1px solid #ccc;
}

.custom-table th,
.custom-table td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: center;
  word-wrap: break-word;
}

.custom-table th {
  background-color: #f2f2f2;
  font-weight: bold;
}

.custom-table tbody tr:nth-child(even) {
  background-color: #f2f2f2;
}

.sort-icon {
  display: inline-block;
  margin-left: 4px;
  width: 16px;
  height: 16px;
  position: relative;
}

.sort-icon .arrow-up,
.sort-icon .arrow-down {
  position: absolute;
  width: 0;
  height: 0;
  border-style: solid;
}

.sort-icon .arrow-up {
  border-width: 0 5px 5px 5px;
  border-color: transparent transparent rgba(0, 0, 0, 0.7) transparent;
  top: 0;
}

.sort-icon .arrow-down {
  border-width: 5px 5px 0 5px;
  border-color: rgba(0, 0, 0, 0.7) transparent transparent transparent;
  bottom: 0;
}

.sort-icon.ascending .arrow-up {
  border-color: transparent transparent #ff5722 transparent;
  opacity: 1;
}

.sort-icon.descending .arrow-down {
  border-color: #ff5722 transparent transparent transparent;
  opacity: 1;
}

.highlighted {
  background-color: rgb(243, 215, 215);
  font-weight: bold;
}
</style>
