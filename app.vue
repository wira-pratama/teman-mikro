<script setup lang="ts">
import Fuse from 'fuse.js'
const db:any = ref();
const openTab:any = ref('object');


/*
    Stock Objects Constants
*/
const stockObjects:any = ref([]);
const stockObjectInput:any = ref({
    supplier_name: '',
    unit_name: ''
});

const stockObjectsFuse:any = ref();
const stockObjectsQuickSearchQuery: any = ref('');
const stockObjectsQuickSearchResult: any = ref([]);

const createStockObjectsFuse:any = () => {
    if (stockObjects.value.length > 0) {
        stockObjectsFuse.value = new Fuse(
            stockObjects.value,
            {
                includeScore: true,
                threshold: 0.2,
                keys: ['id', 'supplier_name', 'unit_name']
            }
        );
    }
    
}

const getStockObjectsQuickSearchResult: any = () => {
    stockObjectsQuickSearchResult.value = stockObjectsFuse.value.search(stockObjectsQuickSearchQuery.value);
}

watch(stockObjectsQuickSearchQuery, (newValue, oldValue) => {
    getStockObjectsQuickSearchResult();
});

const getAllStockObjects: any = () => {
    let stockObjectTable = db.value.transaction("stockObject", "readwrite").objectStore("stockObject");
    let stockObjectTableRequest = stockObjectTable.getAll();

    stockObjectTableRequest.onsuccess = () => {
        if (stockObjectTableRequest.result !== undefined) {
            stockObjects.value = stockObjectTableRequest.result;
            createStockObjectsFuse();
        } else {
            /*
                TODO: handle error and show popup
            */
            console.log("error in stock object");
        }
    };
}

const createStockObject: any = () => {
    let stockObjectTable = db.value.transaction("stockObject", "readwrite").objectStore("stockObject");
    let stockObjectTableRequest = stockObjectTable.add(
        {
            id: `${stockObjectInput.value.supplier_name} ${stockObjectInput.value.unit_name}`,
            ...stockObjectInput.value
        }
    );

    stockObjectTableRequest.onsuccess = () => {
        stockObjectInput.value = {
            supplier_name: '',
            unit_name: ''
        };
    };
    stockObjectTableRequest.onerror = () => {
        console.log("Error", stockObjectTableRequest.error);
    };

    getAllStockObjects();
}


const deleteStockObject: any = (id: string) => {
    let stockObjectTable = db.value.transaction("stockObject", "readwrite").objectStore("stockObject");
    let stockObjectTableRequest = stockObjectTable.delete(id);

    stockObjectTableRequest.onsuccess = () => {
        console.log("Deleted stock object", stockObjectTableRequest.result);
    };
    stockObjectTableRequest.onerror = () => {
        console.log("Error", stockObjectTableRequest.error);
    };

    getAllStockObjects();
}


/*
    Stock Movements Constants
*/
const stockMovements:any = ref([]);
const stockMovementInput:any = ref({
    movement_unit: '',
    movement_type: '',
    movement_value: '',
    movement_volume: '',
    movement_date:  '',
});

const stockMovementsFuse:any = ref();
const stockMovementsQuickSearchQuery: any = ref('');
const stockMovementsQuickSearchResult: any = ref([]);

const createStockMovementsFuse:any = () => {
    if (stockMovements.value.length > 0) {
        stockMovementsFuse.value = new Fuse(
            stockMovements.value,
            {
                includeScore: true,
                threshold: 0.2,
                keys: ['movement_unit', 'movement_date']
            }
        );
    }
    
}

const getQuickSearchStockMovementsResults: any = () => {
    stockMovementsQuickSearchResult.value = stockMovementsFuse.value.search(stockMovementsQuickSearchQuery.value);
}

watch(stockMovementsQuickSearchQuery, (newValue, oldValue) => {
    getQuickSearchStockMovementsResults();
});

const getAllStockMovements: any = () => {
    let stockMovementTable = db.value.transaction("stockMovement", "readwrite").objectStore("stockMovement");
    let stockMovementTableRequest = stockMovementTable.getAll();

    stockMovementTableRequest.onsuccess = () => {
        if (stockMovementTableRequest.result !== undefined) {
            stockMovements.value = stockMovementTableRequest.result;
            createStockMovementsFuse();
        } else {
            /*
                TODO: handle error and show popup
            */
            console.log("error in stock movement");
        }
    };
}

