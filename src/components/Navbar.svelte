<script>
  import { onMount } from "svelte";
  import { goto } from "$app/navigation";
  import { hideLoadingModal, showLoadingModal, Toast } from "../utils/toast";
  import { urlApi } from "../utils/constant";

  let token;
  let user;
  let oldPassword;
  let newPassword;
  let confirmPassword;

  onMount(() => {
    token = localStorage.getItem("token");
    const u = localStorage.getItem("user");
    user = u ? JSON.parse(u) : null;
  });

  async function signOut(event) {
    event.preventDefault();

    localStorage.removeItem("token");
    localStorage.removeItem("user");
    Toast.success("Déconnexion réussie");

    await goto("/admin", { replaceState: true });
  }

  async function changePassword(event) {
    event.preventDefault();
    if (newPassword !== confirmPassword) {
      Toast.error("Les nouveaux mots de passe ne correspondent pas.");
      return;
    }

    showLoadingModal();
    try {
      const response = await fetch(`${urlApi}/change-password`, {
        method: "PUT",
        headers: {
          "Content-Type": "application/json",
          Authorization: `Bearer ${token}`,
        },
        body: JSON.stringify({
          oldPassword,
          newPassword,
        }),
      });

      const resp = await response.json();

      if (resp.status) {
        hideLoadingModal();
        Toast.success("Mot de passe modifié avec succès.");
        oldPassword = "";
        newPassword = "";
        confirmPassword = "";
        const modal = document.getElementById("change_pass_modal");
        modal.close();
      } else {
        hideLoadingModal();
        Toast.error(
          resp.message || "Échec de la modification du mot de passe.",
        );
      }
    } catch (error) {
      hideLoadingModal();
      console.error("Erreur lors de la modification du mot de passe :", error);
      Toast.error("Impossible de modifier le mot de passe.");
    }
  }
</script>

<div class="navbar bg-base-100 rounded-lg">
  <div class="flex-1">
    Bienvenu {#if user}
      {user.surname} !
    {/if}
  </div>

  <div class="flex-none">
    <div class="dropdown dropdown-end">
      <div tabindex="0" role="button" class="btn btn-ghost btn-circle avatar">
        <div class="w-10 rounded-full">
          <div class="avatar avatar-placeholder">
            <div class="bg-primary text-neutral-content w-10 rounded-full">
              <span class="text-xs">
                {#if user}
                  {user.name.charAt(0)}{user.surname.charAt(0)}
                {/if}
              </span>
            </div>
          </div>          
        </div>
      </div>
      <ul
        tabindex="-1"
        class="menu menu-base dropdown-content bg-base-100 rounded-box z-1 mt-3 w-52 p-2 shadow"
      >
        <li>
          <button
            class="justify-between mb-2 font-medium"
            on:click={() => profil_modal.showModal()}>Profil</button
          >
        </li>
        <li>
          <button
            class="justify-between mb-2 font-medium"
            on:click={() => change_pass_modal.showModal()}
            >Changer mot de passe</button
          >
        </li>
        <li>
          <button class="font-medium" on:click={signOut}>Se déconnecter</button>
        </li>
      </ul>
    </div>
  </div>
</div>

<dialog id="change_pass_modal" class="modal">
  <div class="modal-box max-w-md">
    <!-- max-w-md = largeur du modal -->
    <form method="dialog">
      <button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2">
        ✕
      </button>
    </form>

    <h3 class="text-lg font-bold mb-4">Changer mot de passe</h3>

    <fieldset class="fieldset w-full space-y-3">
      <div>
        <label class="label mb-1">Ancien mot de passe</label>
        <input
          type="password"
          class="input input-bordered w-full"
          bind:value={oldPassword}
        />
      </div>
      <div>
        <label class="label mb-1">Nouveau mot de passe</label>
        <input
          type="password"
          class="input input-bordered w-full"
          bind:value={newPassword}
        />
      </div>
      <div>
        <label class="label mb-1">Confirmer mot de passe</label>
        <input
          type="password"
          class="input input-bordered w-full"
          bind:value={confirmPassword}
        />
      </div>

      <button class="btn btn-primary mt-4 w-full" on:click={changePassword}
        >Modifier</button
      >
    </fieldset>
  </div>
</dialog>

<dialog id="profil_modal" class="modal">
  <div class="modal-box max-w-md p-0">
    <form method="dialog">
      <button
        class="btn btn-sm btn-circle btn-ghost absolute right-4 top-4 z-10 bg-white text-primary bg-opacity-20 text-black hover:bg-opacity-30"
      >
        ✕
      </button>
    </form>

    {#if user}
      <div
        class="relative h-32 bg-primary"
      >
        <div class="absolute -bottom-16 left-1/2 transform -translate-x-1/2">
          <div class="relative">
            <div class="avatar">
              <div
                class="w-32 h-32 rounded-full ring-4 bg-primary ring-white shadow-xl flex items-center justify-center text-white text-4xl font-bold"
              >
                {user.name.charAt(0)}{user.surname.charAt(0)}
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- Contenu -->
      <div class="pt-20 pb-6 px-6">
        <!-- Nom et rôle -->
        <div class="text-center mb-6">
          <h2 class="text-2xl font-bold mb-1">{user.name} {user.surname}</h2>
          <p class="text-base-content/60">{user.role.name}</p>
        </div>

        <!-- Informations -->
        <div class="space-y-3 mb-6">
          <!-- Email -->
          <div
            class="flex items-center gap-3 p-3 bg-base-200 rounded-xl hover:bg-base-300 transition-colors"
          >
            <div class="p-2 rounded-lg" style="background-color: #e8eef7;">
              <svg
                xmlns="http://www.w3.org/2000/svg"
                class="h-5 w-5"
                fill="none"
                viewBox="0 0 24 24"
                stroke="#422AD5"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  stroke-width="2"
                  d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"
                />
              </svg>
            </div>
            <span class="flex-1">{user.email}</span>
          </div>

        </div>
      </div>
    {/if}
  </div>
</dialog>
