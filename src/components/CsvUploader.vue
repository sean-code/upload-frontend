<template>
  <div>
    <input type="file" @change="handleFileUpload" />
    <table>
      <thead>
        <tr>
          <th v-for="header in headers" :key="header">
            <select v-model="headerMapping[header]" @change="updateFieldOptions">
              <option v-for="option in fieldOptions" :key="option.value" :value="option.value" :disabled="option.disabled">{{ option.label }}</option>
            </select>
          </th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, rowIndex) in tableData" :key="rowIndex">
          <td v-for="(value, key) in row" :key="key">{{ value }}</td>
        </tr>
      </tbody>
    </table>
    <!-- <button @click="mapHeaders">Map Headers</button> -->
    <button @click="submitData">Submit Data</button>
  </div>
</template>




<script>
import Papa from 'papaparse';



export default {
  components: {
    Papa

  },
  data() {
    return {
      file: null,
      tableData: [],
      headers: [],
      mappedData: [],
      apiResponse: null,
      headerMapping: {
        'Product Name': 'title',
        'Brand': 'brand',
        'Color': 'color',
        'Product Model': 'prod_model',
        'Size': 'size',
        'Weight' : 'weight',
        'Shop': 'shop',
        'Warehouse': 'warehouse'
      },
      fieldOptions: [
        { label: 'Product Name', value: 'title', disabled: false  },
        { label: 'Brand*', value: 'brand', disabled: false  },
        { label: 'Price*', value: 'price', disabled: false  },
        { label: 'Quantity*', value: 'quantity', disabled: false  },
        { label: 'Seller SKU*', value: 'seller_sku', disabled: false  },
        { label: 'Shop SKU*', value: 'shop_sku', disabled: false  },
        { label: 'Warehouse*', value: 'warehouse', disabled: false  },
        { label: 'Color', value: 'color', disabled: false  },
        { label: 'Short Description', value: 'short_description', disabled: false  },
        { label: 'Full Description', value: 'description', disabled: false  },
        { label: 'Size', value: 'size', disabled: false  },
        { label: 'Weight', value: 'weight', disabled: false  },
        { label: 'Wordpress SKU', value: 'wordpress_sku', disabled: false  },
        { label: 'Jumia SKU', value: 'jumia_sku', disabled: false  },
        { label: 'Saleprice', value: 'sale_price', disabled: false  },
        { label: 'Main Image', value: 'main_image', disabled: false  },
        { label: 'Other Images', value: 'images', disabled: false  },
        { label: 'Seller SKU', value: 'seller_sku', disabled: false  },
        { label: 'Product Model', value: 'prod_model' , disabled: false },
        { label: 'Sale Start Date', value: 'sale_start_date', disabled: false  },
        { label: 'Shop', value: 'shop_id', disabled: false  },
        { label: 'Variation', value: 'variation', disabled: false  },
      ],
    };
  },
  methods: {
    handleFileUpload(event) {
    this.file = event.target.files[0];
    Papa.parse(this.file, {
      header: true,
      complete: (results) => {
        const { data, meta: { fields } } = results;
        this.tableData = data;
        this.headers = fields;

        // Initialize header mapping with first field option
        this.headerMapping = {};
        for (const header of this.headers) {
          this.headerMapping[header] = this.fieldOptions[0].value;
        }
      }
    });
  },
  mapHeaders() {
    this.mappedData = this.tableData.map(row => {
      const mappedRow = {};
      for (const header of this.headers) {
        const field = this.headerMapping[header];
        mappedRow[field] = row[header];
      }
      return mappedRow;
    });
  },
  updateFieldOptions() {
    // Reset disabled options
    for (const option of this.fieldOptions) {
      option.disabled = false;
    }

    // Disable selected options
    for (const header of this.headers) {
      const field = this.headerMapping[header];
      for (const option of this.fieldOptions) {
        if (option.value === field) {
          option.disabled = true;
        }
      }
    }
  },
  async submitData() {
    await this.mapHeaders();
    const response = await fetch('https://example.com/api/endpoint', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(this.mappedData)
    });
    this.apiResponse = await response.json();
  }

  },
};
</script>

<style>
.table {
  border-collapse: collapse;
  width: 100%;
}

th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}

th {
  background-color: #f2f2f2;
}

tr:nth-child(even) {
  background-color: #f2f2f2;
}
</style>
