<template>
  <div class="grid">
    <div class="col-13 md:col-7">
      <div class="card">
	  
	<Galleria
	  :value="images" 
	  :responsiveOptions="responsiveOptions" 
	  :numVisible="1" 
	  containerStyle="width: 100%"
	  :containerClass="{
	    'custom-galleria': true, 
		fullscreen: fullScreen,
		'p-image-mask': fullScreen, 
		'p-component-overlay': fullScreen 
	  }"
      :circular="true" 
	  :showItemNavigators="false" 
	  :showThumbnails="false" 
	  v-model:visible="startGallery"
	  @click="toggleFullScreen"
	>
		<template #item="slotProps">
	  <span class="p-image p-component p-image-preview-container">
		<Image :src="slotProps.item.itemImageSrc" :class="{'p-image-preview': fullScreen}" :alt="slotProps.item.alt" :style="[{'width': !fullScreen ? '100%' : '', 'display': !fullScreen ? 'block' : ''}]" />
			<!--img :src="slotProps.item.itemImageSrc" :alt="slotProps.item.alt" style="width: 100%; display: block;" /-->

	  <div class="p-image-preview-indicator">
	    <i class="p-image-preview-icon pi pi-angle-left" style="left: 3rem;position: absolute;" @click="prevSlide"></i>
	    <i class="p-image-preview-icon pi pi-eye" @click="toggleFullScreen"></i>
		<i class="p-image-preview-icon pi pi-angle-right" style="right: 3rem;position: absolute;" @click="nextSlide"></i>
	  </div>
	  </span>
		</template>
    <template #footer>
        <div class="custom-galleria-footer" style="padding: 12px">
            <span v-if="images" class="title-container">
                <span>{{activeIndex + 1}}/{{images.length}}</span>
                <span class="title">{{images[activeIndex].title}}</span>
                <span>{{images[activeIndex].alt}}</span>
            </span>
        </div>
    </template>		
	</Galleria>	

      </div>	  
      <div class="card col-12 md:col-8">

		<Toast />
        <span class="p-buttonset text-center" style="margin-left:auto">
          <Button label="Загрузить" class='p-button-outlined p-button-sm' icon="pi pi-cloud-upload" @click='savePhoto(state.photo)' />
          <Button label="Ложное срабатывание" class='p-button-danger p-button-outlined p-button-success p-button-sm' icon="pi pi-times" />
          <Button label="Подтвердить" class='p-button-success p-button-outlined p-button-secondary p-button-sm' icon="pi pi-check" @click="movePage"/>
        </span>
</div></div>
    <div class="col-11 md:col-5">
<div class="card">
        <h5 style="text-align:center">Статус сканирования - {{ scans.status }} от {{ scans.downloaded }}</h5>
        <!--Accordion :activeIndex="0">
            <AccordionTab :header="$route.params.id"
		<DataTable
		  v-bind:first.sync="first"
		  removableSort
		  selectionMode="single"        		
		-->

        <DataTable
		  ref="dt"
          class="p-datatable-sm" 
          data-key="id"
		  v-model:selection.sync="selectedRow" 
		  :sortOrder="1"
		  :sortField="initSortField" 
		  :lazy="false"
          :value="scans.video_key_post"
          :paginator="true"
          :rows="8"
          :loading="loading"
          responsive-layout="scroll"
		  @rowSelect="onRowSelect"
		  @sort="onSort($event)"
		  @page="onPage($event)"
        >
          <template #empty>
            Ничего не найдено.
          </template>
          <template #loading>
            Режим загрузки данных...
          </template>
		  <Column header="Фото" selectionMode="single" style="min-width: 10rem" :exportable="false"></Column>
          <Column field="time" header="Таймкод" style="min-width:6rem" headerStyle="width:100%;text-align: center" bodyStyle="text-align: center"/>
          <Column field="detection" header="Повреждение" style="min-width:15vw" headerStyle="width:100%;text-align: center" bodyStyle="text-align: center"/>
          <Column field="probability" header="Прогноз" :sortable="true" style="min-width:2rem" headerStyle="width:100%;text-align: center" bodyStyle="text-align: center">
   
			<template #body="{data}">
              {{ startSorting(data) }} {{ data.probability }}
            </template>
		  </Column>
          <!--Column field="photo" header="Изображения" data-type="text" body-class="text-center" style="min-width:12rem" headerStyle="width:100%;text-align: center">
            <template #body="slotProps">
			  <Button type="button" class="p-button-sm" label="Смотреть" :to="{hash: true}" @click="showScan('select', slotProps)"></Button>
            </template>
            <template #filter="{filterModel}">
              <TriStateCheckbox v-model="filterModel.value" />
            </template>
          </Column-->
        </DataTable>
		
            <!--/AccordionTab>
        </Accordion-->		
		
	  <!-- Card / -->	

