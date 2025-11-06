<script>
    import { onMount } from "svelte";
    import { Toast } from "../utils/toast.js";
    import { urlApi } from "../utils/constant.js";

    let currentPage = 1;
    const perPage = 10;
    let totalPages = 0;

    let payments = [];
    $: paginatedPayments = payments.slice(
        (currentPage - 1) * perPage,
        currentPage * perPage,
    );

    async function fetchPayment() {
        try {
            const token = localStorage.getItem("token");

            const response = await fetch(`${urlApi}/payment`, {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${token || ""}`,
                },
            });
            const resp = await response.json();
            console.log(resp);

            if (resp.status) {
                payments = resp.data;
                totalPages = Math.ceil(payments.length / perPage);
            } else {
                Toast.error(
                    resp.message || "Échec du chargement des paiements.",
                );
            }
        } catch (error) {
            console.error("Erreur lors du chargement des paiements :", error);
            Toast.error("Impossible de charger les paiements.");
        }
    }

    onMount(async () => {
        fetchPayment();
    });
</script>

<div>
    <h1 class="text-2xl font-bold mb-4">Livres</h1>
</div>

<div class="breadcrumbs text-sm mt-2 mb-5">
    <ul>
        <li>J-Librairie</li>
        <li>Livres</li>
    </ul>
</div>

<div class="shadow-sm rounded-box border border-base-content/5 bg-base-100 p-4">
    <h2 class="font-semibold text-lg mb-3">Liste des paiements</h2>
    <div
        class="overflow-x-auto rounded-box border border-base-content/5 bg-base-100"
    >
        <table class="table">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Titre du livre</th>
                    <th>Prix du livre</th>
                    <th>Efféctué par</th>
                    <th>Status</th>
                </tr>
            </thead>
            <tbody>
                {#each paginatedPayments as payment, index}
                    <tr class="hover:bg-gray-50 transition">
                        <th>{(currentPage - 1) * perPage + index + 1}</th>
                        <td>{payment.book.title}</td>
                        <td>{payment.book.price}</td>
                        <td>{payment.user.name} {payment.user.surname}</td>
                        <td>
                            <span
                                class={payment.status === "approved"
                                    ? "badge badge-soft badge-success font-semibold"
                                    : payment.status === "pending"
                                      ? "badge badge-soft badge-warning font-semibold"
                                      : "badge badge-soft badge-error font-semibold"}
                            >
                                {payment.status}
                            </span>
                        </td>
                    </tr>
                {/each}
            </tbody>
        </table>
    </div>

    <!-- PAGINATION -->
    <div class="flex justify-end items-center mt-4 gap-2">
        <button
            class="btn btn-sm"
            on:click={() => (currentPage = Math.max(1, currentPage - 1))}
            disabled={currentPage === 1}
        >
            «
        </button>

        {#each Array(totalPages) as _, i}
            <button
                class="btn btn-sm {currentPage === i + 1
                    ? 'btn-active btn-primary'
                    : 'btn-ghost'}"
                on:click={() => (currentPage = i + 1)}
            >
                {i + 1}
            </button>
        {/each}

        <button
            class="btn btn-sm"
            on:click={() =>
                (currentPage = Math.min(totalPages, currentPage + 1))}
            disabled={currentPage === totalPages}
        >
            »
        </button>
    </div>
</div>
