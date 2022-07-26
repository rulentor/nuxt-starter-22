<template>
    <div class="grid panel-demo">
	    <ConfirmDialog/>
          <Dialog header="Добавить субтитры" v-model:visible="showAddSubs" :breakpoints="{'960px': '75vw', '640px': '90vw'}" :style="{width: '50vw'}" :modal="true">
			<div class="card p-fluid">
			  <div class="formgrid grid">
			    <div class="field col">
				<ul id="rendering">
				<template v-for="item in subtitles">
				  <li>{{ item.name }}</li>
				</template>
				</ul>
				</div>
			  </div>
			</div>
            <template #footer>
			    <FileUpload name="subtitles[]" :customUpload="true" @uploader="importSubsUploader" mode="basic" ref='file' accept="" :maxFileSize="50000000000" label="Загрузка субтитра" chooseLabel="Загрузка субтитра" class="mr-2" />
                <Button label="Отмена" icon="pi pi-times" @click="showAddSubs = !showAddSubs" class="p-button-text"/>
            </template>
        </Dialog>		
                <Dialog header="Новое сканирование" @addProject="emitProjectAdd" v-model:visible="showAddVideo" :breakpoints="{ '960px': '75vw' }" :style="{ width: '30vw' }" :modal="true">
					<div class="card p-fluid">

					<div class="formgrid grid">
							<div class="field col">
								<label for="date">Дата и время</label>
								<InputText id="date" type="text" :value="state.datetime"/>

							
							</div>
							<div class="field col">
	
								<label for="frequency">Частота кадров в сек.</label>
								<InputText id="frequency" type="text" :value="state.frequency"/>								
							</div>
					</div>
						
						<div class="field">
								<label for="link">Ссылка на видео</label>
								<InputText id="link" type="text" :value="state.link"/>
						</div>
						<div class="field ">
								<label for="frequency">Направление</label>
                                <Dropdown v-model="state.backward" :options="optionsDirect" optionLabel="name" optionValue="id" placeholder="выберите" />							
						</div>
					<div class="formgrid grid">
						<div class="field col">
								<label for="frequency" class="ml-6">Загрузка видеофайла</label>
                                <FileUpload name="file[]" :customUpload="true" @uploader="importUploader" mode="basic" ref='file' accept="" :maxFileSize="50000000000" label="Видео" chooseLabel="Видео" class="ml-6" />					
							</div>
							<div class="field col">
								<label for="frequency" class="ml-6">Загрузка субтитров</label>
			                    <FileUpload name="subtitles[]" :customUpload="true" @uploader="importSubsUploader" mode="basic" ref='file' accept="" :maxFileSize="50000000000" label="Субтитры" chooseLabel="Субтитры" class="ml-6" />							
							</div>
					</div>

						
					</div>
                    <template #footer>
					    <Button label="Начать сканирование" @click="addVideo" icon="pi pi-check" class="p-button-success" />
						<!--Button label="Субтитры" @click="showAddSubs = !showAddSubs" icon="pi pi-check" class="p-button-success" /-->
                        <Button label="Отменить" @click="showAddVideo = !showAddVideo" icon="pi pi-check" class="p-button-secondary" />
                    </template>
                </Dialog>	
                <Dialog header="Новый проект" v-model:visible="showAddProject" :breakpoints="{ '960px': '75vw' }" :style="{ width: '30vw' }" :modal="true">
					<div class="card p-fluid">

					<div class="formgrid grid">
							<div class="field col">
								<label for="name">Наименование</label>
								<InputText id="name" v-model="state.project_name" type="text" class="p-input-sm mb-3" placeholder="Наименование" style="padding:1rem;" />
							</div>
						</div>

					</div>
                    <template #footer>
					    <Button label="Создать" @click="addProject" icon="pi pi-check" class="p-button-success" />
                        <Button label="Отменить" @click="showAddProject = !showAddProject" icon="pi pi-check" class="p-button-secondary" />
                    </template>
                </Dialog>					
        <div class="col-12 md:col-12">
            <Toast />
			<template v-for="(item, index) of projects">
            <Card v-if="projects[index].id" class="mb-2">
                <template v-slot:title>
                    <div class="flex align-items-center justify-content-between mb-0">
                       <h5>Проект # {{ item.name }}</h5>
                   </div>
                </template>

                <template v-slot:content>

            <Toolbar class="mb-1">
                <template #start>
					<Button icon="pi pi-plus" label='Добавить сканирование' class="p-button-raised p-button-text" @click="startAddVideo(item.id, index, item)" />
                </template>
                <template #end>
					<!--Button icon="pi pi-cloud-download" label='Общий отчёт' class="p-button-raised p-button-text" @click="showFullReport(index)" /-->						
					<Button icon="pi pi-trash" label='Удалить проект' style='margin-left: 6px' class="p-button-raised p-button-text" @click="confirmDelete(index, 'project', 'проект')" />
                </template>
            </Toolbar>

                <DataTable
                    :value="item.video"
                    :paginator="true"
                    class="p-datatable-gridlines"
                    :rows="3"
                    dataKey="id"
                    :rowHover="true"
                    :loading="loading"
                    responsiveLayout="scroll"
                >
                    <template #empty> Ничего не загружено. </template>
                    <template #loading> В процессе... </template>
                    <Column field="id" header="Id сканирования" style="min-width: 12rem">
                        <template #body="{ data }">
                            {{ data.id }} {{ stateuser.last_name }}
                        </template>
                    </Column>
                    <Column field="downloaded" header="Сканирование" data-type="date" style="min-width:2rem" headerStyle="text-align: center" bodyStyle="text-align: center">
                     <template #body="{data}">
                        <span style="width:100%;text-align:center">{{ data.downloaded }} <p>{{ data.video_time }}</p></span>
                     </template>
                    </Column>
					  <Column field="status" header="Статус" :show-filter-match-modes="false" headerStyle="text-align: center" style="min-width: 2rem;" bodyStyle="text-align: center; ">
					<template #body="{data}" class="">
					 <span style="text-align:center" class=''>
					   {{ (data.status > 1 || data.status < 99)? `${data.status}%` : data.status }} 
					   <!--div v-if="data.status != 'Выполнено'" class="preloader-container">
						  <div class="circle circle-1" style="margin-left:3rem"></div>
						  <div class="circle circle-2"></div>
						  <div class="circle circle-3"></div>
						  <div class="circle circle-4"></div>
						  <div class="circle circle-5"></div>
					   </div-->
					   <ProgressBar v-if="data.status != 'Выполнено' && data.status != 'Начало работы'" :value="data.status" :showValue="false" style="height: 1em;" class='m-3 mt-1'/>
					 </span>
					</template>
					  </Column>					
					
					
					

                    <Column header="Действия">
                        <template #body="{data,index}, slotProps">
							  <a href="javascript:goVideo(data.id);" @click="goVideo(data.id)">
								<i class="pi pi-eye mr-4 p-text" style="font-size: 1.5rem; margin-right: 32px !important;" v-badge="data.founded_anomaly"></i>
							  </a>
							  <!--router-link to="#" class="mr-2" @click="(event) => fullReport(data.id)">
								<i class="pi pi-file mr-4 p-text-secondary" style="font-size: 1.5rem"></i>
							  </router-link-->
							  <router-link to="#" class="mr-2" @click="fullDocxReport(data)">
								<i class="pi pi-file mr-4 p-text-secondary" style="font-size: 1.5rem"></i>
							  </router-link>							  
							  <router-link to="#" @click="(event) => confirmDelete(data.id, 'video', 'сканирование')" >
								<i class="pi pi-trash mr-4 p-text-danger" style="font-size: 1.5rem"></i>
							  </router-link>
                        </template>
                    </Column>					
                </DataTable>
                </template>
            </Card>
			</template>
        </div>

