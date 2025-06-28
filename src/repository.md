```bash-vue [下载源码]
  # Download
  git clone {{ gitUrl }} Antdv.pro

```

<script setup lang="ts">
import { ref } from 'vue'
import { onMounted } from 'vue'

const gitUrl = ref('https://github.com/antdvr/vue-template-3.x.git')

onMounted(() => {
  const href = window.location.href
  const atomgit = 'https://antdvr.atomgit.net/'
  const atomgitUrl = 'https://atomgit.com/antdvr/vue-template-3.x.git'

  if (href.startsWith(atomgit)) {
    gitUrl.value = atomgitUrl
  }
})
</script>
