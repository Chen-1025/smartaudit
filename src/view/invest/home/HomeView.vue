<template>
  <div class="home-container" :style="{ 'background-image': 'url(' + backgroundImage + ')' }">
    <div class="card-grid">
      <Card class="card-item" v-for="(item, index) in modules" :key="index" @click.native="goToLink(item.link)">
        <div class="card-content">
          <div class="icon">
            <Icon :type="item.icon" size="32" />
          </div>
          <h4>{{ item.title }}</h4>
        </div>
      </Card>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      backgroundImage: 'https://images.unsplash.com/photo-1517694712202-14dd9538aa97?q=80&w=2070&auto=format&fit=crop',
      modules: [
        {
          title: '案件报告',
          icon: 'ios-laptop',
          // 使用路径名，而不是完整的URL
          link: '/report_page/alert-management' 
        },
        {
          title: '案件串并',
          icon: 'ios-analytics',
          link: '/linking_page/case-elements' 
        },
        {
          title: '数据分析',
          icon: 'ios-bulb',
          link: '/analysis_page/flows-analysis' 
        },
        {
          title: '材料质检',
          icon: 'ios-construct',
          link: '/inspect_page/transcript-inspection' 
        }
      ]
    }
  },
  methods: {
    goToLink (path) {
      // 使用 Vue Router 进行跳转，不会在新窗口打开
      this.$router.push(path).catch(err => {
        if (err.name !== 'NavigationDuplicated') {
          throw err
        }
      });
    }
  }
}
</script>

<style scoped>
/* 样式部分保持不变 */
.home-container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  position: relative;
  
  background-color: rgba(0, 0, 0, 0.1);
}

.card-grid {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 30px;
  padding: 20px;
  background: rgba(255, 255, 255, 0.7);
  border-radius: 10px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(5px);
}

.card-item {
  width: 150px;
  height: 150px;
  background: #ffffff;
  border: 1px solid #dcdee2;
  box-shadow: 0 1px 6px rgba(0, 0, 0, 0.1);
  border-radius: 6px;
  transition: all 0.3s ease;
  cursor: pointer;
  text-align: center;
  
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.card-item:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  transform: translateY(-3px);
}

.card-content {
  padding: 10px;
}

.icon {
  margin-bottom: 10px;
}

h4 {
  margin: 0;
  font-size: 16px;
  color: #515a6e;
  font-weight: normal;
}
</style>