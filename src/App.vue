<template>
  <div class="app">
    <header class="header">
      <div class="container header-inner">
        <div class="brand">
          <div class="brand-mark">☯</div>
          <div class="brand-text">
            <div class="brand-name">灵象 AI 算命馆</div>
            <div class="brand-sub">用 AI 的方式，重讲一次东方玄学</div>
          </div>
        </div>
        <div class="header-badge">实验项目 · 娱乐为主 · 不作决策依据</div>
      </div>
    </header>

    <main class="main">
      <div class="container grid-2">
        <section class="panel intro">
          <p class="pill">AIGC · 运势解析 · 情绪陪伴</p>
          <h1>让 AI 根据你的输入，生成一份今日运势与提醒。</h1>
          <p class="muted">
            结合出生信息、情绪状态和提问内容，AI
            通过规则与语料库生成多维度的运势解读，并给出温柔但清醒的行动建议。
          </p>
          <ul class="list">
            <li>支持昵称、生日、星座、当下心情与具体问题。</li>
            <li>输出整体运势、感情、事业、财富、健康与行动建议。</li>
            <li>所有结果仅供娱乐，不会收集或上传你的隐私数据。</li>
          </ul>
          <p class="tiny muted">
            提示：本项目前端可直接部署到阿里云 ESA Pages，利用边缘节点加速访问体验。
          </p>
        </section>

        <section class="panel form-panel">
          <form class="form" @submit.prevent="onSubmit">
            <div class="form-row">
              <label>
                <span>你的昵称</span>
                <input v-model.trim="form.name" type="text" placeholder="如：阿七 / 小王 / Luna" />
              </label>
            </div>

            <div class="form-row two">
              <label>
                <span>生日</span>
                <input v-model="form.birthday" type="date" />
              </label>
              <label>
                <span>星座（可选）</span>
                <select v-model="form.zodiac">
                  <option value="">不清楚/不想填</option>
                  <option v-for="z in zodiacs" :key="z" :value="z">{{ z }}</option>
                </select>
              </label>
            </div>

            <div class="form-row">
              <label>
                <span>当前心情</span>
                <select v-model="form.mood">
                  <option value="">随便看看</option>
                  <option value="tired">有点累 / 压力大</option>
                  <option value="lost">有些迷茫</option>
                  <option value="excited">比较兴奋 / 期待</option>
                  <option value="calm">平静 / 稳定</option>
                  <option value="sad">有点难过</option>
                </select>
              </label>
            </div>

            <div class="form-row">
              <label>
                <span>你最想问的一件事</span>
                <textarea
                  v-model.trim="form.question"
                  rows="3"
                  placeholder="例如：要不要换工作？这段关系还要不要继续？今年适合做什么大的决定？"
                ></textarea>
              </label>
            </div>

            <div class="form-row actions">
              <button class="btn primary" type="submit" :disabled="loading">
                <span v-if="!loading">生成 AI 运势解读</span>
                <span v-else>AI 正在推演中...</span>
              </button>
              <button class="btn ghost" type="button" @click="resetForm">重置</button>
            </div>

            <p class="tiny muted">
              说明：本应用所有推理在浏览器本地完成，不上传到服务器，仅基于规则和随机权重生成文案。
            </p>
          </form>

          <section v-if="result" class="result">
            <h2>今日灵象解读</h2>
            <p class="muted summary">{{ result.summary }}</p>

            <div class="result-grid">
              <div class="result-block">
                <h3>整体运势 · {{ result.overall }}</h3>
                <p class="muted">{{ result.overallText }}</p>
              </div>
              <div class="result-block">
                <h3>感情</h3>
                <p class="muted">{{ result.love }}</p>
              </div>
              <div class="result-block">
                <h3>事业/学业</h3>
                <p class="muted">{{ result.career }}</p>
              </div>
              <div class="result-block">
                <h3>财富</h3>
                <p class="muted">{{ result.wealth }}</p>
              </div>
              <div class="result-block">
                <h3>健康/状态</h3>
                <p class="muted">{{ result.health }}</p>
              </div>
              <div class="result-block highlight">
                <h3>今日行动建议</h3>
                <p>{{ result.advice }}</p>
                <p class="tiny muted">
                  幸运色：{{ result.luckyColor }} · 幸运数字：{{ result.luckyNumber }} · 幸运方向：{{
                    result.luckyDirection
                  }}
                </p>
              </div>
            </div>
          </section>
        </section>
      </div>
    </main>

    <footer class="footer">
      <div class="container footer-inner">
        <div>灵象 AI 算命馆 · 实验性 AIGC 占卜应用</div>
        <div class="muted tiny">本项目由阿里云ESA提供加速、计算和保护 · 仅供娱乐，请勿替代专业建议。</div>
      </div>
    </footer>
  </div>
</template>

<script setup>
import { reactive, ref } from 'vue'

const zodiacs = [
  '白羊座',
  '金牛座',
  '双子座',
  '巨蟹座',
  '狮子座',
  '处女座',
  '天秤座',
  '天蝎座',
  '射手座',
  '摩羯座',
  '水瓶座',
  '双鱼座',
]

const form = reactive({
  name: '',
  birthday: '',
  zodiac: '',
  mood: '',
  question: '',
})

