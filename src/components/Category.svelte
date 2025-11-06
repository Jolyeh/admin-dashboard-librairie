<script>
    import { onMount } from "svelte";
    import {
        confirmAlert,
        hideLoadingModal,
        showLoadingModal,
        Toast,
    } from "../utils/toast.js";
    import { urlApi } from "../utils/constant.js";

    let currentPage = 1;
    const perPage = 10;
    let totalPages = 0;

    let name;
    let newName;
    let idCategory;

    let categories = [];
    $: paginatedCategories = categories.slice(
        (currentPage - 1) * perPage,
        currentPage * perPage,
    );

    async function fetchCategories() {
        const token = localStorage.getItem("token");
        try {
            const response = await fetch(`${urlApi}/category`, {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${token || ""}`,
                },
            });

            const resp = await response.json();

            if (resp.status) {
                categories = resp.data;
                totalPages = Math.ceil(categories.length / perPage);
            } else {
                Toast.error(
                    resp.message || "Échec du chargement des catégories.",
                );
            }
        } catch (error) {
            console.error("Erreur lors du chargement des catégories :", error);
            Toast.error("Impossible de charger les catégories.");
        }
    }

    onMount(async () => {
        fetchCategories();
    });

    async function addCategory(event) {
        event.preventDefault();

        try {
            if (!name) {
                Toast.error("Veuillez remplir le champ libellé.");
                return;
            }
            showLoadingModal();
            const token = localStorage.getItem("token");

            const response = await fetch(`${urlApi}/category`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${token || ""}`,
                },
                body: JSON.stringify({ name }),
            });

            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                Toast.success("Catégorie ajoutée avec succès.");
                name = "";
                fetchCategories();
                document.getElementById("add_cat_modal").close();
            } else {
                hideLoadingModal();
                Toast.error(
                    resp.message || "Échec de l'ajout de la catégorie.",
                );
            }
        } catch (error) {
            hideLoadingModal();
            console.error("Erreur lors de l'ajout de la catégorie :", error);
            Toast.error("Impossible d'ajouter la catégorie.");
        }
    }

    async function updateCategory(event, categoryId) {
        event.preventDefault();

        try {
            if (!newName) {
                Toast.error("Veuillez remplir le champ libellé.");
                return;
            }

            showLoadingModal();

            const token = localStorage.getItem("token");

            const response = await fetch(`${urlApi}/category/${categoryId}`, {
                method: "PUT",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${token || ""}`,
                },
                body: JSON.stringify({ name: newName }),
            });

            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                Toast.success("Catégorie mise à jour avec succès.");
                newName = "";
                fetchCategories();
                document.getElementById("update_cat_modal").close();
            } else {
                hideLoadingModal();
                Toast.error(
                    resp.message || "Échec de la mise à jour de la catégorie.",
                );
            }
        } catch (error) {
            hideLoadingModal();
            console.error(
                "Erreur lors de la mise à jour de la catégorie :",
                error,
            );
            Toast.error("Impossible de mettre à jour la catégorie.");
        }
    }

    async function deleteCategory(event, categoryId) {
        event.preventDefault();
        try {
            const confirmed = await confirmAlert({
                title: "Supprimer la catégorie ?",
                text: "Cette action est irréversible !",
                confirmButtonText: "Oui, supprimer",
                cancelButtonText: "Annuler",
            });
            if (!confirmed) return;
            showLoadingModal();
            const token = localStorage.getItem("token");

            const response = await fetch(`${urlApi}/category/${categoryId}`, {
                method: "DELETE",
                headers: {
                    "Content-Type": "application/json",
                    Authorization: `Bearer ${token || ""}`,
                },
            });

            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                Toast.success("Catégorie supprimée avec succès.");
                fetchCategories();
            } else {
                Toast.error(
                    resp.message || "Échec de la suppression de la catégorie.",
                );
            }
        } catch (error) {
            hideLoadingModal();
            console.error(
                "Erreur lors de la suppression de la catégorie :",
                error,
            );
            Toast.error("Impossible de supprimer la catégorie.");
        }
    }
</script>

<div>
    <h1 class="text-2xl font-bold mb-4">Catégories</h1>
</div>

<div class="breadcrumbs text-sm mt-2 mb-5">
    <ul>
        <li>J-Librairie</li>
        <li>Catégories</li>
    </ul>
</div>

<div class="shadow-sm rounded-box border border-base-content/5 bg-base-100 p-4">
    <div class="flex justify-between items-center mb-3">
        <h2 class="font-semibold text-lg">Liste des Catégories</h2>
        <button
            class="btn btn-primary flex items-center gap-2"
            on:click={() => add_cat_modal.showModal()}
        >
            <!-- Texte visible sur écran moyen et plus -->
            <span class="hidden md:inline">Ajouter une catégorie</span>

            <!-- Icône visible uniquement sur mobile -->
            <span class="inline md:hidden text-xl font-bold">+</span>
        </button>
    </div>
    <div
        class="overflow-x-auto rounded-box border border-base-content/5 bg-base-100"
    >
        <table class="table">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Libellé</th>
                    <th>Nbres de livres</th>
                    <th>Nbres de vues</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody>
                {#each paginatedCategories as category, index}
                    <tr class="hover:bg-gray-50 transition">
                        <th>{(currentPage - 1) * perPage + index + 1}</th>

                        <!-- Nom de la catégorie -->
                        <td class="font-medium text-gray-700"
                            >{category.name}</td
                        >

                        <!-- Nombre de livres -->
                        <td class="text-center">{category.books.length}</td>

                        <!-- Total des clics sur tous les livres -->
                        <td class="text-center">
                            {category.books && category.books.length > 0
                                ? category.books.reduce(
                                      (sum, book) => sum + (book.clicks || 0),
                                      0,
                                  )
                                : 0}
                        </td>

                        <!-- Actions -->
                        <td class="flex gap-2">
                            <button
                                class="btn btn-sm btn-outline btn-primary"
                                on:click={() => {
                                    newName = category.name;
                                    idCategory = category.id;
                                    document
                                        .getElementById("update_cat_modal")
                                        .showModal();
                                }}
                            >
                                Modifier
                            </button>

                            <button
                                class="btn btn-sm btn-outline btn-error"
                                on:click={(e) => deleteCategory(e, category.id)}
                            >
                                Supprimer
                            </button>
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

<dialog id="add_cat_modal" class="modal">
    <div class="modal-box max-w-md">
        <!-- max-w-md = largeur du modal -->
        <form method="dialog">
            <button
                class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            >
                ✕
            </button>
        </form>

        <h3 class="text-lg font-bold mb-3">Ajouter une catégorie</h3>

        <fieldset class="fieldset w-full space-y-3">
            <div>
                <label class="label mb-1">libellé</label>
                <input
                    type="text"
                    class="input input-bordered w-full"
                    bind:value={name}
                />
            </div>

            <button class="btn btn-primary mt-4 w-full" on:click={addCategory}
                >Ajouter</button
            >
        </fieldset>
    </div>
</dialog>

<dialog id="update_cat_modal" class="modal">
    <div class="modal-box max-w-md">
        <!-- max-w-md = largeur du modal -->
        <form method="dialog">
            <button
                class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            >
                ✕
            </button>
        </form>

        <h3 class="text-lg font-bold mb-3">Modifier la catégorie</h3>

        <fieldset class="fieldset w-full space-y-3">
            <div>
                <label class="label mb-1">libellé</label>
                <input
                    type="text"
                    class="input input-bordered w-full"
                    bind:value={newName}
                />
            </div>

            <button
                class="btn btn-primary mt-4 w-full"
                on:click={(e) => updateCategory(e, idCategory)}>Modifier</button
            >
        </fieldset>
    </div>
</dialog>
