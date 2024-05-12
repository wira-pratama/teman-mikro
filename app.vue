<script setup lang="ts">
const db:any = ref();

const stockObjects:any = ref(null);
const stockMovements:any = ref(null);

const stockObjectInput:any = ref({
    supplier_name: '',
    unit_name: ''
});

const stockMovementInput:any = ref({
    stock_object_id: '',
    movement_type: '',
    movement_value: '',
    movement_volume: '',
    movement_date:  '',
});

const getAllStockObjects: any = () => {
    let stockObjectTable = db.value.transaction("stockObject", "readwrite").objectStore("stockObject");
    let stockObjectTableRequest = stockObjectTable.getAll();

    stockObjectTableRequest.onsuccess = () => {
        if (stockObjectTableRequest.result !== undefined) {
            stockObjects.value = stockObjectTableRequest.result;
        } else {
            /*
                TODO: handle error and show popup
            */
            console.log("error in stock object");
        }
    };
}

const getAllStockMovements: any = () => {
    let stockMovementTable = db.value.transaction("stockMovement", "readwrite").objectStore("stockMovement");
    let stockMovementTableRequest = stockMovementTable.getAll();

    stockMovementTableRequest.onsuccess = () => {
        if (stockMovementTableRequest.result !== undefined) {
            stockMovements.value = stockMovementTableRequest.result;
        } else {
            /*
                TODO: handle error and show popup
            */
            console.log("error in stock movement");
        }
    };
}

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
                    autoIncrement: true
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

const createStockObject: any = () => {
    let stockObjectTable = db.value.transaction("stockObject", "readwrite").objectStore("stockObject");
    let stockObjectTableRequest = stockObjectTable.add({...stockObjectInput.value});

    stockObjectTableRequest.onsuccess = () => {
        console.log("Book added to the store", stockObjectTableRequest.result);
    };
    stockObjectTableRequest.onerror = () => {
        console.log("Error", stockObjectTableRequest.error);
    };

    getAllStockObjects();
}

const createStockMovement: any = () => {
    let stockMovementTable = db.value.transaction("stockMovement", "readwrite").objectStore("stockMovement");
    let stockMovementTableRequest = stockMovementTable.add({...stockMovementInput.value});

    stockMovementTableRequest.onsuccess = () => {
        console.log("Book added to the store", stockMovementTableRequest.result);
    };
    stockMovementTableRequest.onerror = () => {
        console.log("Error", stockMovementTableRequest.error);
    };

    getAllStockMovements();
}
</script>

<template>
    <div>
        <input 
            v-model="stockObjectInput.supplier_name"
            type="text" 
            placeholder="Nama supplier" 
        />
        <input 
            v-model="stockObjectInput.unit_name"
            type="text" 
            placeholder="Nama unit" 
        />
        <button 
            @click="createStockObject()"
        >
            Tambah Sebagai Stok
        </button>
    </div>
    <div>
        {{ stockObjects }}
    </div>
    <div>
        <input 
            v-model="stockMovementInput.stock_object_id"
            type="text" 
            placeholder="Stok yang bergerak" 
        />
        <input 
            v-model="stockMovementInput.movement_type"
            type="text" 
            placeholder="Tipe pergerakan stok" 
        />
        <input 
            v-model="stockMovementInput.movement_value"
            type="number" 
            placeholder="Nilai pergerakan stok" 
        />
        <input 
            v-model="stockMovementInput.movement_volume"
            type="number" 
            placeholder="Besar pergerakan stok" 
        />
        <input 
            v-model="stockMovementInput.movement_date"
            type="date" 
            placeholder="Tanggal pergerakan stok" 
        />
    
        <button 
            @click="createStockMovement()"
        >
            Tambah Sebagai Stok
        </button>
    </div>
    {{ stockMovementInput }}
    <div>
        {{ stockMovements }}
    </div>
</template>
