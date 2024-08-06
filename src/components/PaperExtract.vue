<template>
  <div class="flex gap-5 h-[800px] m-20">
    <div class="flex-1 border flex flex-col overflow-auto">
      <!-- 标题 -->
      <div class="bg-[#f3f3f5] h-[50px] flex items-center px-2">
        <div>输入区</div>
        <div></div>
      </div>

      <!-- 区域 -->
      <div
        class="p-5 flex-1 outline-none input-textarea whitespace-pre"
        contenteditable="true"
        placeholder="请输入内容"
        @input="handleInput"
        @change="handleInput"
      ></div>
    </div>
    <div class="flex-1 border flex flex-col overflow-auto">
      <!-- 标题 -->
      <div class="bg-[#f3f3f5] flex items-center h-[50px] px-2">
        <div>检查区</div>
        <div></div>
      </div>

      <!-- 区域 -->
      <div class="flex-1 p-5">
        <div
          v-for="(question, idx1) in questions"
          :key="'stem' + idx1"
          class="border mb-2 p-2 rounded-2"
        >
          <div v-if="question.title">
            <div>
              <span>{{ idx1 + 1 }}.</span>
              <span v-html="question.title"></span>
            </div>
            <div
              v-for="(option, idx2) in question.options"
              :key="'option' + idx2"
              class="m-5"
            >
              <span>{{ ['A', 'B', 'C', 'D', 'E', 'F'][idx2] }}.</span>
              <span v-html="option"></span>
            </div>
          </div>
          <div v-if="question.other" v-html="question.other"></div>
        </div>
      </div>
    </div>
  </div>
</template>
<script lang="ts" setup>
  import { computed, reactive, ref, watch, onMounted } from 'vue'

  const questions = ref<
    {
      title?: string
      options?: string[]
      other?: string
    }[]
  >([])

  const handleInput = (e) => {
    document.querySelectorAll('.input-textarea img').forEach((img) => {
      img.setAttribute('style', '')
    })
    const content = e.target.innerHTML

    if (content) {
      handleContent(content)
    }
  }

  const handleContent = (input: string) => {
    questions.value = []
    const lines = input.split('\n').filter((line) => line.trim())
    lines.forEach((line) => {
      if (!line.includes('.')) {
        questions.value.push({
          other: line,
        })
        return
      }
      if (getStem(line)) {
        questions.value.push({
          title: getStem(line),
          options: [],
        })
      }
      if (getOption(line)) {
        if (questions.value[questions.value.length - 1].options) {
          questions.value[questions.value.length - 1].options!.push(
            getOption(line)
          )
        }
      }
    })
  }

  const getOption = (content: string) => {
    const isOption = ['A', 'B', 'C', 'D', 'E', 'F'].includes(
      content.split('.')[0].trim()
    )
    if (isOption) {
      return content.split('.')[1].trim()
    }
    return ''
  }

  const getStem = (content: string) => {
    const isStem = /\d/.test(content.split('.')[0].trim())
    if (isStem) {
      return content.split('.')[1].trim()
    }
    return ''
  }

  const handleImage = () => {}
</script>
<style lang="less" scoped>
  .input-textarea:empty::before {
    color: lightgrey;
    content: attr(placeholder);
  }
</style>
