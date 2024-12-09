<script setup>
import { ref, computed, onMounted } from 'vue'
import * as XLSX from 'xlsx'

const dataMappingFile = ref(null)
const dataFile = ref(null)
const dataMapping = ref([])
const data = ref(null)
const selectedNode = ref(null)
const sheet = ref('')
const cell = ref('')
const draggedItems = ref([])

// Dummy data for drag-and-drop
const dummyData = ref([
  { id: 1, label: 'Item 1' },
  { id: 2, label: 'Item 2' },
  { id: 3, label: 'Item 3' },
  { id: 4, label: 'Item 4' },
  { id: 5, label: 'Item 5' }
])

// Grid cells data
const gridCells = ref(Array(10000).fill(null))

const handleFileUpload = (type, event) => {
  const file = event.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = (e) => {
      try {
        const content = JSON.parse(e.target.result)
        if (type === 'dataMapping') {
          dataMapping.value = content
          console.log('dataMapping:', dataMapping.value)
          transposeDataMapping(content)
        } else if (type === 'data') {
          data.value = content
          console.log('data:', data.value)
        }
      } catch (error) {
        console.error('Error parsing JSON:', error)
        alert('Error parsing JSON file. Please ensure the file is in the correct format.')
      }
    }
    reader.readAsText(file)
  } else {
    console.log(`No file selected for ${type}`)
  }
}

const transposeDataMapping = (dataMapping) => {
  // Clear the grid cells
  gridCells.value = Array(10000).fill(null)

  // Transpose the data mapping to the grid cells
  dataMapping.forEach((mapping) => {
    const { spreadsheetLocation } = mapping
    const col = spreadsheetLocation.charCodeAt(0) - 65 // Convert column letter to index (A=0, B=1, ...)
    const row = parseInt(spreadsheetLocation.slice(1)) - 1 // Convert row number to index (1=0, 2=1, ...)
    const cellIndex = row * 100 + col
    gridCells.value[cellIndex] = { label: mapping.dataPath.map(dp => dp.tag).join(' -> ') }
  })
}

const updateSelectedNode = (selected) => {
  console.log('Selected Node:', selected)
}

const generateReport = () => {
  console.log('Generating report...')
  if (!dataMapping.value.length || !data.value) {
    alert('Please upload both JSON files.')
    return
  }

  const workbook = XLSX.utils.book_new()
  const sheet = XLSX.utils.aoa_to_sheet([[]])
  XLSX.utils.book_append_sheet(workbook, sheet, 'Sheet 1')

  dataMapping.value.forEach((mapping) => {
    const value = mapping.dataPath.reduce((acc, curr) => acc[curr.tag], data.value)
    if (value !== null && value !== undefined) {
      XLSX.utils.sheet_add_aoa(sheet, [[value]], { origin: mapping.spreadsheetLocation })
    }
  })

  XLSX.writeFile(workbook, 'report.xlsx')
  console.log('Report generated successfully.')
}

// const createDataMapping = () => {
//   if (!data.value) {
//     alert('Please upload data.json first.')
//     return
//   }

//   // Logic to create a new data_mapping.json file
//   // This can be implemented based on user input or predefined logic
//   console.log('Creating new data_mapping.json...')
// }

const addMapping = () => {
  if (!selectedNode.value || !sheet.value || !cell.value) {
    alert('Please select a node and specify the sheet and cell.')
    return
  }

  const newMapping = {
    dataPath: selectedNode.value.split('.').map((tag) => ({ tag })),
    spreadsheetLocation: cell.value,
    sheet: sheet.value,
  }

  dataMapping.value.push(newMapping)
  console.log('New mapping added:', newMapping)
}

const dataTree = computed(() => {
  if (!data.value) return []

  const convertToTree = (obj, parentKey = '') => {
    return Object.keys(obj).map((key) => {
      const id = parentKey ? `${parentKey}.${key}` : key
      if (typeof obj[key] === 'object' && obj[key] !== null) {
        return {
          label: key,
          id,
          children: convertToTree(obj[key], id),
        }
      } else {
        return {
          label: `${key}: ${obj[key]}`,
          id,
        }
      }
    })
  }

  return convertToTree(data.value)
})

const onDragStart = (event, item, source) => {
  event.dataTransfer.setData('item', JSON.stringify(item))
  event.dataTransfer.setData('source', source)
  event.dataTransfer.setData('index', event.target.dataset.index)
}

const onDrop = (event, target) => {
  const item = JSON.parse(event.dataTransfer.getData('item'))
  const source = event.dataTransfer.getData('source')
  const index = event.dataTransfer.getData('index')

  const dropIndex = Array.from(event.target.parentNode.children).indexOf(event.target)

  if (source === target) {
    const list = source === 'dummyData' ? dummyData.value : draggedItems.value
    list.splice(index, 1)
    list.splice(dropIndex, 0, item)
  } else {
    if (source === 'dummyData' && target === 'draggedItems') {
      dummyData.value = dummyData.value.filter(i => i.id !== item.id)
      draggedItems.value.splice(dropIndex, 0, item)
    } else if (source === 'draggedItems' && target === 'dummyData') {
      draggedItems.value = draggedItems.value.filter(i => i.id !== item.id)
      dummyData.value.splice(dropIndex, 0, item)
    }
  }
}