const createStockMovement: any = () => {
    let stockMovementTable = db.value.transaction("stockMovement", "readwrite").objectStore("stockMovement");
    let stockMovementTableRequest = stockMovementTable.add({
        ...stockMovementInput.value
    });

    stockMovementTableRequest.onsuccess = () => {
        console.log("Book added to the store", stockMovementTableRequest.result);
        stockMovementInput.value = {
            movement_unit: '',
            movement_type: '',
            movement_value: '',
            movement_volume: '',
            movement_date:  '',
        };
    };
    stockMovementTableRequest.onerror = () => {
        console.log("Error", stockMovementTableRequest.error);
    };

    getAllStockMovements();
}

const deleteStockMovement: any = (id: string) => {
    let stockMovementTable = db.value.transaction("stockMovement", "readwrite").objectStore("stockMovement");
    let stockMovementTableRequest = stockMovementTable.delete(id);

    stockMovementTableRequest.onsuccess = () => {
        console.log("Deleted stock object", stockMovementTableRequest.result);
    };
    stockMovementTableRequest.onerror = () => {
        console.log("Error", stockMovementTableRequest.error);
    };

    getAllStockMovements();
}


/*
    Application Start
*/
onMounted(() => {
    let openRequest = window.indexedDB.open("TemanMikro", 1);

    /*
        Initialize database
    */
    openRequest.onupgradeneeded = () => {
        db.value = openRequest.result;

        /*
            Declare stockObject comprising:
            1. Supplier Name
            2. Unit Name
        */
        if (!db.value.objectStoreNames.contains('stockObject')) {
            db.value.createObjectStore(
                'stockObject', 
                {
                    keyPath: 'id',
                }
            );
        }

        /*
            Declare stockMovement comprising:
            1. stockObjectId
            2. movementType [in, out]
            3. movementValue (total value)
            4. movementVolume (in quantity)
            5. movementDate (in Date object)
        */
        if (!db.value.objectStoreNames.contains('stockMovement')) {
            db.value.createObjectStore(
                'stockMovement', 
                {
                    keyPath: 'id',
                    autoIncrement: true
                }
            );
        }
    };

    /*
        TODO: handle error and show popup
    */
    openRequest.onerror = () => {
        console.error("Error", openRequest.error);
    };

    /*
        Fetch data from stockObject and stockMovement
        TODO: handle success and show popup
    */
    openRequest.onsuccess = () => {
        db.value = openRequest.result;

        getAllStockObjects();
        getAllStockMovements();
    };
});
</script>