</div>
  </div>
  </div>
</template>
<script setup lang='ts'>
import { ref, reactive, watchEffect, onMounted } from 'vue'
import { FilterMatchMode, FilterOperator } from 'primevue/api'
import { useCookie } from 'vue-cookie-next'
import fetch from 'cross-fetch'
import { useToast } from 'primevue/usetoast'
import { useRouter, useRoute } from "vue-router"
import { useUserStore } from '~/stores/user'
import { saveAs } from 'file-saver'
const props = defineProps<{ id: string }>()
const videoId = ref(props.id)
const router = useRouter()
const route = useRoute()
const toast = useToast()
const { getCookie, setCookie, removeCookie } = useCookie()
const scans = ref({video_key_post:null})
const stateuser = getCookie('userState')
const dt = ref()
const first = ref(null)
const initSortField = ref('time')
const enableSort = ref(true)
const buttonNextPage = ref()
const buttonPrevPage = ref()
const loading = ref()
const currentPage = ref(1)
const currentSlide = ref(0)
const selectedRow = ref()
const activeIndex = ref(0)
const fullScreen = ref(false)
const fullScreenIcon = ref('pi pi-eye')
const startGallery = ref()
const images = ref()
const responsiveOptions = ref([
    {
        breakpoint: '1024px',
        numVisible: 5
    },
    {
        breakpoint: '768px',
        numVisible: 3
    },
    {
        breakpoint: '560px',
        numVisible: 1
    }
])
const switchPages = []
for (let i = 8; i < 500; i += 8) switchPages.push(i)
//console.log('array',switchPages) 
const state = reactive({
  project_id: null,
  video_id: parseInt(videoId),
  time: null,
  detection: null,
  photo:null,
  probability: null,
  //datetime: '',
  
})
  onMounted(
    async () => { 
    
	}
  )
const startShow = async () => {
  currentSlide.value = -1
  currentPage.value = 1
  nextSlide(null)
}
const init = async (id) => {
  //console.log('Перешли скан:',dt.value.$props) sorted //videoId.value
  //dt.value.sortSingle() first filters resetPage dt.value.sorted(time)
  loading.value = true
  initSortField.value = null //'time'
	
  try {
    const res = await fetch(`http://84.201.177.58:3000/api/videos/${videoId.value}`,{
	  //method: 'post',
	  headers: {
	  "Content-Type": 'application/json',
	  'Authorization':'Token  '+stateuser.token
	  },
	})
    
    if (res.status >= 400) {
      throw new Error("Bad response from server")
    }
    
    scans.value = await res.json()
    console.log('init',scans.value)
	loading.value = false
	startShow()
	buttonNextPage.value = dt.value.$el.getElementsByClassName('pi-angle-right') //[0].click()
    buttonPrevPage.value = dt.value.$el.getElementsByClassName('pi-angle-left') //[0].click()	
  } catch (err) {
    console.error(err)
  }
}
watchEffect(() => {
	videoId.value = props.id
	init(null)
})
const actionGallery = () => {
  startGallery.value = true
}
const toggleFullScreen = () => {
  console.log('toggle')
  fullScreen.value = !fullScreen.value
}
const savePhoto = async (item) => {
  console.log('загружаем..')
  try {
    const res = await fetch(state.photo)
    if (res.status >= 400) {
      throw new Error("Bad response from server")
    }
    
    let blob = await res.blob()
    saveAs(blob, 'Сканирование ('+state.time+') '+state.detection+'.jpg')
  } catch (err) {
    console.error(err)
  }  
}
 
