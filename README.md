<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Control de Inventario Colaborativo</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Estilo para la barra de desplazamiento */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: #f1f1f1;
        }
        ::-webkit-scrollbar-thumb {
            background: #888;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #555;
        }
        /* Animación para el toast */
        @keyframes slideIn {
            from { transform: translateY(100%); opacity: 0; }
            to { transform: translateY(0); opacity: 1; }
        }
        @keyframes slideOut {
            from { transform: translateY(0); opacity: 1; }
            to { transform: translateY(100%); opacity: 0; }
        }
        .toast-slide-in {
            animation: slideIn 0.5s forwards;
        }
        .toast-slide-out {
            animation: slideOut 0.5s forwards;
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <!-- Contenedor Principal -->
    <div class="container mx-auto p-4 md:p-6 lg:p-8 max-w-7xl">

        <header class="mb-6 flex justify-between items-center">
            <div>
                <h1 class="text-3xl font-bold text-gray-700">
                    <i class="fas fa-barcode mr-2"></i>Sistema de Inventario Colaborativo
                </h1>
                <p class="text-sm text-gray-500 mt-1">Todos los usuarios ven y editan el mismo inventario.</p>
            </div>
            <div id="auth-status" class="text-right">
                 <p class="text-sm text-gray-500 mt-1">ID de Sesión: <span id="userId" class="font-mono">Cargando...</span></p>
            </div>
        </header>

        <!-- Pestañas de Navegación -->
        <div class="mb-6">
            <div class="border-b border-gray-200">
                <nav class="-mb-px flex space-x-6" aria-label="Tabs">
                    <button id="tab-movimentar" class="tab-btn border-indigo-500 text-indigo-600 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                        <i class="fas fa-exchange-alt mr-2"></i>Mover Inventario
                    </button>
                    <button id="tab-gerenciar" class="tab-btn border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300 whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">
                        <i class="fas fa-box-open mr-2"></i>Administrar Productos
                    </button>
                </nav>
            </div>
        </div>

        <!-- Contenido de las Pestañas -->
        <main>
            <!-- Pantalla de Carga -->
            <div id="loader" class="text-center py-10">
                <i class="fas fa-spinner fa-spin text-4xl text-indigo-500"></i>
                <p class="mt-2 text-gray-600">Conectando a la base de datos...</p>
            </div>

            <!-- Pestaña: Mover Inventario -->
            <div id="content-movimentar" class="tab-content hidden">
                <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                    <!-- Columna de Lectura -->
                    <div class="lg:col-span-2 bg-white p-6 rounded-lg shadow-sm">
                        <h2 class="text-xl font-semibold mb-4">Operación de Inventario</h2>
                        <div class="mb-4">
                            <label for="barcode-scanner-input" class="block text-sm font-medium text-gray-700 mb-1">Escanear Código de Barras</label>
                            <div class="relative">
                                <i class="fas fa-barcode absolute left-3 top-1/2 -translate-y-1/2 text-gray-400"></i>
                                <input type="text" id="barcode-scanner-input" placeholder="Esperando lectura del código..." class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                            </div>
                        </div>

                        <!-- Detalles del Producto Escaneado -->
                        <div id="scanned-product-details" class="mt-6 p-4 bg-gray-50 rounded-md border border-gray-200 hidden">
                            <h3 class="font-bold text-lg text-gray-800" id="scanned-product-name">Nombre del Producto</h3>
                            <p class="text-sm text-gray-500" id="scanned-product-barcode">Código: </p>
                            <div class="mt-4 flex items-baseline">
                                <p class="text-3xl font-bold text-indigo-600" id="scanned-product-stock">0</p>
                                <p class="ml-2 text-gray-600">unidades en inventario</p>
                            </div>
                        </div>
                        
                        <div id="product-not-found" class="mt-6 p-4 bg-red-50 text-red-700 rounded-md border border-red-200 hidden">
                            <p><i class="fas fa-exclamation-triangle mr-2"></i>Producto no registrado. Ve a la pestaña "Administrar Productos" para agregarlo.</p>
                        </div>

                        <!-- Acciones de Entrada y Salida -->
                        <div id="stock-actions" class="mt-6 hidden">
                            <div class="flex items-center space-x-4">
                                <div>
                                    <label for="quantity-input" class="block text-sm font-medium text-gray-700">Cantidad</label>
                                    <input type="number" id="quantity-input" value="1" min="1" class="w-28 mt-1 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                                </div>
                                <div class="flex-grow pt-6 flex space-x-3">
                                    <button id="btn-stock-in" class="w-full flex-1 bg-green-500 hover:bg-green-600 text-white font-bold py-2 px-4 rounded-md transition duration-200 shadow-sm">
                                        <i class="fas fa-plus-circle mr-2"></i>Entrada
                                    </button>
                                    <button id="btn-stock-out" class="w-full flex-1 bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-md transition duration-200 shadow-sm">
                                        <i class="fas fa-minus-circle mr-2"></i>Salida
                                    </button>
                                </div>
                            </div>
                        </div>
                    </div>
                    <!-- Columna de Historial -->
                    <div class="bg-white p-6 rounded-lg shadow-sm">
                        <h2 class="text-xl font-semibold mb-4">Últimos Movimientos</h2>
                        <div id="movements-history" class="space-y-3 h-96 overflow-y-auto">
                           <p class="text-gray-500 text-sm">Ningún movimiento registrado aún.</p>
                        </div>
                    </div>
                </div>
            </div>

            <!-- Pestaña: Administrar Productos -->
            <div id="content-gerenciar" class="tab-content hidden">
                <div class="bg-white p-6 rounded-lg shadow-sm">
                    <h2 class="text-xl font-semibold mb-4">Agregar Nuevo Producto</h2>
                    <form id="add-product-form" class="grid grid-cols-1 md:grid-cols-4 gap-4 items-end">
                        <div class="md:col-span-2">
                            <label for="barcode" class="block text-sm font-medium text-gray-700">Código de Barras</label>
                            <input type="text" id="barcode" required class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        </div>
                        <div>
                            <label for="description" class="block text-sm font-medium text-gray-700">Descripción del Producto</label>
                            <input type="text" id="description" required class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        </div>
                        <div>
                            <label for="initial-stock" class="block text-sm font-medium text-gray-700">Inventario Inicial</label>
                            <input type="number" id="initial-stock" required value="0" min="0" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500">
                        </div>
                        <div class="md:col-span-4">
                            <button type="submit" class="w-full md:w-auto bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-6 rounded-md transition duration-200 shadow-sm">
                                <i class="fas fa-plus mr-2"></i>Agregar Producto
                            </button>
                        </div>
                    </form>

                    <hr class="my-8">

                    <h2 class="text-xl font-semibold mb-4">Lista de Productos</h2>
                    <div class="overflow-x-auto">
                        <table class="min-w-full bg-white">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Código de Barras</th>
                                    <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Descripción</th>
                                    <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Inventario Actual</th>
                                    <th class="py-3 px-4 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Acciones</th>
                                </tr>
                            </thead>
                            <tbody id="products-table-body" class="divide-y divide-gray-200">
                                <!-- Las filas de productos se insertarán aquí vía JS -->
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </main>
    </div>

    <!-- Modal de Edición -->
    <div id="edit-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-md shadow-lg rounded-md bg-white">
            <div class="flex justify-between items-center mb-4">
                <h3 class="text-lg font-medium">Editar Producto</h3>
                <button id="close-modal-btn" class="text-gray-400 hover:text-gray-600">&times;</button>
            </div>
            <form id="edit-product-form">
                <input type="hidden" id="edit-product-id">
                <div class="space-y-4">
                    <div>
                        <label for="edit-barcode" class="block text-sm font-medium text-gray-700">Código de Barras</label>
                        <input type="text" id="edit-barcode" required class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm">
                    </div>
                    <div>
                        <label for="edit-description" class="block text-sm font-medium text-gray-700">Descripción</label>
                        <input type="text" id="edit-description" required class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm">
                    </div>
                    <div>
                        <label for="edit-stock" class="block text-sm font-medium text-gray-700">Inventario</label>
                        <input type="number" id="edit-stock" required min="0" class="mt-1 block w-full border border-gray-300 rounded-md shadow-sm">
                    </div>
                </div>
                <div class="mt-6 flex justify-end space-x-3">
                    <button type="button" id="cancel-edit-btn" class="bg-gray-200 hover:bg-gray-300 text-gray-800 font-bold py-2 px-4 rounded-md">Cancelar</button>
                    <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-md">Guardar Cambios</button>
                </div>
            </form>
        </div>
    </div>
    
    <!-- Modal de Confirmación de Eliminación -->
    <div id="delete-confirm-modal" class="fixed inset-0 bg-gray-600 bg-opacity-50 overflow-y-auto h-full w-full hidden z-50">
        <div class="relative top-20 mx-auto p-5 border w-full max-w-md shadow-lg rounded-md bg-white">
            <div class="mt-3 text-center">
                <div class="mx-auto flex items-center justify-center h-12 w-12 rounded-full bg-red-100">
                    <i class="fas fa-exclamation-triangle text-red-600 text-xl"></i>
                </div>
                <h3 class="text-lg leading-6 font-medium text-gray-900 mt-4">Eliminar Producto</h3>
                <div class="mt-2 px-7 py-3">
                    <p class="text-sm text-gray-500">
                        ¿Estás seguro de que quieres eliminar este producto? Esta acción no se puede deshacer.
                    </p>
                </div>
                <div class="items-center px-4 py-3 space-x-4">
                    <button id="cancel-delete-btn" class="px-4 py-2 bg-gray-200 text-gray-800 rounded-md hover:bg-gray-300">
                        Cancelar
                    </button>
                    <button id="confirm-delete-btn" class="px-4 py-2 bg-red-600 text-white rounded-md hover:bg-red-700">
                        Eliminar
                    </button>
                </div>
            </div>
        </div>
    </div>

    <!-- Notificación Toast -->
    <div id="toast" class="fixed bottom-5 right-5 bg-gray-800 text-white py-3 px-5 rounded-lg shadow-lg hidden z-50">
        <p id="toast-message"></p>
    </div>


    <script type="module">
        // Importaciones de Firebase
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, getDoc, updateDoc, deleteDoc, query, where, getDocs, serverTimestamp, runTransaction } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- CONFIGURACIÓN Y AUTENTICACIÓN DE FIREBASE ---
        
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        let productsCollectionRef = null;
        let movementsCollectionRef = null;
        let currentProduct = null; 

        const loader = document.getElementById('loader');
        const mainContentMovimentar = document.getElementById('content-movimentar');
        const mainContentGerenciar = document.getElementById('content-gerenciar');
        const userIdSpan = document.getElementById('userId');

        // --- **FIX START**: More Robust Authentication Flow ---
        async function main() {
            try {
                // First, attempt to sign in. This handles all cases.
                let userCredential;
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    userCredential = await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    // If no specific token, sign in anonymously.
                    // This also handles the case where a user is already signed in.
                    userCredential = await signInAnonymously(auth);
                }
                const user = userCredential.user;
                
                // Now that we are GUARANTEED to have a user, we can initialize the app.
                // Set user info on the UI
                userIdSpan.textContent = user.uid.substring(0, 8);
                
                // Define collection paths
                const publicDataPath = `artifacts/${appId}/public/data`;
                productsCollectionRef = collection(db, `${publicDataPath}/products`);
                movementsCollectionRef = collection(db, `${publicDataPath}/movements`);

                // Start the main application logic
                initializeAppLogic();

            } catch (error) {
                console.error("Authentication or Initialization Failed:", error);
                loader.textContent = "Authentication failed. Please reload the page.";
            }
        }

        main(); // Run the main authentication and initialization function
        // --- **FIX END** ---


        // --- LÓGICA DE LA APLICACIÓN ---
        function initializeAppLogic() {
            loader.style.display = 'none';
            mainContentMovimentar.classList.remove('hidden');
            
            setupTabs();
            setupGerenciarProdutos();
            setupMovimentarEstoque();
            listenToProducts();
            listenToMovements();
            setupDeleteModal();
        }

        // --- LÓGICA DE LAS PESTAÑAS ---
        function setupTabs() {
            const tabs = document.querySelectorAll('.tab-btn');
            const contents = document.querySelectorAll('.tab-content');
            const barcodeInput = document.getElementById('barcode-scanner-input');

            tabs.forEach(tab => {
                tab.addEventListener('click', () => {
                    tabs.forEach(t => {
                        t.classList.remove('border-indigo-500', 'text-indigo-600');
                        t.classList.add('border-transparent', 'text-gray-500', 'hover:text-gray-700', 'hover:border-gray-300');
                    });
                    tab.classList.add('border-indigo-500', 'text-indigo-600');
                    tab.classList.remove('border-transparent', 'text-gray-500');

                    contents.forEach(content => content.classList.add('hidden'));
                    const targetContent = document.getElementById(tab.id.replace('tab-', 'content-'));
                    if (targetContent) {
                        targetContent.classList.remove('hidden');
                    }
                    
                    if (tab.id === 'tab-movimentar') {
                        barcodeInput.focus();
                        barcodeInput.select();
                    }
                });
            });
            document.getElementById('tab-movimentar').click();
        }

        // --- LÓGICA DE ADMINISTRACIÓN DE PRODUCTOS ---
        function setupGerenciarProdutos() {
            const addForm = document.getElementById('add-product-form');
            addForm.addEventListener('submit', async (e) => {
                e.preventDefault();
                const barcode = document.getElementById('barcode').value.trim();
                const description = document.getElementById('description').value.trim();
                const initialStock = parseInt(document.getElementById('initial-stock').value);

                if (!barcode || !description || isNaN(initialStock)) {
                    showToast('Por favor, completa todos los campos.', 'error');
                    return;
                }

                try {
                    const q = query(productsCollectionRef, where("barcode", "==", barcode));
                    const querySnapshot = await getDocs(q);
                    if (!querySnapshot.empty) {
                        showToast('Este código de barras ya está registrado.', 'error');
                        return;
                    }

                    await addDoc(productsCollectionRef, {
                        barcode: barcode,
                        description: description,
                        quantity: initialStock,
                        createdAt: serverTimestamp()
                    });
                    showToast('¡Producto agregado con éxito!');
                    addForm.reset();
                } catch (error) {
                    console.error("Error al agregar producto: ", error);
                    showToast('Error al agregar producto.', 'error');
                }
            });
            
            setupEditModal();
        }

        function listenToProducts() {
            const tableBody = document.getElementById('products-table-body');
            onSnapshot(query(productsCollectionRef), (snapshot) => {
                tableBody.innerHTML = '';
                if (snapshot.empty) {
                    tableBody.innerHTML = `<tr><td colspan="4" class="text-center py-4 text-gray-500">Ningún producto registrado.</td></tr>`;
                    return;
                }
                const sortedDocs = snapshot.docs.sort((a, b) => (a.data().description > b.data().description) ? 1 : -1);

                sortedDocs.forEach(doc => {
                    const product = doc.data();
                    const row = `
                        <tr id="${doc.id}">
                            <td class="py-3 px-4 font-mono text-sm">${product.barcode}</td>
                            <td class="py-3 px-4">${product.description}</td>
                            <td class="py-3 px-4 font-bold text-lg">${product.quantity}</td>
                            <td class="py-3 px-4 space-x-2">
                                <button data-id="${doc.id}" class="edit-btn text-indigo-600 hover:text-indigo-900"><i class="fas fa-edit"></i></button>
                                <button data-id="${doc.id}" class="delete-btn text-red-600 hover:text-red-900"><i class="fas fa-trash"></i></button>
                            </td>
                        </tr>
                    `;
                    tableBody.innerHTML += row;
                });
                
                document.querySelectorAll('.edit-btn').forEach(btn => btn.addEventListener('click', () => openEditModal(btn.dataset.id)));
                document.querySelectorAll('.delete-btn').forEach(btn => btn.addEventListener('click', () => confirmDelete(btn.dataset.id)));

                if (currentProduct) {
                    const updatedProduct = snapshot.docs.find(d => d.id === currentProduct.id);
                    if (updatedProduct) {
                        currentProduct = { id: updatedProduct.id, ...updatedProduct.data() };
                        document.getElementById('scanned-product-stock').textContent = currentProduct.quantity;
                    }
                }
            }, (error) => {
                console.error("Error in products listener:", error);
                showToast("Error al cargar productos: " + error.message, "error");
            });
        }
        
        function confirmDelete(id) {
            const deleteModal = document.getElementById('delete-confirm-modal');
            const confirmDeleteBtn = document.getElementById('confirm-delete-btn');
            confirmDeleteBtn.dataset.id = id;
            deleteModal.classList.remove('hidden');
        }

        function setupDeleteModal() {
            const deleteModal = document.getElementById('delete-confirm-modal');
            const cancelDeleteBtn = document.getElementById('cancel-delete-btn');
            const confirmDeleteBtn = document.getElementById('confirm-delete-btn');

            const hideDeleteModal = () => {
                delete confirmDeleteBtn.dataset.id;
                deleteModal.classList.add('hidden');
            };
            
            cancelDeleteBtn.addEventListener('click', hideDeleteModal);

            confirmDeleteBtn.addEventListener('click', async (e) => {
                const id = e.currentTarget.dataset.id;
                if (id) {
                    try {
                        await deleteDoc(doc(productsCollectionRef, id));
                        showToast('¡Producto eliminado con éxito!');
                    } catch (error) {
                        console.error("Error al eliminar producto: ", error);
                        showToast('Error al eliminar producto.', 'error');
                    } finally {
                        hideDeleteModal();
                    }
                }
            });
        }

        function setupEditModal() {
            const modal = document.getElementById('edit-modal');
            const closeBtn = document.getElementById('close-modal-btn');
            const cancelBtn = document.getElementById('cancel-edit-btn');
            const editForm = document.getElementById('edit-product-form');

            const closeModal = () => modal.classList.add('hidden');
            
            closeBtn.addEventListener('click', closeModal);
            cancelBtn.addEventListener('click', closeModal);

            editForm.addEventListener('submit', async (e) => {
                e.preventDefault();
                const id = document.getElementById('edit-product-id').value;
                const barcode = document.getElementById('edit-barcode').value.trim();
                const description = document.getElementById('edit-description').value.trim();
                const stock = parseInt(document.getElementById('edit-stock').value);

                try {
                    const productRef = doc(productsCollectionRef, id);
                    await updateDoc(productRef, { barcode, description, quantity: stock });
                    showToast('¡Producto actualizado con éxito!');
                    closeModal();
                } catch (error) {
                    console.error("Error al actualizar producto: ", error);
                    showToast('Error al actualizar producto.', 'error');
                }
            });
        }
        
        async function openEditModal(id) {
            try {
                const productRef = doc(productsCollectionRef, id);
                const docSnap = await getDoc(productRef);
                if (docSnap.exists()) {
                    const product = docSnap.data();
                    document.getElementById('edit-product-id').value = id;
                    document.getElementById('edit-barcode').value = product.barcode;
                    document.getElementById('edit-description').value = product.description;
                    document.getElementById('edit-stock').value = product.quantity;
                    document.getElementById('edit-modal').classList.remove('hidden');
                } else {
                    showToast('Producto no encontrado.', 'error');
                }
            } catch (error) {
                console.error("Error al buscar producto para editar: ", error);
                showToast('Error al buscar producto.', 'error');
            }
        }

        function setupMovimentarEstoque() {
            const barcodeInput = document.getElementById('barcode-scanner-input');
            barcodeInput.addEventListener('change', async () => {
                const barcode = barcodeInput.value.trim();
                if (!barcode) return;

                const q = query(productsCollectionRef, where("barcode", "==", barcode));
                const querySnapshot = await getDocs(q);

                const detailsDiv = document.getElementById('scanned-product-details');
                const notFoundDiv = document.getElementById('product-not-found');
                const actionsDiv = document.getElementById('stock-actions');

                if (querySnapshot.empty) {
                    currentProduct = null;
                    detailsDiv.classList.add('hidden');
                    actionsDiv.classList.add('hidden');
                    notFoundDiv.classList.remove('hidden');
                } else {
                    const productDoc = querySnapshot.docs[0];
                    currentProduct = { id: productDoc.id, ...productDoc.data() };
                    
                    document.getElementById('scanned-product-name').textContent = currentProduct.description;
                    document.getElementById('scanned-product-barcode').textContent = `Código: ${currentProduct.barcode}`;
                    document.getElementById('scanned-product-stock').textContent = currentProduct.quantity;
                    
                    notFoundDiv.classList.add('hidden');
                    detailsDiv.classList.remove('hidden');
                    actionsDiv.classList.remove('hidden');
                    document.getElementById('quantity-input').focus();
                    document.getElementById('quantity-input').select();
                }
            });

            document.getElementById('btn-stock-in').addEventListener('click', () => updateStock('Entrada'));
            document.getElementById('btn-stock-out').addEventListener('click', () => updateStock('Salida'));
        }

        async function updateStock(type) {
            if (!currentProduct) return;
            const quantity = parseInt(document.getElementById('quantity-input').value);

            if (isNaN(quantity) || quantity <= 0) {
                showToast('Cantidad inválida.', 'error');
                return;
            }
            
            const productRef = doc(productsCollectionRef, currentProduct.id);

            try {
                await runTransaction(db, async (transaction) => {
                    const productDoc = await transaction.get(productRef);
                    if (!productDoc.exists()) {
                        throw "¡El producto ya no existe!";
                    }

                    const currentStock = productDoc.data().quantity;
                    let newStock = currentStock;

                    if (type === 'Entrada') {
                        newStock += quantity;
                    } else { 
                        if (currentStock < quantity) {
                            throw "Inventario insuficiente para esta salida.";
                        }
                        newStock -= quantity;
                    }

                    transaction.update(productRef, { quantity: newStock });
                    
                    const movementRef = doc(movementsCollectionRef);
                    transaction.set(movementRef, {
                        productId: currentProduct.id,
                        barcode: currentProduct.barcode,
                        description: currentProduct.description,
                        type: type,
                        quantity: quantity,
                        timestamp: serverTimestamp()
                    });
                });

                showToast(`¡Movimiento de ${type.toLowerCase()} registrado!`);
                document.getElementById('quantity-input').value = '1';
                document.getElementById('barcode-scanner-input').focus();
                document.getElementById('barcode-scanner-input').select();

            } catch (e) {
                console.error("Transaction failed: ", e);
                const errorMessage = typeof e === 'string' ? e : 'Error al actualizar el inventario.';
                showToast(errorMessage, 'error');
            }
        }

        function listenToMovements() {
            const historyDiv = document.getElementById('movements-history');
            const q = query(movementsCollectionRef); 
            
            onSnapshot(q, (snapshot) => {
                if (snapshot.empty) {
                    historyDiv.innerHTML = `<p class="text-gray-500 text-sm">Ningún movimiento registrado aún.</p>`;
                    return;
                }

                const sortedDocs = snapshot.docs.sort((a, b) => {
                    const timeA = a.data().timestamp?.toDate() || 0;
                    const timeB = b.data().timestamp?.toDate() || 0;
                    return timeB - timeA;
                }).slice(0, 20);

                historyDiv.innerHTML = '';
                sortedDocs.forEach(doc => {
                    const movement = doc.data();
                    const isEntrada = movement.type === 'Entrada';
                    const bgColor = isEntrada ? 'bg-green-50' : 'bg-red-50';
                    const textColor = isEntrada ? 'text-green-800' : 'text-red-800';
                    const icon = isEntrada ? 'fa-arrow-up' : 'fa-arrow-down';
                    const date = movement.timestamp ? new Date(movement.timestamp.seconds * 1000).toLocaleString('es-ES') : 'ahora';

                    const movementHTML = `
                        <div class="p-3 rounded-md ${bgColor} ${textColor}">
                            <div class="flex justify-between items-center">
                                <p class="font-bold"><i class="fas ${icon} mr-2"></i>${movement.type} (${movement.quantity})</p>
                                <p class="text-xs">${date}</p>
                            </div>
                            <p class="text-sm truncate">${movement.description}</p>
                        </div>
                    `;
                    historyDiv.innerHTML += movementHTML;
                });
            }, (error) => {
                console.error("Error in movements listener:", error);
                showToast("Error al cargar movimientos: " + error.message, "error");
            });
        }

        function showToast(message, type = 'success') {
            const toast = document.getElementById('toast');
            const toastMessage = document.getElementById('toast-message');
            
            toastMessage.textContent = message;
            toast.className = 'fixed bottom-5 right-5 text-white py-3 px-5 rounded-lg shadow-lg z-50';
            
            if (type === 'error') {
                toast.classList.add('bg-red-600');
            } else {
                toast.classList.add('bg-gray-800');
            }
            
            toast.classList.remove('hidden');
            toast.classList.add('toast-slide-in');

            setTimeout(() => {
                toast.classList.remove('toast-slide-in');
                toast.classList.add('toast-slide-out');
                setTimeout(() => toast.classList.add('hidden'), 500);
            }, 3000);
        }

    </script>
</body>
</html>
# meuestoque
