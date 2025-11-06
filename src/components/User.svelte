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

    let users = []; // ⚠️ doit être let, pas const
    let totalPages = 0;
    let name;
    let surname;
    let email;
    let password;

    $: paginatedUsers = users.slice(
        (currentPage - 1) * perPage,
        currentPage * perPage,
    );

    async function fetchUsers() {
        try {
            const response = await fetch(`${urlApi}/user`, {
                method: "GET",
                headers: {
                    "Content-Type": "application/json",
                },
            });

            const resp = await response.json();

            if (resp.status) {
                users = resp.data;
                totalPages = Math.ceil(users.length / perPage);
            } else {
                Toast.error(
                    resp.message || "Échec du chargement des utilisateurs.",
                );
            }
        } catch (error) {
            console.error(
                "Erreur lors du chargement des utilisateurs :",
                error,
            );
            Toast.error("Impossible de charger les utilisateurs.");
        }
    }

    onMount(async () => {
        fetchUsers();
    });

    async function addAdmin(event) {
        event.preventDefault();
        try {
            if (!name || !surname || !email || !password) {
                Toast.error("Veuillez remplir tous les champs.");
                return;
            }

            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

            if (!emailRegex.test(email)) {
                Toast.error("Veuillez entrer une adresse email valide.");
                return;
            }
            showLoadingModal();
            const response = await fetch(`${urlApi}/user/add-admin`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({
                    name,
                    surname,
                    email,
                    password,
                }),
            });

            const resp = await response.json();
            if (resp.status) {
                hideLoadingModal();
                Toast.success("Administrateur ajouté avec succès !");
                fetchUsers();
                // Réinitialiser les champs du formulaire
                name = "";
                surname = "";
                email = "";
                password = "";
                // Fermer le modal
                const modal = document.getElementById("add_admin_modal");
                modal.close();
            } else {
                hideLoadingModal();
                Toast.error(
                    resp.message || "Échec de l'ajout de l'administrateur.",
                );
            }
        } catch (error) {
            hideLoadingModal();
            console.error(
                "Erreur lors de l'ajout de l'administrateur :",
                error,
            );
            Toast.error("Impossible d'ajouter l'administrateur.");
        }
    }

    async function deleteUser(event, userId) {
        event.preventDefault();

        // Afficher l'alerte de confirmation
        const result = await confirmAlert({
            title: "Supprimer l'utilisateur ?",
            text: "Cette action est irréversible !",
            confirmButtonText: "Oui, supprimer",
            cancelButtonText: "Annuler",
        });

        if (result) {
            showLoadingModal();
            try {
                const response = await fetch(
                    `${urlApi}/user/delete/${userId}`,
                    {
                        method: "DELETE",
                        headers: {
                            "Content-Type": "application/json",
                        },
                    },
                );

                const resp = await response.json();

                if (resp.status) {
                    hideLoadingModal();
                    Toast.success("Utilisateur supprimé avec succès !");
                    fetchUsers();
                } else {
                    hideLoadingModal();
                    Toast.error(
                        resp.message ||
                            "Échec de la suppression de l'utilisateur.",
                    );
                }
            } catch (error) {
                hideLoadingModal();
                console.error(
                    "Erreur lors de la suppression de l'utilisateur :",
                    error,
                );
                Toast.error("Impossible de supprimer l'utilisateur.");
            }
        }
    }
</script>

<div>
    <h1 class="text-2xl font-bold mb-4">Utilisateurs</h1>
</div>

<div class="breadcrumbs text-sm mt-2 mb-5">
    <ul>
        <li>J-Librairie</li>
        <li>Utilisateurs</li>
    </ul>
</div>

<div class="shadow-sm rounded-box border border-base-content/5 bg-base-100 p-4">
    <!-- HEADER -->
    <div class="flex justify-between items-center mb-3">
        <h2 class="font-semibold text-lg">Liste des utilisateurs</h2>
        <button
            class="btn btn-primary flex items-center gap-2"
            on:click={() => add_admin_modal.showModal()}
        >
            <!-- Texte visible sur écran moyen et plus -->
            <span class="hidden md:inline">Ajouter un administrateur</span>

            <!-- Icône visible uniquement sur mobile -->
            <span class="inline md:hidden text-xl font-bold">+</span>
        </button>
    </div>

    <!-- TABLE -->
    <div
        class="overflow-x-auto rounded-box border border-base-content/5 bg-base-100"
    >
        <table class="table">
            <thead>
                <tr>
                    <th>#</th>
                    <th>Nom</th>
                    <th>Prénoms</th>
                    <th>Email</th>
                    <th>Type de compte</th>
                    <th>Action</th>
                </tr>
            </thead>
            <tbody>
                {#each paginatedUsers as user, index}
                    <tr class="hover:bg-gray-50 transition">
                        <th>{(currentPage - 1) * perPage + index + 1}</th>
                        <td>{user.name}</td>
                        <td>{user.surname}</td>
                        <td>{user.email}</td>
                        <td>{user.role.name}</td>
                        <td>
                            <button
                                class="btn btn-sm btn-error btn-outline"
                                on:click={(event) => deleteUser(event, user.id)}
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

<dialog id="add_admin_modal" class="modal">
    <div class="modal-box max-w-md">
        <!-- max-w-md = largeur du modal -->
        <form method="dialog">
            <button
                class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            >
                ✕
            </button>
        </form>

        <h3 class="text-lg font-bold">Ajouter un administrateur</h3>
        <p class="py-4">Renseignez les informations du nouveau admin</p>

        <fieldset class="fieldset w-full space-y-3">
            <div>
                <label class="label mb-1">Nom</label>
                <input
                    type="text"
                    class="input input-bordered w-full"
                    bind:value={name}
                />
            </div>

            <div>
                <label class="label mb-1">Prénoms</label>
                <input
                    type="text"
                    class="input input-bordered w-full"
                    bind:value={surname}
                />
            </div>

            <div>
                <label class="label mb-1">Email</label>
                <input
                    type="email"
                    class="input input-bordered w-full"
                    bind:value={email}
                />
            </div>

            <div>
                <label class="label mb-1">Mot de passe</label>
                <input
                    type="password"
                    class="input input-bordered w-full"
                    bind:value={password}
                />
            </div>

            <button class="btn btn-primary mt-4 w-full" on:click={addAdmin}
                >Ajouter</button
            >
        </fieldset>
    </div>
</dialog>
