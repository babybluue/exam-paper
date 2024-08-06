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
    const textArea = document.querySelector('.input-textarea')
    textArea?.querySelectorAll('img').forEach((img) => {
      img.setAttribute('style', '')
    })

    const savedSel = saveSelection(textArea)
    handleRemoveTags(textArea)
    if (savedSel) {
      restoreSelection(textArea, savedSel)
    }
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

  const handleRemoveTags = (container: any) => {
    const nodes = container.childNodes

    for (let i = 0; i < nodes.length; i++) {
      const node = nodes[i]

      if (
        node.nodeType === Node.ELEMENT_NODE &&
        (node.tagName === 'DIV' || node.tagName === 'SPAN')
      ) {
        // 将子节点插入到当前节点之前
        while (node.firstChild) {
          container.insertBefore(node.firstChild, node)
        }
        // 移除当前节点
        container.removeChild(node)
        // 递归调用以处理嵌套的 div 和 span
        i-- // 调整索引以检查新的当前节点
      } else if (node.tagName === 'BR') {
        // 创建一个换行符文本节点
        const newline = document.createTextNode('\n')
        // 替换 <br> 标签
        container.replaceChild(newline, node)
      } else if (node.nodeType === Node.ELEMENT_NODE) {
        // 递归处理嵌套的元素
        handleRemoveTags(node)
      }
    }
  }

  const saveSelection = (container) => {
    const selection = window.getSelection()
    if (!selection) {
      return
    }
    if (selection.rangeCount > 0) {
      const range = selection.getRangeAt(0)
      const preSelectionRange = range.cloneRange()
      preSelectionRange.selectNodeContents(container)
      preSelectionRange.setEnd(range.startContainer, range.startOffset)
      const start = preSelectionRange.toString().length

      return {
        start: start,
        end: start + range.toString().length,
      }
    }
    return null
  }

  const restoreSelection = (container, savedSel) => {
    let charIndex = 0
    const range = document.createRange()
    range.setStart(container, 0)
    range.collapse(true)
    const nodeStack = [container]
    let node,
      foundStart = false,
      stop = false

    while (!stop && (node = nodeStack.pop())) {
      if (node.nodeType === 3) {
        let nextCharIndex = charIndex + node.length
        if (
          !foundStart &&
          savedSel.start >= charIndex &&
          savedSel.start <= nextCharIndex
        ) {
          range.setStart(node, savedSel.start - charIndex)
          foundStart = true
        }
        if (
          foundStart &&
          savedSel.end >= charIndex &&
          savedSel.end <= nextCharIndex
        ) {
          range.setEnd(node, savedSel.end - charIndex)
          stop = true
        }
        charIndex = nextCharIndex
      } else {
        let i = node.childNodes.length
        while (i--) {
          nodeStack.push(node.childNodes[i])
        }
      }
    }

    const selection = window.getSelection()
    if (!selection) {
      return
    }
    selection.removeAllRanges()
    selection.addRange(range)
  }

  const handleImage = () => {}
</script>
<style lang="less" scoped>
  .input-textarea:empty::before {
    color: lightgrey;
    content: attr(placeholder);
  }
</style>
