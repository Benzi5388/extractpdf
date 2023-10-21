<template>
  <div>
    <label for="fileInput" class="file-label">
      <input type="file" id="fileInput" @change="handlePDFUpload" accept=".pdf" class="hidden-input" />
      Choose a PDF File
    </label>
    <span class="file-name spacer">{{ pdfFile ? pdfFile.name : 'No file chosen' }}</span>
    <button @click="extractFormData" class="extract-button">Extract Form Data</button>
    <div class="button-space"></div>
    <div class="table-container">
      <table v-if="formData.length > 0" class="table">
        <thead>
          <tr>
            <th>Field Name</th>
            <th>Field Value</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(field, index) in formData" :key="index">
            <td>{{ field.name }}</td>
            <td>{{ field.value }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script>
import { ref } from 'vue';
import pdfjsLib from 'pdfjs-dist/build/pdf';

export default {
  setup() {
    const pdfFile = ref(null);
    const formData = ref([]);

    const handlePDFUpload = (event) => {
      pdfFile.value = event.target.files[0];
    };

    const extractFormData = async () => {
      if (pdfFile.value) {
        try {
          const pdfData = new Uint8Array(await pdfFile.value.arrayBuffer());
          const script = document.createElement('script');
          script.src = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.10.377/pdf.min.js';
          script.onload = () => {
            const pdfjsLib = window['pdfjs-dist/build/pdf'];
            pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.10.377/pdf.worker.min.js';
            const loadingTask = pdfjsLib.getDocument({ data: pdfData });

            loadingTask.promise.then((pdfDocument) => {
              const formDataArray = [];

              pdfDocument.getMetadata().then((metadata) => {
                const numPages = pdfDocument.numPages;

                for (let pageNumber = 1; pageNumber <= numPages; pageNumber++) {
                  pdfDocument.getPage(pageNumber).then((pdfPage) => {
                    pdfPage.getAnnotations().then((annotations) => {
                      annotations.forEach((annotation) => {
                        if (annotation.fieldName) {
                          formDataArray.push({
                            name: annotation.fieldName,
                            value: annotation.fieldValue || '',
                          });
                        }
                      });
                      if (pageNumber === numPages) {
                        formData.value = formDataArray;
                      }
                    });
                  });
                }
              });
            });
          };

          document.head.appendChild(script);
        } catch (error) {
          console.error(error);
        }
      }
    };

    return {
      pdfFile,
      formData,
      handlePDFUpload,
      extractFormData,
    };
  },
};
</script>

<style>
.file-label {
  display: inline-block;
  background-color: #333;
  color: #fff;
  padding: 10px 20px;
  cursor: pointer;
  border: none;
  border-radius: 5px;
  margin-right: 10px;
}

.hidden-input {
  display: none;
}

.file-name {
  font-size: 14px;
  margin-left: 10px;
  font-weight: bold; /* Make "No file chosen" bold */
  color: #333;
}

.extract-button {
  background-color: #333; /* Darken the button */
  color: #fff;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  margin-top: 10px;
}

.button-space {
  margin-top: 20px;
}

.table-container {
  max-width: 100%;
  overflow-x: auto;
}

.table {
  width: 100%;
  border-collapse: collapse;
  border: 1px solid #ddd;
  table-layout: auto;
}

.table th,
.table td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.table th {
  background-color: #f2f2f2;
}
.spacer {
  margin-right: 20px; /* Adjust the margin to create space between elements */
}
</style>