</div>
</template>

<script setup lang='ts'>
import { FilterMatchMode, FilterOperator } from 'primevue/api'
import { ref, reactive, onMounted, nextTick } from 'vue'
import { useCookie } from 'vue-cookie-next'
import fetch from 'cross-fetch'
import { useToast } from 'primevue/usetoast'
import { useRouter, useRoute } from "vue-router"
import { useConfirm } from "primevue/useconfirm"
import { useMenuStore } from '~/stores/menu'
import { useFetch } from '@vueuse/core'
import { Readable } from 'stream'
import * as XLSX from 'xlsx/xlsx.mjs'
import { read, writeFileXLSX } from 'xlsx'
import { set_cptable } from 'xlsx'
import * as cptable from 'xlsx/dist/cpexcel.full.mjs'
import * as cheerio from 'cheerio'
import * as htmlparser2 from 'htmlparser2'
import { saveAs } from 'file-saver'
import { useWebSocket } from '@vueuse/core'
set_cptable(cptable)
const props = defineProps({
  foo: String
})
const menustate = useMenuStore()
const route = useRoute()
const router = useRouter()
const emit = defineEmits(['addProject'])
const showAddProject = ref(false)
const showAddSubs = ref(false)
emit('addProject',showAddProject)
menustate.setMenuEmit(showAddProject)
const emitProjectAdd = () => {
  console.log('Emit',emit('addProject'))
  showAddVideo.value = true
}
menustate.setMenu({menu: 'stuff'})
console.log('menu-state',menustate.usedstate)
watchEffect(() => {
  console.log('Emit',emit('addProject'))
})
const toast = useToast()
const confirm = useConfirm()
const { getCookie, setCookie, removeCookie } = useCookie()
const file = ref()
const menu = ref()
const projects = ref()
const stateuser = getCookie('userState')
const showAddVideo = ref(false)
//const selectDirect = ref()
const state = reactive({
  project_id: null,
  project_name:'',
  video_id: null,
  file: null,
  frequency: 10,
  //direct: 'normal',
  backward: false,
  datetime: ''
})
const display = ref(false)
const loading = ref(false)
const customer = ref(null)
const filters = ref({global: []})
const deleteProductsDialog = ref(false)
const optionsDirect = ref([
  { id: 0, name: 'прямое' },
  { id: 1, name: 'обратное' },  
])
const toolbarItems = ref([
                {
                    label: 'Save',
                    icon: 'pi pi-check',
                },
                {
                    label: 'Update',
                    icon: 'pi pi-upload',
                },
                {
                    label: 'Delete',
                    icon: 'pi pi-trash',
                },
                {
                    label: 'Home Page',
                    icon: 'pi pi-home',
                },
])
const cardMenu = ref([
                { label: 'Save', icon: 'pi pi-fw pi-check' },
                { label: 'Update', icon: 'pi pi-fw pi-refresh' },
                { label: 'Delete', icon: 'pi pi-fw pi-trash' },
])
onMounted (
  () => {
    projects.value = []
	customer.value = []
	clearFilter()
	getProjects()
	const my = getCookie('userState')
    console.log('user',my)	
  }
)
const sheetName = ref()
const addSheet = () => {
console.log('add-sheet')
}
const startAddProjects = (event) => {
  console.log('add-projects',event)
}
const menuSnake = async () => { 
let result = []
projects.value.map((data,key) => {
 result.push({ label: data.name, project_id: data.id, icon: 'pi pi-fw pi-bookmark', class: 'p-list-menu', items: []})
 data.video.map(item => {
   result[key].items.push({ label: `${item.founded_anomaly} Сканирование ${item.downloaded}`, icon: 'pi pi-fw pi-pause', to: `/projects-scans/${item.id}` })
 })
})
router.addMenu(result)
setCookie('userMenu', JSON.stringify(result))
console.log('записали кук-меню',getCookie('userMenu'))
}
const sockets = ref({})
let interval = {}
const watchVideoStatus = (colId, rowId, id) => {
  //console.log('project-video',colId, rowId, id)
  //sockets[id] = useWebSocket(`ws://84.201.177.58/ws/test/5/?token=d9c02de1db59ca8d07c25dd15ac2de1bb7cb9065`,{   
  sockets[id] = useWebSocket(`ws://84.201.177.58/ws/ready/${id}/?token=${stateuser.token}`,{  
    heartbeat: {
      message: 'ping',
      interval: 1000,
    },
  })  
  
  interval[id] = window.setInterval(() => {  
    //console.log(sockets[id].data._value)
	const raw = JSON.parse(sockets[id].data._value) || {}
	let status = (raw)? raw.complete :'Начало работы'
	if(status === 0) status = 'Начало работы'
	if(status === 100) status = 'Выполнено'
	projects.value[colId].video[rowId].status = status
	//raw.status === 'Ready'
	if(status === 'Выполнено') window.clearInterval(interval[id])
	if(status === 'Выполнено') projects.value[colId].video[rowId].status = 'Выполнено'	
  },1000)
}
const videoList = () => {
  projects.value.map((items, colId) => {
	items.video.map((item, rowId) => {
	  watchVideoStatus(colId, rowId, item.id)
	})
  })
}
const getProjects = async () => {
  loading.value = false
  try {
    const res = await fetch(`http://84.201.177.58:3000/api/cases`,{
	  //method: 'post',
	  headers: {
	  "Content-Type": 'application/json',
	  'Authorization':'Token  '+stateuser.token
	  },
	})
    
    if (res.status >= 400) {
      throw new Error("Bad response from server")
    }
    
    projects.value = await res.json()
	menuSnake() 
	videoList()
    //console.log('projects',projects.value)
	loading.value = false
  } catch (err) {
    console.error(err)
  }
  
}
const obj = ref({})
//const subtitles = ref([])
let reader = null
const formData = ref(null)
const importUploader = async (event) => {
  console.log('получили файл с кнопки')
  const direction = (state.backward)? 'обратное' :'прямое'
  formData.value = new FormData()
  reader = new FileReader()
     
  const fileVideo = event.files[0]
  formData.value.append(`file`, event.files[0],event.files[0].name)
  toast.add({severity: 'warn', summary: `Файл подгружен успешно. Направление ${direction}`, detail: 'название ' + event.files[0].name, life: 3000})
}
const importSubsUploader = async (event) => {
  console.log('получили субтитр с кнопки')  
  if(!reader) return
  const direction = (state.backward)? 'обратное' :'прямое'  
  //subtitles.value.push(event.files[0])
  //const len = subtitles.value.length
  formData.value.append(`subtitles`, event.files[0],event.files[0].name)
  toast.add({severity: 'warn', summary: `Субтитры подгружены успешно. Направление ${direction}`, detail: 'название ' + event.files[0].name, life: 3000})
}
const goVideo = async (id) => {
  state.video_id = id
    router.push(`/projects-scans/${encodeURIComponent(id)}`)
}
import fs from 'vite-plugin-fs/browser'

