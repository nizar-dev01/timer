<template>
  <div
    id="app"
    :class="currentState"
  >
      <div class="time-container">
          <div class="time-box">
              <h2 class="time-text">
                {{ currentTime }}
              </h2>
          </div>
          <div class="metadata-box">
              <input
                type="text"
                class="metadata-input"
                v-model="metadataText"
                ref="InputText"
                placeholder="Enter Name/Metadata"
              />
          </div>
      </div>
      <div class="action-box-container">
          <button
              type="button"
              class="action-button"
              @click.prevent="runButtonAction(
                has_started ?
                  ( is_on_pause ? 'START' : 'PAUSE')
                : 'START'
              )"
          >
            {{ has_started ? is_on_pause ? 'RESUME' : 'PAUSE' : 'START' }}
          </button>
          <button
              type="button"
              class="action-button"
              @click.prevent="runButtonAction(
                has_started ? 'END' : 'REPORT'
              )"
          >{{has_started ? 'END' : 'View Records'}}</button>
      </div>
      <div class="records-container" @click="hideRecords" v-if="toggleRecords">
        <div class="records-box" @click.stop>
          <table class="tbl-records" cellspacing="1">
            <thead>
              <tr>
                <th>
                  Name
                </th>
                <th>
                  Time Taken
                </th>
                <th></th>
              </tr>
            </thead>
            <tbody>
              <tr
              v-for="data in data_store"
              class="body"
              :class="{red: !data.is_qualified}"
              >
                <td>
                  {{ data.metadata || "-" }}
                </td>
                <td>
                  {{ data.time }}
                </td>
                <td class="remove-text" @click="_remove_store(data.id)">
                  Remove
                </td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
  </div>
</template>
<script setup>
const metadataText = ref("")

const time_template = {
  sec: 0,
  min: 0,
}

const current_time = ref({...time_template})

const currentTime = computed(()=> {
  const minute = current_time.value.min
  const second = current_time.value.sec
  return `00${minute}`.slice(-2) + ":" + `00${second}`.slice(-2)
})
let timer_interval;

const active_time = 4
const warning_time = 5
const danger_time = 6
const stop_time = 7

const currentState = computed(() => {
  const minute = current_time.value.min
  if(minute >= active_time && minute < warning_time){
    return "active"
  }
  if(minute >= warning_time && minute < danger_time){
    return "warning"
  }
  if(minute >= danger_time && minute < stop_time){
    return "danger"
  }
  if(minute >= stop_time) return "stop"
  return "default"
})

const is_actively_running = ref(false)
const is_on_pause = ref(false)
const has_started = ref(false)

const startTimer = (
  initTime = is_on_pause.value ?
    current_time.value :
    time_template
) => {
  // Set the initial Values
  for (let time_field in initTime){
    current_time.value[time_field] = initTime[time_field]
  }
  // Start the timer
  is_actively_running.value = true
  is_on_pause.value = false
  has_started.value = true
  const runner = () => {
    const _timer = current_time.value
    if(_timer.sec >= 60) {
      _timer.sec = 0
    } else{
      return  _timer.sec+=30
    }
    if(_timer.min >= 60) {
      _timer.min = 0
    } else {
      return _timer.min++
    }
  }
  runner()
  timer_interval = setInterval(runner, 1000)
}

const pauseTimer = () => {
  clearInterval(timer_interval)
  is_on_pause.value = true
}

const resetTimer = () => {
  is_actively_running.value = false
  is_on_pause.value = false
  has_started.value = false
  for(let key in time_template){
    current_time.value[key] = time_template[key]
  }
  metadataText.value = ""
}

const _field = '__timer_data__'
const _get_store = () => {
  const store = process.client && JSON.parse(localStorage.getItem(_field)) || []
  data_store.value = store
  return store
}
const _set_store = store => {
  localStorage.setItem(_field, JSON.stringify(store))
  data_store.value = store
}

const _insert_store = (_data) => {
  const stored_data = _get_store()
  const timestamp = new Date().getTime()
  _data.id = `${stored_data.length}-${Math.random()}-${timestamp}`
  stored_data.push(_data)
  _set_store(stored_data)
}

const _remove_store = (id) => {
  const store = _get_store()
  let i = 0
  let element = null
  while(true){
    if(i >= store.length || !!element){
      break
    }
    if(store[i] && store[i].id === id){
      element = store[i]
      break
    } else i++
  }
  store.splice(i, 1)
  _set_store(store)

}

const takeSnapshot = () => {
  const data = {
    time: currentTime.value,
    metadata: metadataText.value,
    is_qualified: current_time.value.min >= active_time && current_time.value.min < stop_time
  }
  _insert_store(data)
  // debugger
}

const endTimer = () => {
  pauseTimer()
  takeSnapshot()
  resetTimer()
  showRecords()
}

const toggleRecords = ref(false)
const data_store = ref([])

const showRecords = () => {
  _get_store()
  toggleRecords.value = true
}

const hideRecords = () => {
  toggleRecords.value = false
}

const runButtonAction = (action) => {
  switch(action){
    case "START":
      startTimer()
    break
    case "PAUSE":
      pauseTimer()
    break
    case "END":
      endTimer()
    break
    case "REPORT":
      showRecords()
    break
    default:
      console.log("The default action")
  }
}

const InputText = ref(null)
</script>
<style lang="scss" src="./assets/style/index.scss">
</style>
