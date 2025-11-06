<script>
    import { onMount } from "svelte";
    import { Toast } from "../utils/toast";
    import { urlApi } from "../utils/constant";

    const date = new Date();
    const today = date.toLocaleDateString("fr-FR", {
        year: "numeric",
        month: "long",
        day: "numeric",
    });

    let payments = [];
    let requests = [];
    let books = [];
    let users = [];
    let transactions = [];

    let totalBooks = 0;
    let totalUsers = 0;
    let totalRequestsPending = 0;
    let totalDownloads = 0;
    let totalTransactions = 0;
    let totalRevenue = 0;
    let requestsConfirmed = 0;
    let requestsRefused = 0;
    let booksByCategory = [];

    async function fetchData() {
        const token = localStorage.getItem("token");

        try {
            // Fetch books
            const booksResp = await fetch(`${urlApi}/book`, {
                headers: { Authorization: `Bearer ${token || ""}` },
            });
            const booksData = await booksResp.json();
            if (booksData.status) {
                books = booksData.data;
                totalBooks = books.length;
                totalDownloads = books.reduce(
                    (acc, book) => acc + (book.download || 0),
                    0,
                );

                // Books by category
                const categoryMap = {};
                books.forEach((book) => {
                    const cat = book.category?.name || "Autres";
                    categoryMap[cat] = (categoryMap[cat] || 0) + 1;
                });
                booksByCategory = Object.entries(categoryMap).map(
                    ([name, count]) => ({ name, count }),
                );
            }

            // Fetch users
            const usersResp = await fetch(`${urlApi}/user`, {
                headers: { Authorization: `Bearer ${token || ""}` },
            });
            const usersData = await usersResp.json();
            if (usersData.status) {
                users = usersData.data.filter((u) => u.role?.name !== "ADMIN");
                totalUsers = users.length;
            }

            // Fetch requests
            const requestsResp = await fetch(`${urlApi}/request`, {
                headers: { Authorization: `Bearer ${token || ""}` },
            });
            const requestsData = await requestsResp.json();
            if (requestsData.status) {
                requests = requestsData.data;
                totalRequestsPending = requests.filter(
                    (r) => r.status?.name === "PENDING",
                ).length;
                requestsConfirmed = requests.filter(
                    (r) => r.status?.name === "CONFIRM",
                ).length;
                requestsRefused = requests.filter(
                    (r) => r.status?.name === "REFUSE",
                ).length;
            }

            // Fetch transactions
            const transactionsResp = await fetch(`${urlApi}/transaction`, {
                headers: { Authorization: `Bearer ${token || ""}` },
            });
            const transactionsData = await transactionsResp.json();
            if (transactionsData.status) {
                transactions = transactionsData.data;
                totalTransactions = transactions.length;
                totalRevenue = transactions.reduce(
                    (acc, t) => acc + (t.book?.price || 0),
                    0,
                );
            }
        } catch (error) {
            console.error("Erreur lors du chargement des données :", error);
            Toast.error(
                "Impossible de charger les données du tableau de bord.",
            );
        }
    }

    onMount(() => {
        fetchData();
    });
</script>

<!-- Header -->
<div class="mb-6">
    <h1 class="text-2xl font-bold text-base-content">Tableau de bord</h1>
    <!-- <p class="text-base-content/60 mt-1">{today}</p> -->
</div>

