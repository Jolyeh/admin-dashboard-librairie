<script>
    import { goto } from "$app/navigation";
    import { urlApi } from "../../utils/constant.js";
    import { hideLoadingModal, showLoadingModal, Toast } from "../../utils/toast.js";

    let email;
    let password;
    let showPassword = false;
    let verifyEmail;
    let opt;
    let optUser;
    let newPassword;
    let confirmPassword;

    function togglePasswordVisibility() {
        showPassword = !showPassword;
    }

    async function login(event) {
        event.preventDefault();

        if (!email || !password) {
            Toast.error("Veuillez remplir tous les champs.");
            return;
        }

        try {
            showLoadingModal();
            const response = await fetch(`${urlApi}/login`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({ email, password }),
            });

            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                if (resp.data.role.name !== "ADMIN") {
                    Toast.error(
                        "Accès refusé. Vous n'êtes pas administrateur.",
                    );
                    return;
                }

                Toast.success("Connexion réussie !");

                localStorage.setItem("token", resp.data.token);
                localStorage.setItem("user", JSON.stringify(resp.data));
                goto("/admin/dashboard", { replaceState: true });
            } else {
                hideLoadingModal();
                Toast.error(resp.message || "Échec de la connexion.");
            }
        } catch (error) {
            hideLoadingModal();
            Toast.error("Une erreur est survenue lors de la connexion.");
            console.error("Login error:", error);
        }
    }

    async function getOtp(event) {
        event.preventDefault();
        try {
            showLoadingModal();
            const response = await fetch(`${urlApi}/send-otp`, {
                method: "POST",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({ email: verifyEmail }),
            });

            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                opt = resp.data.otp;
                email_modal.close();
                opt_modal.showModal();
                Toast.success("OTP envoyé à votre email.");
            } else {
                hideLoadingModal();
                Toast.error(resp.message || "Échec de l'envoi de l'OTP.");
            }
        } catch (error) {
            hideLoadingModal();
            Toast.error("Une erreur est survenue lors de l'envoi de l'OTP.");
            console.error("OTP error:", error);
        }
    }

    async function verifyOtp(event) {
        event.preventDefault();
        if (optUser === opt) {
            opt_modal.close();
            reset_pass_modal.showModal();
            Toast.success("OTP vérifié. Vous pouvez réinitialiser votre mot de passe.");
        } else {
            Toast.error("OTP incorrect. Veuillez réessayer.");
        }
    }

    async function resetPassword(event) {
        event.preventDefault();

        if (newPassword !== confirmPassword) {
            Toast.error("Les mots de passe ne correspondent pas.");
            return;
        }

        try {
            showLoadingModal();
            const response = await fetch(`${urlApi}/reset-password`, {
                method: "PUT",
                headers: {
                    "Content-Type": "application/json",
                },
                body: JSON.stringify({ email: verifyEmail, newPassword }),
            });

            const resp = await response.json();

            if (resp.status) {
                hideLoadingModal();
                reset_pass_modal.close();
                Toast.success("Mot de passe réinitialisé avec succès.");
                verifyEmail = "";
            } else {
                hideLoadingModal();
                Toast.error(resp.message || "Échec de la réinitialisation du mot de passe.");
            }
        } catch (error) {
            hideLoadingModal();
            Toast.error("Une erreur est survenue lors de la réinitialisation du mot de passe.");
            console.error("Reset Password error:", error);
        }
    }
</script>