const showScan = (event, item) => {
  window.scrollTo(0,0)
  console.log('show-scan',currentSlide.value,item)
  if(!selectedRow.value || event == 'event') {
    item = scans.value.video_key_post[currentSlide.value]
    selectedRow.value = item
  }
  state.photo = item.photo //'http://'+photo[1]
  state.detection = item.detection
  state.time = item.time
  state.probability = item.probability
  
  images.value = [
    {
      "itemImageSrc": item.photo,
      "thumbnailImageSrc": item.photo,
      "alt": "Тип повреждения: "+state.detection+ " Вероятность "+state.probability,
      "title": "Таймкод "+state.time
    },    
  ]
}
const movePage = (event) => {
  console.log('page+', button2[0].click())
  console.log('page+',dt.value)
}
const checkPage = (page) => {
  switch(page){
    case 'prev':
	if(switchPages.includes(currentSlide.value + 1)) {
	  buttonPrevPage.value[0].click()
	}
	break;
    case 'next':
	if(switchPages.includes(currentSlide.value)) {
	  buttonNextPage.value[0].click()
	}
	break;	
  }
}
const prevPage = () => {
  currentPage.value --
  if(currentPage.value < 1) currentPage.value = 1
}
const nextPage = () => {
  currentPage.value ++
}
const prevSlide = (event) => {
  currentSlide.value --
  if(currentSlide.value < 0) currentSlide.value = 0  
  showScan('event', null)
  checkPage('prev')
}
const nextSlide = (event) => {
  currentSlide.value ++
  if(currentSlide.value >= scans.value.video_key_post.length) currentSlide.value = 0
  showScan('event', null) 
  checkPage('next')
}
const onRowSelect = (event) => {
  console.log('on-row-select',currentPage.value,event)
  currentSlide.value = ((currentPage.value - 1) * 8) + event.index
  showScan('select',event.data)
}
const startSorting = (data) => {
  //console.log('data-start-sort',dt.value.$data)
}
const onPage = (event) => {
  console.log('on-page',event)
  currentPage.value = event.page + 1
}
const onSort = (event) => {
  console.log('on-sort!',event)
  let k = 1
  initSortField.value = event.sortField
  if(event.sortOrder > 0) scans.value.video_key_post.sort((a, b) => a[event.sortField] - b[event.sortField])
  if(event.sortOrder < 0) scans.value.video_key_post.sort((a, b) => b[event.sortField] - a[event.sortField]) 
  currentSlide.value = 0
  currentPage.value = 0
  startShow()  
}
</script>
<style lang="scss" scoped>
::v-deep(.custom-galleria) {
    &.fullscreen {
        display: flex;
        flex-direction: column;
        .p-galleria-content {
            flex-grow: 1;
            justify-content: center;
        }
    }
    .p-galleria-content {
        position: relative;
    }
    .p-galleria-thumbnail-wrapper {
        position: absolute;
        bottom: 0;
        left: 0;
        width: 100%;
    }
    .p-galleria-thumbnail-items-container {
        width: 100%;
    }
    .custom-galleria-footer {
        display: flex;
        align-items: center;
        background-color: rgba(0, 0, 0, .9);
        color: #ffffff;
        > button {
            background-color: transparent;
            color: #ffffff;
            border: 0 none;
            border-radius: 0;
            margin: .2rem 0;
            &.fullscreen-button {
                margin-left: auto;
            }
            &:hover {
                background-color: rgba(255, 255, 255, 0.1);
            }
        }
    }
    .title-container {
        > span {
            font-size: .9rem;
            padding-left: .829rem;
            &.title {
                font-weight: bold;
            }
        }
    }
}
</style>
