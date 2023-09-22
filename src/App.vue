<template>
  <div>
    <h1 class="title">轉(交)換債月交易彙總表</h1>
    <div class="selected-rows-table">
      <h2>被選中項目</h2>
      <button
        class="clear-button"
        @click="clearSelectedRows"
      >
        清除所有選中項目
      </button>
      <table>
        <thead>
          <tr>
            <th>代號</th>
            <th>名稱</th>
            <th>成交量</th>
            <th>成交金額</th>
            <th>成交筆數</th>
            <th>平均</th>
            <th>最高</th>
            <th>最低</th>
            <th>當月最後營業日</th>
            <th>月份</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(row, rowIndex) in selectedRows"
            :key="rowIndex"
          >
            <td
              v-for="(cell, cellIndex) in row"
              :key="cellIndex"
            >
              {{ cell }}
            </td>
            <td>{{ selectedRowsMonth[rowIndex] }}</td>
          </tr>
        </tbody>
      </table>
    </div>
    <div class="search-container">
      <input
        v-model="searchTerm"
        placeholder="Search"
      />
    </div>
    <div
      v-for="group in groupedData"
      :key="group.identifier"
    >
      <h2>{{ group.identifier }}</h2>
      <div class="table-container">
        <table>
          <thead class="sticky-thead">
            <tr>
              <th>選取</th>
              <th>代號</th>
              <th>名稱</th>
              <th>成交量</th>
              <th>成交金額</th>
              <th>成交筆數</th>
              <th>平均</th>
              <th>最高</th>
              <th>最低</th>
              <th>當月最後營業日</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(row, rowIndex) in filteredData(group.data)"
              :key="rowIndex"
            >
              <td>
                <input
                  type="checkbox"
                  :checked="selectedRows.includes(row)"
                  @change="toggleRowSelection(row, group.identifier)"
                />
              </td>
              <td
                v-for="(cell, cellIndex) in row"
                :key="cellIndex"
                :class="{ highlighted: shouldHighlight(cell) }"
              >
                {{ cell }}
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";

const groupedData = ref([]);
const searchTerm = ref("");
const selectedRows = ref([]);
const selectedRowsMonth = ref([]);

onMounted(async () => {
  try {
    const response = await fetch("output.json");
    const jsonData = await response.json();

    if (Array.isArray(jsonData)) {
      const grouped = groupDataByMonth(jsonData);
      groupedData.value = grouped;
    }
  } catch (error) {
    console.error("Error fetching data:", error);
  }
});

function groupDataByMonth(data) {
  const grouped = [];

  data.forEach((item) => {
    const identifier = item.identifier;
    const existingGroup = grouped.find(
      (group) => group.identifier === identifier
    );

    if (!existingGroup) {
      grouped.push({
        identifier: identifier,
        headers: item.data[0],
        data: item.data.slice(1),
      });
    } else {
      existingGroup.data = existingGroup.data.concat(item.data.slice(1));
    }
  });

  return grouped;
}

const filteredData = computed(() => {
  return (data) => {
    if (!searchTerm.value) {
      return data;
    }
    const lowerSearchTerm = searchTerm.value.toLowerCase();
    return data.filter((row) =>
      row.some((cell) =>
        cell.toString().toLowerCase().includes(lowerSearchTerm)
      )
    );
  };
});

function shouldHighlight(cell) {
  if (!searchTerm.value) {
    return false;
  }
  const lowerSearchTerm = searchTerm.value.toLowerCase();
  return cell.toString().toLowerCase().includes(lowerSearchTerm);
}

function toggleRowSelection(row, month) {
  if (selectedRows.value.includes(row)) {
    selectedRows.value = selectedRows.value.filter(
      (selectedRow) => selectedRow !== row
    );
    selectedRowsMonth.value = selectedRowsMonth.value.filter(
      (selectedMonth) => selectedMonth !== month
    );
  } else {
    selectedRows.value.push(row);
    selectedRowsMonth.value.push(month);
  }
}

function clearSelectedRows() {
  selectedRows.value = [];
  selectedRowsMonth.value = [];
}
</script>

<style>
.title {
  text-align: center;
}

@media screen and (max-width: 768px) {
  table {
    font-size: 14px;
    width: auto;
  }

  th,
  td {
    padding: 6px;
  }

  .table-container {
    max-width: 100%;
    overflow-x: auto;
  }
}

.selected-rows-table {
  margin: 20px auto 20px auto;
  overflow: auto;
}

table {
  border-collapse: collapse;
  width: 100%;
}

table,
th,
td {
  border: 1px solid #ccc;
}

th,
td {
  padding: 8px;
  text-align: left;
  white-space: nowrap;
}

th {
  background-color: #f2f2f2;
}

.search-container {
  margin-bottom: 10px;
}

input {
  width: 100%;
  padding: 8px;
}

.sticky-thead {
  position: sticky;
  top: 0;
  background-color: white;
  z-index: 1;
}

.highlighted {
  background-color: rgb(243, 215, 215);
}

.clear-button {
  margin-bottom: 20px;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s ease;
}

.clear-button:hover {
  background-color: #0056b3;
}
</style>
