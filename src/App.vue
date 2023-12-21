<script setup lang="ts">
import { ref, computed } from 'vue';
import DiffMatchPatch from 'diff-match-patch';
import { PhCopy, PhCheckCircle } from "@phosphor-icons/vue";

const dmp = new DiffMatchPatch();

const text = ref('(masterpiece), 1girl,solo, blue hair,, long hair,pink eyes,hat, from above, hat, holding flowers, , beach,');
const copied = ref(false);

const normalizedText = computed(() =>
  text
    .value
    .split(",")
    .map(item => item.trim())
    .filter(Boolean)
    .filter((item, index, self) => self.indexOf(item) === index)
    .join(", ")
);

const changeObjects = computed(() => {
  const d = dmp.diff_main(text.value, normalizedText.value);
  dmp.diff_cleanupSemantic(d);
  return d;
});

async function copy() {
  try {
    await navigator.clipboard.writeText(normalizedText.value);
  } catch(err) {
    console.error(err);
    window.alert("ERROR: Could not copy text to clipboard");
    return;
  }
  copied.value = true;
  await new Promise(s => setTimeout(s, 2000));
  copied.value = false;
}
</script>

<template>
  <div>
    <h1>sd-text-normalizer</h1>

    <p>Normalize the Stable Diffusion (a1111) prompt.<br>Put a space after the comma and remove duplicate words.</p>
    <p>The text is not transmitted or stored externally.</p>

    <textarea class="text-input" cols="60" rows="8" v-model="text"></textarea>

    <div class="card">
      <div class="result">
        <span v-for="(changeObject, i) of changeObjects" :key="`${i}-${changeObject[1]}`" :class="{ added: changeObject[0] === 1, removed: changeObject[0] === -1 }">
          {{ changeObject[1] }}
        </span>
      </div>
    </div>
    
    <div class="card">
      <button class="unko" type="button" @click="copy">
        <ph-check-circle v-if="copied" color="rgb(71, 142, 88)" :size="20" class="icon" />
        <ph-copy v-if="!copied" :size="20" class="icon" />
        Copy (without the <span class="removed">red words</span>)
      </button>
    </div>
  </div>
</template>

<style scoped>
.text-input, .result {
  font-family: ui-monospace,SFMono-Regular,SF Mono,Menlo,Consolas,Liberation Mono,monospace;
}

.added {
  font-weight: 600;
  background: rgb(71, 142, 88);
  border: 1px solid rgb(171, 242, 188);
  white-space: pre-wrap;
}

.removed {
  font-weight: 600;
  background: rgb(142, 71, 71);
  border: 1px solid rgb(242, 171, 171);
  white-space: pre-wrap;
}

.icon {
  transform: translateY(4px);
}

@media (prefers-color-scheme: light) {
  .added {
    background: rgb(171, 242, 188);
    border: 1px solid rgb(71, 142, 88);
  }

  .removed {
    background: rgb(242, 171, 171);
    border: 1px solid rgb(142, 71, 71);
  }
}
</style>