<div class="min-h-screen bg-base-200 flex items-center justify-center p-4">
    <div class="w-full max-w-lg items-center">

        <div class="w-full">
            <div class="card bg-base-100 shadow-2xl">
                <div class="card-body p-8 lg:p-10">
                    <div class="mb-8">
                        <h2 class="text-2xl font-bold text-base-content mb-2">
                            Espace d'administration
                        </h2>
                        <p class="text-base-content/60">
                            Accédez à votre tableau de bord administrateur
                        </p>
                    </div>

                    <!-- Formulaire -->
                    <form on:submit|preventDefault={login} class="space-y-6">
                        <!-- Email -->
                        <div class="form-control">
                            <label class="label" for="email">
                                <span class="label-text font-medium">Adresse email</span>
                            </label>
                            <div class="relative">
                                <input
                                    id="email"
                                    type="email"
                                    class="input input-bordered w-full"
                                    placeholder="admin@jlibrairie.com"
                                    bind:value={email}
                                    required
                                />
                            </div>
                        </div>

                        <!-- Mot de passe -->
                        <div class="form-control">
                            <label class="label" for="password">
                                <span class="label-text font-medium">Mot de passe</span>
                            </label>
                            <div class="relative">
                                <input
                                    id="password"
                                    type={showPassword ? "text" : "password"}
                                    class="input input-bordered w-full pr-12"
                                    placeholder="••••••••"
                                    bind:value={password}
                                    required
                                />
                                <button
                                    type="button"
                                    class="absolute inset-y-0 right-0 pr-4 flex items-center"
                                    on:click={togglePasswordVisibility}
                                >
                                    {#if showPassword}
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-base-content/40 hover:text-base-content/60" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13.875 18.825A10.05 10.05 0 0112 19c-4.478 0-8.268-2.943-9.543-7a9.97 9.97 0 011.563-3.029m5.858.908a3 3 0 114.243 4.243M9.878 9.878l4.242 4.242M9.88 9.88l-3.29-3.29m7.532 7.532l3.29 3.29M3 3l3.59 3.59m0 0A9.953 9.953 0 0112 5c4.478 0 8.268 2.943 9.543 7a10.025 10.025 0 01-4.132 5.411m0 0L21 21" />
                                        </svg>
                                    {:else}
                                        <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 text-base-content/40 hover:text-base-content/60" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" />
                                            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" />
                                        </svg>
                                    {/if}
                                </button>
                            </div>
                        </div>

                        <button
                            type="submit"
                            class="btn btn-primary w-full h-12"
                        >
                                Se connecter
                        </button>
                    </form>

                    <div class="divider my-3">OU</div>
                    <div class="text-center">
                        <p class="text-sm text-base-content/60">
                            <button class="link link-primary font-medium" on:click={()=> email_modal.showModal()}>
                                Mot de passe oublié ?
                            </button>
                        </p>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

<dialog id="email_modal" class="modal">
    <div class="modal-box max-w-md">
        <!-- max-w-md = largeur du modal -->
        <form method="dialog">
            <button
                class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            >
                ✕
            </button>
        </form>

        <h3 class="text-lg font-bold">Vérification du email</h3>
        <p class="py-4">Renseignez l'email du compte</p>

        <fieldset class="fieldset w-full space-y-3">
            <div>
                <label class="label mb-1">Email</label>
                <input
                    type="text"
                    class="input input-bordered w-full"
                    bind:value={verifyEmail}
                />
            </div>

            <button class="btn btn-primary mt-4 w-full" on:click={getOtp}
                >Valider</button
            >
        </fieldset>
    </div>
</dialog>

<dialog id="opt_modal" class="modal">
    <div class="modal-box max-w-md">
        <!-- max-w-md = largeur du modal -->
        <form method="dialog">
            <button
                class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            >
                ✕
            </button>
        </form>

        <h3 class="text-lg font-bold">Confirmation du email</h3>
        <p class="py-4">Renseignez le code envoyer sur votre email</p>

        <fieldset class="fieldset w-full space-y-3">
            <div>
                <label class="label mb-1">Code</label>
                <input
                    type="text"
                    class="input input-bordered w-full"
                    bind:value={optUser}
                />
            </div>

            <button class="btn btn-primary mt-4 w-full" on:click={verifyOtp}
                >Valider</button
            >
        </fieldset>
    </div>
</dialog>

<dialog id="reset_pass_modal" class="modal">
    <div class="modal-box max-w-md">
        <!-- max-w-md = largeur du modal -->
        <form method="dialog">
            <button
                class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
            >
                ✕
            </button>
        </form>

        <h3 class="text-lg font-bold mb-3">Modifier mot de passe</h3>

        <fieldset class="fieldset w-full space-y-3">
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

            <button class="btn btn-primary mt-4 w-full" on:click={resetPassword}
                >Valider</button
            >
        </fieldset>
    </div>
</dialog>