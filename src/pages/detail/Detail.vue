<style lang="scss" scoped >
.page-title {
  text-align: center;
}
</style>
<template>
<v-ons-page>
  <navbar></navbar>
  <v-ons-card modifier="material">
    <div class="page-content">
      <h3 class="page-title">
        {{ this.$route.params.year }} - {{ this.$route.params.month }} - {{ this.$route.params.day }}
      </h3>
    </div>
  </v-ons-card>
</v-ons-page>
</template>

<script>
import Navbar from '../../components/navbar/Navbar';

export default {
  name: 'posts-page',
  components: {
    Navbar,
  },
  params :{
    year: {
      default: null
    },
    month: {
      default: null
    },
    day: {
      default: null,
    }
  },
};
</script>
