<script>
    import { onMount } from "svelte";
    import { Toast, confirmAlert, hideLoadingModal, showLoadingModal } from "../utils/toast.js";
    import { urlApi } from "../utils/constant.js";

    let currentPage = 1;
    const perPage = 10;
    let totalPages = 0;
    let selectedRequest;
    let reason;

    let requests = [];
    $: paginatedRequests = requests.slice(
        (currentPage - 1) * perPage,
        currentPage * perPage,
    );

    async function fetchRequest() {
        try {
            const token = localStorage.getItem("token");

            const response = await fetch(`${urlApi}/request`, {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${token || ""}`,
                },
            });
            const resp = await response.json();

            if (resp.status) {
                requests = resp.data;
                totalPages = Math.ceil(requests.length / perPage);
            } else {
                Toast.error(
                    resp.message || "Échec du chargement des demandes.",
                );
            }
        } catch (error) {
            console.error("Erreur lors du chargement des demandes :", error);
            Toast.error("Impossible de charger les demandes.");
        }
    }

    onMount(async () => {
        fetchRequest();
    });

    async function deleteRequest(event, requestId) {
        event.preventDefault();

        const result = await confirmAlert({
            title: "Supprimer la demande ?",
            text: "Cette action est irréversible !",
            confirmButtonText: "Oui, supprimer",
            cancelButtonText: "Annuler",
        });

        if (result) {
            showLoadingModal();
            try {
                const response = await fetch(`${urlApi}/request/${requestId}`, {
                    method: "DELETE",
                    headers: {
                        "Content-Type": "application/json",
                    },
                });

                const resp = await response.json();

                if (resp.status) {
                    hideLoadingModal();
                    Toast.success("Demande supprimé avec succès !");
                    fetchRequest();
                } else {
                    hideLoadingModal();
                    Toast.error(
                        resp.message ||
                            "Échec de la suppression de la demande.",
                    );
                }
            } catch (error) {
                hideLoadingModal();
                console.error(
                    "Erreur lors de la suppression de la demande :",
                    error,
                );
                Toast.error("Impossible de supprimer la demande.");
            }
        }
    }

    async function confirmRequest(event) {
        event.preventDefault();
        showLoadingModal();
        try {
            const token = localStorage.getItem("token");

            const response = await fetch(
                `${urlApi}/request/${selectedRequest.id}/confirm`,
                {
                    method: "PUT",
                    headers: {
                        "Content-Type": "application/json",
                        Authorization: `Bearer ${token || ""}`,
                    },
                },
            );
            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                Toast.success("Demande confirmée avec succès !");
                detail_modal.close();
                fetchRequest();
            } else {
                hideLoadingModal();
                Toast.error(
                    resp.message || "Échec de la confirmation de la demande.",
                );
            }
        } catch (error) {
            hideLoadingModal();
            console.error(
                "Erreur lors de la confirmation de la demande :",
                error,
            );
            Toast.error("Impossible de confirmer la demande.");
        }
    }

    async function refuseRequest(event) {
        event.preventDefault();
        showLoadingModal();
        try {
            const token = localStorage.getItem("token");

            const response = await fetch(
                `${urlApi}/request/${selectedRequest.id}/refuse`,
                {
                    method: "PUT",
                    headers: {
                        "Content-Type": "application/json",
                        Authorization: `Bearer ${token || ""}`,
                    },
                    body: JSON.stringify({ reason })
                },
            );
            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                Toast.success("Demande rejetée avec succès !");
                reason_modal.close();
                fetchRequest();
            } else {
                hideLoadingModal();
                Toast.error(resp.message || "Échec du rejet de la demande.");
            }
        } catch (error) {
            hideLoadingModal();
            console.error("Erreur lors du rejet de la demande :", error);
            Toast.error("Impossible de rejeter la demande.");
        }
    }
</script>

<div>
    <h1 class="text-2xl font-bold mb-4">Demandes</h1>
</div>

<div class="breadcrumbs text-sm mt-2 mb-5">
    <ul>
        <li>J-Librairie</li>
        <li>Demandes</li>
    </ul>
</div>

<div class="shadow-sm rounded-box border border-base-content/5 bg-base-100 p-4">
    <h2 class="font-semibold text-lg mb-3">Liste des demandes</h2>
    <div
        class="overflow-x-auto rounded-box border border-base-content/5 bg-base-100"
    >
        <table class="table">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Demande de</th>
                    <th>Type de compte</th>
                    <th>Reponse</th>
                    <th>Status</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody>
                {#each paginatedRequests as request, index}
                    <tr class="hover:bg-gray-50 transition">
                        <th>{(currentPage - 1) * perPage + index + 1}</th>
                        <td>{request.user.name} {request.user.surname}</td>
                        <td>{request.type}</td>
                        <td>
                            {#if request.reponse?.split(" ").length > 7}
                                {request.reponse
                                    .split(" ")
                                    .slice(0, 7)
                                    .join(" ")}...
                            {:else}
                                {request.reponse}
                            {/if}
                        </td>
                        <td>
                            {#if request.status.name === "PENDING"}
                                <span class="badge badge-soft badge-warning"
                                    >En attente</span
                                >
                            {:else if request.status.name === "CONFIRM"}
                                <span class="badge badge-soft badge-success">Confirmé</span
                                >
                            {:else if request.status.name === "REFUSE"}
                                <span class="badge badge-soft badge-error">Rejeté</span>
                            {/if}
                        </td>
                        <td>
                            <button
                                class="btn btn-sm btn-success btn-outline"
                                on:click={() => {
                                    selectedRequest = request;
                                    detail_modal.showModal();
                                }}>Consulter</button
                            >
                            <button
                                class="btn btn-sm btn-error btn-outline"
                                on:click={(event) => {
                                    deleteRequest(event, request.id);
                                }}>Supprimer</button
                            >
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

<dialog id="detail_modal" class="modal">
    <div class="modal-box max-w-md bg-base-100 shadow-xl">
        <form method="dialog">
            <button
                class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            >
                ✕
            </button>
        </form>

        {#if selectedRequest}
            <h3 class="text-xl font-semibold mb-3">
                Demande de <span class="text-primary"
                    >{selectedRequest.user.name}
                    {selectedRequest.user.surname}</span
                >
            </h3>

            <div class="divider my-2"></div>

            <p class="py-1">
                <span class="font-medium text-gray-700"
                    >Type de compte vendeur :</span
                >
                <span class="badge badge-outline badge-primary ml-1"
                    >{selectedRequest.type}</span
                >
            </p>

            <div class="mt-4">
                <p class="font-medium text-gray-700 mb-3">
                    Réponse aux questions :
                </p>
                <div
                    class="bg-base-300 rounded-lg py-3 px-3 text-sm leading-relaxed"
                >
                    {selectedRequest.reponse}
                </div>
            </div>

            {#if selectedRequest.status.name !== "CONFIRM" && selectedRequest.status.name !== "REFUSE"}
                <div class="flex justify-end mt-6 gap-3">
                    <button
                        class="btn btn-outline btn-error"
                        on:click={() => {
                            detail_modal.close();
                            reason_modal.showModal();
                        }}
                    >
                        Rejeter
                    </button>
                    <button class="btn btn-primary" on:click={confirmRequest}>
                        Confirmer
                    </button>
                </div>
            {:else}
                <div class="mt-6">
                    <p class="font-medium text-gray-600">
                        Statut actuel :
                        <span
                            class={`badge badge-soft ${
                                selectedRequest.status.name === "CONFIRM"
                                    ? "badge-success"
                                    : "badge-error"
                            }`}
                        >
                            {selectedRequest.status.name}
                        </span>
                    </p>
                </div>
            {/if}
        {/if}
    </div>
</dialog>

<dialog id="reason_modal" class="modal">
    <div class="modal-box max-w-md">
        <!-- max-w-md = largeur du modal -->
        <form method="dialog">
            <button
                class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            >
                ✕
            </button>
        </form>

        {#if selectedRequest}
            <h3 class="text-lg font-bold">
                Rejet de la demande de {selectedRequest.user.name}
                {selectedRequest.user.surname}
            </h3>
        {/if}
        <p class="py-4">Veuillez fournir les raisons du rejet.</p>

        <fieldset class="fieldset w-full space-y-3">
            <div>
                <label class="label mb-1">Raisons</label>
                <textarea class="textarea w-full" bind:value={reason}
                ></textarea>
            </div>

            <button class="btn btn-primary mt-4 w-full" on:click={refuseRequest}
                >Valider</button
            >
        </fieldset>
    </div>
</dialog>