<script>
    import { onMount } from "svelte";
    import { confirmAlert, hideLoadingModal, showLoadingModal, Toast } from "../utils/toast.js";
    import { urlApi } from "../utils/constant.js";

    let currentPage = 1;
    const perPage = 10;
    let totalPages = 0;

    let books = [];
    $: paginatedBooks = books.slice(
        (currentPage - 1) * perPage,
        currentPage * perPage,
    );

    async function fetchBooks() {
        try {
            const response = await fetch(`${urlApi}/book/adminBook`, {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                },
            });

            const resp = await response.json();

            if (resp.status) {
                books = resp.data;
                totalPages = Math.ceil(books.length / perPage);
            } else {
                Toast.error(resp.message || "Échec du chargement des livres.");
            }
        } catch (error) {
            console.error("Erreur lors du chargement des livres :", error);
            Toast.error("Impossible de charger les livres.");
        }
    }

    onMount(async () => {
        fetchBooks();
    });

    async function deleteBook(event, bookId) {
        event.preventDefault();
        
        try {
            const confirmed = await confirmAlert({
                title: "Supprimer le livre ?",
                text: "Cette action est irréversible !",
                confirmButtonText: "Oui, supprimer",
                cancelButtonText: "Annuler",
            });
            if (!confirmed) return;
            showLoadingModal();

            const token = localStorage.getItem("token");

            const response = await fetch(`${urlApi}/book/${bookId}`, {
                method: "DELETE",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${token || ""}`,
                },
            });

            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                Toast.success("Livre supprimé avec succès.");
                fetchBooks();
            } else {
                hideLoadingModal();
                Toast.error(
                    resp.message || "Échec de la suppression du livre.",
                );
            }
        } catch (error) {
            hideLoadingModal();
            console.error("Erreur lors de la suppression du livre :", error);
            Toast.error("Impossible de supprimer le livre.");
        }
    }
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
    <h2 class="font-semibold text-lg mb-3">Liste des livres</h2>
    <div
        class="overflow-x-auto rounded-box border border-base-content/5 bg-base-100"
    >
        <table class="table">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Image</th>
                    <th>Titre</th>
                    <th>Description</th>
                    <th>Auteur</th>
                    <th>Prix</th>
                    <th>Stock</th>
                    <th>Click</th>
                    <th>Téléc...</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody>
                {#each paginatedBooks as book, index}
                    <tr class="hover:bg-gray-50 transition">
                        <th>{(currentPage - 1) * perPage + index + 1}</th>
                        <td>
                            <img
                                src="{book.image}"
                                alt={book.title}
                                class="w-12 h-12 object-cover rounded"
                            />
                        </td>
                        <td>{book.title}</td>
                        <td>
                            {#if book.description?.split(" ").length > 7}
                                {book.description
                                    .split(" ")
                                    .slice(0, 7)
                                    .join(" ")}...
                            {:else}
                                {book.description}
                            {/if}
                        </td>
                        <td>{book.author}</td>
                        <td>{book.price}</td>
                        <td>{book.stock}</td>
                        <td>{book.click}</td>
                        <td>{book.download}</td>
                        <td>
                            <button
                                class="btn btn-sm btn-error btn-outline"
                                on:click={(event) => deleteBook(event, book.id)}
                                >Supprimer</button
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
