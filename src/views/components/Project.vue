<template>
  <div class="project">
    <div class="head__color"></div>
    <div class="project__content">
      <div class="left-side">
        <div class="project__content__title">
          <h3>{{ project.name }}</h3>
        </div>
        <div class="employees-tasks__total">
          <div class="employees-total">
            {{ project.employeeNum }} employee(s)
          </div>
          <div class="tasks-total">{{ project.taskNum }} task(s)</div>
        </div>
        <div class="project__status">
          <div class="new">{{ project.taskNumByStatus.NEW }} New</div>
          <div class="in-progress">{{ project.taskNumByStatus.IN_PROGRESS }} In Progress</div>
          <div class="ready">{{ project.taskNumByStatus.READY_FOR_REVIEW }} Ready For Review</div>
          <div class="done">{{ project.taskNumByStatus.DONE }} Done</div>
        </div>
      </div>
      <div class="export-btn" @click.stop="handleExportExcel">
        <span>Export to Excel</span>
        <img src="../../assets/logo/Microsoft_Excel_2013-2019_logo.svg.png" alt="" />
      </div>
    </div>
    <div class="tail_color"></div>
  </div>
</template>

<script setup>
import axios from "axios"
import { inject } from "vue"
const { project } = defineProps(["project"]);
import { VUE_APP_BACKEND_URL } from '../../../env'
import * as XLSX from 'xlsx';
import { saveAs } from 'file-saver';

const store = inject('store');


const handleExportExcel = async () => {
  const config = {
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${store.state.accessToken}`,
    },

  };

  store.state.isLoading = true;
  const response = await axios.get(`${VUE_APP_BACKEND_URL}/api/manager/project-details/${project.id}`, config);
  store.state.isLoading = false;

  const data = response.data.data;
  console.log(data.tasks);
  exportToExcel(data)

}

const exportToExcel = async (data) => {
  const workbook = XLSX.utils.book_new();
  const sheet1 = XLSX.utils.aoa_to_sheet([
    ["Project Name", project.name],
    ["Total employees involved", project.employeeNum],
    ["Total tasks", project.taskNum],
    [''],
  ]);

  const tableHeading = [
    'TT',
    "Tên công việc",
    "Miêu tả công việc",
    "% Hoàn thành",
    "Từ ngày",
    "Đến ngày",
    "Estimate hours",
    "In-charge",
    "CTV tự đánh giá",
    "LDDV đánh giá"
  ]
  XLSX.utils.sheet_add_aoa(sheet1, [tableHeading], { origin: -1 });


  data.tasks.forEach((task, index) => {
    const row = [(index + 1),
    task.title,
    task.description,
    task.completion + "%",
    task.startDate,
    task.endDate,
    task.estimateHours,
    task.assigneeName,
    task.employeeReview,
    task.managerReview]


    XLSX.utils.sheet_add_aoa(sheet1, [row], { origin: -1 });
  })

  
  sheet1['!cols'] = [
    { width: 15, alignment: { horizontal: 'center' }}, //TT',
    { width: 30, alignment: { horizontal: 'left' } }, //"Tên công việc",
    { width: 30, alignment: { horizontal: 'left' } }, //"Miêu tả công việc",
    { width: 15, alignment: { horizontal: 'center' }  }, //"% Hoàn thành",
    { width: 15, alignment: { horizontal: 'center' }  }, //"Từ ngày",
    { width: 15, alignment: { horizontal: 'center' }  }, //"Đến ngày",
    { width: 15, alignment: { horizontal: 'center' }  }, //"Estimate hours",
    { width: 15, alignment: { horizontal: 'left' }  }, //"In-charge",
    { width: 30, alignment: { horizontal: 'left' }  }, //"CTV tự đánh giá",
    { width: 30, alignment: { horizontal: 'left' }  }, //"LDDV đánh giá"
  ];

  sheet1['!rows'] = [{ font: { bold: true } }, {}, {}, {}, { font: { bold: true } }]; // make the 5th row bold, (table heading)
  XLSX.utils.book_append_sheet(workbook, sheet1, 'Project Details');

  const wbout = await XLSX.write(workbook, { bookType: 'xlsx', type: 'array' });
  const blob = new Blob([wbout], { type: 'application/octet-stream' });
  saveAs(blob, `${project.name}.xlsx`);

}

</script>

<style lang="scss" scoped>
.project {
  display: flex;
  border-radius: 10px;
  box-shadow: 0px 4px 4px rgba(0, 0, 0, 0.25);
  width: 100%;

  .head__color {
    background-color: var(--primary);
    border-radius: 10px 0 0 10px;
    width: 10px;
  }

  .project__content {
    display: flex;
    justify-content: space-between;
    align-items: flex-end;
    padding: 20px;
    width: 100%;
    font-size: 14px;

    .left-side {
      display: flex;
      flex-direction: column;
      gap: 10px;

      .project__content__title {}

      .employees-tasks__total {
        display: flex;
        gap: 20px;
      }

      .project__status {
        display: flex;
        gap: 20px;

        .new {
          background-color: #4c4ad0;
          border-radius: 10px;
          padding: 10px;
          color: #fff;
          font-size: 10px;
        }

        .in-progress {
          background-color: #ab7c0f;
          border-radius: 10px;
          padding: 10px;
          color: #fff;
          font-size: 10px;
        }

        .ready {
          background-color: #f2994a;
          border-radius: 10px;
          padding: 10px;
          color: #fff;
          font-size: 10px;
        }

        .done {
          background-color: #27ae60;
          border-radius: 10px;
          padding: 10px;
          color: #fff;
          font-size: 10px;
        }
      }
    }

    .export-btn {
      display: flex;
      align-items: center;
      gap: 10px;
      background-color: gray;
      padding: 5px;
      border-radius: 5px;
      cursor: pointer;
      color: #fff;

      &:hover {
        background-color: #0000005d;
      }
    }
  }

  .tail_color {
    background-color: var(--primary);
    border-radius: 0 10px 10px 0;
    width: 10px;
  }
}
</style>