<!-- Cartes statistiques principales -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
    <!-- Livres téléchargés -->
    <div class="card bg-base-100 shadow-lg hover:shadow-xl transition-shadow">
        <div class="card-body">
            <div class="flex items-center justify-between">
                <div>
                    <p class="text-base-content/60 text-sm font-medium">Téléchargements</p>
                    <h3 class="text-3xl font-bold mt-2 text-primary">{totalDownloads}</h3>
                    <p class="text-xs text-base-content/50 mt-2">Total des téléchargements</p>
                </div>
                <div class="bg-primary/10 p-3 rounded-xl">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M7 16a4 4 0 01-.88-7.903A5 5 0 1115.9 6L16 6a5 5 0 011 9.9M9 19l3 3m0 0l3-3m-3 3V10" />
                    </svg>
                </div>
            </div>
        </div>
    </div>

    <!-- Utilisateurs actifs -->
    <div class="card bg-base-100 shadow-lg hover:shadow-xl transition-shadow">
        <div class="card-body">
            <div class="flex items-center justify-between">
                <div>
                    <p class="text-base-content/60 text-sm font-medium">Utilisateurs</p>
                    <h3 class="text-3xl font-bold mt-2 text-primary">{totalUsers}</h3>
                    <p class="text-xs mt-2 text-primary">
                        <span class="font-semibold">up {Math.floor(totalUsers * 0.1)}%</span> ce mois-ci
                    </p>
                </div>
                <div class="bg-primary/10 p-3 rounded-xl">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M17 20h5v-2a3 3 0 00-5.356-1.857M17 20H7m10 0v-2c0-.656-.126-1.283-.356-1.857M7 20H2v-2a3 3 0 015.356-1.857M7 20v-2c0-.656.126-1.283.356-1.857m0 0a5.002 5.002 0 019.288 0M15 7a3 3 0 11-6 0 3 3 0 016 0zm6 3a2 2 0 11-4 0 2 2 0 014 0zM7 10a2 2 0 11-4 0 2 2 0 014 0z" />
                    </svg>
                </div>
            </div>
        </div>
    </div>

    <!-- Total de livres -->
    <div class="card bg-base-100 shadow-lg hover:shadow-xl transition-shadow">
        <div class="card-body">
            <div class="flex items-center justify-between">
                <div>
                    <p class="text-base-content/60 text-sm font-medium">Catalogue</p>
                    <h3 class="text-3xl font-bold mt-2 text-primary">{totalBooks}</h3>
                    <p class="text-xs text-base-content/50 mt-2">Livres disponibles</p>
                </div>
                <div class="bg-primary/10 p-3 rounded-xl">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M12 6.253v13m0-13C10.832 5.477 9.246 5 7.5 5S4.168 5.477 3 6.253v13C4.168 18.477 5.754 18 7.5 18s3.332.477 4.5 1.253m0-13C13.168 5.477 14.754 5 16.5 5c1.747 0 3.332.477 4.5 1.253v13C19.832 18.477 18.247 18 16.5 18c-1.746 0-3.332.477-4.5 1.253" />
                    </svg>
                </div>
            </div>
        </div>
    </div>

    <!-- Demandes en attente -->
    <div class="card bg-base-100 shadow-lg hover:shadow-xl transition-shadow">
        <div class="card-body">
            <div class="flex items-center justify-between">
                <div>
                    <p class="text-base-content/60 text-sm font-medium">En attente</p>
                    <h3 class="text-3xl font-bold mt-2 text-primary">{totalRequestsPending}</h3>
                    <p class="text-xs mt-2 text-primary">
                        <span class="font-semibold">up {Math.floor(totalRequestsPending * 0.08)}%</span> ce mois-ci
                    </p>
                </div>
                <div class="bg-primary/10 p-3 rounded-xl">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-8 w-8" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
                        <path stroke-linecap="round" stroke-linejoin="round" d="M12 8v4l3 3m6-3a9 9 0 11-18 0 9 9 0 0118 0z" />
                    </svg>
                </div>
            </div>
        </div>
    </div>
</div>

