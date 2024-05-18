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
    unit_name: '',
    unit_stock: 0,
    unit_average_in_value: 0,
    unit_average_out_value: 0,
    unit_out: 0,
    last_unit_out_in_period: '',
    unit_in: 0,
    last_unit_in_in_period: ''
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
            unit_name: '',
            unit_stock: 0,
            unit_average_in_value: 0,
            unit_average_out_value: 0,
            unit_out: 0,
            last_unit_out_in_period: '',
            unit_in: 0,
            last_unit_in_in_period: ''
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
const stockMovementUnitIndex:any = ref();

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
    Stock Count Constants
*/

const updateStockCount:any = (
    stockObject: any,
) => {
    let stockMovementTable = db.value.transaction("stockMovement").objectStore("stockMovement");
    let stockMovementUnitIndex = stockMovementTable.index('movementUnitIndex');

    let stockMovementUnitIndexRequest = stockMovementUnitIndex.getAll(stockObject.id);

    stockMovementUnitIndexRequest.onsuccess = () => {
        if (stockMovementUnitIndexRequest.result !== undefined) {
            let targetStockMovementUnits: any = stockMovementUnitIndexRequest.result;

            let stockMovementIns: any = targetStockMovementUnits.filter((targetStockMovementUnit:any) => targetStockMovementUnit.movement_type == 'in');
            let stockMovementInQuantity = stockMovementIns.map((stockMovementIn:any) => stockMovementIn.movement_volume);
            let stockMovementInValue = stockMovementIns.map((stockMovementIn:any) => stockMovementIn.movement_value);

            let stockMovementOuts: any = targetStockMovementUnits.filter((targetStockMovementUnit:any) => targetStockMovementUnit.movement_type == 'out');
            let stockMovementOutQuantity = stockMovementOuts.map((stockMovementOut:any) => stockMovementOut.movement_volume);
            let stockMovementOutValue = stockMovementIns.map((stockMovementIn:any) => stockMovementIn.movement_value);
            
            
            stockObject.unit_in = stockMovementInQuantity.reduce((partialSum:number, a:number) => partialSum + a, 0);
            stockObject.unit_out = stockMovementOutQuantity.reduce((partialSum:number, a:number) => partialSum + a, 0);
            stockObject.unit_stock = stockObject.unit_in - stockObject.unit_out;
            stockObject.unit_average_in_value = stockMovementInValue.reduce((partialSum:number, a:number) => partialSum + a, 0) / stockObject.unit_in;
            stockObject.unit_average_out_value = stockMovementOutValue.reduce((partialSum:number, a:number) => partialSum + a, 0) / stockObject.unit_in;

            let {stockObjectId, ...stockObjectFields} = stockObject
            let stockObjectTable = db.value.transaction("stockObject", "readwrite").objectStore("stockObject");
            let stockObjectTableRequest = stockObjectTable.put(stockObjectFields, stockObjectId);

            stockObjectTableRequest.onsuccess = () => {
                console.log("Updated stock object");
            };
        } else {
            console.log("No such books");
        }
    };
}

const updateStockCounts: any = () => {
    for (let i=0; i < stockObjects.value.length; i++) {
        updateStockCount(stockObjects.value[i]);
    };
    console.log('hello world')

    getAllStockObjects(); 
}

/*
    Charts
*/
// const getDaysInMonth = (month:number, year:number) => {
//     const date = new Date(year, month, 1);
//     const days = [];
//     while (date.getMonth() === month) {
//         days.push(new Date(date));
//         date.setDate(date.getDate() + 1);
//     }
//     return days;
// }
// const currentMonth = new Date().getMonth();
// const currentYear = new Date().getFullYear();
// const allDatesInCurrentMonth = getDaysInMonth(currentMonth, currentYear).map(date => date.toISOString().split('T')[0]);

const currentLabels: any = ref([])
const currentDataset: any = ref([])

const getAggregateDataOnDate = (from:string, to:string) => {
    let stockMovementTable = db.value.transaction("stockMovement").objectStore("stockMovement");
    let stockMovementDateIndex = stockMovementTable.index('movementDateIndex');
    let stockMovementDateIndexRequest = stockMovementDateIndex.getAll(IDBKeyRange.bound(from, to));


    let currentDate = from;
    let stockMovementDateTransactions:any = [];
    let stockMovementDates:any = [];

    stockMovementDateIndexRequest.onsuccess = () => {
        let targetStockMovementUnits: any = stockMovementDateIndexRequest.result;
        let stockMovementOuts: any = targetStockMovementUnits.filter((targetStockMovementUnit:any) => targetStockMovementUnit.movement_type == 'out');
        
        stockMovementDateTransactions.push(stockMovementOuts[0].movement_value);
        stockMovementDates.push(stockMovementOuts[0].movement_date);
        for (let i=1; i < stockMovementOuts.length; i++){
            if (stockMovementOuts[i].movement_date == currentDate) {
                stockMovementDateTransactions[stockMovementDateTransactions.length-1] += stockMovementOuts[i].movement_value
            } else {
                stockMovementDateTransactions.push(stockMovementOuts[i].movement_value);
                currentDate = stockMovementOuts[i].movement_date;
                stockMovementDates.push(stockMovementOuts[i].movement_date);
            }
        }
    }

    currentLabels.value = stockMovementDates;
    currentDataset.value = stockMovementDateTransactions;
}



