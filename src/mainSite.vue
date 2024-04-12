<template>
    <v-app-bar class="pl-4">
      <RouterLink to="/" class="text-h6 text-md-h5 text-lg-h3 text-decoration-none text-white">Boxifyy</RouterLink>
        <div class="w-25 ml-3">
          <v-text-field v-model="text" variant="underlined" placeholder="Add new box"></v-text-field>
        </div>
        <div class="w-25 ml-3">
          <v-select v-model="select" label="Box type" :items="items" variant="underlined" single-line></v-select>
        </div>
        <v-btn @click="addBox" variant="outlined" class="ml-3">Add</v-btn>
    </v-app-bar>
    <v-navigation-drawer>
      <li v-for="i in allBox" :key="i.boxTitle" @click="$router.push('/box/'+i.id)" class="ma-2 cursor-pointer">{{ i.boxTitle }}</li>
    </v-navigation-drawer>
    <v-main class="d-flex justify-center">
      <v-container>
        <v-btn v-if="isSorting == false" @click="changeState">Sort alphabetically</v-btn>
        <v-btn v-else @click="changeState">Sort by date</v-btn>
            <div v-for="item in allBox" :key="item.id" class="border-sm mt-3 pa-8 rounded-lg elevation-6">
              <p to="/box" v-if="item.isEditing == 0" @click="$router.push('/box/'+item.id)" class="text-h5 cursor-pointer">
                <img v-if="item.boxType == 0" style="vertical-align:middle" height="30px" src="./img/box-solid.svg" alt="Standard type box">
                <img v-else-if="item.boxType == 1" style="vertical-align:middle" height="30px" src="./img/list-solid.svg" alt="Standard type box">
                <img v-else-if="item.boxType == 2" style="vertical-align:middle" height="30px" src="./img/flipboard.svg" alt="Standard type box">
                {{ item.boxTitle }}</p>
                <v-text-field v-model="item.boxTitle" class="w-75" placeholder="Type new box name" v-else></v-text-field>
                <div v-for="i in allItems.filter(e => {return e.boxId == item.id})" :key="i.id" class="ma-1 text-grey-darken-2">
                  <p v-if="i.isCharacterize == 1" class="text-decoration-line-through text-pink-darken-2">{{ i.itemName }}</p>
                  <p v-else-if="item.boxType != 2">{{ i.itemName }}</p>
                </div>
                <div v-if="item.boxType == 2" class="ma-1 text-grey-darken-2">
                  {{ inColumnNumber(item) }}
                  <p>First column: {{ columnOne }}</p>
                  <p>Second column: {{ columnTwo }}</p>
                  <p>Third column: {{ columnThree }}</p>
                </div>
              <v-btn rounded="lg" :ripple="false" @click="changeEditState(item)" class="mr-3 mt-3" v-if="item.isEditing == 0">Edit</v-btn>
              <v-btn rounded="lg" :ripple="false" @click="changeEditState(item)" class="mr-3 mt-3" v-else>Confirm</v-btn>
              <v-btn rounded="lg" @click="deleteBox(item)" class="mt-3">Delete</v-btn>
            </div>
      </v-container>
      
    </v-main>
  </template> 
  
  <script setup>
    import {onMounted, ref} from 'vue'
    import { createDirectus, createItem, deleteItem, deleteItems, readItems, rest, updateItem } from '@directus/sdk'
    const client = createDirectus('https://rndii-193-160-11-29.a.free.pinggy.link/').with(rest());
    
    const allBox = ref([])
    const allItems = ref([])
    const isSorting = ref(false)
    const text = ref("")
    const items = ref(['Standard', 'List', 'Board'])
    const select = ref(items.value[0])
    let columnOne = 0
    let columnTwo = 0
    let columnThree = 0

  
    async function addBox() {
      let temp = -1
      switch (select.value) {
        case "Standard":
          temp = 0
          break
        case "List": 
          temp = 1
          break
        case "Board":
          temp = 2
          break
      }
      console.log(temp)
      if (text.value != "") {
        await client.request(createItem('box', {boxTitle: text.value, boxType: temp, isEditing: 0}))
        text.value = ''
        allBox.value = await client.request(readItems('box'))
      }
    }
  
    async function deleteBox(item) {
      await client.request(deleteItem('box', item.id))
      await client.request(deleteItems('item', {
        filter: {
          boxId: {
            _eq: item.id,
          },
        },
      }))
      allBox.value = await client.request(readItems('box'))
    }


    async function changeEditState(item) {
      if (item.isEditing == 1) {
        item.isEditing = 0
        await client.request(updateItem('box', item.id, {boxTitle: item.boxTitle, isEditing: 0}))
      } else {
        item.isEditing = 1
      }
    }

    async function changeState() {
        isSorting.value = !isSorting.value
        
        if (isSorting.value) {
            allBox.value = await client.request(readItems('box', {
                sort: ['boxTitle']
            }))
        } else {
            allBox.value = await client.request(readItems('box'))
        }
    }

    function inColumnNumber(box) {
      columnOne = 0
      columnTwo = 0
      columnThree = 0
      
      allItems.value.forEach(element => {
        if (element.boxId == box.id) {
          if (element.col == 1) {
            columnOne += 1
          } else if (element.col == 2) {
            columnTwo += 1
          } else {
            columnThree += 1
          }
        }
      });
    }


    onMounted(async () => {
      allBox.value = await client.request(readItems('box'));
      allItems.value = await client.request(readItems('item'))
      
    })
  
  </script>