<template>
    <div>
      <label for="company-select">Select a Company:</label>
      <select id="company-select" v-model="selectedCompany" @change="onCompanyChange">
        <option v-for="company in companies" :key="company.id" :value="company.id">
          {{ company.name }}
        </option>
      </select>
    </div>
  </template>
  
  <script>
  export default {
    data() {
      return {
        companies: [], // 회사 목록
        selectedCompany: null // 선택된 회사 ID
      };
    },
    mounted() {
      this.fetchCompanies();
    },
    methods: {
      // 서버에서 회사 목록을 가져오는 함수
      fetchCompanies() {
        fetch('/api/companies') // 서버에서 회사 목록을 가져오는 API 엔드포인트
          .then(response => response.json())
          .then(data => {
            this.companies = data;
          });
      },
      // 회사 선택이 변경될 때 호출되는 함수
      onCompanyChange() {
        this.$emit('company-changed', this.selectedCompany);
      }
    }
  };
  </script>
  
  <style>
  /* 필요한 스타일 추가 */
  </style>