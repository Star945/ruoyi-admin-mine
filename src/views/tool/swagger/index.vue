<!--
 * @Author: your name
 * @Date: 2021-07-05 13:52:43
 * @LastEditTime: 2021-07-05 16:03:21
 * @LastEditors: your name
 * @Description: In User Settings Edit
 * @FilePath: \qingyuan-resturant-admin\src\views\tool\swagger\index.vue
-->
<template>
  <i-frame :src="url" />
</template>
<script>
import iFrame from "@/components/iFrame/index";
export default {
  name: "Swagger",
  components: { iFrame },
  data() {
    return {
      url: process.env.VUE_APP_BASE_API + "/swagger-ui/index.html"
    };
  },
};
</script>