const allowDrop = (event) => {
  event.preventDefault()
}

const onGridDrop = (event) => {
  const item = JSON.parse(event.dataTransfer.getData('item'))
  const cellIndex = event.target.dataset.index
  gridCells.value[cellIndex] = item
  console.log(`Dropped item ${item.label} into cell ${cellIndex}`)
}

const resetCell = (index) => {
  gridCells.value[index] = null
}

onMounted(() => {
  const dropAreas = document.querySelectorAll('.drop-area')
  dropAreas.forEach(dropArea => {
    dropArea.addEventListener('dragover', allowDrop)
    dropArea.addEventListener('drop', (event) => onDrop(event, dropArea.dataset.target))
  })

  const gridCells = document.querySelectorAll('.grid-cell')
  gridCells.forEach(cell => {
    cell.addEventListener('dragover', allowDrop)
    cell.addEventListener('drop', onGridDrop)
  })
})
</script>

<template>
  <q-page class="q-pa-md">
    <q-card flat bordered>
      <q-card-section>
        <div class="text-h6">Generate Report</div>
        <!-- <div class="text-subtitle2">by John Doe</div> -->
      </q-card-section>
      <q-card-section>
        <q-form @submit.prevent="generateReport" class="q-gutter-y-sm">
          <q-file
            outlined
            v-model="dataMappingFile"
            label="Upload data_mapping.json"
            accept=".json"
            @input="handleFileUpload('dataMapping', $event)"
          />
          <q-file
            outlined
            v-model="dataFile"
            label="Upload data.json"
            accept=".json"
            @input="handleFileUpload('data', $event)"
          />
          <q-btn type="submit" label="Generate Report" color="primary" />
        </q-form>
      </q-card-section>
    </q-card>

    <q-card flat bordered v-if="data" class="q-mt-md">
      <q-card-section>
        <q-tree
          :nodes="dataTree"
          node-key="id"
          selection="single"
          v-model:selected="selectedNode"
          @update:selected="updateSelectedNode"
          @dragstart="onDragStart($event, selectedNode, 'dataTree')"
          draggable="true"
        />
      </q-card-section>
    </q-card>

    <q-card flat bordered v-if="selectedNode" class="q-mt-md">
      <q-card-section>
        <div>Selected Node: {{ selectedNode }}</div>
        <q-input v-model="sheet" label="Sheet" />
        <q-input v-model="cell" label="Cell" />
        <q-btn @click="addMapping" label="Add Mapping" color="secondary" />
      </q-card-section>
    </q-card>

    <q-card flat bordered class="q-mt-md">
      <q-card-section>
        <div class="text-h6">Drag and Drop Items</div>
        <div class="drop-area" data-target="dummyData" v-mutation>
          <div v-for="(item, index) in dummyData" :key="item.id" draggable="true" @dragstart="onDragStart($event, item, 'dummyData')" :data-index="index">
            {{ item.label }}
          </div>
        </div>
      </q-card-section>
    </q-card>

    <q-card flat bordered class="q-mt-md">
      <q-card-section>
        <div class="text-h6">Dragged Items</div>
        <div class="drop-area" data-target="draggedItems" v-mutation>
          <div v-for="(item, index) in draggedItems" :key="item.id" draggable="true" @dragstart="onDragStart($event, item, 'draggedItems')" :data-index="index">
            {{ item.label }}
          </div>
        </div>
      </q-card-section>
    </q-card>

    <q-card flat bordered class="q-mt-md">
      <q-card-section>
        <div class="text-h6">Infinite Grid</div>
        <div class="infinite-grid">
          <div v-for="(cell, index) in gridCells" :key="index" class="grid-cell" :data-index="index">
            <div v-if="cell">
              {{ cell.label }}
              <button @click="resetCell(index)">Reset</button>
            </div>
          </div>
        </div>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<style scoped>
.drop-area {
  min-height: 100px;
  border: 1px dashed #ccc;
  padding: 10px;
}

.infinite-grid {
  display: grid;
  grid-template-columns: repeat(100, 100px);
  grid-template-rows: repeat(100, 100px);
  gap: 10px;
  overflow: auto;
  height: 400px;
  width: 100%;
}

.grid-cell {
  display: flex;
  align-items: center;
  justify-content: center;
  border: 1px solid #ccc;
  background-color: #f9f9f9;
  height: 100px;
  width: 100px;
  position: relative;
}

.grid-cell button {
  position: absolute;
  top: 5px;
  right: 5px;
}
</style>