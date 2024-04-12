<template>
    <v-app-bar class="pl-4">
        <RouterLink to="/" class="text-h3 text-decoration-none text-white">Boxifyy</RouterLink>
        <div class="w-25 ml-3">
            <v-text-field v-model="text" variant="underlined" placeholder="Add notes"></v-text-field>
        </div>
        <v-btn @click="addNewItem" variant="outlined" class="ml-3">Add</v-btn>
    </v-app-bar>
    <v-navigation-drawer>
      <li v-for="i in allBox" :key="i.boxTitle" @click="$router.push('/box/'+i.id)" class="ma-2 cursor-pointer">{{ i.boxTitle }}</li>
    </v-navigation-drawer>
    <v-main v-for="box in allBox.filter(f => {return f.id == $route.params.id})" :key="box.id">
        <p class="text-h4 ma-3 ml-5">
            {{ box.boxTitle }}
        </p>
        <div class="d-sm-flex justify-space-between">
            <v-btn v-if="isSorting == false && box.boxType != 2" @click="changeState" class="ml-5 ma-1">Sort alphabetically</v-btn>
            <v-btn v-else-if="isSorting == true && box.boxType != 2" @click="changeState" class="ml-5 ma-1">Sort by date</v-btn>
            <v-btn class="ma-1 ml-5 mr-5" v-if="box.boxType == 1" @click="deleteAllCharacterize">Delete all characterize</v-btn>
            <v-btn class="ma-1 ml-5 mr-5" v-if="box.boxType == 2" @click="saveBoardChanges">Save changes</v-btn>
        </div>
        <v-container v-if="box.boxType == 0">
            <v-row>
                <v-col v-for="i in allItems.filter(e => {return e.boxId == $route.params.id})" :key="i.id" @click="deleteItemFromBox(i)">
                    <v-card class="border-sm ma-2 pa-5 rounded-lg elevation-6 bg-white text-center cursor-pointer">{{ i.itemName }}</v-card>
                </v-col>
            </v-row>
        </v-container>
        <v-container v-if="box.boxType == 1">
            <v-row>
                <v-col v-for="i in allItems.filter(e => {return e.boxId == $route.params.id})" :key="i.id" @click="characterizeNote(i)">
                    <v-card v-if="i.isCharacterize == 0" class="border-sm ma-2 pa-5 rounded-lg elevation-6 bg-white text-center cursor-pointer">{{ i.itemName }}</v-card>
                    <v-card v-else class="border-sm ma-2 pa-5 rounded-lg elevation-6 bg-white text-center cursor-pointer text-decoration-line-through text-pink-darken-2">{{ i.itemName }}</v-card>
                </v-col>
            </v-row>
        </v-container>
        <v-container v-if="box.boxType == 2" class="h-100">
            <div class="d-flex justify-space-between h-100">
                <div class="w-25 h-100">
                    <p class="text-h4">First</p>
                    <draggable v-model="first_column" tag="ul" group="kolumny" class="h-100">
                        <template #item="{element: oneItem}">
                            <li style="list-style: none;">
                                <v-card class="border-sm ma-2 pa-sm-8 pa-2 rounded-lg elevation-6 bg-white text-center cursor-pointer" @dblclick="deleteItemFromBox(oneItem)"> {{ oneItem.itemName }}</v-card>
                            </li>
                        </template>
                    </draggable>
                </div>
                <div class="w-25 h-100">
                    <p class="text-h4">Second</p>
                    <draggable v-model="second_column" tag="ul" group="kolumny" class="h-100">
                        <template #item="{ element: oneItem}">
                            <li style="list-style: none;">
                                <v-card class="border-sm ma-2 pa-sm-8 pa-2 rounded-lg elevation-6 bg-white text-center cursor-pointer" @dblclick="deleteItemFromBox(oneItem)"> {{ oneItem.itemName }}</v-card>
                            </li>
                        </template>
                    </draggable>
                </div>
                <div class="w-25 h-100">
                    <p class="text-h4">Third</p>
                    <draggable v-model="third_column" tag="ul" group="kolumny" class="h-100">
                        <template #item="{ element: oneItem}">
                            <li style="list-style: none;">
                                <v-card class="border-sm ma-2 pa-sm-8 pa-2 rounded-lg elevation-6 bg-white text-center cursor-pointer" @dblclick="deleteItemFromBox(oneItem)"> {{ oneItem.itemName }}</v-card>
                            </li>
                        </template>
                    </draggable>
                </div>
            </div>
        </v-container>
    </v-main>