<!-- Section statistiques détaillées -->
<div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
    <!-- Revenus et transactions -->
    <div class="card bg-base-100 shadow-lg">
        <div class="card-body">
            <h2 class="card-title text-lg mb-4">Statistiques financières</h2>
            <div class="space-y-4">
                <div class="flex items-center justify-between p-4 bg-base-200 rounded-lg">
                    <div class="flex items-center gap-3">
                        <div class="bg-green-100 p-2 rounded-lg">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-green-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
                            </svg>
                        </div>
                        <div>
                            <p class="text-sm text-base-content/60">Revenus totaux</p>
                            <p class="text-2xl font-bold">{totalRevenue.toLocaleString()} FCFA</p>
                        </div>
                    </div>
                </div>
                <div class="flex items-center justify-between p-4 bg-base-200 rounded-lg">
                    <div class="flex items-center gap-3">
                        <div class="bg-blue-100 p-2 rounded-lg">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-blue-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-3 7h3m-3 4h3m-6-4h.01M9 16h.01" />
                            </svg>
                        </div>
                        <div>
                            <p class="text-sm text-base-content/60">Transactions</p>
                            <p class="text-2xl font-bold">{totalTransactions}</p>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <!-- Statut des demandes -->
    <div class="card bg-base-100 shadow-lg">
        <div class="card-body">
            <h2 class="card-title text-lg mb-4">État des demandes</h2>
            <div class="space-y-3">
                <!-- En attente -->
                <div class="flex items-center justify-between">
                    <div class="flex items-center gap-3">
                        <div class="badge badge-soft badge-warning badge-lg">En attente</div>
                        <span class="font-semibold">{totalRequestsPending}</span>
                    </div>
                    <div class="w-32">
                        <progress class="progress progress-warning" value={totalRequestsPending} max={requests.length}></progress>
                    </div>
                </div>

                <!-- Confirmées -->
                <div class="flex items-center justify-between">
                    <div class="flex items-center gap-3">
                        <div class="badge badge-soft badge-success badge-lg">Confirmées</div>
                        <span class="font-semibold">{requestsConfirmed}</span>
                    </div>
                    <div class="w-32">
                        <progress class="progress progress-success" value={requestsConfirmed} max={requests.length}></progress>
                    </div>
                </div>

                <!-- Refusées -->
                <div class="flex items-center justify-between">
                    <div class="flex items-center gap-3">
                        <div class="badge badge-soft badge-error badge-lg">Refusées</div>
                        <span class="font-semibold">{requestsRefused}</span>
                    </div>
                    <div class="w-32">
                        <progress class="progress progress-error" value={requestsRefused} max={requests.length}></progress>
                    </div>
                </div>

                <!-- Total -->
                <div class="divider my-2"></div>
                <div class="flex items-center justify-between font-bold">
                    <span>Total</span>
                    <span>{requests.length} demandes</span>
                </div>
            </div>
        </div>
    </div>
</div>

<!-- Livres par catégorie -->
<div class="card bg-base-100 shadow-lg">
    <div class="card-body">
        <h2 class="card-title text-lg mb-4">Répartition par catégorie</h2>
        {#if booksByCategory.length > 0}
            <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                {#each booksByCategory as category, index}
                    <div class="flex items-center justify-between p-4 bg-base-200 rounded-lg hover:shadow-md transition-shadow">
                        <div class="flex items-center gap-3">
                            <div class="avatar placeholder">
                                <div class="w-10 h-10 rounded-lg bg-primary text-white flex items-center justify-center">
                                    <span class="text-lg font-bold">{category.count}</span>
                                </div>                                
                            </div>
                            <div> 
                                <p class="font-semibold">{category.name}</p>
                                <p class="text-sm text-base-content/60">
                                    {category.count} livre{category.count > 1 ? 's' : ''}
                                </p>
                            </div>
                        </div>
                        <div class="text-right">
                            <p class="text-sm font-semibold text-primary">
                                {((category.count / totalBooks) * 100).toFixed(1)}%
                            </p>
                        </div>
                    </div>
                {/each}
            </div>
        {:else}
            <div class="text-center py-8 text-base-content/60">
                <p>Aucune catégorie disponible</p>
            </div>
        {/if}
    </div>
</div>