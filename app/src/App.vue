<template>
  <div>
    <h1>Select a Company</h1>
    <select v-model="selectedCompanyId">
      <option disabled value="">Please select one</option>
      <option v-for="company in companies" :key="company.id" :value="company.id">
        {{ company.name }}
      </option>
    </select>

    <!-- companyId가 변경될 때마다 ChartComponent를 재생성 -->
    <ChartComponent :key="selectedCompanyId" :companyId="selectedCompanyId" v-if="selectedCompanyId" />
  </div>
</template>

<script>
import ChartComponent from './components/ChartComponent.vue';

export default {
  components: {
    ChartComponent
  },
  data() {
    return {
      selectedCompanyId: "", // 기본값을 빈 문자열로 설정
      companies: []
    };
  },
  watch: {
    selectedCompanyId(newVal) {
      console.log("Selected Company ID:", newVal);
    }
  },
  mounted() {
    this.fetchCompanies();
  },
  methods: {
    fetchCompanies() {
      // 서버에서 회사 목록을 가져오는 API 호출
      const apiUrl = `${process.env.VUE_APP_API_URL}/company`;
      fetch(apiUrl)
        .then(response => response.json())
        .then(data => {
          this.companies = data;
        });
    }
  }
}
</script>

<style scoped>
/* 스타일링을 원하는 대로 추가하세요 */
</style>