</template>

<script setup>
    import { ref, onMounted } from 'vue'
    import { createDirectus, createItem, readItems, deleteItem, rest, updateItem, deleteItems } from '@directus/sdk'
    import draggable from 'vuedraggable'
    const client = createDirectus('https://rndii-193-160-11-29.a.free.pinggy.link/').with(rest());

    const text = ref("")
    const allBox = ref([])
    const allItems = ref([])
    let uri = window.location.href.split('/')
    const isSorting = ref(false)
    

    const first_column = ref([])
    const second_column = ref([])
    const third_column = ref([])


    async function addNewItem() {
        if (text.value != "") {
            await client.request(createItem('item', {itemName: text.value, boxId: uri[4], isCharacterize: 0, col: 1}))
            text.value = ''
            allItems.value = await client.request(readItems('item'))
        }

        loadOnBoardType()
    }

    async function deleteItemFromBox(item) {
        const confirmText = "Are you sure you want to delete?"
        if (confirm(confirmText) == true) {
            await client.request(deleteItem('item', item.id))
            allItems.value = await client.request(readItems('item'))
        }

        loadOnBoardType()
    }

    async function changeState() {
        isSorting.value = !isSorting.value
        
        if (isSorting.value) {
            allItems.value = await client.request(readItems('item', {
                sort: ['itemName']
            }))
        } else {
            allItems.value = await client.request(readItems('item'))
        }
    }

    async function characterizeNote(note) {
        if (note.isCharacterize == 0) {
            await client.request(updateItem('item', note.id, {boxId: note.boxId, itemName: note.itemName, isCharacterize: 1}))
            allItems.value = await client.request(readItems('item'))
        } else {
            await client.request(updateItem('item', note.id, {boxId: note.boxId, itemName: note.itemName, isCharacterize: 0}))
            allItems.value = await client.request(readItems('item'))
        }
    }

    async function deleteAllCharacterize() {
        const confirtmText = "Are you sure you want to delete?"
        if (confirm(confirtmText)) {
            await client.request(deleteItems('item', {
                filter: {
                    isCharacterize: {
                        _eq: 1
                    }
                }
            }))
            allItems.value = await client.request(readItems('item'))
        }
    }
    

    async function saveBoardChanges() { 
        allItems.value.forEach(element => {
            for (let i = 0; i < first_column.value.length; i++) {
                if (element == first_column.value.at(i)){
                    client.request(updateItem('item', element.id, {
                        col: 1
                    }))
                }
            }
            for (let i = 0; i < second_column.value.length; i++) {
                if (element == second_column.value.at(i)){
                    client.request(updateItem('item', element.id, {
                        col: 2
                    }))
                }
            }
            for (let i = 0; i < third_column.value.length; i++) {
                if (element == third_column.value.at(i)){
                    client.request(updateItem('item', element.id, {
                        col: 3
                    }))
                }
            }
        });
    }

    function loadOnBoardType() {
        first_column.value = []
        second_column.value = []
        third_column.value = []
        allItems.value.forEach(e => {
            if (e.boxId == uri[4]) {
                if (e.col == 1) {
                    first_column.value.push(e)
                } else if (e.col == 2) {
                    second_column.value.push(e)
                } else if (e.col == 3) {
                    third_column.value.push(e)
                }
            }
        });
    }

    onMounted(async () => {
        allBox.value = await client.request(readItems('box'))
        allItems.value = await client.request(readItems('item'))

        loadOnBoardType()
    })

</script>