const loading = ref(false)
const result = ref(null)

function hashString(str) {
  let h = 0
  for (let i = 0; i < str.length; i++) {
    h = (h * 31 + str.charCodeAt(i)) >>> 0
  }
  return h
}

function pick(arr, seed, salt) {
  if (!arr.length) return ''
  const v = (seed + salt * 131) >>> 0
  return arr[v % arr.length]
}

function generateFortune(payload) {
  const base = `${payload.name || '无名客'}|${payload.birthday || '0000-00-00'}|${payload.zodiac || '未知'}|${
    payload.mood || 'normal'
  }|${payload.question || '随缘一卦'}`
  const seed = hashString(base)

  const moodLabelMap = {
    tired: '有点疲惫的你',
    lost: '略显迷茫的你',
    excited: '兴致不错的你',
    calm: '情绪稳定的你',
    sad: '心里有点酸的你',
    '': '今天的你',
  }

  const overallLevels = ['偏稳', '平衡', '小吉', '吉', '小凶但可解']
  const summaryTemplates = [
    '整体气场相对平稳，更多是对过往选择的复盘期，适合慢下来感受当下。',
    '外部节奏略快，但你真正需要的是给自己一段不被打扰的空白时间。',
    '有一些不经意的小好运，来自你过去某个认真对待过的细节。',
    '若能主动迈出一小步，运势会顺着你的行动排布出新的可能性。',
    '情绪容易被放大，记得在做重要决定前多睡一觉再确认。',
  ]

  const loveTexts = [
    '感情上更适合“慢热诚实”，适当表达你的在意，不必用试探证明安全感。',
    '若有伴，建议把近期的压力说清楚，对方比你想象中更愿意分担；若单身，可以主动多参加一点社交场合。',
    '这两天适合给旧关系一个温柔的收尾，或者给新关系一个轻松的开场。',
    '不要把所有对未来的焦虑都投射在某个人身上，你值得先把注意力拉回到自己身上。',
  ]

  const careerTexts = [
    '工作/学业更需要的是“做减法”，先完成最重要的 1-2 件事，其余的延后也来得及。',
    '可以尝试把一个复杂任务拆成三个可执行的小步骤，你会发现事情比想象中好推进。',
    '适合整理资料、复盘过往项目，给未来的自己留下一些清晰的路径和注释。',
    '如果你正在考虑跳槽或转向，先从“试水”和“副业尝试”开始，而不是一次性all in。',
  ]

  const wealthTexts = [
    '财运偏稳，不宜冲动消费或大额投资，适合做一些长期理财知识的学习。',
    '可以尝试记录一周的支出，你会发现一些“习惯性花费”其实并不能真正带来快乐。',
    '适合处理旧账和财务整理，把模糊变成清单，本身就是对自己的一种保护。',
    '若有合作机会，建议把边界与分成写清楚，模糊地带最容易滋生误会。',
  ]

  const healthTexts = [
    '身体更需要的是规律作息和好好吃饭，别再用熬夜奖励自己了。',
    '注意肩颈和眼睛，减少长时间久坐，哪怕每小时站起来走一走也有帮助。',
    '情绪上的小波动属于正常范围，试着用写日记或散步的方式帮自己消化。',
    '适合做一些低门槛的运动，比如拉伸、瑜伽或楼梯代替电梯，让身体重新变暖。',
  ]

  const adviceTexts = [
    '今天可以为自己做一件小小但明确的事情，例如整理桌面、清理手机相册，给生活腾出一点空白。',
    '把你最在意的问题写下来，再写出三种最坏结果和三种最好结果，你会发现“未知”本身才是最让人害怕的部分。',
    '如果很纠结，不妨先给自己一段“观察期”，在这段时间里只收集信息，不做重大决定。',
    '为自己安排一个与屏幕无关的小时，比如看纸质书、发呆、手写一页纸，让心跟上身体的步伐。',
    '试着先把关注点放在“我现在能做的一小步”上，而不是“如果我完美就好了”。',
  ]

  const colors = ['雾蓝', '暖白', '星空紫', '杏仁奶咖', '薄荷绿', '珊瑚橙']
  const directions = ['东', '西', '南', '北', '东南', '西北']

  const overall = pick(overallLevels, seed, 1)
  const summary =
    `${moodLabelMap[payload.mood || ''] || '今天的你'}，正在经历一个关于「${payload.question || '当下状态'}」的小节点。` +
    pick(summaryTemplates, seed, 2)

  return {
    overall,
    summary,
    overallText: pick(summaryTemplates, seed, 3),
    love: pick(loveTexts, seed, 4),
    career: pick(careerTexts, seed, 5),
    wealth: pick(wealthTexts, seed, 6),
    health: pick(healthTexts, seed, 7),
    advice: pick(adviceTexts, seed, 8),
    luckyColor: pick(colors, seed, 9),
    luckyNumber: (seed % 9) + 1,
    luckyDirection: pick(directions, seed, 10),
  }
}

function onSubmit() {
  loading.value = true
  setTimeout(() => {
    result.value = generateFortune(form)
    loading.value = false
  }, 500)
}

function resetForm() {
  form.name = ''
  form.birthday = ''
  form.zodiac = ''
  form.mood = ''
  form.question = ''
  result.value = null
}
</script>
