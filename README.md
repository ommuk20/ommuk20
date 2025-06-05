<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SBT Lounge - Billing System (Simple)</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f2f5;
        }
        .salon-header {
            background: linear-gradient(to right, #6366f1, #8b5cf6);
            color: white;
            padding: 1rem;
            text-align: center;
            border-radius: 0.5rem;
            margin-bottom: 1rem;
        }
        .card {
            background-color: white;
            border-radius: 0.75rem;
            box-shadow: 0 4px 6px -1px rgba(0, 0, 0, 0.1), 0 2px 4px -1px rgba(0, 0, 0, 0.06);
            padding: 1.5rem;
            margin-bottom: 1.5rem;
        }
        .btn {
            padding: 0.75rem 1.5rem;
            border-radius: 0.5rem;
            font-weight: 500;
            transition: background-color 0.3s ease;
            cursor: pointer;
            display: inline-flex;
            align-items: center;
            justify-content: center;
        }
        .btn-sm {
            padding: 0.5rem 1rem;
            font-size: 0.875rem;
        }
        .btn-xs {
            padding: 0.25rem 0.5rem;
            font-size: 0.75rem;
        }
        .btn-primary {
            background-color: #4f46e5;
            color: white;
        }
        .btn-primary:hover {
            background-color: #4338ca;
        }
        .btn-secondary {
            background-color: #6b7280;
            color: white;
        }
        .btn-secondary:hover {
            background-color: #4b5563;
        }
        .btn-danger {
            background-color: #ef4444;
            color: white;
        }
        .btn-danger:hover {
            background-color: #dc2626;
        }
        .btn-outline {
            border: 1px solid #4f46e5;
            color: #4f46e5;
        }
        .btn-outline:hover {
            background-color: #e0e7ff;
        }

        /* Bill Preview Area */
        .bill-preview {
            border: 1px solid #e5e7eb;
            padding: 0.5rem;
            margin-top: 1rem;
            background-color: #fff;
            border-radius: 0.25rem;
            max-width: 300px; 
            margin-left: auto;
            margin-right: auto;
            font-size: 9pt;
            color: #000;
        }
        .bill-preview h3 {
            font-size: 12pt;
            font-weight: 600;
            text-align: center;
            margin-bottom: 0.3rem;
        }
        .bill-preview p, .bill-preview .salon-details p {
            margin-bottom: 0.1rem;
            line-height: 1.2;
            font-size: 8pt;
        }
        .bill-preview .salon-details {
            text-align: center;
            margin-bottom: 0.5rem;
        }
        .bill-preview hr {
            margin-top: 0.3rem;
            margin-bottom: 0.3rem;
            border-top: 1px dashed #555;
        }
        .bill-preview .customer-info p {
             font-size: 8pt;
        }
        .bill-preview .item-list-header, .bill-preview .item-list-item {
            display: flex;
            justify-content: space-between;
            font-size: 8pt;
            padding: 2px 0;
        }
        .bill-preview .item-list-header {
            font-weight: bold;
            border-bottom: 1px solid #555;
            margin-bottom: 3px;
        }
        .bill-preview .item-list-item .item-name {
            flex-grow: 1;
            text-align: left;
            padding-right: 5px;
        }
        .bill-preview .item-list-item .item-price {
            text-align: right;
            min-width: 50px;
        }
        .bill-preview .totals-summary {
            margin-top: 0.5rem;
            text-align: right;
        }
        .bill-preview .totals-summary p {
            font-size: 9pt;
            margin-bottom: 0.1rem;
        }
        .bill-preview .grand-total {
            font-size: 10pt;
            font-weight: bold;
        }
        .bill-preview .footer-thankyou {
            text-align: center;
            font-size: 8pt;
            margin-top: 0.5rem;
        }
        .bill-preview .item-table { display: none; }

        #pastBillsTable th, #pastBillsTable td {
            padding: 0.75rem;
            border: 1px solid #e5e7eb;
        }
        .clickable-customer {
            color: #4f46e5;
            text-decoration: underline;
            cursor: pointer;
        }
        .clickable-customer:hover {
            color: #4338ca;
        }
        #loadingSpinner {
            position: fixed;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            z-index: 9999;
        }
        .spinner {
            border: 4px solid rgba(0, 0, 0, 0.1);
            width: 36px;
            height: 36px;
            border-radius: 50%;
            border-left-color: #4f46e5;
            animation: spin 1s ease infinite;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        .price-input {
            width: 70px;
            text-align: right;
            padding: 0.25rem 0.5rem;
            border: 1px solid #d1d5db;
            border-radius: 0.375rem;
        }

        /* Print specific styles */
        @media print {
            body {
                width: 76mm; margin: 0; padding: 2mm; background-color: #fff; font-size: 9pt; color: #000;
            }
            .no-print { display: none !important; }
            .salon-header, .card, .md\:col-span-1, .md\:col-span-2>div:not(#billPreviewModal), 
            #billPreviewModal > div > div:last-child /* buttons in modal */ {
                display: none !important; 
            }
            #billPreviewModal {
                position: static !important; background: none !important; display: block !important; 
                box-shadow: none !important; overflow: visible !important; padding: 0 !important;
                width: 100% !important; max-width: 100% !important; max-height: none !important;
            }
            #billPreviewArea, #billPreviewArea * { visibility: visible !important; }
            #billPreviewArea {
                position: static !important; width: 100% !important; max-width: none !important;
                margin: 0 !important; padding: 0 !important; border: none !important; box-shadow: none !important;
                font-size: 9pt !important; color: #000 !important; background-color: #fff !important;
            }
            .price-input { display: none !important; }
            .price-display-print { display: inline !important; }
        }
        .price-display-print { display: none; }
    </style>