<template>
    <Head>
        <Title>Nuxt: The Intuitive Web Framework</Title>
        <Meta
            name="description"
            content="Build your next Vue.js application with confidence using Nuxt...."
        />
    </Head>
    <main class="p-16 grid grid-cols-5 gap-8">
        <aside class="col-span-1 row-span-2">
            <div>
                <h1 class="text-2xl font-bold">TemanMikro</h1>
                <div class="flex flex-col">
                    <button 
                        @click="openTab = 'dashboard'"
                        class="border-b text-left py-1"
                    >
                        Dashboard
                    </button>
                    <button 
                        @click="openTab = 'object'"
                        class="border-b text-left py-1"
                    >
                        Supplier Stok
                    </button>
                    <button 
                        @click="openTab = 'movement'"
                        class="border-b text-left py-1"
                    >
                        Pergerakan Stok
                    </button>
                </div>
            </div>
        </aside>
        <section
            v-if="openTab == 'object'"
            class="col-span-4 flex flex-col space-y-4 border-l p-4 bg-gray-50 rounded-md"
        >
            <h2 class="text-xl">Tambah Stok</h2>
            <div class="shadow-sm flex flex-col">
                <input 
                    v-model="stockObjectInput.supplier_name"
                    type="text" 
                    placeholder="Nama supplier"
                    class="border-b-2 px-2 py-1"
                />
                <input 
                    v-model="stockObjectInput.unit_name"
                    type="text" 
                    placeholder="Nama unit" 
                    class="border-b-2 px-2 py-1"
                />
                <button 
                    @click="createStockObject()"
                    class="px-2 py-1 bg-green-700 text-white"
                >
                    Tambah Sebagai Stok
                </button>
            </div>
            <div class="flex flex-col">
                <div 
                    v-for="stockObject, idx in stockObjects" 
                    :key="`${stockObject.id}_${idx}`" 
                    class="w-full flex flex-row text-sm p-2 border-b"
                >
                    <p class="w-4/5"><span class="font-medium">{{ stockObject.supplier_name }}</span> <span>{{ stockObject.unit_name }}</span></p>
                    <button 
                        class="w-1/5 text-xs text-red-600"
                        @click="deleteStockObject(stockObject.id)"
                    >
                        Hapus
                    </button>
                </div>
            </div>
        </section>
        <section 
            v-if="openTab == 'movement'"
            class="col-span-4 flex flex-col space-y-4 border-l p-4 bg-gray-50 rounded-md"
        >
            <h2 class="text-xl">Tambah Pergerakan Stok</h2>
            <div class="shadow-sm flex flex-col">
                
                <div class="border-b-2 py-4">
                    <p 
                        class="w-full py-1"
                        :class="stockMovementInput.movement_unit ? 'text-black' : 'text-gray-400'"
                    >
                        {{ stockMovementInput.movement_unit ? stockMovementInput.movement_unit : 'Cari Unit yang Bergerak' }}
                    </p>
                    <input 
                        v-model="stockObjectsQuickSearchQuery"
                        type="text" 
                        placeholder="..."
                        class="w-full border ml-4 pl-4 py-1"
                    />
                    <button
                        v-for="searchResult, idx in stockObjectsQuickSearchResult"
                        :key="`${searchResult.item.id}_${searchResult.refIndex}_${idx}`"
                        @click="
                            stockMovementInput.movement_unit = searchResult.item.id;
                            stockObjectsQuickSearchQuery = '';
                        "
                        class="ml-4 pl-4 w-full text-left border-b-2 border-blue-100 py-2"
                    >
                        {{  searchResult.item.id }}
                    </button>
                </div>
                <select 
                    v-model="stockMovementInput.movement_type" 
                    placeholder="Tipe pergerakan stok" 
                    class="border-b-2 px-1 py-1"
                    :class="stockMovementInput.movement_type ? 'text-black' : 'text-gray-400'"
                >
                    <option value="''" class="text-gray-400" disabled selected>Pilih Tipe Pergerakan</option>
                    <option value="in" class="text-black">Barang Masuk</option>
                    <option value="out" class="text-black">Barang Keluar</option>
                </select>
                <input 
                    v-model="stockMovementInput.movement_value"
                    type="number" 
                    placeholder="Nilai pergerakan stok" 
                    class="border-b-2 px-2 py-1"
                />
                <input 
                    v-model="stockMovementInput.movement_volume"
                    type="number" 
                    placeholder="Besar pergerakan stok" 
                    class="border-b-2 px-2 py-1"
                />
                <input 
                    v-model="stockMovementInput.movement_date"
                    type="date" 
                    placeholder="Tanggal pergerakan stok" 
                    class="border-b-2 px-2 py-1"
                />
                <button 
                    @click="createStockMovement()"
                    class="px-2 py-1 bg-green-700 text-white"
                >
                    Tambah Pergerakan Stok
                </button>
            </div>
            <div class="flex flex-col">
                <div 
                    v-for="stockMovement, idx in stockMovements" 
                    :key="`${stockMovement.id}_${idx}`" 
                    class="w-full flex flex-row text-sm p-2 border-b"
                >
                    <div class="w-4/5">
                        <p class="text-xs">
                            <span 
                                class="border pl-1 pr-2 rounded-r-lg"
                                :class="stockMovement.movement_type == 'in' ? 'text-white bg-indigo-700 border-indigo-700' : 'text-green-900 border-green-900'"
                            >
                                {{ stockMovement.movement_type == 'in' ? 'Barang Masuk' : 'Barang Keluar' }}
                            </span>
                            {{ stockMovement.movement_date }}
                        </p>
                        <p class="py-1 text-lg">{{ stockMovement.movement_unit }}</p>
                        <p class="text-md">Rp. {{ String(stockMovement.movement_value).replace(/(.)(?=(\d{3})+$)/g,'$1,') }} | {{ stockMovement.movement_volume }} pcs</p>
                    </div>
                    <button 
                        class="w-1/5 text-xs text-red-600"
                        @click="deleteStockMovement(stockMovement.id)"
                    >
                        Hapus
                    </button>
                </div>
            </div>
        </section>
    </main>
    
    
</template>
