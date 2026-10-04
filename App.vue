<script setup>
import { ref } from 'vue'

// 注意：这次数据用 ref 包了一层。
// ref 的作用：数据一变，页面自动跟着变。普通数组做不到这一点。
// 记住规矩：<script> 里读写要加 .value，模板里写不用加（Vue 帮你解包）
const projects = ref([
  {
    name: '智能体',
    description: '通过温湿度传感器与摄像头，实时采集温室环境数据并自动调节通风与灌溉。',
    members: ['朱育田', '李雪'],
    tech: ['Vue 3', 'Node.js', 'MQTT', 'Python'],
    statusType: 'doing',
    status: '进行中',
    github: 'https://github.com/example/greenhouse',
  },
  {
    name: '实验室设备预约平台',
    description: '面向实验室设备的在线预约、审批与超时释放，替代原来的微信群登记方式。',
    members: ['王强'],
    tech: ['Vue 3', 'Element Plus'],
    statusType: 'done',
    status: '已结题',
    github: 'https://github.com/example/device-booking',
  },
  {
    name: '课堂表情识别实验平台',
    description: '接入摄像头采集课堂表情样本，用于心理学课程的课堂反馈分析（实验阶段，暂不开放使用）。',
    members: ['陈晓', '刘洋', '赵敏'],
    tech: ['Python', 'OpenCV', 'Vue 3'],
    statusType: 'pause',
    status: '暂停中',
    github: 'https://github.com/example/face-expression',
  },
])

// 表单开关：false 只显示按钮，true 显示表单
const showForm = ref(false)

// 表单里正在填的内容，每个输入框用 v-model 绑到对应字段
const form = ref({
  name: '',
  description: '',
  members: '',
  tech: '',
  statusType: 'doing',
  github: '',
})

// 下拉框存的是英文值（doing/done/pause），这里对应到页面上要显示的中文
const statusMap = {
  doing: '进行中',
  done: '已结题',
  pause: '暂停中',
}

// 输入框里是「张明、李雪」或「Vue 3, Node.js」这种一整串文字，
// 这里把它切成数组，好和上面现有数据的格式保持一致
function toList(text) {
  return text
    .split(/[,，]/)
    .map((item) => item.trim())
    .filter((item) => item)
}

// 点「保存」时执行：把表单内容加进项目列表
function submitProject() {
  projects.value.push({
    name: form.value.name,
    description: form.value.description,
    members: toList(form.value.members),
    tech: toList(form.value.tech),
    statusType: form.value.statusType,
    status: statusMap[form.value.statusType],
    github: form.value.github,
  })

  showForm.value = false
  // 清空表单，下次新增时不会残留上次填的内容
  form.value = {
    name: '',
    description: '',
    members: '',
    tech: '',
    statusType: 'doing',
    github: '',
  }
}

// 点「取消」：关掉表单，不保存
function cancelForm() {
  showForm.value = false
}

// 正在确认删除的是第几个项目（null = 现在没有在确认）
// 用下标而不用项目名，是因为名字可能重复
const pendingDelete = ref(null)

// 点「删除」：先问一句，不真的删
function askDelete(index) {
  pendingDelete.value = index
}

// 点「确认删除」：真的删
function confirmDelete(index) {
  projects.value.splice(index, 1)
  pendingDelete.value = null
}

// 点「不删了」：退出确认状态
function cancelDelete() {
  pendingDelete.value = null
}
</script>

<template>
  <main class="page">
    <header>
      <div class="head-row">
        <div>
          <h1>实验室项目展示板</h1>
          <p class="subtitle">共 {{ projects.length }} 个项目</p>
        </div>
        <button v-if="!showForm" class="btn-primary" @click="showForm = true">
          + 新增项目
        </button>
      </div>
    </header>

    <!-- .prevent 很关键：不加它，点保存页面会整页刷新，填的东西全没了 -->
    <form v-if="showForm" class="form" @submit.prevent="submitProject">
      <div class="field">
        <label for="name">项目名称</label>
        <input id="name" v-model="form.name" required placeholder="例如：智能温室监测系统" />
      </div>

      <div class="field">
        <label for="description">项目简介</label>
        <textarea
          id="description"
          v-model="form.description"
          rows="3"
          placeholder="一句话说明这个项目是做什么的"
        ></textarea>
      </div>

      <div class="field">
        <label for="members">项目成员</label>
        <input
          id="members"
          v-model="form.members"
          placeholder="顿号或逗号分隔，例如：张明、李雪"
        />
      </div>

      <div class="field">
        <label for="tech">技术栈</label>
        <input
          id="tech"
          v-model="form.tech"
          placeholder="逗号分隔，例如：Vue 3, Node.js"
        />
      </div>

      <div class="field">
        <label for="status">项目状态</label>
        <select id="status" v-model="form.statusType">
          <option value="doing">进行中</option>
          <option value="done">已结题</option>
          <option value="pause">暂停中</option>
        </select>
      </div>

      <div class="field">
        <label for="github">GitHub 地址</label>
        <input id="github" v-model="form.github" placeholder="https://github.com/..." />
      </div>

      <div class="form-actions">
        <button type="submit" class="btn-primary">保存</button>
        <button type="button" class="btn-ghost" @click="cancelForm">取消</button>
      </div>
    </form>

    <ul class="list">
      <li v-for="p in projects" :key="p.name" class="card">
        <div class="card-head">
          <h2 class="card-title">{{ p.name }}</h2>
          <span class="tag" :class="'tag-' + p.statusType">{{ p.status }}</span>
        </div>

        <p class="desc">{{ p.description }}</p>

        <div class="meta">
          <p class="meta-line">
            <span class="label">成员</span>
            {{ p.members.join('、') }}
          </p>
          <p class="meta-line">
            <span class="label">技术栈</span>
            <span v-for="t in p.tech" :key="t" class="tech">{{ t }}</span>
          </p>
        </div>

        <a class="link" :href="p.github" target="_blank" rel="noopener">
          GitHub 仓库 →
        </a>
      </li>
    </ul>
  </main>
</template>