</head>
<body class="p-4 md:p-8">

    <div id="loadingSpinner" class="hidden">
        <div class="spinner"></div>
    </div>

    <div class="salon-header no-print">
        <h1 class="text-3xl font-bold">SBT Lounge</h1>
        <p class="text-lg">Unisex Salon - Billing System (Simple)</p>
        <p class="text-sm">User ID: <span id="currentUserId">Loading...</span></p>
    </div>

    <div class="container mx-auto no-print">
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6">
            <!-- Billing Section -->
            <div class="md:col-span-2">
                <div class="card">
                    <h2 class="text-2xl font-semibold mb-4 text-indigo-600">New Bill</h2>
                    <div class="mb-4">
                        <label for="customerName" class="block text-sm font-medium text-gray-700">Customer Name:</label>
                        <input type="text" id="customerName" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="Enter customer name">
                    </div>
                    <div class="mb-6">
                        <label for="customerMobile" class="block text-sm font-medium text-gray-700">Mobile Number:</label>
                        <input type="tel" id="customerMobile" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="Enter mobile number">
                    </div>

                    <div class="mb-6">
                        <h3 class="text-xl font-semibold mb-3 text-gray-800">Select Services</h3>
                        <div id="serviceCategories" class="space-y-4 max-h-80 overflow-y-auto p-2 border rounded-md">
                        </div>
                    </div>

                    <div class="mb-6 p-4 border border-dashed border-indigo-300 rounded-md">
                        <h3 class="text-lg font-semibold mb-3 text-indigo-700">Add Custom Service</h3>
                        <div class="grid grid-cols-1 sm:grid-cols-3 gap-4 items-end">
                            <div class="sm:col-span-2">
                                <label for="customServiceName" class="block text-sm font-medium text-gray-700">Service Name:</label>
                                <input type="text" id="customServiceName" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="e.g., Special Treatment">
                            </div>
                            <div>
                                <label for="customServicePrice" class="block text-sm font-medium text-gray-700">Price (₹):</label>
                                <input type="number" id="customServicePrice" min="0" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="e.g., 250">
                            </div>
                        </div>
                        <button id="addCustomServiceBtn" class="btn btn-primary btn-sm mt-3">
                             <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                                <path stroke-linecap="round" stroke-linejoin="round" d="M12 4v16m8-8H4" />
                            </svg>
                            Add Custom Service
                        </button>
                    </div>

                    <div class="mb-6">
                        <h3 class="text-xl font-semibold mb-3 text-gray-800">Current Bill Items</h3>
                        <div id="currentBillItems" class="space-y-2 min-h-[50px] p-2 border rounded-md bg-gray-50">
                            <p class="text-gray-500 italic">No items added yet.</p>
                        </div>
                        
                        <div class="mt-4 pt-4 border-t border-gray-200">
                            <div class="grid grid-cols-1 sm:grid-cols-2 gap-4 mb-2">
                                <div>
                                    <label for="discountPercent" class="block text-sm font-medium text-gray-700">Discount (%):</label>
                                    <input type="number" id="discountPercent" min="0" max="100" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="e.g., 10">
                                </div>
                                <div>
                                    <label for="discountFixed" class="block text-sm font-medium text-gray-700">Discount (₹ Fixed):</label>
                                    <input type="number" id="discountFixed" min="0" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="e.g., 50">
                                </div>
                            </div>

                            <div class="text-right space-y-1">
                                <p class="text-md text-gray-600">Subtotal: ₹<span id="currentBillSubtotal">0.00</span></p>
                                <p class="text-md text-red-500">Discount Applied: - ₹<span id="currentBillDiscountApplied">0.00</span></p>
                                <p class="text-2xl font-bold text-indigo-700">Grand Total: ₹<span id="currentBillGrandTotal">0.00</span></p>
                            </div>
                        </div>
                    </div>

                    <div class="flex flex-col sm:flex-row space-y-2 sm:space-y-0 sm:space-x-3">
                        <button id="finalizeBillBtn" class="btn btn-primary w-full sm:w-auto flex-grow">Finalize & Prepare Bill</button>
                        <button id="clearBillBtn" class="btn btn-danger w-full sm:w-auto">Clear Bill</button>
                    </div>
                </div>
            </div>

            <div class="md:col-span-1">
                <div class="card">
                    <h2 class="text-2xl font-semibold mb-4 text-indigo-600">Past Bills</h2>
                    <div class="mb-4 flex items-center space-x-2">
                        <input type="text" id="searchPastBills" class="block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" placeholder="Search by Name or Bill No.">
                        <button id="clearSearchBtn" class="btn btn-secondary btn-sm">Clear</button>
                    </div>
                    <div class="max-h-[600px] overflow-y-auto">
                        <table id="pastBillsTable" class="min-w-full divide-y divide-gray-200 text-sm">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="text-left font-medium text-gray-500 uppercase tracking-wider">Bill #</th>
                                    <th class="text-left font-medium text-gray-500 uppercase tracking-wider">Customer</th>
                                    <th class="text-left font-medium text-gray-500 uppercase tracking-wider">Total</th>
                                    <th class="text-left font-medium text-gray-500 uppercase tracking-wider">Date</th>
                                    <th class="text-left font-medium text-gray-500 uppercase tracking-wider">Actions</th>
                                </tr>
                            </thead>
                            <tbody id="pastBillsList" class="bg-white divide-y divide-gray-200">
                                <tr><td colspan="5" class="text-center p-4 text-gray-500">No past bills found.</td></tr>
                            </tbody>
                        </table>
                    </div>
                </div>
            </div>
        </div>

        <!-- Bill Preview Modal -->
        <div id="billPreviewModal" class="fixed inset-0 bg-gray-800 bg-opacity-75 flex items-center justify-center p-4 hidden z-50">
            <div class="bg-white p-6 rounded-lg shadow-xl w-full max-w-md max-h-[90vh] overflow-y-auto no-print"> 
                <div id="billPreviewArea" class="bill-preview">
                    <!-- Bill content injected here -->
                </div>
                <div class="mt-6 flex flex-wrap justify-end space-x-0 sm:space-x-3 space-y-2 sm:space-y-0">
                    <button id="printBillBtn" class="btn btn-primary btn-sm w-full sm:w-auto">Print Bill</button>
                    <button id="downloadPdfBtn" class="btn btn-primary btn-sm w-full sm:w-auto">Download PDF</button>
                    <button id="closePreviewBtn" class="btn btn-secondary btn-sm w-full sm:w-auto">Close / New Bill</button>
                </div>
            </div>
        </div>
    </div>

    <script type="module">
        // Firebase Imports
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, doc, addDoc, setDoc, getDoc, collection, query, onSnapshot, serverTimestamp, orderBy, where, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'sbt-lounge-billing-simple';

        let app, auth, db, userId;
        let currentBillItemsArray = []; // Stores { name, price, isCustom }

        // DOM Elements
        const customerNameEl = document.getElementById('customerName');
        const customerMobileEl = document.getElementById('customerMobile');
        const serviceCategoriesEl = document.getElementById('serviceCategories');
        const currentBillItemsEl = document.getElementById('currentBillItems');
        const customServiceNameEl = document.getElementById('customServiceName');
        const customServicePriceEl = document.getElementById('customServicePrice');
        const addCustomServiceBtn = document.getElementById('addCustomServiceBtn');
        const discountPercentEl = document.getElementById('discountPercent');
        const discountFixedEl = document.getElementById('discountFixed');
        const currentBillSubtotalEl = document.getElementById('currentBillSubtotal');
        const currentBillDiscountAppliedEl = document.getElementById('currentBillDiscountApplied');
        const currentBillGrandTotalEl = document.getElementById('currentBillGrandTotal');
        const finalizeBillBtn = document.getElementById('finalizeBillBtn');
        const clearBillBtn = document.getElementById('clearBillBtn');
        const billPreviewModal = document.getElementById('billPreviewModal');
        const billPreviewArea = document.getElementById('billPreviewArea');
        const printBillBtn = document.getElementById('printBillBtn');
        const downloadPdfBtn = document.getElementById('downloadPdfBtn');
        const closePreviewBtn = document.getElementById('closePreviewBtn');
        const pastBillsListEl = document.getElementById('pastBillsList');
        const searchPastBillsEl = document.getElementById('searchPastBills');
        const clearSearchBtn = document.getElementById('clearSearchBtn');
        const currentUserIdEl = document.getElementById('currentUserId');
        const loadingSpinner = document.getElementById('loadingSpinner');


        const salonDetails = {
            name: "SBT Lounge",
            type: "Unisex Salon",
            address: "Tarulia, Krishnapur Main Road, Kolkata 102",
            phone: "Ph: +91 1234567890" // Example phone
        };

        const allServices = [
            { category: "Men's Services", name: "Men's Haircut - Standard", price: 150 },
            { category: "Men's Services", name: "Men's Haircut - Style", price: 200 },
            { category: "Men's Services", name: "Beard Trim", price: 80 },
            { category: "Men's Services", name: "Shave", price: 70 },
            { category: "Men's Services", name: "Haircut & Beard Trim Combo", price: 220 },
            { category: "Men's Services", name: "Head Massage (Men)", price: 100 },
            { category: "Men's Services", name: "Hair Wash (Men)", price: 60 },
            { category: "Girls' Services", name: "Girls' Haircut - Standard", price: 300 },
            { category: "Girls' Services", name: "Girls' Haircut - Style", price: 450 },
            { category: "Girls' Services", name: "Hair Wash & Blow Dry", price: 250 },
            { category: "Girls' Services", name: "Hair Styling (Curls/Straight)", price: 350 },
            { category: "Girls' Services", name: "Basic Facial", price: 600 },
            { category: "Girls' Services", name: "Deluxe Facial", price: 1000 },
            { category: "Girls' Services", name: "Manicure", price: 300 },
            { category: "Girls' Services", name: "Pedicure", price: 350 },
            { category: "Girls' Services", name: "Waxing - Full Arms", price: 250 },
            { category: "Girls' Services", name: "Waxing - Full Legs", price: 350 },
            { category: "Girls' Services", name: "Eyebrow Threading", price: 40 },
            { category: "Nail Art", name: "Nail Art (per nail)", price: 50 },
            { category: "Nail Art", name: "Full Hand Basic Nail Art", price: 400 },
            { category: "Nail Art", name: "Gel Polish with Art", price: 700 },
            { category: "Unisex Hair Treatments", name: "Hair Spa (Short Hair)", price: 700 },
            { category: "Unisex Hair Treatments", name: "Hair Spa (Long Hair)", price: 1000 },
            { category: "Unisex Hair Treatments", name: "Hair Color - Global (Short)", price: 1500 },
            { category: "Unisex Hair Treatments", name: "Hair Color - Global (Long)", price: 2500 },
            { category: "Unisex Hair Treatments", name: "Keratin Treatment (Starts)", price: 3000 },
            { category: "Other Services", name: "Head Massage (Unisex)", price: 200 },
            { category: "Other Services", name: "Detan Pack (Face)", price: 300 },
        ];

        function showLoading() { loadingSpinner.classList.remove('hidden'); }
        function hideLoading() { loadingSpinner.classList.add('hidden'); }

        function showCustomAlert(message, type = "error", duration = 3000) {
            const alertId = 'custom-alert-modal-unique'; 
            let existingAlert = document.getElementById(alertId);
            if (existingAlert) existingAlert.remove(); 

            const alertModal = document.createElement('div');
            alertModal.id = alertId;
            alertModal.className = `fixed top-5 right-5 p-4 rounded-md shadow-lg text-white text-sm z-[10000] transition-all duration-300 ease-in-out transform translate-x-full no-print`;
            
            if (type === "success") alertModal.classList.add('bg-green-500');
            else if (type === "info") alertModal.classList.add('bg-blue-500');
            else alertModal.classList.add('bg-red-500');
            
            alertModal.textContent = message;
            document.body.appendChild(alertModal);

            setTimeout(() => { alertModal.classList.remove('translate-x-full'); alertModal.classList.add('translate-x-0'); }, 50);
            setTimeout(() => {
                alertModal.classList.add('opacity-0', 'translate-x-full');
                setTimeout(() => { if (document.body.contains(alertModal)) alertModal.remove(); }, 350); 
            }, duration);
        }

        async function initializeFirebase() {
            showLoading();
            try {
                if (Object.keys(firebaseConfig).length === 0) {
                    console.error("Firebase config is missing.");
                    showCustomAlert("Critical Error: Firebase configuration is missing. App cannot start.");
                    hideLoading(); return;
                }
                app = initializeApp(firebaseConfig);
                auth = getAuth(app);
                db = getFirestore(app);

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        currentUserIdEl.textContent = userId;
                        await loadServices();
                        await loadPastBills();
                    } else {
                        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                            await signInWithCustomToken(auth, __initial_auth_token);
                        } else {
                            await signInAnonymously(auth);
                        }
                    }
                    hideLoading();
                });
            } catch (error) {
                console.error("Firebase initialization error:", error);
                showCustomAlert(`Error initializing Firebase: ${error.message}.`);
                hideLoading();
            }
        }
        
        async function loadServices() {
            serviceCategoriesEl.innerHTML = ''; 
            const categories = {};
            allServices.forEach(service => {
                if (!categories[service.category]) categories[service.category] = [];
                categories[service.category].push(service);
            });

            for (const categoryName in categories) {
                const categoryDiv = document.createElement('div');
                categoryDiv.innerHTML = `<h4 class="text-lg font-semibold mt-3 mb-1 text-indigo-600">${categoryName}</h4>`;
                const ul = document.createElement('ul');
                ul.className = 'space-y-1';
                categories[categoryName].forEach(service => {
                    const li = document.createElement('li');
                    li.className = 'service-item p-2 hover:bg-indigo-50 rounded-md transition-colors duration-150 flex justify-between items-center';
                    li.innerHTML = `
                        <span class="flex-grow">${service.name} (₹${service.price.toFixed(2)})</span>
                        <button class="add-service-btn btn btn-outline btn-xs py-1 px-2" data-name="${service.name}" data-price="${service.price}">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-4 w-4 mr-1" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2"><path stroke-linecap="round" stroke-linejoin="round" d="M12 4v16m8-8H4" /></svg>Add
                        </button>`;
                    ul.appendChild(li);
                });
                categoryDiv.appendChild(ul);
                serviceCategoriesEl.appendChild(categoryDiv);
            }
            document.querySelectorAll('.add-service-btn').forEach(button => {
                button.addEventListener('click', (e) => {
                    const targetButton = e.target.closest('.add-service-btn');
                    addServiceToBill(targetButton.dataset.name, parseFloat(targetButton.dataset.price));
                });
            });
        }

        function addServiceToBill(name, price, isCustom = false) {
            if (!name || !name.trim()) { showCustomAlert("Service name cannot be empty.", "error"); return; }
            if (isNaN(price) || price < 0) { showCustomAlert("Service price must be a non-negative number.", "error"); return; }
            currentBillItemsArray.push({ name: name.trim(), price: parseFloat(price), isCustom });
            renderCurrentBill();
            if (isCustom) { customServiceNameEl.value = ''; customServicePriceEl.value = ''; }
        }

        function removeServiceFromBill(index) {
            currentBillItemsArray.splice(index, 1);
            renderCurrentBill();
        }
        
        function updateItemPrice(index, newPrice) {
            if (index >= 0 && index < currentBillItemsArray.length) {
                const priceValue = parseFloat(newPrice);
                if (!isNaN(priceValue) && priceValue >= 0) currentBillItemsArray[index].price = priceValue;
                else showCustomAlert("Invalid price. Use non-negative number.", "error", 2000);
                renderCurrentBill(); 
            }
        }

        function renderCurrentBill() {
            currentBillItemsEl.innerHTML = '';
            let subtotal = 0;
            if (currentBillItemsArray.length === 0) {
                currentBillItemsEl.innerHTML = '<p class="text-gray-500 italic">No items added yet.</p>';
            } else {
                currentBillItemsArray.forEach((item, index) => {
                    const itemDiv = document.createElement('div');
                    itemDiv.className = 'flex justify-between items-center p-2 bg-white border border-gray-200 rounded-md';
                    const priceInput = document.createElement('input');
                    priceInput.type = 'number'; priceInput.className = 'price-input ml-2';
                    priceInput.value = item.price.toFixed(2); priceInput.min = "0"; priceInput.step = "0.01"; 
                    priceInput.dataset.index = index;
                    priceInput.addEventListener('change', (e) => updateItemPrice(index, e.target.value));
                    priceInput.addEventListener('input', (e) => updateItemPrice(index, e.target.value));
                    const priceDisplayPrint = document.createElement('span');
                    priceDisplayPrint.className = 'price-display-print'; priceDisplayPrint.textContent = `₹${item.price.toFixed(2)}`;
                    const itemDetails = document.createElement('span'); itemDetails.textContent = `${item.name} - ₹`;
                    const priceWrapper = document.createElement('span');
                    priceWrapper.appendChild(itemDetails); priceWrapper.appendChild(priceInput); priceWrapper.appendChild(priceDisplayPrint);
                    const removeButton = document.createElement('button');
                    removeButton.className = 'remove-item-btn btn btn-danger btn-xs py-1 px-2 ml-2';
                    removeButton.dataset.index = index; removeButton.innerHTML = '&times; Remove';
                    removeButton.addEventListener('click', (e) => removeServiceFromBill(parseInt(e.target.closest('.remove-item-btn').dataset.index)));
                    itemDiv.appendChild(priceWrapper); itemDiv.appendChild(removeButton);
                    currentBillItemsEl.appendChild(itemDiv);
                    subtotal += item.price; 
                });
            }
            currentBillSubtotalEl.textContent = subtotal.toFixed(2);
            let discountApplied = 0;
            const discountPercent = parseFloat(discountPercentEl.value) || 0;
            const discountFixed = parseFloat(discountFixedEl.value) || 0;
            if (discountPercent > 0 && discountPercent <= 100) discountApplied = (subtotal * discountPercent) / 100;
            else if (discountFixed > 0) discountApplied = discountFixed;
            discountApplied = Math.min(discountApplied, subtotal); 
            currentBillDiscountAppliedEl.textContent = discountApplied.toFixed(2);
            currentBillGrandTotalEl.textContent = (subtotal - discountApplied).toFixed(2);
        }

        function clearBill() {
            customerNameEl.value = ''; customerMobileEl.value = ''; currentBillItemsArray = [];
            discountPercentEl.value = ''; discountFixedEl.value = '';
            customServiceNameEl.value = ''; customServicePriceEl.value = '';
            renderCurrentBill(); 
        }

        async function finalizeBill() {
            const customerName = customerNameEl.value.trim();
            if (!customerName) { showCustomAlert("Please enter customer name."); customerNameEl.focus(); return; }
            if (currentBillItemsArray.length === 0) { showCustomAlert("Please add services to the bill."); return; }
            for(const item of currentBillItemsArray) {
                if (isNaN(item.price) || item.price < 0) { showCustomAlert(`Invalid price for item: ${item.name}.`, "error"); return; }
            }
            if (!db) { showCustomAlert("Database not initialized. Cannot save bill."); return; }
            showLoading();
            const billNumber = `SBT-${Date.now()}`;
            const billData = {
                billNumber, customerName, customerMobile: customerMobileEl.value.trim(),
                items: [...currentBillItemsArray], 
                subtotal: parseFloat(currentBillSubtotalEl.textContent),
                discountInputPercent: parseFloat(discountPercentEl.value) || 0,
                discountInputFixed: parseFloat(discountFixedEl.value) || 0,
                discountApplied: parseFloat(currentBillDiscountAppliedEl.textContent),
                grandTotal: parseFloat(currentBillGrandTotalEl.textContent),
                billedBy: userId || 'anonymous', salonDetails, timestamp: serverTimestamp() 
            };
            try {
                const docRef = await addDoc(collection(db, `artifacts/${appId}/public/data/bills`), billData);
                const previewData = {...billData, timestamp: { toDate: () => new Date() } };
                populateBillPreview(previewData, docRef.id); 
                billPreviewModal.classList.remove('hidden');
                showCustomAlert("Bill finalized and saved successfully!", "success");
            } catch (error) {
                console.error("Error saving bill: ", error);
                showCustomAlert(`Failed to save bill: ${error.message}`);
            } finally {
                hideLoading();
            }
        }
        
        function populateBillPreview(billData, billDocId = null) { 
            const itemsHtml = (billData.items && Array.isArray(billData.items)) ? billData.items.map((item, index) => `
                <div class="item-list-item">
                    <span class="item-name">${index + 1}. ${item.name}</span>
                    <span class="item-price">₹${parseFloat(item.price).toFixed(2)}</span>
                </div>`).join('') : '<p>No items found.</p>';
            billPreviewArea.innerHTML = `
                <div class="salon-details"><h3>${billData.salonDetails.name}</h3><p>${billData.salonDetails.type}</p><p>${billData.salonDetails.address}</p>${billData.salonDetails.phone ? `<p>${billData.salonDetails.phone}</p>` : ''}</div><hr>
                <div class="customer-info" style="display: flex; justify-content: space-between; margin-bottom: 5px;"><p style="text-align: left;"><strong>Bill:</strong> ${billData.billNumber}</p><p style="text-align: right;"><strong>Date:</strong> ${new Date(billData.timestamp?.seconds ? billData.timestamp.toDate() : Date.now()).toLocaleDateString()}</p></div>
                <div class="customer-info" style="margin-bottom: 5px;"><p><strong>Cust:</strong> ${billData.customerName} ${billData.customerMobile ? `(${billData.customerMobile})` : ''}</p></div><hr>
                <div class="item-list-header"><span>Service</span><span>Price</span></div><div class="item-list-body">${itemsHtml}</div><hr>
                <div class="totals-summary"><p>Subtotal: ₹${billData.subtotal.toFixed(2)}</p>${billData.discountApplied > 0 ? `<p>Discount: - ₹${billData.discountApplied.toFixed(2)}</p>` : ''}<p class="grand-total">TOTAL: ₹${billData.grandTotal.toFixed(2)}</p></div>
                <p class="footer-thankyou">Thank you for visiting!</p>`;
            downloadPdfBtn.billDataForPdf = billData; 
        }

        function printCurrentBill() { window.print(); }

        async function downloadCurrentBillAsPdf() {
            const billData = downloadPdfBtn.billDataForPdf; 
            if (!billData) { showCustomAlert("Bill data not found for PDF generation."); return; }
            showLoading();
            try {
                const { jsPDF } = window.jspdf;
                const thermalWidthPt = 210; 
                const originalMaxWidth = billPreviewArea.style.maxWidth;
                const originalModalPadding = billPreviewModal.querySelector('.bg-white').style.padding; 
                const originalPreviewPadding = billPreviewArea.style.padding;
                
                billPreviewArea.style.maxWidth = `${thermalWidthPt}pt`;
                billPreviewArea.style.padding = '5pt';
                billPreviewModal.querySelector('.bg-white').style.padding = '0';

                const canvas = await html2canvas(billPreviewArea, { scale: 2, width: thermalWidthPt, windowWidth: thermalWidthPt, useCORS: true });
                
                billPreviewArea.style.maxWidth = originalMaxWidth;
                billPreviewArea.style.padding = originalPreviewPadding;
                billPreviewModal.querySelector('.bg-white').style.padding = originalModalPadding;

                const imgData = canvas.toDataURL('image/png');
                const imgProps = new Image(); 
                imgProps.onload = function() {
                    const naturalWidth = this.naturalWidth, naturalHeight = this.naturalHeight;
                    const pdfContentWidth = thermalWidthPt - 10; 
                    const pdfImgHeight = (naturalHeight * pdfContentWidth) / naturalWidth;
                    const pdfPageHeight = pdfImgHeight + 10; 
                    const pdf = new jsPDF({ orientation: 'p', unit: 'pt', format: [thermalWidthPt, pdfPageHeight] });
                    pdf.addImage(imgData, 'PNG', 5, 5, pdfContentWidth, pdfImgHeight); 
                    pdf.save(`SBT-Lounge-Bill-${billData.billNumber}.pdf`);
                    hideLoading();
                }
                imgProps.onerror = function() { showCustomAlert("Error processing image for PDF."); hideLoading(); }
                imgProps.src = imgData; 
            } catch (error) {
                console.error("Error generating PDF:", error);
                showCustomAlert("Could not generate PDF. " + error.message);
                hideLoading();
            }
        }

        let allPastBillsData = []; 
        async function loadPastBills() {
            if (!db) return;
            showLoading();
            const q = query(collection(db, `artifacts/${appId}/public/data/bills`)); 
            onSnapshot(q, (querySnapshot) => {
                allPastBillsData = [];
                querySnapshot.forEach((doc) => allPastBillsData.push({ id: doc.id, ...doc.data() }));
                allPastBillsData.sort((a, b) => (b.timestamp?.seconds || 0) - (a.timestamp?.seconds || 0));
                renderPastBills(allPastBillsData);
                hideLoading();
            }, (error) => {
                console.error("Error fetching past bills: ", error);
                showCustomAlert(`Error fetching past bills: ${error.message}`);
                hideLoading();
            });
        }
        
        function renderPastBills(billsToRender) {
            pastBillsListEl.innerHTML = '';
            if (billsToRender.length === 0) {
                pastBillsListEl.innerHTML = '<tr><td colspan="5" class="text-center p-4 text-gray-500">No past bills found.</td></tr>'; return;
            }
            billsToRender.forEach(bill => {
                const tr = document.createElement('tr');
                tr.className = "hover:bg-gray-50 transition-colors duration-150";
                const billDate = bill.timestamp?.seconds ? new Date(bill.timestamp.seconds * 1000).toLocaleDateString() : 'N/A';
                tr.innerHTML = `
                    <td class="whitespace-nowrap">${bill.billNumber}</td>
                    <td class="whitespace-nowrap"><span class="clickable-customer" data-name="${bill.customerName}" data-mobile="${bill.customerMobile || ''}">${bill.customerName}</span></td>
                    <td class="whitespace-nowrap text-right">₹${(bill.grandTotal !== undefined ? bill.grandTotal : 0).toFixed(2)}</td>
                    <td class="whitespace-nowrap">${billDate}</td>
                    <td><button class="view-bill-btn btn btn-outline btn-xs py-1 px-2" data-bill-id="${bill.id}">View</button></td>`;
                pastBillsListEl.appendChild(tr);
            });
            document.querySelectorAll('.view-bill-btn').forEach(button => {
                button.addEventListener('click', (e) => {
                    const billId = e.target.closest('.view-bill-btn').dataset.billId;
                    const billData = allPastBillsData.find(b => b.id === billId);
                    if (billData) { populateBillPreview(billData, billId); billPreviewModal.classList.remove('hidden'); } 
                    else showCustomAlert("Could not find bill details.");
                });
            });
            document.querySelectorAll('.clickable-customer').forEach(span => {
                span.addEventListener('click', (e) => {
                    const target = e.target.closest('.clickable-customer');
                    customerNameEl.value = target.dataset.name; customerMobileEl.value = target.dataset.mobile;
                    showCustomAlert(`Customer: ${target.dataset.name} pre-filled.`, "success", 2000);
                    customerNameEl.focus(); 
                });
            });
        }

        // Standard Event Listeners
        searchPastBillsEl.addEventListener('input', (e) => {
            const searchTerm = e.target.value.toLowerCase();
            if (!searchTerm) { renderPastBills(allPastBillsData); return; }
            const filteredBills = allPastBillsData.filter(bill => 
                bill.customerName.toLowerCase().includes(searchTerm) || bill.billNumber.toLowerCase().includes(searchTerm) ||
                (bill.customerMobile && bill.customerMobile.toLowerCase().includes(searchTerm))
            );
            renderPastBills(filteredBills);
        });
        clearSearchBtn.addEventListener('click', () => { searchPastBillsEl.value = ''; renderPastBills(allPastBillsData); });
        discountPercentEl.addEventListener('input', () => { if (discountPercentEl.value) discountFixedEl.value = ''; renderCurrentBill(); });
        discountFixedEl.addEventListener('input', () => { if (discountFixedEl.value) discountPercentEl.value = ''; renderCurrentBill(); });
        addCustomServiceBtn.addEventListener('click', () => addServiceToBill(customServiceNameEl.value, customServicePriceEl.value, true));
        finalizeBillBtn.addEventListener('click', finalizeBill);
        clearBillBtn.addEventListener('click', clearBill);
        printBillBtn.addEventListener('click', printCurrentBill);
        downloadPdfBtn.addEventListener('click', downloadCurrentBillAsPdf);
        closePreviewBtn.addEventListener('click', () => { billPreviewModal.classList.add('hidden'); clearBill(); });

        // Initial Load
        document.addEventListener('DOMContentLoaded', () => { initializeFirebase(); renderCurrentBill(); });
    </script>
</body>
</html>