/*
    Application Start
*/
onMounted(() => {
    let openRequest = window.indexedDB.open("MikroStok", 1);

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
            let stockMovementTable:any = db.value.createObjectStore(
                'stockMovement', 
                {
                    keyPath: 'id',
                    autoIncrement: true
                }
            )
            stockMovementTable.createIndex('movementUnitIndex', 'movement_unit');
            stockMovementTable.createIndex('movementDateIndex', 'movement_date');
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
    <main class="p-4 md:p-16 grid grid-cols-5 gap-8">
        <aside class="col-span-5 md:col-span-1 md:row-span-2">
            <div>
                <h1 class="text-2xl font-bold">MikroStok</h1>
                <p class="text-md">Solusi manajemen stok sederhana untuk bisnis mikro dan kecil</p>
                <div class="flex flex-col my-2">
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
                        Stok
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
            v-if="openTab == 'dashboard'"
            class="col-span-5 md:col-span-4 flex flex-col space-y-4 border-l p-4 bg-gray-50 rounded-md shadow-md"
        >
            <h2 class="text-xl font-bold">Penjualan Harian</h2>
            <div class="flex flex-row">
                <button 
                    @click="getAggregateDataOnDate('2024-05-12', '2024-05-21')"
                    class="px-2 py-1 bg-gray-700 text-sm text-white w-1/4 font-semibold"
                >
                    Update Grafik
                </button>
            </div>
            <div  class="h-[40vh]">
                <TimeSeriesChart  :labels="currentLabels" :dataset="currentDataset" />
            </div>
        </section>
        <section
            v-if="openTab == 'dashboard'"
            class="col-span-5 md:col-span-4 flex flex-col space-y-4 border-l p-4 bg-gray-50 rounded-md shadow-md"
        >
            <h2 class="text-xl font-bold">Nilai Stok</h2>
        </section>
        <section
            v-if="openTab == 'object'"
            class="col-span-5 md:col-span-4 flex flex-col space-y-4 border-l p-4 bg-gray-50 rounded-md shadow-md"
        >
            <h2 class="text-xl font-bold">Tambah Stok</h2>
            <div class="shadow-sm flex flex-col">
                <span class="text-md">Nama Supplier:</span>
                <input 
                    v-model="stockObjectInput.supplier_name"
                    type="text" 
                    placeholder="Nama supplier"
                    class="border-b-2 bg-gray-100 mb-3 px-2 py-1 rounded-md"
                />
                <span class="text-md">Nama Unit:</span>
                <input 
                    v-model="stockObjectInput.unit_name"
                    type="text" 
                    placeholder="Nama unit" 
                    class="border-b-2 bg-gray-100 mb-3 px-2 py-1 rounded-md"
                />
                <div class="flex flex-row">
                    <button 
                        @click="createStockObject()"
                        class="px-2 py-1 bg-green-700 text-sm text-white rounded-l-lg w-3/4 font-semibold"
                    >
                        Tambah Sebagai Stok
                    </button>
                    <button 
                        @click="updateStockCounts()"
                        class="px-2 py-1 bg-gray-700 text-sm text-white rounded-r-lg w-1/4 font-semibold"
                    >
                        Update Stok
                    </button>
                </div>
                
            </div>
        </section>
        <section
            v-if="openTab == 'object' && stockObjects.length > 0"
            class="col-span-5 md:col-span-4 flex flex-col space-y-4 rounded-md"
        >
            <div class="w-full flex flex-row text-sm p-2 border-b">
                <p class="w-5/12 text-xs">
                    Nama Barang
                </p>
                <p class="w-2/12 text-xs">
                    Harga Jual
                </p>
                <p class="w-2/12 text-xs">
                    Harga Beli
                </p>
                <p class="w-3/12 text-xs">
                    Jumlah Stok
                </p>
                <p class="w-1/12 text-xs">
                    Aksi
                </p>
            </div>
            <div 
                v-for="stockObject, idx in stockObjects" 
                :key="`${stockObject.id}_${idx}`" 
                class="w-full flex flex-row text-sm p-2 border-b"
            >
                <p class="w-5/12">
                    <span class="font-medium">{{ stockObject.supplier_name }}</span> <span>{{ stockObject.unit_name }}</span>
                </p>
                <p class="w-2/12">
                    Rp. {{ String(stockObject.unit_average_out_value).replace(/(.)(?=(\d{3})+$)/g,'$1,') }}
                </p>
                <p class="w-2/12">
                    Rp. {{ String(stockObject.unit_average_in_value).replace(/(.)(?=(\d{3})+$)/g,'$1,') }}
                </p>
                <p class="w-3/12 font-bold">
                    {{ stockObject.unit_stock }}
                </p>
                <button 
                    class="w-1/12 text-xs text-red-600"
                    @click="deleteStockObject(stockObject.id)"
                >
                    Hapus
                </button>
            </div>
        </section>
        <section 
            v-if="openTab == 'movement'"
            class="col-span-5 md:col-span-4 flex flex-col space-y-4 border-l p-4 bg-gray-50 rounded-md shadow-md"
        >
            <h2 class="text-xl font-bold">Tambah Pergerakan Stok</h2>
            <div class="flex flex-col">
                <span class="text-md">Pilih Unit yang Bergerak:</span>
                <div class="border-b-2 bg-gray-100 mb-3 px-2 py-1 rounded-md">
                    <p 
                        class="w-full py-1"
                        :class="stockMovementInput.movement_unit ? 'text-black' : 'text-gray-400'"
                    >
                        {{ stockMovementInput.movement_unit ? `terpilih: ${stockMovementInput.movement_unit}` : 'Unit yang dipilih:' }}
                    </p>
                    <input 
                        v-model="stockObjectsQuickSearchQuery"
                        type="text" 
                        placeholder="..."
                        class="w-full px-4 py-1 bg-gray-100 border-2 rounded-lg"
                    />
                    <button
                        v-for="searchResult, idx in stockObjectsQuickSearchResult"
                        :key="`${searchResult.item.id}_${searchResult.refIndex}_${idx}`"
                        @click="
                            stockMovementInput.movement_unit = searchResult.item.id;
                            stockObjectsQuickSearchQuery = '';
                        "
                        class="w-full text-left border-b-2 border-gray-400 py-1 px-4"
                    >
                        {{  searchResult.item.id }}
                    </button>
                </div>
                <span class="text-md">Pilih Tipe Pergerakan Stok:</span>
                <select 
                    v-model="stockMovementInput.movement_type" 
                    placeholder="Tipe pergerakan stok" 
                    class="border-b-2 bg-gray-100 mb-3 px-2 py-1 rounded-md"
                    :class="stockMovementInput.movement_type ? 'text-black' : 'text-gray-400'"
                >
                    <option value="''" class="text-gray-400" disabled selected>Pilih Tipe Pergerakan</option>
                    <option value="in" class="text-black">Barang Masuk</option>
                    <option value="out" class="text-black">Barang Keluar</option>
                </select>
                <span class="text-md">Besar (Kuantitas) Pergerakan Stok:</span>
                <input 
                    v-model="stockMovementInput.movement_volume"
                    type="number" 
                    placeholder="Besar pergerakan stok" 
                    class="border-b-2 bg-gray-100 mb-3 px-2 py-1 rounded-md"
                />
                <span class="text-md">Nlai (Rupiah) Pergerakan Stok:</span>
                <input 
                    v-model="stockMovementInput.movement_value"
                    type="number" 
                    placeholder="Nilai pergerakan stok" 
                    class="border-b-2 bg-gray-100 mb-3 px-2 py-1 rounded-md"
                />
                <span class="text-md">Nilai (Rupiah) Pergerakan Stok:</span>
                <input 
                    v-model="stockMovementInput.movement_date"
                    type="date" 
                    placeholder="Tanggal pergerakan stok" 
                    class="border-b-2 bg-gray-100 mb-3 px-2 py-1 rounded-md"
                />
                <div class="flex flex-row">
                    <button 
                        @click="createStockMovement()"
                        class="px-2 py-1 bg-green-700 text-sm text-white rounded-lg w-full font-semibold"
                    >
                        Tambah Pergerakan Stok
                    </button>
                </div>
            </div>
        </section>
        <section
            v-if="openTab == 'movement' && stockMovements.length > 0"
            class="col-span-5 md:col-span-4 flex flex-col space-y-4 rounded-md"
        >
            <div 
                v-for="stockMovement, idx in stockMovements" 
                :key="`${stockMovement.id}_${idx}`" 
                class="w-full flex flex-row text-sm border-b py-1"
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
                    <p class="text-lg">{{ stockMovement.movement_volume }}x {{ stockMovement.movement_unit }}</p>
                    <p class="text-md">Rp. {{ String(stockMovement.movement_value).replace(/(.)(?=(\d{3})+$)/g,'$1,') }} (Rp. {{ String(stockMovement.movement_value / stockMovement.movement_volume).replace(/(.)(?=(\d{3})+$)/g,'$1,') }}/pcs)</p>
                    <p class="font-thin"></p>
                </div>
                <button 
                    class="w-1/5 text-xs text-red-600"
                    @click="deleteStockMovement(stockMovement.id)"
                >
                    Hapus
                </button>
            </div>
        </section>
    </main>
</template>