const fullDocxReport = async (item) => {  
  //report_path, organization
  saveAs(item.report_path, 'Репорт.docx')
}  

const fullReport = async (id) => {  
  console.log(' ..',id)
  try{
    const res1 = await fetch(`http://84.201.177.58:3000/api/report/${id}`,{
      headers: { 
        "Content-Type": 'application/json',  
        'Authorization':'Token '+ stateuser.token 
      },
    })  
    if (res1.status >= 400) {
      throw new Error("Bad response from server")  
    }
    const reports = await res1.json()
    const res2 = await fetch('/data/report.html')
    if (res2.status >= 400) {
      throw new Error("Bad response from server") 
    }
    const html = await res2.text()
    const parser = new DOMParser()
    const $ = cheerio.load(html, null, false)    
    const tpl = `
      <tr>
      <td data-t="n" data-v="2890105" id="sjs-A7">2890105</td>
      <td data-t="n" data-v="0">0</td>
      <td data-t="n" data-v="0">0</td>
      <td>1</td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td></td>
      <td>100</td>
      <td></td>
      <td></td>
      <td></td>
      </tr>
`
  let $tpl_main = cheerio.load('<div id="container">'+$('div').html()+'</div>', null, false)
  let $tpl = cheerio.load('<div></div>', null, false)
  const tmp = reports.shift()
  reports.pop()
  reports.reverse()
  let count = 4
  let start = null
  reports.map((d,key) => {
    count = 4
    start = d.message.split(' ')
    $tpl('div').append(tpl)
    $tpl('tr:eq('+key+') td:eq(1)').attr('data-v',start[2])
    $tpl('tr:eq('+key+') td:eq(2)').attr('data-v',start[5])
      Object.keys(d.codes).map(id => {
        let step = new Array(d.codes[id])
		
		//привет Женя твоя мать ждёт
		step = [1]
		if(id < 10) id = parseInt(id) + 50
        for(const i of step){
          $tpl('tr:eq('+key+') td:eq('+count+')').attr('data-v',id)
          count++
        }
      })
      $tpl_main('<tr>'+$tpl('tr:eq('+key+')').html()+'</tr>').insertAfter('#insert-data')
   })
   const dom = parser.parseFromString($tpl_main('div').html(), 'text/html')
   const wbexport = XLSX.utils.table_to_book(dom)//,{sheet:'-1'}
   XLSX.writeFile(wbexport, `Отчет по видео ID сканирования - (${id}).xlsx`) 
  } catch (err) {
    console.error(err)
  }
}
const fullReport2 = async (id) => {
  console.log('полный отчёт..',id)
  try {
    const res = await fetch(`http://84.201.177.58:3000/api/videos/${id}`,{
	  //method: 'post',
	  headers: {
	  "Content-Type": 'application/json',
	  'Authorization':'Token  '+stateuser.token
	  },
	})
    
    if (res.status >= 400) {
      throw new Error("Bad response from server")
    }
    const scans = await res.json()
    console.log('projects-video',scans.value)	
	let result = {
	  videoId: 0,
	  fio: '',
	  date: '',
	  count: 0,
	  status: '',
	  videoUrl: '',
	  videoTime: 0,
	  items: [],
	  //header: ["Сканирование id", "Оператор", "Дата загрузки", "Найдено", "Статус", "Видео", "Длительность видео", "Фото", "Дефект", "Таймкод"],
	  //photoUrl: '',
	  //detection: '',
	  //timecode: ''
	}
   
    scans.video_key_post.forEach((data, key) => {
	  result.items.push({
	  "Сканирование id": id,
	  "Оператор": scans.user,
	  "Дата загрузки": scans.downloaded,
	  "Найдено": scans.founded_anomaly,
	  "Статус": scans.status,
	  "Ссылка видео": scans.video, //'https://yandex.cloud/videos/id',
	  "Длительность видео": scans.video_time,
	  "Фото": data.photo,
	  "Дефект": data.detection,
	  "Таймкод": data.time	  
	  })
	})
	console.log('Excell',result.items)
	    const items = XLSX.utils.json_to_sheet(result.items)
        const wb = XLSX.utils.book_new()
        XLSX.utils.book_append_sheet(wb, items, 'data')
        XLSX.writeFile(wb,'Отчёт по сканированию - '+'.xlsx')
  } catch (err) {
    console.error(err)
  }
const jsonData = {
  fileName: 'Полный отчёт по #'+event+'.xlsx',
  data: [
    { name: "Tom", phone: "+86 01012", email: "000@gmail.com" },
    { name: "Jack", phone: "+86 01012", email: "000@gmail.com" },
    { name: "Alice", phone: "+86 01012", email: "000@gmail.com" }
  ],
  
}  
}
const addProject = async () => {  
  loading.value = true
  try {
    const res = await fetch(`http://84.201.177.58:3000/api/cases`,{
	  method: 'post',
	  headers: {
	  "Content-Type": 'application/json',
	  'Authorization':'Token  '+stateuser.token
	  },
	  body: JSON.stringify({ name: state.project_name })
	})
    
    if (res.status >= 400) {
      throw new Error("Bad response from server")
    }
	getProjects()
	showAddProject.value = false
	loading.value = false
  } catch (err) {
    console.error(err)
  }   
}
const startAddVideo = (projectId, index) => {
  state.project_id = projectId
  showAddVideo.value = true
}
const autoUpdate = () => {
nextTick(() => {
    let interval = window.setInterval(() => {
      getProjects()
    },2500)
})
}
const addVideo = async () => {
  formData.value.append('project_id', state.project_id)  
  formData.value.append('freq', state.frequency)  
  formData.value.append('backward', state.backward)   
  loading.value = true
  showAddVideo.value = false
  try {
    const res = await fetch(`http://84.201.177.58:3000/api/videos`,{
	  method: 'post',
	  headers: {
	  'Authorization':'Token  '+stateuser.token
	  },
	  body: formData.value
	})
    
    if (res.status >= 400) {
      throw new Error("Bad response from server")
    }
	autoUpdate()
  } catch (err) {
    console.error(err)
  } 
}
const confirmDelete = (id, action, title) => {
  confirm.require({
    message: 'Необходимо подтверждение операции',
    header: 'Удалить '+title,
    icon: 'pi pi-info-circle',
    acceptClass: 'p-button-danger',
      accept: () => {
	    if(action == 'project') deleteProject(id)
		if(action == 'video') deleteVideo(id)
        toast.add({severity:'info', summary:'Успех', detail:'Запись удалена успешно', life: 3000})
      },
      reject: () => {
        toast.add({severity:'error', summary:'Неуспех', detail:'Отмена удаления', life: 3000})
      }
  })
}
const deleteProject = async (id) => {
   loading.value = true
  
  try {
    const res = await fetch(`http://84.201.177.58:3000/api/cases/${projects.value[id].id}`,{
	  method: 'delete',
	  headers: {
	  "Content-Type": 'application/json',
	  'Authorization':'Token  '+stateuser.token
	  },
	})
    
    if (res.status >= 400) {
      throw new Error("Bad response from server")
    }
	projects.value[id].id = null
	loading.value = false
  } catch (err) {
    console.error(err)
  }  
}
const deleteVideo = async (id) => {
  loading.value = true
  try {
    const res = await fetch(`http://84.201.177.58:3000/api/videos/${id}`,{
	  method: 'delete',
	  headers: {
	  "Content-Type": 'application/json',
	  'Authorization':'Token  '+stateuser.token
	  },
	  
	})
    
    if (res.status >= 400) {
      throw new Error("Bad response from server")
    }
	getProjects()
  } catch (err) {
    console.error(err)
  }  
}
const showFullReport = async (id) => {
  console.log('full-report')
}
const open = (event) => {
  display.value = true
}
const close = (event) => {
  display.value = true
}
const toggle = (event) => {
  menu.value.toggle(event)
}
const initFilters = () => {
            filters.value = {
                global: { value: null, matchMode: FilterMatchMode.CONTAINS },
                name: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
                'country.name': { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.STARTS_WITH }] },
                representative: { value: null, matchMode: FilterMatchMode.IN },
                date: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.DATE_IS }] },
                balance: { operator: FilterOperator.AND, constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }] },
                status: { operator: FilterOperator.OR, constraints: [{ value: null, matchMode: FilterMatchMode.EQUALS }] },
                activity: { value: null, matchMode: FilterMatchMode.BETWEEN },
                verified: { value: null, matchMode: FilterMatchMode.EQUALS },
            }
}
const clearFilter = () => initFilters()
</script>
