<script setup lang="ts">
import { reactive, ref } from 'vue'
import axios from 'axios'
import dayjs from 'dayjs'
import data from '../data.json'

const statusMessage = ref('未开始')
const countdownTime = ref(60000 + Math.floor(Math.random() * 15000))
const countdownShow = ref(-1)
const formInfo = reactive({
  card: '',
  name: '',
  dept: '',
  testPaperId: '1798915139650629633',
})

const beginInfo = reactive({
  dataScope: null,
  handTime: null,
  isPass: null,
  number: null,
  officeId: null,
  score: null,
  status: 1,
  tel: null,
  testPaperName: '2024安全生产知识竞赛（正式考试）',
  testPassword: null,
  testStatus: 0,
  type: 1,
  useTime: null,
  userId: null,
})
async function go() {
  try {
    statusMessage.value = '答题中'
    const userInfo = await axios.post('/api/inspection/public/examinee/save', {
      ...formInfo,
    })
    const allQuestions = await axios.post('/api/inspection/public/details/beginExam', {
      ...beginInfo,
      ...formInfo,
      id: userInfo.data.data.id,
      createDate: dayjs().format('YYYY-MM-DD HH:mm:ss'),
      testEndTime: dayjs().add(1, 'hour').format('YYYY-MM-DD HH:mm:ss'),
    })
    const answers = allQuestions.data.data.map((item: any) => {
      return {
        answer: answer(item.name),
        questionId: item.questionId,
        serialNumber: item.serialNumber,
      }
    })

    const allQuestionsLength = answers.length
    const answered = answers.filter((item: any) => item.answer)
    statusMessage.value = `回答了${answered.length}/${allQuestionsLength}道题`
    countdown(Math.floor(countdownTime.value / 1000))
    if (answered.length !== 30) {
      statusMessage.value = '有题没找到答案'
      return
    }
    setTimeout(() => {
      axios.post('/api/inspection/public/examinee/submitExamResult', {
        examResult: answers,
        testPaperId: formInfo.testPaperId,
        examineeId: userInfo.data.data.id,
        card: formInfo.card,
        endTime: dayjs().valueOf(),
      })
      statusMessage.value = '交卷成功'
    }, countdownTime.value)
  }
  catch (error) {
  }
}

function answer(question: string) {
  const answerItem: any = data.question.find(item => item.Column3!.trim() === question)
  if (!answerItem)
    return null

  if (answerItem.Column2 === '判断题')
    return answerItem.Column4 === '正确' ? '0' : '1'

  if (answerItem.Column2 === '多项选择题')
    return answerItem.Column4.trim().split('').join(',')

  return answerItem.Column4
}

function countdown(time: number) {
  countdownShow.value = time
  const timeout = setTimeout(() => {
    countdownShow.value -= 1
    if (countdownShow.value > 0)
      countdown(countdownShow.value)
    else
      clearTimeout(timeout)
  }, 1000)
}
</script>

<template>
  <p style="font-weight: bold;">
    {{ statusMessage }}
  </p>
  <p v-if="countdownShow !== -1 || statusMessage === '交卷成功'" style="font-weight: bold;">
    {{ countdownShow }}秒后交卷
  </p>
  <div mt-4 flex flex-col items-center gap-4>
    <TheInput v-model="formInfo.name" type="text" placeholder="姓名" />
    <TheInput v-model="formInfo.card" type="text" placeholder="身份证" />
    <TheInput v-model="formInfo.dept" type="text" placeholder="单位" />
    <button text-sm btn @click="go">
      go
    </button>
  </div>
</template>

<style scoped>

</style>
